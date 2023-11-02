---
title: 认识SLAM算法
tags:
  - XR
  - Study Notes
comment: true
categories: XR
date: 2023-11-03 01:33:42
---




## 前言

因为工作原因，接触到了SLAM算法。一开始只是知道了有这么个词儿，慢慢的才试着去了解SLAM算法的含义。好记性不如烂笔头，更何况我现在没有好记性。



## 什么是SLAM算法

SLAM的全拼为Simultaneous Localization And Mapping，中文一般译为同步定位与地图构建。从名字可以看出，SLAM算法的核心有两点：

- 定位
- 建图

定位即获取自己当前所在空间环境中的位置，建图即将看到或感知到的环境记录下来，构建成环境地图。

<!-- more -->

因此，SLAM算法的目标可以总结为：**在没有任何先验知识的情况下，根据传感器数据实时构建周围的地图环境，同时根据这个地图推测自身的定位**。



![SLAM目标](https://raw.githubusercontent.com/treeknows/blog_pic/master/imgs/SLAM%E7%9B%AE%E6%A0%87.PNG)



SLAM算法的工作内容，可以形象的描述为：

当我们进入一个未知的环境时，

1. 特征标记：用”眼睛“观察周围地标，并记住他们的特征。
2. 在自己的”脑海“中，根据双眼获得的信息，把特征地标在三维地图中重建出来。
3. 当自己在行走时，不断获取新的特征地标，并且校正头脑中的地图模型。
4. 根据自己前一段时间行走获得的地表特征，确定自己的位置。

在此过程中，定位和建图是同时进行的。

### VR中的SLAM

VR中的SLAM所依靠的传感器为IMU和Camera，通过传感器定位自己在空间中的哪个位置，我在哪里（移动位置x,y,z），我在看哪个方向（旋转角度），同时建图以确定周边的环境是什么样子的。

#### 定位

在VR中，SLAM算法的输出就是**6DOF**(degree of freedom)，即orientation + position。其实这个6DOF指的头6，在VR中还有手6DOF和物体6DOF的概念。

#### 建图

 SLAM建立的地图可以分为三种：

- 稀疏地图
- 稠密地图
- 语义地图

![slam_map_result](https://raw.githubusercontent.com/treeknows/blog_pic/master/imgs/slam_map_result.png)

## SLAM Overview

### 硬件方案

#### 6DOF的硬件方案：Inside-Out和Outside-In

|     方案     | Outside-in 外向内                  | Inside-out 内向外                          |
| :----------: | :--------------------------------- | ------------------------------------------ |
|     原理     | 基于标记，需要固定在外部的追踪设备 | 基于无标记，只需VR上的摄像头，使用SLAM算法 |
|   追踪精度   | 精度高，小于1mm                    | 精度略低                                   |
|  可移动范围  | 仅限于传感器监测范围               | 移动范围无限制                             |
|   跟踪死角   | 正常几乎无死角                     | 有跟踪死角                                 |
|     延迟     | 延迟相对少                         | 有一定延迟                                 |
|    抗遮挡    | 易受遮挡影响                       | 无遮挡问题                                 |
| 事前环境准备 | 需要                               | 不需要                                     |

#### SLAM硬件：相机

| 单目相机     | 双目相机   | RGB-D相机        |
| ------------ | ---------- | ---------------- |
| 成本低       | 计算深度   | 主动测深度       |
| 距离不受限   | 距离不受限 | 重建效果好       |
| 尺度不确定性 | 配置复杂   | 测量范围小       |
| 初始化问题   | 计算量大   | 受日光和材质干扰 |

#### SLAM硬件： IMU

| 加速计                                                       | 陀螺仪                               | 磁力计                                   |
| ------------------------------------------------------------ | ------------------------------------ | ---------------------------------------- |
| 检测物体在载体坐标系统独立三轴的加速度信号，对单方向加速度积分即可得到方向速度 | 检测载体相对于导航坐标系的角速度信号 | 检测载体相对于地球磁场的东南西北方向信号 |
| ”我们又前进了几米“                                           | ”我们转了几圈“                       | ”我们向西偏北方向“                       |

#### SLAM硬件：相机和IMU互补

| 方案 | IMU                                                    | Camera                                         |
| ---- | ------------------------------------------------------ | ---------------------------------------------- |
| 优势 | 快速响应<br />不受成像质量影响<br />角速度普遍比较准确 | 不产生漂移                                     |
| 劣势 | 存在零偏<br />低精度IMU积分位姿发散                    | 受图像遮挡、运动物体干扰<br />快速移动时易丢失 |

#### SLAM硬件：摆放要求

![camera_place](https://raw.githubusercontent.com/treeknows/blog_pic/master/imgs/202311030004005.png)

-  下面两个相机是主相机，需要下偏，确保充足的双目重叠
- 上面两个相机是辅相机，上偏，扩大跟踪FOV的同时兼顾前方盲区深度
- SLAM双目匹配，以及手柄、手势的主要活动区域，基本是以下面两个相机为主
- 四目相机和IMU之间要保证刚性连接，否则一旦发生位移，就需要重新标定

### 6DOF标定

#### 标定的目的

拿拍照来讲，拍照即为将三维的场景通过某种变换得到一张二维的照片，标定的目的就在于计算出一个数学模型，从而使得通过该数学模型，能够由二维的照片反推出三维的场景。

![purpose_of_calibration_result](https://raw.githubusercontent.com/treeknows/blog_pic/master/imgs/purpose_of_calibration_result.png) 

#### 标定的好坏

重投影误差：真实三维空间点在图像平面上的投影（图像上的像素点）和重投影（用计算值得到的虚拟像素点）之间的差值。

![reprojection_error_result](https://raw.githubusercontent.com/treeknows/blog_pic/master/imgs/reprojection_error_result.png)

图中**e**即为计算得到的重投影误差值

### SLAM算法模块

- 传感器模块：数据采集

- 视觉里程计模块：特征匹配

- 后端模块：消除视觉里程计累计误差

- 地图模块：构建三维地图

- 回环检测模块：空间积累误差消除

SLAM系统可以分为前台线程和后台线程，其中：

前台线程接收传感器数据，进行SLAM初始化和特征跟踪与位姿实时求解，并输出设备实时位姿和三维点云。

后台线程进行局部或全局优化，减少误差累积；进行场景回路检测，并支持场景重定位。

### SLAM业务架构

![SLAM_ARCH](https://raw.githubusercontent.com/treeknows/blog_pic/master/imgs/SLAM_ARCH.png)

在当前主流的解决方案中，VR Service是比较重要的一个模块。比如高通的QVRService，QVR负责接收sensor的数据，并交由DSP上的SLAM算法做融合，生成的6DOF pose通过QVR传递给SDK/APP端。

### SLAM轨迹误差

绝对轨迹误差：真实轨迹点和估计轨迹点之间的差值

![ads_pose_error](https://raw.githubusercontent.com/treeknows/blog_pic/master/imgs/abs_pose_error.png)

相对轨迹误差：位姿变化量的插值

![relative_pose_error](https://raw.githubusercontent.com/treeknows/blog_pic/master/imgs/relative_pose_error.png)

### SLAM性能指标

| 绝对位姿误差 | 位置最大误差 < 10cm，位置平均误差 < 4cm<br />最大角度误差 < 4 degree，平均角度误差 < 1 degree |
| ------------ | ------------------------------------------------------------ |
| 相对位姿误差 | 位置最大误差 < 10mm，位置平均误差 < 1mm<br />最大角度误差 < 0.5 degree，平均角度误差 < 0.1 degree |
| 漂移/draft   | 无漂移，出现漂移，精度会非常差                               |
| 抖动/jitter  | 位置最大误差 < 3 mm，位置平均误差 < 0.5mm<br />最大角度误差 < 0.2 degree，平均角度误差 < 0.02 degree |
| 延迟         | 延迟 < 5ms                                                   |
| 重定位       | 重定位时间 < 1s，位置精度 < 1cm                              |
| 追踪范围     | 追踪范围 < 10m * 10m                                         |
| 光照敏感性   | 明亮、灰暗、漆黑                                             |
| 纹理敏感性   | 高纹理、低纹理、无纹理                                       |
| 跟踪敏感性   | 位姿多变、快速距离切换、移动物体背景、传感器短暂失效         |

## Video See Through

AR/VR显示技术可以分为OST和VST

- OST：光学透视，真实世界通过放置在用户眼前的半透明光学合成器看到的，同时光学合成器也被用来将计算机生成的图像反射到用户的眼睛里，从而将真实世界和虚拟世界结合起来。
- VST：视频透视，通过相机实时捕捉真实画面，再和虚拟世界该呈现的画面融合，最终呈现在显示屏幕上。

## 三维重建

常常被简写为3DR，全称为**3D Reconstruction**，其主要目的为实现更好的交互。

### 基本流程

稀疏点云重建--->稠密点云重建--->点云网格重建--->三维语义重建

![3DR_process](https://raw.githubusercontent.com/treeknows/blog_pic/master/imgs/3DR_process.png)

3DR的实现依赖于很多算法，如场景理解、语义分割、目标识别、位姿估计等

### 3DR框架

![3DR_framework](https://raw.githubusercontent.com/treeknows/blog_pic/master/imgs/3DR_framework.png)

当前还处于1.0时代，有了平面检测，但是也只是平面的检测，而不是桌面的检测或地面的检测或具体的物体平面。

## 后记

囫囵的介绍了一下SLAM算法，但是很多模块并没有写的很详细。6DOF标定部分还有很多不曾提到的内容：6DOF标定过程、6DOF标定文件、畸变矫正等等等等，酝酿酝酿再补充。



---

> 总的来讲，行文断断续续，越断越爽，越续越差。:expressionless:
