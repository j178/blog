---
title: 'Python 图片处理库: Pillow'
slug: python-pillow
tags:
- Python
- Pillow
categories: [Coding]
date: 2017-01-21 00:15:07
---


- Image.open()
  im.mode, im.size, im.format
- Image.save()
- Image.show()
  性能很慢, 仅作调试用.
- Image.convert()
  比较常用的函数
- Image.crop()
- Image.paste()
- Image.split()
  将图像分为不同的 band
- Image.getpixel()
- Image.putpixel()
  很慢
- Image.point()
  可以接受一个 lookup table, 或者一个函数, 函数接受一个参数, 这个参数可以看做像素点的颜色, 返回值为像素点的新颜色
- Image.thumbnail()
  生成缩略图, 会直接修改原图, 如果要保留原图需要先用 `Image.copy()`
- Image.getbbox()
  这个函数好像很好用的样子, 待会儿再尝试
- Image.getcolors(maxcolors=256)
  获取图片中用到的所有颜色, 返回 [ (count, color),... ]
  如果是 RGB 等格式, color 也是一个 `tuple`, 所以不同的颜色会有很多, 需要传入较大的 maxcolors, 否则会返回 `None`
- Image.histogram()
  获取图片的直方图, 返回一个`list`, 如果是灰度图像, 则 list 大小为256, 每个元素表示0-255各个灰度的像素点的个数. 如果是 RGB 图像, 则 list 大小为 256*3.