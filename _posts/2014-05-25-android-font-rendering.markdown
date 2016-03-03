---
layout: post
title: "Android Font Rendering"
summary:
---

任何一个有一定年限客户端应用工作经验的开发者都会深切地意识到字符渲染可以多么复杂.
至少这是直到 2010 年我开始写 libwui 时候我的想法,
libhwui 是一个 Android 3.0 2D 绘制 API 的 OpenGl 后端.
随后我发现当你尝试使用 GPU 绘制的时候, 文本甚至变得更加复杂.

# Text and Android

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

All this means that when you call Canvas.drawText(), directly or indirectly, the OpenGL renderer does not receive the arguments you send, but an array of numbers — the glyph identifiers — and an array of x/y positions.

所有这些意味着当你直接或间接地调用 `Canvas.drawText()` 的时候, OpenGL 渲染器不会收到你的参数, 而是一个字形标志的数组 - 一组 x/y 坐标

# 栅格化和缓存
Every draw call to the font renderer is associated to a single font. A font is used to cache individual glyphs. A glyph is in turn stored in a cache texture (but a cache texture can contain glyph from multiple fonts). The cache texture is an important object that holds multiple buffers: a list of free blocks, a pixel buffer, a handle to the OpenGL texture and a vertex buffer (the mesh).

每一次字体渲染器的绘制调用都与单独一个字体关联. 一个字体用于缓存单独字形. 一个自行按顺序存储缓存 texture( 但是一个缓存 texture 能存储多个字体的字形). 缓存 texture 是重要的对象, 其持有多个缓存: 一系列空闲区块, 像素缓存, 一个 OpenGL 的 Handler 以及一个端点缓存(网格)

![]()

The data structures used to store all these objects are fairly simple:

存储这些对象的数据结构是相似的:

* Fonts are stored in a LRU cache in the font renderer

* 字体存储在字体渲染器的一个 LRU 缓存中.
* Glyphs are stored in a map in each font (the key is the glyph identifier)
* 字形存储在每一个字体的一个 map 中.(key 是字形标志)
* Cache textures track free space with a linked list of blocks

* A pixel buffer is an array of uint8_t or uint32_t (for alpha and RGBA caches)
* 像素缓存是 uint8_t 或 uint32_t 的数组(用于 alpha 和 RGBA 缓存)
* A mesh is a buffer of vertex with two attributes: x/y positions and u/v coordinates
* 网格是端点的数组, 端点有两个参数: x/y 位置和 u/v 坐标
* A texture is a GLuint handle
* texture 是一个 GLuunt handle


When the font renderer initializes, it creates two types of cache textures: alpha and RGBA. Alpha textures are used to store regular glyphs; since fonts do not contain color information, we only need to store anti-aliasing information. The RGBA caches are used to store emojis.

当字体渲染器初始化的时候, 它创建两个类型的缓存 texture: alpha 和 RGBA. Alpha textures 用于存储常规字形; 因为字体不包含颜色信息, 只有康据此信息是需要存储的. RGBA 缓存用于存储 emojis.

For each type of cache texture, the font renderer creates several instances of CacheTexture, of various sizes. The size of the caches can be different from device to device but here are the default sizes (the number of caches is hard coded):

对于每一种缓存 texture, 字体渲染器会创建几个不同尺寸的 `CacheTexture` 实例, 缓存的尺寸因设备而异, 这里是不同设备的默认值(缓存数量是被写死的)
* 1024x512 alpha cache
* 2048x256 alpha cache
* 2048x256 alpha cache
* 2048x512 alpha cache
* 1024x512 RGBA cache
* 2048x256 RGBA cache

When a CacheTexture object is created, its underlying buffers are not automatically allocated. The font renderer allocates them as needed, except for the 1024x512 alpha cache, which is always allocated.

当一个 `CacheTexture` 被创建的时候, 除了 1024x512 alpha cache 以外, 其下其它的各个缓存并没有自动分配空间, 字体渲染器在需要的时候自动分配它们.

Glyphs are packed in the textures in columns. Whenever the renderer encounters a glyph that is not cached, it asks each CacheTexture of the appropriate type — in the order listed above — to cache that glyph.

字形按列封装在 textures 中. 无论何时当时渲染器遇到一个未缓存的字形是, 它都要求响应类型的每一个 CacheTexture -在上面的顺序列出- 缓存此字形.

This is where the list of blocks gets used. That list contains, for a given cache texture, the list of currently allocated columns plus all the available empty space. If a glyph fits in an existing column, it is added at the bottom of the occupied space in said column.
If all columns are occupied, a new column is carved out of the left side of the remaining space. Since few fonts are monospaced, the renderer rounds the width of each glyph to a multiple of 4 pixels (by default). This is a good compromise between columns reuse and texture packing. The packing is not optimum, but it offers as fast implementation.
All glyphs are stored in the textures with an empty border of 1 pixel around them. This is necessary to avoid artifacts when the font textures are sampled with bilinear filtering.

这是



glyph identifiers: 字形标志
