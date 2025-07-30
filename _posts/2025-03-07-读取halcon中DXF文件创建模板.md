---
layout: post
title: "读取halcon中DXF文件创建模板"
date: 2025-03-07
tags: [shape match]
comments: true
author: mwwz
---

## halcon中的DXF文件

在HDevelop的帮助窗口里，选择“算子”标签页，输入`dxf`，一共出现四个算子：

![dxf相关算子](/images/dxf_0.png "dxf相关算子")

- **read_contour_xld_dxf**：Read XLD contours from a DXF file.
- **read_polygon_xld_dxf**：Read XLD polygons from a DXF file.
- **write_contour_xld_dxf**：Write XLD contours to a file in DXF format.
- **write_polygon_xld_dxf**：Write XLD polygons to a file in DXF format.

**XLDs**（eXtended Line Descriptions，扩展线描述）是HALCON中特定的图标对象，用于表示亚像素精度的轮廓和多边形(Hcont, Hpoly)。

从上述描述可知，XLD主要包含轮廓和多边形两种类型，四个算子的功能分别为**从DXF文件读取XLD对象** 、**将XLD对象写入DXF文件** 。

在**DeepSeek**中输入`C++读取DXF文件并解析`得到如下代码：

```cpp
#include <iostream>
#include <fstream>
#include <string>
#include <vector>
#include <map>

// 定义一个结构体来存储DXF文件中的实体
struct DxfEntity {
    std::string type;
    std::map<int, std::string> properties;
};

// 读取DXF文件并解析
std::vector<DxfEntity> readDxfFile(const std::string& filePath) {
    std::ifstream file(filePath);
    std::vector<DxfEntity> entities;
    DxfEntity currentEntity;
    bool inEntity = false;

    if (!file.is_open()) {
        std::cerr << "无法打开文件: " << filePath << std::endl;
        return entities;
    }

    std::string line;
    while (std::getline(file, line)) {
        // 去除行首尾的空白字符
        line.erase(0, line.find_first_not_of(" \t\r\n"));
        line.erase(line.find_last_not_of(" \t\r\n") + 1);

        if (line.empty()) continue;

        // 解析组码和值
        int groupCode = std::stoi(line);
        std::string value;
        if (std::getline(file, value)) {
            value.erase(0, value.find_first_not_of(" \t\r\n"));
            value.erase(value.find_last_not_of(" \t\r\n") + 1);

            if (groupCode == 0) {
                // 新的实体开始
                if (inEntity) {
                    entities.push_back(currentEntity);
                    currentEntity = DxfEntity();
                }
                currentEntity.type = value;
                inEntity = true;
            } else {
                // 添加属性到当前实体
                currentEntity.properties[groupCode] = value;
            }
        }
    }

    // 添加最后一个实体
    if (inEntity) {
        entities.push_back(currentEntity);
    }

    file.close();
    return entities;
}

// 打印解析的实体
void printEntities(const std::vector<DxfEntity>& entities) {
    for (const auto& entity : entities) {
        std::cout << "实体类型: " << entity.type << std::endl;
        for (const auto& prop : entity.properties) {
            std::cout << "  组码: " << prop.first << ", 值: " << prop.second << std::endl;
        }
        std::cout << std::endl;
    }
}

int main() {
    std::string filePath = "example.dxf";
    std::vector<DxfEntity> entities = readDxfFile(filePath);
    printEntities(entities);
    return 0;
}
```

利用上述代码，分别读取由**write_contour_xld_dxf** 和**write_polygon_xld_dxf** 保存的DXF文件，输出结果如下：

![dxf contour](/images/dxf_1.png "dxf contour")
![dxf polygon](/images/dxf_2.png "dxf polygon")

不难看出，**POLYLINE** 标示每一个XLD对象的开始，其后的**VERTEX** 即为XLD中的顶点，10，20，30分别对应**X，Y，Z** 坐标，1040对应**edge_direction** ，对于轮廓可以提取到`(X,Y,edge_direction)`，对于多边形可以提取到`(X,Y)`，基于这些信息，即可创建模板。

## 使用DXF文件创建模板

**mwwz**图像库支持从轮廓或者多边形创建模板，读取halcon的DXF文件后，解析出位置、方向等信息，即可创建适用于**mwwz**的模板。此项功能已加入测试软件。需要注意的是，轮廓本身自带方向，而多边形仅包含顶点位置信息，其方向需要额外指定。在测试软件中加载模板时，**多边形模板+** 表示方向在轮廓的右侧，**多边形模板-** 表示方向在轮廓的左侧，此外**形状模板(*.shm)** 与halcon并不兼容。

[测试软件下载地址](https://pan.baidu.com/s/1FP6wA8KOwCYJhKI1cc93xg?pwd=aabb)

![dxf contour](/images/dxf_3.png "dxf contour")
![dxf polygon](/images/dxf_4.png "dxf polygon")
