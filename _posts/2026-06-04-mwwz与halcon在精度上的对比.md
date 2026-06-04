---
layout: post
title: "mwwz与halcon在精度上的对比"
date: 2026-06-04
tags: [shape match]
comments: true
author: mwwz
---

## 1. 准备工作

在 `HDevelop`中使用以下代码生成模板图像和待测试图像：

- **模板图像**：尺寸为`401×301`，在中心坐标`(200,150)`处绘制角度为0、半宽`160`、半高`110`的矩形。

- **待测试图像**：尺寸为`641×481`，在中心坐标`(320,240)`处绘制半宽`160`、半高`110`、角度为`θ`的矩形。

- 角度`θ`分别取：`0°`、`11.25°`、`22.5°`、`30°`、`37.5°`、`45°`、`60°`。

![gen image](/images/gen_ls.png "gen image")

## 2. 开始对比

- 起始角度设为`0°`，角度范围为`90°`。

- 启用最小二乘法：

  - mwwz中`亚像素`选择`最小二乘法`

  - halcon中`亚像素`选择`least_squares_very_high`

理论上，匹配结果应为`(320,240,θ)`。详细对比结果如下图所示，如有错误，欢迎指正。

![halcon-0](/images/hls_0.png "halcon-0")

![mwwz-0](/images/mls_0.png "mwwz-0")

![halcon-11.25](/images/hls_1.png "halcon-11.25")

![mwwz-11.25](/images/mls_1.png "mwwz-11.25")

![halcon-22.5](/images/hls_2.png "halcon-22.5")

![mwwz-22.5](/images/mls_2.png "mwwz-22.5")

![halcon-30](/images/hls_3.png "halcon-30")

![mwwz-30](/images/mls_3.png "mwwz-30")

![halcon-37.5](/images/hls_4.png "halcon-37.5")

![mwwz-37.5](/images/mls_4.png "mwwz-37.5")

![halcon-45](/images/hls_5.png "halcon-45")

![mwwz-45](/images/mls_5.png "mwwz-45")

![halcon-60](/images/hls_6.png "halcon-60")

![mwwz-60](/images/mls_6.png "mwwz-60")

## 立即下载

**[下载地址](https://www.mwwztech.com/)**
