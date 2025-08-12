---
layout: post
title: "实现halcon中的gen_region_polygon_filled"
date: 2025-08-12
tags: [polygon]
comments: true
author: mwwz
---

## Scan-Line Polygon Fill

扫描线填充算法是计算机图形学中一种高效的填充多边形内部区域的算法。与逐个像素检测的泛滥填充(Flood Fill)或边界填充(Boundary Fill)算法不同，它利用了图形的连贯性，一次处理一条水平线上的所有像素，效率很高。

 关键步骤与数据结构

  为了高效地实现这一过程，算法通常使用两个关键的数据结构：

   1. 边表 (Edge Table, ET)：一个数组或链表列表，用于存放多边形的所有边的信息。整个表按边的最小 y 坐标进行索引。每条边信息通常包括：
       * ymax: 该边的最大 y 坐标。
       * x_at_ymin: 该边在最小 y 坐标处的 x 值。
       * 1/m: 斜率的倒数 (dx/dy)，用于在扫描线每向上移动一行时，快速更新边与扫描线的 x 交点。

   2. 活性边表 (Active Edge Table, AET)：一个链表，用于存放当前扫描线正在相交的边。AET 中的边始终按其与当前扫描线交点的 x 坐标从小到大排序。

  算法流程如下：

   1. 初始化：遍历多边形的所有顶点，建立边表(ET)。水平的边通常被忽略。
   2. 开始扫描：设置一个 y 值，从多边形的最低扫描线开始。
   3. 处理循环：只要活性边表(AET)不为空，或者边表(ET)中还有未处理的边，就重复以下步骤：
      a. 更新AET：检查边表(ET)，将所有 ymin == y 的边移入活性边表(AET)中。
      b. 排序AET：将 AET 中的所有边按当前的 x 交点坐标进行排序。
      c. 填充像素：将 AET 中的边两两配对（第1条和第2条，第3条和第4条，以此类推），然后填充每对边之间在当前扫描线上的所有像素。
      d. 进入下一行：将 y 值加 1 (y = y + 1)。
      e. 更新AET：
           * 移除 AET 中满足 ymax == y 的边（因为扫描线已经越过了这些边）。
           * 对于 AET 中剩余的每一条边，根据其斜率倒数更新 x 交点的值 (x = x + 1/m)，为下一行做准备。
   4. 结束：循环直到处理完所有扫描线。

## openCV实现

openCV中的fillPoly实现了上述的扫描线填充算法，主要用到的函数有两个：CollectPolyEdges和FillEdgeCollection，第一个函数是收集多边形边缘，第二个函数是填充多边形，大概过程就是将与扫描线相交的边表

## halcon实现

opencv里的椭圆拟合函数并不是很好使，参照文献[Numerically Stable Direct Least Squares Fitting of Ellipses](https://www.semanticscholar.org/paper/Numerically-Stable-Direct-Least-Squares-Fitting-of-oy-Flusser/9a8607575ba9c6016e9f3db5e52f5ed4d14d5dfd)实现会好很多，该方法是数值稳定的，对于只占椭圆很小一段比例的圆弧效果很好。

![示例图片](/images/ellipse_dist_4.png)
