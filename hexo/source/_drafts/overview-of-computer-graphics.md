---
title: 【CG学习笔记】Overview of Computer Graphics
tags:
- Study Notes
- Computer Graphics
categories:
- 学习笔记
comment: true

---

>  开始自学计算机图形学，学习视频来自B站[GAMES101-现代计算机图形学入门-闫令琪](https://www.bilibili.com/video/BV1X7411F744?p=1)
>
>  本文仅为个人笔记

## 导航

[Games Lecture 1]()

 ## Topics

### What is Computer Grahpics?

视频中没有提到什么内容，好像只说了一个**合成和操作**？摘抄一段百度百科的解释：

> 计算机图形学(Computer Graphics，简称CG)是一种使用数学算法将[二维](https://baike.baidu.com/item/二维/380405?fromModule=lemma_inlink)或[三维图形](https://baike.baidu.com/item/三维图形/5612976?fromModule=lemma_inlink)转化为计算机显示器的[栅格](https://baike.baidu.com/item/栅格/7368256?fromModule=lemma_inlink)形式的科学。简单地说，计算机图形学的主要研究内容就是研究如何在计算机中表示图形、以及利用计算机进行图形的计算、处理和显示的相关[原理](https://baike.baidu.com/item/原理/85014?fromModule=lemma_inlink)与[算法](https://baike.baidu.com/item/算法/209025?fromModule=lemma_inlink)。

个人理解的话大致差不多，将二维或三维的图形显示在计算机屏幕上。只能说比较抽象，个人理解处于一种可思不可道的玄妙状态。

### Why study Computer Graphics?

- 这部分最后总结的一句话很nice：

  - Computer Graphics is **AWESOME**!

- 什么是好的画面：

  - 从技术角度来说，画面直接看起来是否足够亮

    > 体现在渲染中的一个关键技术：全局光照。如果全局光照做的好，画面看起来会比较明亮、舒服。如果画面看起来比较暗，可能就是因为技术不足。

- 图形学操作可视化的视觉信息

- 图形学在游戏、电影、动画、设计、可视化、虚拟现实、模拟（仿真）、GUI、Typography 等方面都有普遍的应用 

- **The Quick Brown Fox Jumps Over The Lazy Dog**，一句话包括了所有的26个字母

- 图形学包括很多东西，OpenGL只是其中的一个API

以上，都是一些零碎化的内容。

### Course Topics

#### Rasterization - 光栅化

什么是光栅化

- 把三维空间的几何形体显示在屏幕上

#### Curves and Meshes - 曲线和曲面

如何表示曲线，如何表示曲面

#### Ray Tracing - 光线追踪 

实时光线追踪

一切需要猜测的内容都是**计算机视觉**的范畴，

#### Animation/Simulation - 动画/仿真

模拟小球弹跳运动轨迹

### Course Logistics

- 推荐阅读书籍：
  - 虎书 - Fundamentals Of Computer Graphics

- 推荐使用IDE：
  - VS Code
