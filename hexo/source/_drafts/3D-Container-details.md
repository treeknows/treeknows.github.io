---
title: 3D-Container-details
tags:

---

# 背景

如何在Unity中显示一个Android App的画面？

# 实现思路

在Unity中申请一个Texture，但是Texture的source部分由Android提供。



> OpenGL ES中使用最多的有两种纹理类型：
>
> - TEXTURE_2D：标准的RGB格式图像数据
> - TEXTURE_EXTERNAL_OES：Android特有的OES纹理，预览相机或者视频使用此纹理，通过SurfaceTexture转换得到
