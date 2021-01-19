---
title: swift函数派发探究
tags:
  - iOS
  - swift
  - 函数派发
  - 泛型
  - 源码
categories:
  - iOS开发
date: 2020-08-20 10:37:31
---

终于有时间来填坑了😂

之前在[《swift 协议与函数派发的几点问题》](https://www.jianshu.com/p/7d05578242fd)中发现了几个让我很费解的问题，不知是 swift 设计如此还是所谓的 bug，现在我们尝试通过一些工具如[SIL](https://github.com/apple/swift/blob/main/docs/SIL.rst)、[LLVM IR](http://llvm.org/docs/LangRef.html)或者源码去探究一下。

<!-- more -->

## 准备

通过`swiftc  `命令:

```shell
swiftc -emit-sil main.swift > main.sil && open main.sil
swiftc -emit-ir main.swift > main.ll && open main.ll
```

 可以分别输出编译后的 SIL 代码和 IR 代码。

中间代码都是 [SSA from](https://zh.wikipedia.org/wiki/%E9%9D%99%E6%80%81%E5%8D%95%E8%B5%8B%E5%80%BC%E5%BD%A2%E5%BC%8F) 形式的代码，并且是 human readable assembly language representation，理解起来并没有想象中的那么困难，况且我们的目的只是探索实现逻辑，并不需要每一行代码都搞懂。

## protocol 不同的派发方式

先来看一段代码：

```swift
// 1
protocol Movable {
    func move() // 该行注释会影响 print 结果
}

extension Movable {
    func move() {
        print("Movable")
    }
}

class Vehicle: Movable{
    func move() {
        print("Vehicle")
    }
}
let v1: Movable = Vehicle()
v1.move()
```

输出：  

> **Vehicle**

注释掉第三行后，输出：

> **Movable**

为什么`不在 protocol 声明`的方法会有这样的表现？来看一下 SIL 代码：

![](/Users/hexo_images/WeChatee95656b9520f777.png)

可以看到，在 procotol 中声明过得的方法使用的是 `witness_method（PWT）` 的派发方式, 而直接在 extension 中声明的方法是采用的**静态派发**的方式直接调用。

### 声明类型对于函数派发的影响

对于如下代码：

```swift
// 1. 声明为 class 类型
let v1: Vehicle = Vehicle()

// 2. 声明为 protocol 类型
let v2: Movable = Vehicle()
```

![](/Users/hexo_images/WX20201230-173920@2x.png)

可见 1 中采用的是 `class_method` 的派发方式，而 2 采用的是 `PWT` 的派发方式。

#### 声明类型的内存大小

可以用`MemoryLayout`来打印对象的内存大小：

```swift
print(MemoryLayout.size(ofValue: v1)) // print: 8
print(MemoryLayout.size(ofValue: v2)) // print: 40
```

第一行打印的 8 字节很好理解，第二行的 40 字节是怎么回事？

实际上声明为协议类型时，内存中保存的是结构体类型：

![](/Users/hexo_images/WX20210105-000757@2x.png)

`ValueBuffer `的最大容量为 3*8 = 24 字节，即只能存储三个变量，当变量的个数大于 3 个时，会在堆上开辟空间将变量复制过去，并将指针地址保存在 Value1 中。

同时，协议类型中还保存了必要的 `PWT（Protocol Witness Table）` 与 `VWT（Value Witness Table））`。

### 父类是否声明协议方法影响函数派发

对于如下代码：

```swift
protocol Movable {
    func move()
}

extension Movable {
    func move() {
        print("可移动接口")
    }
}

class Vehicle: Movable{
    func move() {      // 该方法声明与否影响函数派发
        print("交通工具")
    }
}
class Car: Vehicle {
     override func move() {
        print("汽车")
    }
}
```

![](/Users/hexo_images/WX20210105-165740@2x.png)

从 `SIL` 可以看出，在父类声明之后，协议中的函数是采用的 `class_method` 以 `vtable` 的形式派发，而不再父类中声明的采用直接派发的方式，直接调用 `protocol` 中的方法。

实际上协议中的方法是通过 `PWT` -> `vtable` 找到函数地址的。

我们可以通过命令：

> xcrun swift-demangle <符号>

来还原混淆过的原始符号。

派发流程如下(`SIL`代码)：

1. 找到协议

![](/Users/hexo_images/methid_dispatch1.png)

2. 通过协议，得到`PWT`

![](/Users/hexo_images/methid_dispatch2.png)

3. 通过协议的实现找到父类，这时实际上就得到了实例的 `vtable` ，可以通过函数表派发。

![](/Users/hexo_images/methid_dispatch3.png)

![](/Users/hexo_images/methid_dispatch4.png)





根据上文，问题就很清楚了：

**1、协议中的`optional`方法, 实现的时候是通过何种方式派发的？**

协议中的`optional`方法, 因为声明过，所以采用的是 **PWT **的派发方式。



s**2、为什么自己实现的协议与系统表现的不一样，系统提供的协议做了什么？**

例子中使用的 `UITableViewDelegate ` 从 OC 继承过来，使用的是 `msgSend() `的派发方式，直接通过拿到对象地址发送消息，这点与 Java 的表现一致。

**3、swift在编译的过程中，对有范型的类除了做了泛型擦除还做了什么，以至于影响到了函数的派发？**

