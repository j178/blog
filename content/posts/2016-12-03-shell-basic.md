---
title: Shell 基础
slug: shell-basic
date: 2016-12-03 23:45:38
tags: [shell]
categories: [Linux]
---

- source

  也可以简写为 `.` ，这也是通用的兼容写法

  在当前的shell中执行文件中命令，所以文件中的命令可以读取到当前shell中的变量(不一定是环境变量)，等同于在命令行一条一条地敲入命令执行。<!--more-->

- export
  将变量导出到**当前进程的**环境变量中，后续执行命令都可以读取到这些变量。如果用新的bash进程来export，那么改变的是新进程的环境变量，当执行结束后，本来的shell不会(父进程)的环境变量不会改变。
  使用source之后，才在shell，父进程中执行，所以改变了父进程的环境变量，于是在shell中执行的后续的命令都会继承这个环境变量。

- 子进程会继承哪些东西？
  当前目录？环境变量？