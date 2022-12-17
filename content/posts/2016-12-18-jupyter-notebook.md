---
title: Jupyter Notebook 初试
slug: jupyter-notebook
date: 2016-12-18 02:39:56
tags: [Python]
categories: [Coding]
---

### 安装

`pip3 install jupyter --user`

Jupyter Notebook的kernel是`IPython`, 这样Notebook才可以使用`Python`. 也可以安装kernel, 比如 R, Julia.<!--more-->

Pycharm其实自带了jupyter notebook的支持，而且把包都装好了。新建一个jupyter notebook文件，就相当于打开了一个notebook前端。点运行时会启动server.

### 运行

开启notebook服务器： `jupyter notebook`

`jupyter`是主命令，用来调用具体的`jupyter-notebook`程序。如果在`PATH`中能找到类似`jupyter-xxx`的程序，那么就可以用`jupyter xxx`来启动它。`jupyter --paths`可以显示jupyter一些相关的目录配置。

这个命令会启动基于Tornado的 notebook服务器，并用默认的浏览器作为客户端打开链接，初始的页面 *Notebook Dashboard*. 

Notebook Dashboard会列出服务器启动目录下的文件。

客户端和jupyter kernel通过ZeroMQ传送JSON消息。Notebook前端除了运行代码之外，还会将代码/输出，以及Markdown格式的笔记等保存为 `.ipynb`后缀的JSON文件，这称为一个notebook文件。

### Notebook使用

`Shitf+Enter`执行代码并移动到新的单元格

有返回值的计算结果会有`Out[x]`的标识，代码中最后一个表达式的值将在输出区域显示。如果希望屏蔽输出，可以在最后一个语句后添加分号：";"。在Cell中按`Tab`可以像IPython中一样有自动补全。

可以修改之前的单元格，对其重新计算，这样可以更新整个文档。

Cell支持Markdown格式，使用Shift+Enter渲染为富文本。Markdown可以嵌入LaTex, 图片，表格等内容。

从`from IPython.display`可以引入许多有用的展示内容，用他们可以在notebook中导入音频，视频，网页等内容。

### 魔法命令

所有`%`开头的方法，称为魔法命令，也就是IPython内置的一些方法。`%`称为line magic，对一行起作用，`%%`针对多行起作用。

通过`%lsmagic%`查看所有的魔法命令。使用`?`可以查看命令的信息，比如`%alias?`。

Notebook内置支持matplotlib, 用`%matplotlib`魔术方法调用。





### 参考

[IPython](http://ipython.readthedocs.io/en/stable/interactive/python-ipython-diff.html)