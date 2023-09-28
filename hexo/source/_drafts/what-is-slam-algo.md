---
title: 认识SLAM算法
tags:
- XR
- Study Notes
comment: true

---



## 前言

因为工作原因，接触到了SLAM算法。一开始只是知道了有这么个词儿，慢慢的才试着去了解SLAM算法的含义。好记性不如烂笔头，更何况我现在没有好记性。



## 什么是SLAM算法

SLAM的全拼为Simultaneous Localization And Mapping，中文一般译为同步定位与地图构建。从名字可以看出，SLAM算法的核心有两点：

- 定位
- 建图

定位即获取自己当前所在空间环境中的位置，建图即将看到或感知到的环境记录下来，构建成环境地图。

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

在VR中，SLAM算法的输出就是**6DOF**(degree of freedom)，即orientation + position。其实这个6DOF指的头6，在VR中还有手6DOF和物体6DOF的概念。

## SLAM业务图

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

-  下面两个相机是主相机，需要下偏，确保充足的双目重叠
- 上面两个相机是辅相机，上偏，扩大跟踪FOV的同时兼顾前方盲区深度
- SLAM双目匹配，以及手柄、手势的主要活动区域，基本是以下面两个相机为主
- 四目相机和IMU之间要保证刚性连接，否则一旦发生位移，就需要重新标定



### 6DOF标定



### 业务架构


### SLAM算法

### SLAM测试

### 三维重建
