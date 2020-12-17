---
title: iOS启动优化-二进制重排
tags:
  - 启动优化
categories:
  - iOS开发
date: 2020-03-04 19:33:23
---

随着业务开发进入迭代稳定期，App 的各种优化任务随之而来。[抖音研发实践](https://mp.weixin.qq.com/s/Drmmx5JtjG3UtTFksL6Q8Q)中提出了二进制重排的优化经验，号称提升了 15%的启动速度，本文尝试探究一下二进制重排的原理及 clang 插桩的优化方案。

### 启动过程

> **启动以 main 函数为界限分为两个部分 ： `pre-main` 和 `post-main` 阶段**，
>
> pre-main 阶段包含动态库的加载链接过程、符号绑定、class 加载、以及 c++静态初始化的过程
>
> post-main 一般定义为从 main 函数的调用到首屏渲染完成的过程

#### 启动时间检测：

1. `pre-main`

- 配置 `DYLD `环境变量检测启动时间：**DYLD_PRINT_STATISTICS**

  ![](/images/WX20201216-143742@2x.png)

  通过控制台输出查看耗时：

  ![](/images/WX20201216-152625@2x.png)

  - **dylib loading time**

    动态库的加载耗时，Apple建议为 6 个，所以如果超过这个数量，最好进行库的合并

  - **rebase/binding time**

    符号绑定耗时，由 DYLD 实现，和依赖的动态库数量正相关

  - **ObjC setup time**

    OC class 的注册时间，class 越多越耗时

  - **initializer time**

    执行 load和c++静态初始化的耗时

2. `post-main` 

   post-main 阶段可控，耗时可通过埋点、 instruments 、[AppleTrace](https://github.com/everettjf/AppleTrace)等工具监控，和业务相关性较大，不是本文重点

### 二进制重排原理

> 安全考虑，iOS 使用了虚拟内存，并在iOS4.3 后引入了 ASLR 安全机制。由于启动过程所调用的函数可能在多个 page 中，为了尽量减少 Page Fault的次数，通过重排将启动时期所调用的函数尽量排在同一个 page 中，进而达到减少启动耗时的目的。

可通过配置查看当前符号的排列顺序：

​	![](/images/WX20201216-162043@2x.png)

找到 linkmap 文件：

![](/images/WX20201216-163608@2x.png)

![](/images/WX20201216-163816@2x.png)

上图标注的地方即为当前符号的排序

>**原始符号排列规则：**
>
>1. `Build Phases -> Compile Sources` 中的文件**编译顺序**
>2. 方法/函数在文件中的**书写顺序**



Xcode 的连接器为**ld**，可通过`Build Setting -> Order File` 设置二进制重排的索引文件：

![](/images/WX20201216-173003@2x.png)

![](/images/WX20201216-173355@2x.png)

注意此处**方法**与**函数**的符号区别，二进制中没有的符号会自动忽略

然后查看 linkmap文件即可看到重排后的结果



### Clang插桩

二进制重排的难点在于获取启动过程中调用的`函数/Block/方法`的符号，[抖音方案](https://mp.weixin.qq.com/s/Drmmx5JtjG3UtTFksL6Q8Q)也仅覆盖80%～90%的符号，clang插桩是个更有效更简单的方式。

#### 原理



**附：WWDC 中对于启动优化的介绍** [Optimizing App Launch](https://developer.apple.com/videos/play/wwdc2019/423/ )

