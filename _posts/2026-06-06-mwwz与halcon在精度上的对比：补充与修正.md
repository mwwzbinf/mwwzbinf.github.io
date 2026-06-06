---
layout: post
title: "mwwz与halcon在精度上的对比：补充与修正"
date: 2026-06-06
tags: [shape match]
comments: true
author: mwwz
---

## 1. 待测试图像生成说明

- 旋转中心固定于图像中心`(320,240)`，始终不变。

- 生成的旋转矩形关于旋转中心对称，且适用于任意角度。

 使用`gen_rectangle2_contour_xld`算子验证对称性，如下图所示：

![gen_rectangle2_contour_xld 22.5](/images/gen_rect2_xld_0.png "gen_rectangle2_contour_xld 22.5")

![gen_rectangle2_contour_xld 75](/images/gen_rect2_xld_1.png "gen_rectangle2_contour_xld 75")

## 2. halcon中的两种坐标系

- **Pixel Centered（标准坐标系）**：原点位于左上角像素的中心。

- **Edge Centered（像素坐标系）**：原点位于左上角像素的左上角。

上一篇博客中，`find_shape_model`返回的`（x,y）`为像素坐标系下的坐标。使用`affine_trans_pixel`对其进行转换，对比结果如下：

![result](/images/result_ls2.png "result")

![halcon-a](/images/hls_a.png "halcon-a")

![halcon-b](/images/hls_b.png "halcon-b")

![halcon-c](/images/hls_c.png "halcon-c")

![halcon-d](/images/hls_d.png "halcon-d")

![halcon-e](/images/hls_e.png "halcon-e")

## 立即下载

**[下载地址](https://www.mwwztech.com/)**
