---
layout: post
title: "mwwz与halcon在精度上的对比：终结篇"
date: 2026-06-08
tags: [shape match]
comments: true
author: mwwz
---

在`halcon`的`find_shape_model`算子中，`SubPixel`参数提供了`least_squares`、 `least_squares_high`和`least_squares_very_high`等选项。不难推测，它们之间的核心区别在于：**精度要求越高，参与拟合的边缘点越多、最大迭代次数越多，且收敛阈值越苛刻。** 因此，我改为让所有边缘点参与拟合，新的对比结果如下：

![result](/images/result_ls3.png "result")

![mwwz-a](/images/mls_a.png "mwwz-a")

![mwwz-b](/images/mls_b.png "mwwz-b")

![mwwz-c](/images/mls_c.png "mwwz-c")

![mwwz-d](/images/mls_d.png "mwwz-d")

![mwwz-e](/images/mls_e.png "mwwz-e")

![mwwz-f](/images/mls_e.png "mwwz-f")

![mwwz-g](/images/mls_e.png "mwwz-g")

![mwwz-h](/images/mls_e.png "mwwz-h")

![mwwz-i](/images/mls_e.png "mwwz-i")

![mwwz-j](/images/mls_e.png "mwwz-j")

![mwwz-k](/images/mls_e.png "mwwz-k")

## 立即下载

**[下载地址](https://www.mwwztech.com/)**
