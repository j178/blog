---
title: '设计模式(三): 适配器模式'
slug:  adapter-patter
date: 2016-09-29 21:18:06
tags: [java, 设计模式, 适配器模式]
categories: [Coding]
---

## Java 回顾

- `String` 内部实现是一个 `char` 数组
- `String` 可以由`char[]` ,`int[]`, `byte[]` 生成, 其中`char`代表每一个字符, `int`代表一个`code point`, `byte` 需要根据相应的 `charset` 解码为`char`

## 要点

1. 新旧两个接口
2. 实现旧接口的对象无法在支持新接口的设备上使用
3. 新建一个转换器(adapter), 实现新的接口, 内部聚合一个旧接口的对象, 将对新接口的所有调用全部委托到旧的接口上去.

## 讲课内容

- DIP 接口由高层定义, 低层实现
- 类适配器, 使用继承, 只能适配一个类
- 对象适配器, 使用聚合
