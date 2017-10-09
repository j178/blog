---
title: Unicode 与 UTF-8
tags: [UTF-8, Unicode]
---

## Unicode

{% blockquote Wikipedia %}
Unicode defines a codespace of 1,114,112 [code points](https://en.wikipedia.org/wiki/Code_point) in the range 0~hex~ to 10FFFF~hex~. Normally a Unicode code point is referred to by writing "U+" followed by its [hexadecimal](https://en.wikipedia.org/wiki/Hexadecimal) number. For code points in the [Basic Multilingual Plane](https://en.wikipedia.org/wiki/Basic_Multilingual_Plane) (BMP), four digits are used (e.g., U+0058 for the character LATIN CAPITAL LETTER X); for code points outside the BMP, five or six digits are used, as required (e.g., U+E0001 for the character LANGUAGE TAG and U+10FFFD for the character PRIVATE USE CHARACTER-10FFFD)
{% endblockquote %}

> ​:question:​ 为什么最大的 code point 也只是 `0x10FFFF`, 不到三个字节, 为什么 `utf-8` 可能需要 5-6 个字节呢?
> ​:anchor: `utf-8` 等变长编码方式, 需要占用额外的字节, 比如每个字节的 `10` 前缀.

### Code point planes and blocks

Unicode 被分为 17 个平面(plane), 标号为 0-16,  每个平面都由连续的 65536(=2^16^) 个 code point 组成. 其中 0平面(0-FFFF) 被称为 Basic Multilingual Plane (BMP), 其他平面被称为 Supplementary Planes.

### Basic Multilingual Plane



## UTF-8

可以表示 1,112,064 个有效的 code point (1,114,111 -  2048 个 surrogate code point)



## 其他

Python 或其他语言中, 使用文本模式打开文件, 其实会根据指定或者默认的编码对从文件中读取的二进制流进行解码(decode), 所以字符在内存中的形式都是 code point ? 在保存或者网络传输的时候, 就会再根据需要的 encoding 编码为二进制流.


## 参考链接
- [Unicode](https://en.wikipedia.org/wiki/Unicode)
- [UTF-8](https://en.wikipedia.org/wiki/UTF-8)
- [Plane - Unicode](https://en.wikipedia.org/wiki/Plane_(Unicode))