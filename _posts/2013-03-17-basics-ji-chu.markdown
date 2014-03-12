---
layout: post
title: "Basics 基础"
date: 2013-03-17 22:27
comments: true
categories: Python 
---

> [Python GTK+3 教程翻译](/blog/categories/python-gtk-plus-3-tutorial/)

这一节介绍 GTK+ 里最要用的一些东东.

## 主循环和信号

GTK+ 如多数 GUI 工具一样使用**事件驱动**(event-driven)编程模型.
当用户没有操作的时候, GTK+ 只是呆在主循环里等待输入. 
如果用户执行了某个动作, 比如说, 点击了一下鼠标, 主循环就会"醒来"
发送一个**事件**(**event**)给 GTK+. 

当**控件**(**widegets**)接收到一个**事件**(**event**), 它们经常发出一个一个或者多个**信号**(**signals**). 
信号调用与自己连接的方法通知程序:"发生了一件有趣的事情".这样的工程通常被称作**回调**(**callbacks**).
当回调函数被调用时, 你通常会采取一些动作, 比如点击打开按钮时, 你可能会显示一个文件选择对话框. 
GTK+ 的一个回调完成后, 将返回到主循环, 等待用户的更多输入.


