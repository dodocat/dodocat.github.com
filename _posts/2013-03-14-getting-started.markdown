---
layout: post
title: "Getting Started"
date: 2013-03-14 00:48
comments: true
---

## 一个简单的例子

我们尽可能创建一个最简单的程序来开始我们的教程. 这个程序将创建一个 200x200 像素的空窗口.

!["simple examle"](/image/python-gtk-3-tutorial/simple_example.png)

``` python
#!/usr/bin/python
from gi.repository import Gtk

win = Gtk.Window()
win.connect("delete-event", Gtk.main_quit)
win.show_all()
Gtk.main()
```

现在逐行解释这个例子.

``` python
#!/usr/bin/python
```
所有 Python 程序第一行应该以 `#!`开头, 其后紧跟着你想调用的 Python 解释器的路径.
``` python
from gi.repository import Gtk
```
为了访问 GTK+ 类和函数, 首先要导入 GTK 模块.  下一行创建一个空窗口.
``` python
win = Gtk.Window()
```
紧接着连接窗口的删除事件以保证程序会再 x 点击下的时候结束.
``` python
win.connnect("delete_event", Gtk.main_quit)
```
下一步将窗口显示出来
``` python
win.show_all()
```
最后, 启动 GTK+ 工作循环, 循环会在窗口被关闭的事后退出（参见第五行）.
``` python
Gtk.main()
```
打开终端以运行程序, 进入文件的目录, 输入:
``` sh
python simple_example.py
```

## 延伸例子
这是一个有那么一点点用处的 PyGObject 的经典`Hello World` 程序.

!["Extended Example"](/image/python-gtk-3-tutorial/extended_example.png)

``` python
#!/usr/bin/python
from gi.repository import Gtk

class MyWindow(Gtk.Window):
    def __init__(self):
        Gtk.Window.__init__(self, title="Hello World")
        self.button = Gtk.Button(label="Click Here")
        self.button.connect("clicked", self.on_button_clicked)
        self.add(self.button)

    def on_button_clicked(self, widget):
        print "Hello World"

win = MyWindow()
win.connect("delete-event", Gtk.main_quit)
win.show_all()
Gtk.main()
```

这个例子与上面简单例子不同的地方是, 使用创建 `Gtk.Window` 子类的方法自定义了 `MyWindow` 类.
``` python
class MyWindow(Gtk.Window):
```

在子类(sub class)的构造函数中必须调用父类(super)的构造函数. 此外,我们还告诉父类的构造函数为 Hello World 设置正确的标题:
``` python
Gtk.Window.__init__(self, title="Hello World")
```

接下来的三行用来创建按钮控件,连接它们的信号还有把它们作为子控件加入到 top-level 窗口中.
``` python
self.button = Gtk.Button(label="Click Here")
self.button.connect("clicked", self.on_button_clicked)
self.add(self.button)
```

这些代码让 `on_button_clicked()` 方法会将在按钮被被按下时候被调用.
``` python
def on_button_clicked(self, widget):
    print "Hello World"
```

最后一块再类外面的代码, 和上面的简单例子很像, 不过这里创建是的是类 `MyWindow` 的实例, 而非通用类 `Gtk.Window` 的实例.
The last block, outside of the class, is very similar to the simple example above, but instead of creating an instance of the generic Gtk.Window class, we create an instance of MyWindow.

> 第一篇完成 以后争取平均一周 3-4 章的速度更新
> TODO
