---
title: ATW概述-by Michael Antonov
date: 2024-01-04 13:57:50
tags:
  - XR
  - Rendering
comment: true
categories: XR
---


---

 **翻译文章，原文链接：[Asynchronous Timewarp Examined](https://developer.oculus.com/blog/asynchronous-timewarp-examined/)**

**Written by: Michael Antonov • 2015年3月3日**

---

文章格式安排如下：

> 英文原文

中文译文

（自言自语）

---

<!-- more-->

> ***TL;DR:** Asynchronous timewarp (ATW) is a technique that generates intermediate frames in situations when the game can’t maintain frame rate, helping to reduce judder. However, ATW is not a silver bullet and has limitations that developers should be aware of.*

*Too long, don't read. 异步时间扭曲(ATW)是一种在游戏无法保持帧率时生成中间帧的技术，帮助减少抖动。然而，ATW并不是万能的，它也有一些局限性，开发人员应当知晓。*

> **Intro**
>
> Over the past year there’s been a lot of excitement around asynchronous timewarp (ATW). Many hoped that ATW would allow engines to run and render at a lower frame rate, using ATW to artificially fill in dropped frames without a significant drop in the VR quality.
>
> On Gear VR Innovator Edition, ATW has been a key part of delivering a great experience. Unfortunately, it turns out there are intrinsic limitations and technical challenges that prevent ATW from being a universal solution for judder on PC VR systems with positional tracking like the Rift. Under some conditions, the perceptual effects of timewarp judder in VR can be almost as bad as judder due to skipped frames.
>
> In this blog, we analyze these limitations and situations that cause particular difficulties. As you’ll see, while ATW may be helpful at times, there is no substitute for hitting the full frame rate when it comes to delivering great VR.

**介绍**

过去一年里人们对ATW表现出了很大的兴奋。许多人希望ATW能让引擎以更低的帧率运行和渲染，利用ATW认为的补充丢帧，而不会明显降低VR的质量。

在Gear VR创新版上，ATW成为了提供出色体验的关键部分。遗憾的是，由于存在一些内在限制和技术难题，ATW无法成为解决像Rift等带有位置追踪功能的PC VR系统上抖动问题的通用方案。在一些情况下，在虚拟现实中应用ATW后的效果和由跳帧引起的抖动效果一样糟糕。

在本博客中，我们将分析这些限制以及造成特殊困难的情况。正如你将看到的，ATW有时可能是有帮助的，但要提供出色的VR效果，全帧率是无可替代的。

（应该是指，帧率拉满的效果是ATW望尘莫及的）

> **Timewarp, Asynchronous Timewarp, and Judder**
>
> Timewarp is a technique that warps the rendered image before sending it to the display in order to correct for head motion that occurred after the scene was rendered and thereby reduce the perceived latency. The basic version of this is orientation-only timewarp, in which only the rotational change in the head pose is corrected for; this has the considerable advantage of being a 2D warp, so it does not cost much performance when combined with the distortion pass. For reasonably complex scenes, this can be done with much less computation than rendering a whole new frame.

**时间扭曲、异步时间扭曲和抖动**

TW是一种技术，在送显之前将渲染过的图像扭曲，从而纠正场景渲染后（渲染后到送显前这一段）发生的头部运动，减少感知延迟。最基础的时间扭曲是基于朝向（头部）的，这种方式只对头部姿势的旋转变化进行了校正。这种方法的显著优点是这是二维的扭曲，在与畸变矫正结合时不会太影响性能。对于相当复杂的场景，这比渲染一帧全新画面所需的计算量要少得多。

> Asynchronous timewarp refers to doing this on another thread in parallel (i.e. asynchronously) with rendering. Before every vsync, the ATW thread generates a new timewarped frame from the latest frame completed by the rendering thread.

ATW是指在一条和渲染线程并行的（扭曲）线程上工作。在每次VSync之前，ATW线程基于渲染线程完成的最新帧生成一帧扭曲过的帧。

> Judder and its consequences are covered in detail by Michael Abrash in his [2013 blog posts here](http://blogs.valvesoftware.com/abrash/). Reviewing Michael’s notes on judder would be helpful to get the most out of this post.

Michael Abrash（作者本人）在其 2013 年的博文（原文中贴的链接已失效）中详细介绍了抖动及其后果。阅读迈克尔关于抖动的笔记将有助于您更好地理解本篇文章。

> In order to produce a perceptually correct representation of the virtual world, the images on the displays must be updated with every vsync refresh. However, if rendering takes too long, a frame will be missed, resulting in judder. This is because when no new frame has been rendered, the video adapter scans out the same image a second time. Here is what an object location looks like if the same rendered frame is displayed two frames in a row before updating:
>
> ![double-image-judder](https://raw.githubusercontent.com/treeknows/blog_pic/master/imgs/double-image-judder.jpg)
>
> *Here, the eye is rotating to the left. When the same image is displayed again, its light falls on a different part of the retina, resulting in double image judder.*

为了在虚拟世界呈现正确的感知，屏幕图像必须在每次VSync时刷新。然而，如果渲染耗费太长时间，就会错过一帧，这就导致了抖动。这是因为当没有渲染新的帧时，视频适配器对同一张图像进行了二次扫描。在更新前连续显示同一渲染帧时，对象位置的情况如下图所示：

![double-image-judder](https://raw.githubusercontent.com/treeknows/blog_pic/master/imgs/double-image-judder.jpg)

*在这里，眼睛向左旋转。当再次显示同一图像时，光线落在视网膜的不同部分，导致重影抖动。*

> Of course, doubling is not the only possible effect. If we displayed the same frame three times in a row, you would get a triple image on your retina and so on.

当然，重影不是唯一可能的效果。如果我们连续显示同一帧三次，你会看到三重图像，以此类推。

> Orientation-only ATW can be used to help address judder: if the rendered game frame is not submitted before vsync, timewarp can interrupt and generate the image instead, by warping the last frame to reflect the head motion since the last frame was rendered. Although this new image will not be exactly correct, it will have been adjusted for head rotation, so displaying it will reduce judder as compared to displaying the original frame again, which is what would have happened without ATW.

基于方向的ATW可以帮助解决抖动问题：如果渲染的游戏帧没有在VSync到来前提交，TW会中断（渲染线程）并生成图像，通过扭曲最后一帧来代表最后一帧以来的头部运动。虽然新的图像并不完全正确，但它已经根据头部旋转进行了调整，因此显示这一帧（扭曲生成的帧）相比在没有ATW的情况下显示同一帧（上一帧），可以减少抖动。

> In certain situations, simple rotation-warping can work well. It has been implemented on Gear VR Innovator Edition, where it fills in the frames whenever games can’t meet the frame rate. This smooths many glitches to the point where they’re mostly not noticeable. Because Gear VR lacks position tracking and the content generally avoids near-field objects, many of the artifacts discussed below are lessened or avoided.

在某些确定的场景下，简单的旋转-扭曲效果会很好。在Gear VR创新版上已经实现：当游戏帧无法满足要求时，它就会填充帧数。这使许多故障变得平滑，在大多数情况下都不易察觉。由于 Gear VR 缺乏位置跟踪功能，而且内容一般都会避开近场物体，因此下面讨论的许多假象都会减少或避免。

（在计算机图形学中，我们以绘制出以假乱真的图景为目标，但是经常会绘制出来锯齿状边缘，或者一些颜色错误，我们会称之为**artifact**，意思是这里绘制得不自然）

> There are several reasons why ATW on the PC is significantly more challenging than on Gear VR, starting with the Rift’s support for positional tracking.

PC 上的 ATW 比 Gear VR 上的更具挑战性，原因有几个，首先是 Rift 支持定位追踪。

> **Positional Judder**
>
> Positional judder is one of the most obvious artifacts with orientation-only timewarp. When you move your head, only the additional rotational component is reflected in the ATW-generated images, while any translational head movement since the frame was rendered is ignored. This means that as you move your head from side to side, or even just rotate your head which translates your eyes, you will see multiple-image judder on objects that are close to you. The effect is very noticeable in spaces with near field objects, such as the submarine screenshot below.
>
> ![multiple-image-judder-with-near-field-objects](https://raw.githubusercontent.com/treeknows/blog_pic/master/imgs/multiple-image-judder-with-near-field-objects.jpg)

**位置抖动**

位置抖动是基于方向的时间扭曲带来的最明显的瑕疵之一。当移动头部时，只有额外的旋转部分会反映在ATW生成的图像中，而自帧被渲染以来的所有头部平移都会被忽略。这就意味着当左右移动头部，甚至只是旋转头部使眼睛平移时，将在靠近的物体上观察到多重图像抖动。这种效果在有近场物体时非常明显，例如下面的潜艇截图：

![multiple-image-judder-with-near-field-objects](https://raw.githubusercontent.com/treeknows/blog_pic/master/imgs/multiple-image-judder-with-near-field-objects.jpg)

> So, how bad is this effect?
>
> The magnitude of positional judder depends on the environment the player is in and the types of movements they make. If you keep your head relatively still and only look around at the scenery, the positional error will be small and the judder will not be very noticeable.
>
> **Note:** This is normally the case for Gear VR Innovator Edition, which doesn’t include positional tracking. Nevertheless, the head model generates virtual translations, so when a game is running at half-rate on Gear VR, you can still observe positional judder on near-field objects.

那么，这种影响有多严重？

位置抖动的程度取决于玩家所处的环境和所做的动作类型。如果你的头部保持相对静止，只是环顾四周的景物，那么位置误差会很小，抖动也不会很明显。（说白了就是不做头部平移？）

**注意**：Gear VR 创新版通常就是这种情况，因为它不包含位置跟踪功能。不过，头部模型会产生虚拟平移，因此当游戏在 Gear VR 上以半速率运行时，您仍然可以观察到近场物体的位置抖动。

> If you’re looking at objects far away, the displacement change due to your head movement is unlikely to be significant enough to be noticeable. In these cases, ATW allows you to look around a scene with mid-to-far-field geometry without any noticeable judder.
>
> On the other hand, if you’re in a environment with near-field detail, and you translate your head, the positional judder will be nearly as bad as running without ATW. This will also be true when you look down at a textured ground plane, which is not far enough away to avoid artifacts. The resulting perceptual effect is that of a glitchy, unstable world, which can be disorienting and uncomfortable.

如果你正在看远处的物体，头部移动造成的位移变化不太可能大到足以引起注意。在这种情况下，ATW使你环视带有中远景几何图形的场景而不引起明显的抖动。

另一方面，如果处于带有近景细节的环境，并且移动了头部，那么位置抖动将与不使用ATW时一样严重。低头看有纹理的地面也是如此。因为距离不够远，无法避免伪影。由此产生的感知效果是一个闪烁、不稳定的时间，会让人迷失方向，感到不舒服。

> **Positional Timewarp**
>
> One possible way to address positional judder is to implement full positional warping, which applies both translation and orientation fixups to the original rendered frame. Positional warping needs to consider the depth of the original rendered frame, displacing parts of the image by different amounts. However, such displacement generates dis-occlusion artifacts at object edges, where areas of space are uncovered that don’t have data in the original frame.
>
> Additionally, positional warping is more expensive, can’t easily handle translucency, has trouble with certain anti-aliasing approaches, and doesn’t address the other ATW artifacts discussed below.\

**位置扭曲**

解决位置抖动的一种可能的方法为实现全位置扭曲，对原始渲染帧进行方向（朝向）和平移修正。位置扭曲需要考虑原始渲染帧的深度信息，对图像的不同部分进行不同程度的位移。然而，这样产生的中间帧会使物体边缘不闭合，由于没有原始帧中的数据，会导致中间帧某些区域不能被覆盖。

此外，位置扭曲成本更高，无法轻松处理半透明效果，在使用某些抗锯齿方法时会出现问题，而且无法解决下文中讨论的其他ATW的情形。

> **Moving and Animated Objects**
>
> Animated or moving objects cause another artifact with ATW: because a new image is generated just by warping the original image without knowledge of the movement of objects, for all ATW-generated frames they are effectively frozen in time. This artifact manifests as multiple images of these moving objects — i.e. judder.
>
> ![moving-object-affected-by-judder](https://raw.githubusercontent.com/treeknows/blog_pic/master/imgs/moving-object-affected-by-judder.jpg)
>
> <center>Image of scene with a moving object affected by judder.</center>

**移动和动画物体**

动画或移动中的物体导致了另一个ATW中的瑕疵：由于新图像是在不知道物体移动的情况下通过扭曲原始图像生成的，因此在所有ATW生成的帧中，这些物体实际上都被冻结在时间中。这种伪像表现为这些移动物体的多幅图像，即抖动。

![moving-object-affected-by-judder](https://raw.githubusercontent.com/treeknows/blog_pic/master/imgs/moving-object-affected-by-judder.jpg)

<center>Image of scene with a moving object affected by judder.</center>

> The impact of this artifact depends on the number, projected area, and speed of animated objects in the game scene: if the number or size of moving objects is small or they are not moving fast, the multiple images may not be particularly noticeable. However, when moving objects or animation covers a large portion of the screen it can be disturbing.
>
> Additionally, the frame rate ratio between the game rendering and device refresh rate affects the perceived quality of the motion judder. In our experience, ATW should run at a fixed fraction of the game frame rate. For example, at 90Hz refresh rate, we should either hit 90Hz or fall down to the half-rate of 45Hz with ATW. This will result in image doubling, but the relative positions of the double images on the retina will be stable. Rendering at an intermediate rate, such as 65Hz, will result in a constantly changing number and position of the images on the retina, which is a worse artifact.

这种伪影的影响取决于游戏场景中动画的数量、投影面积和速度：如果正在移动的物体的数量较少或形状较小或移动速度不快，那么重影可能不会太明显。但是，当移动的物体或动画占据了屏幕的大部分区域，就会造成干扰。

此外，游戏渲染帧率和屏幕刷新率之间的比值也会影响抖动的感知质量。根据经验，ATW应该以游戏帧率的固定比例运行。例如，在90赫兹的刷新率下，ATW要么达到90Hz，要么降到45Hz运行。这将导致图像加倍，但双图像在视网膜上的相对位置将保持稳定。以65Hz这样的中间帧率渲染会导致视网膜上图像的数量和位置不断改变，从而产生更糟糕的伪影。

> **Specular and Reflection**
>
> Calculations for reflections and specular lighting consider the direction of the eye vector, or rendering camera vector, to produce an image that is custom rendered for each eye.
>
> ![specular-and-reflection](https://raw.githubusercontent.com/treeknows/blog_pic/master/imgs/specular-and-reflection.jpg)
>
> ![specular-and-reflection2](https://raw.githubusercontent.com/treeknows/blog_pic/master/imgs/specular-and-reflection2.jpg)

**镜面反射**

计算镜面反射需考虑眼睛的方向， 或摄像机（Unity里的camera？）的方向， 由此为每只眼睛生成一张定制的渲染画面。

![specular-and-reflection](https://raw.githubusercontent.com/treeknows/blog_pic/master/imgs/specular-and-reflection.jpg)

![specular-and-reflection2](https://raw.githubusercontent.com/treeknows/blog_pic/master/imgs/specular-and-reflection2.jpg)

> *Diagrams courtesy of Wikipedia and http://ogldev.atspace.co.uk/www/tutorial19/tutorial19.html respectively.*
>
> Since this eye vector changes with head movement, specular and reflection rendering is no longer correct after timewarp. This may result in reflections and specular highlights juddering.
>
> While specular highlights and reflections are two of the most common cases where shading relies on the eye vector, many other eye vector-dependent shading tricks will have similar issues. For example, parallax mapping and relief mapping (aka. parallax occlusion mapping) will show similar artifacts.

图片来自维基百科和[网站]( http://ogldev.atspace.co.uk/www/tutorial19/tutorial19.html)

由于眼睛向量（？）会随着头部运动而改变，因此在TW后，镜面反射渲染将不再正确。这可能会导致镜面反射抖动。

虽然镜面高光和反射是依靠眼球矢量着色的两种最常见情况，许多其他依赖眼球向量的着色技巧也有类似的问题。例如，视差贴图和浮雕映射(又称为视差遮挡映射)也会有类似的现象。

> **Implementation**
>
> Implementing ATW is challenging for two primary reasons:
>
> - It requires GPU HW to support preemption at reasonable granularity
> - It requires OS and driver support to expose GPU preemption
>
> Let’s start with preemption granularity. At 90Hz, the interval between frames is roughly 11ms. This means that in order for ATW to have any chance of generating a frame, it must be able to preempt the main thread rendering commands and run in under 11ms.

**ATW实现**

实现ATW有两个主要的技术挑战：

- 需要GPU硬件以合理的粒度支持抢占
- 需要操作系统和驱动支持GPU抢占

让我们从抢占粒度开始。在90Hz时，帧间隔大约是11ms。这意味着为了使ATW能有机会生成一帧，必须能够抢占主线程的渲染命令，并在11ms内运行。

> However, 11ms isn’t actually good enough — If ATW runs at randomly scheduled points within the frame, its latency (ie. the amount of time between execution and frame scan-out) will also be random. And, we need to make sure we don’t skip any game-rendered frames.
>
> What we really want is for ATW to run consistently shortly before the video card flips to a new frame for scan-out, with just enough time to complete the generation of the new timewarped frame. Short of having custom vsync-triggered ATW interrupt routines, we can achieve this with high-priority preemption granularity and scheduling of around 2ms or less.

然而，11ms实际上还不够好：如果ATW在一帧内随机的时间点开始运行，其延迟(执行和帧扫描之间的时间间隔)也将是随机的。而且，我们需要确保不会调过任何游戏渲染的帧。

我们真正想要的是ATW能够有刚刚足够的时间，在显卡切到新帧扫描输出前结束生成新的一帧。如果不使用定制的垂直同步触发的ATW中断例程，我们可以通过高优先级的抢占性调度，以及大约2毫秒或更短的时间来实现这一点。（它在说啥 :sweat_smile:）

> It turns out that 2ms preemption on general rendering is a tall order for modern video cards and driver implementations. Although many GPUs support limited forms of preemption, the implementation varies significantly:
>
> - Some vendors and drivers allow preemption on either batch or draw call granularities. While helpful, this is not perfect (eg. in an extreme case, a single large draw call with a complex shader can easily take 10 ms).
> - Other vendors and drivers allow preemption of compute shaders, yet require vendor-specific extensions to support preemption of rendering with compute.

事实证明，对于现代显卡及驱动实现来讲，在一般的渲染中实现2ms抢占是一项艰巨的任务。虽然许多GPU都支持有限形式的抢占，但实现的方式确不尽相同。

- 一些供应商和驱动允许在批处理或绘制调用粒度上进行抢占，这虽然有用，但是并不完美(例如，在极端情况下，复杂着色器的单个大型绘制调用可能轻松耗费10ms时间)。
- 其他供应商和驱动允许计算着色器的抢占，但是需要特定供应商的拓展来支持计算渲染的抢占。

> If the preemption doesn’t occur quickly enough, ATW will not complete warping a new frame before vsync, and the last frame will be shown again, resulting in judder. This means that a correct implementation should be able to preempt and resume rendering arbitrarily, regardless of the pipeline state. In theory, even triangle-granularity preemption is not good enough because with complex shaders we don’t know how long rendering a triangle will take. We’re working with GPU manufacturers to implement better preemption, but it will be awhile before it’s ubiquitous.
>
> Another part of the equation is rendering preemption support in the OS. Prior to Windows 8, Windows Display Driver Model (WDDM) supported limited preemption using “batch queue” granularity, where batches were built by the graphics driver. Unfortunately, graphics drivers tend to accumulate large batches for rendering efficiency, resulting in preemption that is too coarse to support ATW well.
>
> With Windows 8, the situation improved as WDDM 1.2 added support for preemption at finer granularities; however, these preemption modes are currently not universally supported by graphics drivers. Rendering pipeline management is expected to improve significantly with Windows 10 and DirectX 12, which gives developers lower-level rendering control. This is good news, but we’re still left without a standard way to support rendering preemption until Windows 10 becomes mainstream. As a result, ATW will require vendor-specific driver extensions for the foreseeable future.

如果抢占动作不够快，ATW将无法在VSync到来前扭曲生成新的帧，最后一帧将再次显示，导致抖动。这就意味着无论流水线的状态如何，正确的实现都应该能够随意的抢占和渲染。理论上，即便是三角粒度抢占（<--没太明白是个啥）也不够好，因为对于复杂着色器，我们不知道渲染一个三角形需要多长时间。我们正与GPU制造商合作，实现更好的抢占功能，但要想普及还需要一段时间。

另一个因素是操作系统对渲染抢占的支持（equation 方程式）。在Win8之前，Windows显示驱动程序模型(WDDM)支持使用“批队列”粒度的有限抢占，其中批次由图形驱动程序构建。不幸的是，图形驱动往往会积累大量批次以提高渲染效率，这就导致抢占过于粗糙，无法很好地支持ATW。

在Win8中，情况有所改善，因为WDDM1.2支持了更细粒度的抢占；不过这些抢占模式目前还没有被图形驱动程序普遍支持。渲染管线管理预计将在Win10和DirectX 12上有显著改善，为开发人员提供更低级别的渲染控制。这是一个好消息，但是在Win10成为主流之前我们仍然缺少支持渲染抢占的标准方法。因此，在可见的将来，ATW仍将依赖特定供应商的驱动程序拓展。

> **ATW is helpful, but it’s not a silver bullet**
>
> Once we have ubiquitous GPU rendering pipeline management and preemption, ATW may become another tool to help developers increase performance and reduce judder in VR. However, due to the issues and challenges we’ve outlined here, ATW is not a silver bullet — VR applications will want to sustain high framerates to deliver the best quality of experience. In the worst cases, ATW’s artifacts can cause users to have an uncomfortable experience. Or stated differently: in the worst cases, ATW can’t prevent an experience from being uncomfortable.
>
> Given the complexities and artifacts involved, it’s clear that ATW, even with positional timewarp, won’t become a perfect universal solution. This means that both orientation-only and positional ATW are best thought of as pothole insurance, filling in when a frame is occasionally skipped. To deliver comfortable, compelling VR that truly generates presence, developers will still need to target a sustained frame rate of 90Hz+.

**ATW有用，但不万能**

一旦我们拥有了无处不在的GPU渲染管线管理和抢占，ATW可能会成为帮助开发者提升VR性能，减少抖动的另一种工具。然而，基于上述的一些问题和挑战，ATW并不是万能的——VR应用期望维持高帧率以提供最好的体验效果。在最坏的情况下，ATW的伪影可能导致用户有不舒服的使用体验。换句话说，在最坏的情况下，ATW无法阻止体验变得不舒服。

鉴于所涉及的复杂性和人工痕迹，很明显对于ATW，甚至位置扭曲，不会成为一个完美的通用解决方案。这就意味着无论3dof的扭曲还是6dof的扭曲最好作为一个保险，在偶尔跳帧时填补空白。想要提供舒适、真实的VR体验，开发者仍需要将帧率保持在90Hz以上。

> Thankfully, Crysis-level graphics are by no means required to deliver incredible VR experiences. It’s perfectly reasonable to reduce the number of lights, the shadow detail, and the shader complexity if it means reaching that 90Hz sweet spot.
>
> Dual-mode titles that try to support traditional monitors and VR will have the most performance difficulties, as the steep performance requirements for good VR quickly become a challenge to engine scalability. For developers in this situation, ATW may look very attractive despite the artifacts. However, as is typical with new mediums, ports are unlikely to be the best experiences; made-for-VR experiences that target 90Hz are likely to be substantially more successful in generating comfort, presence, and the true magic of VR.
>
> — Michael Antonov, Chief Software Architect

幸好，要提供惊艳的VR体验，不需要《孤岛危机》级别的图形。只要能够达到90Hz的最佳频率，减少灯光数量、阴影细节或着色器复杂度都是完全合理的。

试图同时支持传统显示器和VR的双模式游戏会遇到很大的性能问题，因为良好的VR对性能的很快就会成为对引擎可拓展性的挑战。在这种情况下，对开发者来说ATW会有十足的吸引力，尽管看起来会不自然。然而，与新媒体的典型情况一样，移植不可能带来最佳体验；以90Hz为目标的专门的VR体验可能会在产生舒适感、临场感和其他VR的神奇之处收获更多的成功。

—Michael Antonov, Chief Software Architect

---

正文结束😀

