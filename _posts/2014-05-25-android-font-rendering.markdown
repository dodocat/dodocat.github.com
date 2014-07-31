---
layout: post
title: "android font rendering"
summary:
---

任何一个有一定年限客户端应用工作经验的开发者都会深切地意识到字符渲染可以多么复杂.
至少这是直到 2010 年我开始写 libwui 时候我的想法, 
libhwui 是一个 Android 3.0 2D 绘制 API 的 OpenGl 后端.
随后我发现当你尝试使用 GPU 绘制的时候, 文本甚至变得更加复杂.

# Text and adnroid

Android 的硬件加速字体渲染原本有我在 Renderscript 团队的一个同事写的,
随后由包括我的好基友 Chet House 和我在内的几个工程师优化提升.
容易找到很多关于如何使用 OpenGL 渲染文本的教程, 
但是至少大部分文章集中在游戏和简单的避免处理复杂问题上.


这里描述的方法绝不是小说, 
但是我认为它会方便一些想得到一份完整的关于基于 GPU 文本渲染系统是如何实现的的高端概括的开发者. 
本文还描述一些容易实现的优化.

用 OpenGL 渲染文本的一个常见方法是,
计算一个包含所有字形的纹理集. 
这经常用相当复杂的算法封装离线完成以让 texture 过程的浪费最小化.
创建这样一个集合明显需要事先知道所用字体,
包括了字体风格 大小 和其它众多属性 -- 而且图形字形会被应用在运行环境使用.


提前字体纹理生成并不是 Android 的实际解决方案.
UI 工具包无法提前知道应用程序需要什么字体和字形; 
应用甚至需要在运行时加载自定义字体. 这是一个主要的约束, 
    但是只是 Android 字体渲染运行必备条件之一:

* 必须在运行时构建字体缓存
* 必须能够处理大量字体 
* 必须能够处理大量字形
* 必须最小化 texture 消耗
* 必须快
* 低端高端设备上表现同样出色
* 完美地运行在任何驱动 GPU 组合上


# 实现字体渲染

在检验底级 OpenGL 字体渲染器如何工作之前,
让我先从应用直接使用的上层 API 开始. 这些 API 对理解 libhwui 如何工作很重要.

# Text APIs

有四个应用用于布局绘制文本的主要 API:

* `android.widget.TextView` 处理渲染和布局的 View
* `android.text.*` 一些列创建格式化的文本和布局的类
* `android.graphics.Paint` 计量文本
* `android.graphics.Canvas` 渲染文本

both TextView and android.text are high-level implementations on top of Paint and Canvas. Up until Android 3.0, both Paint and Canvas were implemented directly on top of Skia, a software rendering library. Skia provides a nice abstraction of Freetype, a popular Open Source font rasterizer.

TextView 和 android.text 都是 Paint 和 Canvas 的上层实现.
直到 Android 3.0, Paint 和 Canvas 是直接在 Skia 上实现的.
Skia 是一个渲染库提供很棒的 FreeType(开源...) 的抽象.

![Android software text rendering](./)


Android 4.4 的情况复杂一点. 
Paint 和 Canvas 都是使用一个 叫做`TextLayoutCache` 的内部 JNI API,
用于处理浮渣文本布局 (CTL). 此 API 依赖 Harfbuzz, 一个开源 text shaping 引擎. 
文本布局缓存 (TextLayoutCache) 的输入是一个字体和一个 Java UTF-16 字符串,
输出是一个字形及其 x/y 坐标的序列.


TextLayoutCache 是良好地支持如阿拉伯语 希伯来语 泰语等非拉丁语的关键.
我不打算解释 TextLayoutCache 和 Harfbuzz 如何运行的. 
但是如果你想, 你也应该, 在你的应用中正确地支持非拉丁语环境, 
我强烈你建议学习一下 CTL. 
在有关使用 OpenGL 渲染文本的的教程中极少有讨论这个问题的.
绘制文本比简单地从左到右依次放置字形复杂得多.
一些语言, 比如阿拉伯语, 从右到左, 还有一些, 像泰语, 甚至需要字形定位在前面字符的上面或下面.

![Android hardware accelerated text rendering](/image/android_hardware_accelerated_text_rendering.png)

待续...




