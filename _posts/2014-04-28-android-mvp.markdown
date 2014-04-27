---
layout: post
title: "Android MVP"
summary:
---

MVP (Model View Presenter) pattern is a derivative from the well known MVC (Model View Controller), which for a while now is gaining importance in the development of Android applications. There are more and more people talking about it, but yet very few reliable and structured information. That is why I wanted to use this blog to encourage the discussion and bring all our knowledge to apply it in the best possible way to our projects.


MVP 模式是由人人皆知的 MVC (Model View Controller), 派生而来的. MVP 在 Android 开发中愈发重要. 讨论它的人越来越多, 但是依然没有可靠的成体系的信息. 我想用这篇博文鼓励关于 MVP 的讨论, 用大家的学识把 MVP 以最好的方式应用到我们的工程中.

What is MVP?
The MVP pattern allows separate the presentation layer from the logic, so that everything about how the interface works is separated from how we represent it on screen. Ideally the MVP pattern would achieve that same logic might have completely different and interchangeable views.
First thing to clarify is that MVP is not an architectural pattern, it’s only responsible for the presentation layer . In any case it is always better to use it for your architecture that not using it at all.

## 什么是 MVP?

MVP 模式可以使展现层*展现*层与逻辑分离, 如此接口工作的方式就与展现层如何在在屏幕上展现完全分离了.
理想情况下, MVP 模式能实现有着完全不同切可置换的视图的相同逻辑.
首先要澄清, MVP 不是架构模式, 它只负责展现层. 在我的应用实例中, 使用它总是比完全不用对架构更有利.

Why use MVP?
In Android we have a problem arising from the fact that Android activities are closely coupled to both interface and data access mechanisms. We can find extreme examples such as CursorAdapter, which mix adapters, which are part of the view, with cursors, something that should be relegated to the depths of data access layer .
For an application to be easily extensible and maintainable we need to define well separated layers. What do we do tomorrow if, instead of retrieving the same data from a database, we need to do it from a web service? We would have to redo our entire view .

## 为什么使用 MVP

Android activities 近乎是 interface 和数据访问结合的部件给我们带来了一个问题.
我们可以找到极端的例子, 比如 `CursorAdapter` 混合了adaptor(view 的一部分) 与 Cursor(某种应该视为深度数据访问层的东西).

对于一个能轻易扩展和维护的应用, 我们定义良好的分离层. 


