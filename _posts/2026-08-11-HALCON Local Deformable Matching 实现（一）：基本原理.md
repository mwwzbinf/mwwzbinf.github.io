---
layout: post
title: "HALCON Local Deformable Matching 实现(一)
date: 2026-08-11
tags: [shape match]
comments: true
author: mwwz
---

## 核心思想

`Halcon`中`find_local_deformable_model`算子的核心机制，源于论文`Edge-Based Template Matching with a Harmonic Deformation Model`。其基本思路可概括为四个步骤：

1. **构建局部邻域：** 为每个特征点选取最近的`k`个邻居，组成局部点集，匹配时以该整体为单位进行相似度度量，增强鲁棒性。

2. **估计稀疏位移：** 对每个局部点集进行匹配，计算其整体平移量`(dx,dy)`，得到仅覆盖特征点位置的稀疏形变场。

3. **插值生成稠密场** 以稀疏形变场为约束，通过求解拉普拉斯方程(调和函数)将位移平滑传播至全图所有像素，得到稠密形变场。

4. **图像校正** 利用最终的稠密形变场对图像进行校正，或将模板扭曲至目标形状，完成匹配。

## 具体实现

### 1. 特征点提取与邻域构建

- **特征点提取：** 采用`Sobel`算子提取边缘点，并通过亚像素插值将定位精度提升至亚像素级。

- **邻域构建：** 基于`KD-Tree` 对每个特征点搜索最近的 `k`个邻居点（实现上使用 OpenCV flann 或 nanoflann 库），从而将孤立点扩展为局部点集，为后续以“块”为单位的匹配做准备。

<!-- ![Feature Point Extraction](/images/local_deform_0.png "Feature Point Extraction")
![Neighborhood Construction](/images/local_deform_1.png "Neighborhood Construction")
![Neighborhood Construction](/images/local_deform_2.png "Neighborhood Construction") -->

<div align="center">
  <img src="/images/local_deform_0.png" width="90%" alt="Neighborhood Construction" style="display:inline-block; margin:0 5px;">
</div>

<div align="center">
  <img src="/images/local_deform_1.png" width="45%" alt="Neighborhood Construction" style="display:inline-block; margin:0 5px;">
  <img src="/images/local_deform_2.png" width="45%" alt="Neighborhood Construction" style="display:inline-block; margin:0 5px;">
</div>

### 2. 局部块匹配与稀疏位移估计

将每个局部点集作为独立块，与全局形状匹配类似进行独立匹配。对每个块的相似度响应图寻找峰值区域，提取亚像素位移(dx,dy)，汇总后得到稀疏形变场，为后续稠密插值提供约束。

<!-- ![Score map](/images/local_deform_3.png "Score map")
![Score map](/images/local_deform_4.png "Score map")
![Sparse field](/images/local_deform_5.png "Sparse field")
![Zoom-in](/images/local_deform_6.png "Zoom-in") -->

<div align="center">
  <img src="/images/local_deform_3.png" width="45%" alt="Score map" style="display:inline-block; margin:0 5px;">
  <img src="/images/local_deform_4.png" width="45%" alt="Score map" style="display:inline-block; margin:0 5px;">
</div>

<div align="center">
  <img src="/images/local_deform_5.png" width="45%" alt="Sparse field" style="display:inline-block; margin:0 5px;">
  <img src="/images/local_deform_6.png" width="45%" alt="Zoom-in" style="display:inline-block; margin:0 5px;">
</div>

### 3. 调和插值生成稠密形变场

#### 3.1  为什么位移场要满足拉普拉斯方程？

想象位移场 $u(x,y)$ 是一张**橡皮膜**的高度。你希望这张膜：

- 在已知点（控制点）处被固定到指定高度
- 在其他地方尽可能"平坦"、没有褶皱

在数学上，**曲面的弯曲程度**由它的**拉普拉斯算子**衡量：

$$
\nabla^2 u = \frac{\partial^2 u}{\partial x^2} + \frac{\partial^2 u}{\partial y^2}
$$

- 如果 $\nabla^2 u = 0$，这个曲面在局部是"平的"——没有局部凸起或凹陷
- 满足 $\nabla^2 u = 0$ 的函数叫做**调和函数（Harmonic Function）**

所以，**要求位移场满足拉普拉斯方程，等价于要求它在约束点之间以最平滑的方式过渡**。

#### 3.2 二维拉普拉斯算子

把 x 和 y 方向加起来：

$$
\nabla^2 u \approx [u(x+1,y) + u(x-1,y) + u(x,y+1) + u(x,y-1)] - 4u(x,y)
$$

令这个等于 0（拉普拉斯方程）：

$$
u(x,y) = \frac{u(x+1,y) + u(x-1,y) + u(x,y+1) + u(x,y-1)}{4}
$$

**这就是代码中 用到的kernel ！**

<!-- ![Dense deformation field](/images/local_deform_7.png "Dense deformation field")
![Close-up view](/images/local_deform_8.png "Close-up view") -->

<div align="center">
  <img src="/images/local_deform_7.png" width="45%" alt="Dense deformation field" style="display:inline-block; margin:0 5px;">
  <img src="/images/local_deform_8.png" width="45%" alt="Close-up view" style="display:inline-block; margin:0 5px;">
</div>

### 4. 图像校正与模板扭曲

获得稠密形变场后，即可对图像或模板进行几何变换以消除形变影响。

- **图像校正：** 利用稠密形变场提供的逐像素位移映射，直接调用 OpenCV 的 remap 函数对目标图像进行校正，将形变后的图像还原至模板坐标系。

- **模板扭曲：** 若需将模板变换至目标图像空间，则根据模板各点位置查询对应的位移量，逐点施加偏移即可完成模板扭曲。

<!-- ![Corrected image](/images/local_deform_9.png "Corrected image")
![Corrected template](/images/local_deform_10.png "Corrected template") -->

<div align="center">
  <img src="/images/local_deform_9.png" width="45%" alt="Corrected image" style="display:inline-block; margin:0 5px;">
  <img src="/images/local_deform_10.png" width="45%" alt="Corrected template" style="display:inline-block; margin:0 5px;">
</div>

## 问题与后续探索

## 总结与展望

![False and missing displacements](/images/local_deform_11.png "False and missing displacements")

目前，基于稠密形变场的图像校正与模板扭曲已基本实现对齐，但部分细节仍有待完善。理论上，只要位移估计足够精确，变换效果便不是瓶颈。

实现过程中发现，局部匹配存在**匹配错误**或**匹配缺失**的问题，且难以完全避免。例如，当局部点集呈平行线结构时，匹配容易产生歧义，导致异常位移；某些区域则因纹理匮乏而无法匹配，造成位移缺失。对此，可考虑以下策略加以抑制：

- **峰值区域筛选**：对分值图的峰值区域进行形态判断，剔除横纵比异常的峰值；
- **位移一致性校验**：利用当前点周围 8 个邻域位移进行平滑性检查，剔除异常值；
- **后向插值约束**：参考原文献中的后向插值方法，进一步消除错误位移；
- **置信度加权融合**：对不可靠匹配分配较低置信度，将拉普拉斯平滑中 Jacobi 迭代的数据融合升级为逐像素动态加权。

上述问题的深入分析与改进方案，将在后续文章中详细展开。
