---
layout: post
title: "具有状态栏指示 (indicator) 自动重连 ssh tunnel 辅助工具"
date: 2013-03-28 04:14
comments: true
categories: Python 作品 ubuntu 科学上网
---

## bingo

[代码在 github 上](https://github.com/dodocat/indicator-ssht)

## 想法是这样的
我需要一个这样的 ssh tunnel 辅助工具

* 能够断线自动重新连接
* 能够在判断合适放弃重连，并通知我
* 在状态栏以图标显示当前链接状态
* 能够使用 ubuntu HUD 操作

## 所以重复造了轮子

现在有一个 gstm 和一个 autossh 分别具备了我需求的部分功能。
但是没有一个完全满足我的需求的。


