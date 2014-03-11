---
layout: post
title: "android.graphics.drawable.LayerDrawable 的使用"
date: 2013-03-26 01:23
comments: true
categories: Android 
---

一个应用界面，有图片，动态画笔轨迹，动态图片好几种资源同时显示。原本采用 canvas 直接分别绘制不同的区域, 感觉比较纠结. 考虑使用缓存, 而后一次性 canvas 绘制出来. 

查阅后发现可以使用 LayerDrawable 实现, 以上各资源各为一层, 各自的任务循环里更新各自的层. 绘图线程中合并各层. 

## 任务线程

代码 
``` java
biubiu
```

### 绘图线程

**未完.....**
