---
title: Lecture-7-Software-Tools
date: 2021-07-13 19:43:27
tags:
---

{% youtube 'A3LyARGg8kU' %}

本节课零散地介绍了一些摄影爱好者的常用软件，罗列如下：

| 软件名称                        | 运行平台      | 功能简介                                                     |
| ------------------------------- | ------------- | ------------------------------------------------------------ |
| Manual - Custom exposure camera | iOS           | 支持手动模式控制 iPhone 相机                                 |
| Camera+                         | iOS           | iPhone 相机拍摄、后期功能增强                                |
| GaiaGPS                         | iOS/Android   | GPS 路线追踪，可与相片拍摄时间结合，知道照片是在何时、何地拍摄 |
| Photoshop (PS)                  | MacOS/Windows | 强大、完备的照片后期处理功能                                 |

后半节课集中在讲解 PS 的一些基本功能，以实践为主，内容的系统性、完备性肯定不如专门的 PS 教程，因此这里不会详细记录，简单罗列一下要点：

* Adobe Camera Raw: Adobe 提供的各厂商相机 raw 格式图片导入工具，因为 raw 格式保留更多的图片信息，通常会在这时候对照片进行整体调整。
  * Middle Gray：选择中间灰，使得曝光调整地尽量准确
  * Exposure：raw 格式图片通常支持两个 stops 的曝光量调节，因为相机本身的特点，从暗调亮的效果要比从亮调暗的好
  * Contrast：图片在拍摄的时候实际上就已经按照人眼的敏感感光习惯做了[映射](/opencourse-notes/DGMD-E-10/Lecture-4-Exposure-Continued/)，在此基础上调节对比度，实际上就是再叠加一层映射关系，将中间亮度附近的像素往黑白两个方向拉扯
  * Saturation：调节色彩的饱和度，通常是无差别调整，让画面中的所有颜色更鲜艳或更黯淡
  * Viberance：是不同后期软件对 Saturation 调节的升级，通常体现在
    * 调整饱和度的同时，不影响画面中人物皮肤的饱和度
    * 如果一些色彩已经很饱和的时候，不再提升它们的饱和度
  * Camera Calibration：一些相机，特别是无反相机，由于其工艺方面的原因，在四个角上的表现力受限，这时候可以通过选择相机的厂商和型号来自动修补
* Levels：这里的 level 指的是灰度，可以被用来调节黑、白、中间灰的边界，用以压缩图片的黑白边界，提高黑白的表现力
* Curves：curves 在功能上是 levels 的超集，但可以支持非线性、多通道 (RGB) 的映射，比 levels 更加灵活。
* Layers：主要用于图片的非破坏性编辑，可以通过 mask 对局部区域应用
