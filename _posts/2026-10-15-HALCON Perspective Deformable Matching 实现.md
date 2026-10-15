---
layout: post
title: "HALCON Perspective Deformable Matching 实现"
date: 2026-10-15
tags: [shape match]
comments: true
author: mwwz
---

继实现 `find_local_deformable_model` 之后，我们继续实现 `find_planar_uncalib_deformable_model`（平面可变形模板匹配）。

这个算子的实现相对简单。与局部匹配需要将特征点划分为多个局部 Patch 不同，平面匹配只需要将模板作为一个整体，核心目标是求解其透视变换，即单应性矩阵（Homography）。

单应性矩阵的计算可以直接使用 OpenCV 提供的 `findHomography`，因此整体实现并不复杂，主要分为两个步骤：

1. **顶层初估**：在图像金字塔的最高层估计初始单应性矩阵。
2. **逐层精炼**：从高层逐步向下，在每一层对单应性矩阵进行修正和优化，直到最底层，得到最终的精确结果。

目前，MWWZ 视觉库已经实现了 HALCON 中大部分常用的匹配算子。不过在使用方式上，MWWZ 与 HALCON 还是存在一些区别。

HALCON 的不同匹配算子通常都有对应的模板创建算子，例如不同类型的匹配需要分别创建不同的模板。而在 MWWZ 中，我们将模板创建进行了统一：**所有匹配函数共用一个模板创建函数**。

模板只需要创建一次，之后可以被不同的匹配算子重复使用，也就是：

**一次创建，多处使用。**

这样不仅减少了模板管理的复杂度，也让不同匹配算法之间的切换更加方便。

![matching arch.png](/images/matching arch.png "matching arch.png")
![a.png](/images/a.png "a.png")
![c.png](/images/c.png "c.png")
