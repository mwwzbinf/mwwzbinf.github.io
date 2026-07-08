---
layout: post
title: "复现 Halcon 经典例程：find_shape_model_max_deformation.hdev"
date: 2026-07-08
tags: [shape match]
comments: true
author: mwwz
---

## 例程说明

`find_shape_model_max_deformation.hdev` 通过对比实验，直观展示了是否允许变形（**max_deformation**）对搜索轻微变形物体的影响。该例程使用 MVTec Logo 作为模板，在一系列不同程度变形的测试图像中进行搜索，并统计两种模式下的匹配成功率、定位精度和运行时间。

两种模式的参数配置如下：

- **不允许变形**：`SubPixel := 'least_squares_high'`
- **允许变形**：`SubPixel := ['least_squares_high', 'max_deformation 10']`

核心函数 `create_mvtec_logo_broadened` 负责从原始图像（`pen-01`）中提取 MVTec Logo 的高精度亚像素边缘，将其缩放、居中并绘制到一张全新的彩色图像上，最后根据输入参数对 Logo 进行加粗或变细处理。

![halcon-example](/images/deform_mvtec_logo_0.png "halcon-example")

## 改进要点

关于 Halcon 中 **max_deformation** 的实现原理，可以参考我之前的博客：[局部可变形模板匹配算法的初步实现](https://mp.weixin.qq.com/s/fFDyVdf2u4yZul078oOScw)。本文重点说明改进方向与要点：

- 搜索模板点对应的图像边缘点时，搜索距离应适当增加（与变形量相关）。
- 最小二乘法获得的 `dx`、`dy`、`dθ` 变化幅度会增大，需增加容错处理。
- 由于变形量较大，最小二乘法需要多次迭代才能收敛。
- 使用普通（未加权）最小二乘法，避免结果偏向目标的一侧。

经改进，当`最大变形`设为10，`亚像素`设为 `LS_High`，测试效果与 Halcon 基本一致。

![mwwz-0.0.png](/images/deform_mvtec_logo_0_0.png "mwwz-0.0.png")

![mwwz-0.3.png](/images/deform_mvtec_logo_0_3.png "mwwz-0.3.png")

![mwwz-1.1.png](/images/deform_mvtec_logo_1_1.png "mwwz-1.1.png")

![mwwz-5.5.png](/images/deform_mvtec_logo_5_5.png "mwwz-5.5.png")

![mwwz-7.7.png](/images/deform_mvtec_logo_7_7.png "mwwz-7.7.png")

![mwwz-8.8.png](/images/deform_mvtec_logo_8_8.png "mwwz-8.8.png")

![mwwz-9.9.png](/images/deform_mvtec_logo_9_9.png "mwwz-9.9.png")

## 立即下载

**[下载地址](https://www.mwwztech.com/)**
