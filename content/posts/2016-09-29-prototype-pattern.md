---
title: '设计模式(一): 原型模式'
date: 2016-09-29 11:19:46
tags: [java, 设计模式, 原型模式]
categories: [Coding]
---

## Java 回顾

### Java 访问权限控制

|     作用域     | 当前类  | 同一 package |  子类  | 其他 package |
| :---------: | :--: | :--------: | :--: | :--------: |
|  `public`   |  ✔   |     ✔      |  ✔   |     ✔      |
| `protected` |  ✔   |     ✔      |  ✔   |            |
| `包作用域` (默认) |  ✔   |     ✔      |      |            |
|  `private`  |  ✔   |            |      |            |

### 方法覆盖 Method Override
方法覆盖中有一些必须要满足的条件, 这些条件基本都是为了保证多态的有效性, 或者说为了符合 [里氏替换原则](https://en.wikipedia.org/wiki/Liskov_substitution_principle). 
<!--more-->
1. 子类方法的返回类型可以是父类方法的返回类型的子类.
   > Return types may vary among methods that override each other if the return types are reference types. The notion of return-type-substitutability supports covariant returns, that is, the specialization of the return type to a subtype.
2. 子类方法不可以比父类方法抛出更多的 checked 异常, 但可以抛出父类方法抛出异常的子类.
   可以抛出更少的异常, 但无法引入新的异常. 但是抛出 unchecked 异常却不受限制.


## 原型模式

原型模式是一个创建型模式
### 定义
通过给出一个原型对象来指明所要创建的对象的类型，然后用复制这个原型对象的办法创建出更多同类型的对象。
### 使用的原因
- 某些对象可能有复杂的内部结构
- 某些对象可能很难或者无法创建
- 某些对象可能有很复杂的初始窗台

###  Java 中 clone 的问题
`cloneable` 是一个标识接口, 不含任何方法.

`clone()` 方法在 `java.lang.Object` 中定义, 因此所有对象都有 `clone()` 方法, 但是要使用 **`clone()` 的类或者其父类**必须实现 `cloneable` 接口才能调用, 否则会抛出 `CloneNotSupportedException`.

使用 Java 提供的克隆机制, `Object` 类相当于抽象的原型类, 每个实现 `Clone()` 方法的子类都是具体的原型类, `clone()` 作为原型方法.

```java
  protected native Object clone() throws CloneNotSupportedException;
```
因为 `Object` 中 `clone()` 的定义为 `protected` , 要想使用 `clone()` 就必须**自己或者父类覆盖 `clone()` **, 把访问权限覆盖为 `public`.

由于 `clone()` 是一种 native实现, JNI(Java Native Interface), 所以程序员无法自己实现 ` clone()` 方法, 只能调用父类的: `super.clone()` .

抽象的引用(接口而不是抽象类) 无法调用 `clone()` 方法, 只有具体的引用才有 `clone()` 方法, 这违反了针对抽象编程的原则 (DIP, 依赖倒转原则).

因为单例模式中必须维持一个全局唯一的对象, 所以必须(?) 覆盖 `clone()` 方法并显示地抛出 `CloneNotSupportedException` , 以防其某个父类覆盖了 `clone()`.




### 浅拷贝与深拷贝
Java 的提供克隆机制只是 shallow copy

### 浅拷贝

新的对象中的引用类型属性也跟原来的对象一样, 即指向的是同一个对象, 这样的拷贝只拷贝了第一层, 所以叫做浅拷贝.

### 深拷贝

深拷贝不仅仅为原始类型生成新的副本, 而且所有的引用类型属性也都新创建一份. 新的对象中的引用属性的值与原来的对象不同.

深拷贝与 `final` 属性不兼容, 深拷贝实际上相当于一个没有参数的构造函数, 在实例化一个对象的时候 `final`属性都已经被赋值了, 就无法再次赋值了, 所以无法深拷贝.

### 通过序列化和反序列化实现深拷贝
要保证将要序列化的对象的所有属性都实现了 `Serializable` 接口

使用 `ObjectOutputStream` 将对象序列化到流中, 然后从 `ObjectInputStream` 中读取并强制转型为传入时的对象.

序列化的开销非常大, 比 `clone()` 方法要慢数百倍.