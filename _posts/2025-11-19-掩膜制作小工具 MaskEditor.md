---
layout: post
title: "掩膜制作小工具 MaskEditor"
date: 2025-11-19
tags: [Mask Generation]
comments: true
author: mwwz
---

自己用Qt开发了一个掩膜制作小工具，便于后续在 TemplateMatchQt.exe 中进行模板创建。

**基本操作：**
- 打开图像后，可使用以下方式交互：
  - 【鼠标滚轮】：缩放图像
  - 【左键拖拽】：
    - 在移动模式下（小手图标），平移图像
    - 在涂鸦模式下（十字图标），绘制掩膜区域
  - 【右键单击】：切换移动/涂鸦模式

**绘制设置：**
- 支持调节画笔大小与透明度
- 提供撤销上一步与重置全部绘制的功能

**保存输出：**
- 掩膜将保存为黑白图像，与原始图像位于同一目录
- 文件名自动添加"_mask"后缀（如：image_mask.png）
- TemplateMatchQt.exe 将自动加载该掩膜图像以创建模板

![示例图片](/images/mask_0.png "create_mask")
![示例图片](/images/mask_1.png "save_mask")
![示例图片](/images/mask_2.png "create_model_with_mask")