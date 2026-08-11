---
layout: post
title: "HALCON Local Deformable Matching 实现(一)"
date: 2026-08-11
tags: [shape match]
comments: true
author: mwwz
---

`find_local_deformable_model` 是 HALCON 中用于处理局部形变的模板匹配算子。为了理解其背后的局部形变建模思想，本文参考 Hofhauser、Steger 和 Navab 提出的 Edge-Based Template Matching with a Harmonic Deformation Model，并结合自己的实现过程，对 Local Deformable Matching 的核心流程进行分析。

## 核心思想

该算子的基本思路可概括为四个步骤：

1. **构建局部邻域：** 为每个特征点选取最近的 `k` 个邻居，组成局部点集，匹配时以该整体为单位进行相似度度量。

2. **估计稀疏位移：** 对每个局部点集进行匹配，计算其整体平移量 `(dx, dy)`，得到仅覆盖特征点位置的**稀疏形变场**。

3. **插值生成稠密场：** 以稀疏形变场为约束，通过求解拉普拉斯方程（调和函数）将位移平滑传播至全图所有像素，得到稠密形变场。

4. **图像校正：** 利用最终的稠密形变场对图像进行校正，也可将模板扭曲至目标形状，完成匹配。

## 具体实现

### 1. 特征点提取与邻域构建

- **特征点提取：** 采用 Sobel 算子提取边缘点，并通过亚像素插值进行精确定位。

- **邻域构建：** 基于 KD-Tree 为每个特征点搜索最近的 `k` 个邻居（使用 OpenCV flann 或 nanoflann），将孤立点扩展为局部点集。

<div align="center">
  <img src="/images/local_deform_0.png" width="90%" alt="特征点提取结果" style="display:inline-block; margin:0 5px;">
</div>
<p align="center"><b>图1. 特征点提取结果</b></p>

<div align="center">
  <img src="/images/local_deform_1.png" width="45%" alt="邻域构建示例1" style="display:inline-block; margin:0 5px;">
  <img src="/images/local_deform_2.png" width="45%" alt="邻域构建示例2" style="display:inline-block; margin:0 5px;">
</div>
<p align="center"><b>图2. 邻域构建结果</b>（不同特征点的局部点集示例）</p>

### 2. 局部块匹配与稀疏位移估计

将每个局部点集视作独立块，与全局形状匹配类似，对每个块独立进行匹配。在相似度响应图中寻找峰值区域，提取亚像素位移 `(dx, dy)`，汇总后得到稀疏形变场，作为后续稠密插值的约束。

<div align="center">
  <img src="/images/local_deform_3.png" width="45%" alt="分值图1" style="display:inline-block; margin:0 5px;">
  <img src="/images/local_deform_4.png" width="45%" alt="分值图2" style="display:inline-block; margin:0 5px;">
</div>
<p align="center"><b>图3. 局部块分值图</b></p>

<div align="center">
  <img src="/images/local_deform_5.png" width="45%" alt="稀疏形变场" style="display:inline-block; margin:0 5px;">
  <img src="/images/local_deform_6.png" width="45%" alt="局部放大" style="display:inline-block; margin:0 5px;">
</div>
<p align="center"><b>图4. 稀疏形变场</b>（左：全局稀疏位移，右：局部放大）</p>

### 3. 调和插值生成稠密形变场

#### 3.1 为什么位移场要满足拉普拉斯方程？

想象位移场 $u(x,y)$ 是一张**橡皮膜**的高度。你希望这张膜：

- 在已知点（控制点）处被固定到指定高度；
- 在其他地方尽可能"平坦"，没有褶皱。

数学上，**曲面的弯曲程度**由它的**拉普拉斯算子**衡量：

$$
\nabla^2 u = \frac{\partial^2 u}{\partial x^2} + \frac{\partial^2 u}{\partial y^2}
$$

- 若 $\nabla^2 u = 0$，曲面在局部是"平"的，没有凸起或凹陷；
- 满足 $\nabla^2 u = 0$ 的函数称为**调和函数**。

因此，**要求位移场满足拉普拉斯方程，等价于要求它在约束点之间以最平滑的方式过渡**。

#### 3.2 二维拉普拉斯算子

将 $x$ 和 $y$ 方向的二阶导数合并，离散形式为：

$$
\nabla^2 u \approx [u(x+1,y) + u(x-1,y) + u(x,y+1) + u(x,y-1)] - 4u(x,y)
$$

令其等于 0（拉普拉斯方程）：

$$
u(x,y) = \frac{u(x+1,y) + u(x-1,y) + u(x,y+1) + u(x,y-1)}{4}
$$

上式即标准的五点法平滑核，代码中可直接用卷积实现。

#### 3.3 Jacobi 迭代求解稠密形变场

以稀疏位移场中的匹配点为约束，对其余像素反复应用 3.2 节的五点平滑核进行 Jacobi 迭代，直至收敛，即可得到平滑的稠密形变场。其中，匹配点的位移在迭代过程中始终保持不变。

<div align="center">
  <img src="/images/local_deform_7.png" width="45%" alt="稠密形变场" style="display:inline-block; margin:0 5px;">
  <img src="/images/local_deform_8.png" width="45%" alt="局部放大" style="display:inline-block; margin:0 5px;">
</div>
<p align="center"><b>图5. 稠密形变场</b>（左：逐像素位移场，右：局部放大）</p>

### 4. 图像校正与模板扭曲

获得稠密形变场后，即可对图像或模板进行几何变换以消除形变影响。

- **图像校正：** 利用稠密形变场提供的逐像素位移映射，直接调用 OpenCV 的 `remap` 函数对目标图像进行校正，将形变后的图像还原至模板坐标系。

- **模板扭曲：** 若需将模板变换至目标图像空间，则根据模板各点位置查询对应的位移量，逐点施加偏移即可完成模板扭曲。

<div align="center">
  <img src="/images/local_deform_9.png" width="45%" alt="校正后图像" style="display:inline-block; margin:0 5px;">
  <img src="/images/local_deform_10.png" width="45%" alt="校正后模板" style="display:inline-block; margin:0 5px;">
</div>
<p align="center"><b>图6. 校正结果</b>（左：校正后图像，右：扭曲后模板）</p>

## 总结与展望

<div align="center">
  <img src="/images/local_deform_11.png" width="80%" alt="错误与缺失位移示例" style="display:inline-block; margin:0 5px;">
</div>
<p align="center"><b>图7. 错误位移与匹配缺失示例</b></p>

目前，基于稠密形变场的图像校正与模板扭曲已基本实现配准，但部分细节仍有待完善。理论上，只要位移估计足够精确，变换效果便不是瓶颈。

实现过程中发现，局部匹配存在**匹配错误**或**匹配缺失**的问题，且难以完全避免。例如，当局部点集呈平行线结构时，匹配容易产生歧义，导致异常位移；某些区域则因纹理匮乏或对比度不足而无法匹配，造成位移缺失。对此，可考虑以下策略加以抑制：

- **峰值区域筛选：** 对分值图的峰值区域进行形态判断，剔除横纵比异常的峰值；
- **位移一致性校验：** 利用当前点周围 8 个邻域位移进行平滑性检查，剔除异常值；
- **后向插值约束：** 参考原文献中的后向插值方法，进一步消除错误位移；
- **置信度加权融合：** 对不可靠匹配分配较低置信度，将拉普拉斯平滑中 Jacobi 迭代的数据融合升级为逐像素动态加权。

上述问题的深入分析与改进方案，将在后续文章中详细展开。
