---
layout: post
title: "Efficient String Concatenation in Python"
date: 2013-07-08 12:48
comments: true
categories: 
---

# Python 中的高效字符串连接
** 几种方法的性能评估 **

## 引言
有时候使用 Python 构建长字符串会产生运行速度很慢的代码. 在本文章中，我考察了几种字符串连接方法的计算性能.

Python 中的字符串实例是不可变的 - 每次
