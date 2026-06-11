---
layout: post
title: "mwwz与halcon在精度上的对比：HDevelop官方例程验证"
date: 2026-06-11
tags: [shape match]
comments: true
author: mwwz
---

为了进一步评估`mwwz`与`halcon`的形状匹配精度，我使用`HDevelop`中的经典例程`first_example_shape_matching.hdev`进行测试，仅将参数`SubPixel`修改为`least_squares_very_high`。两者采用相同测试集，测试分为两组：第一组，`mwwz`在最小二乘拟合时使用部分边缘点；第二组，`mwwz`使用全部边缘点。测试结果如下：

![result-partial](/images/result_ls4.png "result-partial")

![result-all](/images/result_ls5.png "result=all")

![halcon-board-02](/images/hls_board_02.png "halcon-board-02")

![mwwz-board-02](/images/mls_board_02.png "mwwz-board-02")

![halcon-board-05](/images/hls_board_05.png "halcon-board-05")

![mwwz-board-05](/images/mls_board_05.png "mwwz-board-05")

![halcon-board-10](/images/hls_board_10.png "halcon-board-10")

![mwwz-board-10](/images/mls_board_10.png "mwwz-board-10")

## 立即下载

**[下载地址](https://www.mwwztech.com/)**
