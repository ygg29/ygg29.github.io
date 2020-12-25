---
title: iOS启动优化-二进制重排
tags:
  - 启动优化
categories:
  - iOS开发
date: 2020-03-04 19:33:23
---

随着业务开发进入迭代稳定期，App 的各种优化任务随之而来。[抖音研发实践](https://mp.weixin.qq.com/s/Drmmx5JtjG3UtTFksL6Q8Q)中提出了二进制重排的优化经验，号称提升了 15%的启动速度，本文尝试探究一下二进制重排的原理及 clang 插桩的优化方案。

<!-- more -->

### 启动过程

> App 启动根据 Apple 划分有三种形式：
>
> **冷启动(`cold`):** 指手机刚刚重启，完全清除dyld缓存，用户点击App lcon到完
> 全可用状态。
> **暖启动(`warm`) :** 指app刚被kill掉，但是app的可执行程序仍旧留在dyld缓
> 存里面，下次启动由于会命中部分缓存，启动速度会快很多。
> **热启动(`Resume`):** 指app从后台切到前台，完全在内存里。

> **启动以 main 函数为界限分为两个部分 ： `pre-main` 和 `post-main` 阶段**，
>
> pre-main 阶段包含动态库的加载链接过程、符号绑定、class 加载、以及 c++静态初始化的过程，如下：
>
> ![](/images/WX20201217-110408@2x.png)
>
> post-main 一般定义为从 main 函数的调用到首屏渲染完成的过程

#### 启动时间检测：

1. ` pre-main `

- 可配置 `DYLD `环境变量：**DYLD_PRINT_STATISTICS**

  ![](/images/WX20201216-143742@2x.png)

  通过控制台输出查看耗时：

  ![](/images/WX20201216-152625@2x.png)

  - **`dylib loading time`**

    动态库的加载耗时，Apple建议为 6 个，所以如果超过这个数量，最好进行库的合并

  - **`rebase/binding time`**

    由于ASLR(address space layout randomization)的存在，可执行文件和动态链接库在虚拟内存中的加载地址每次启动都不固定，所以需要这两步来修正。Rebase就是用来修正动态库内部指针的指向。而动态库之间存在依赖关系，在刚加载完所有动态库后，它们之间还是相互独立的状态。Bind就是将动态库内部符号表指向与所依赖的动态库的资源。

    耗时与 `class`、`selector`、`category`这些元数据数量的有关。

  - **`ObjC setup time`**

    OC class 的注册时间，包含启动 runtime、插入分类方法到方法列表中等

  - **`initializer time`**

    `Objc`的`+load()`函数、`C++`的构造函数属性函数 形如`attribute((constructor)) void DoSomeInitializationWork()`、非基本类型的`C++`静态全局变量的创建(通常是类或结构体)`(non-trivial initializer)` 比如一个全局静态结构体的构建，如果在构造函数中有繁重的工作，那么会拖慢启动速度

    **动态库的`load`方法早于主二进制的所有`load`方法被调用**

2. `post-main` 

   post-main 阶段耗时可通过埋点、 instruments 、[AppleTrace](https://github.com/everettjf/AppleTrace)等工具监控，和业务相关性较大，主要优化思路是启动任务分层，非必须任务采用懒加载方式进行。不是本文重点。

### 二进制重排原理

> 安全考虑，iOS 使用了虚拟内存，并在iOS4.3 后引入了 ASLR 安全机制。由于启动过程所调用的函数可能在多个 page 中，为了尽量减少 Page Fault的次数，通过重排将启动时期所调用的函数尽量排在同一个 page 中，进而达到减少启动耗时的目的。

可通过配置的 linkmap 文件查看当前符号的排列顺序：

​	![](/images/WX20201216-162043@2x.png)

找到 linkmap 文件：

![](/images/WX20201216-163608@2x.png)

![](/images/WX20201216-163816@2x.png)

上图标注的地方即为当前二进制文件中，符号的排序。

>**原始符号排列规则：**
>
>1. `Build Phases -> Compile Sources` 中的文件**编译顺序**
>2. 方法/函数在文件中的**书写顺序**



Xcode 的连接器为**ld**，可通过在`Build Setting -> Order File` 中设置 order 文件，来让连接器根据order文件来进行二进制的重排。如下图：

![](/images/WX20201216-173003@2x.png)

![](/images/WX20201216-173355@2x.png)

注意此处**方法**与**函数**的符号区别，二进制中没有的符号会自动忽略。

然后查看 linkmap 文件即可看到重排后的结果。



### Clang插桩

二进制重排的难点在于获取启动过程中调用的`函数/Block/方法`的**符号**，[抖音方案](https://mp.weixin.qq.com/s/Drmmx5JtjG3UtTFksL6Q8Q)也仅覆盖80%～90%的符号，所以 Clang 插桩是个更有效更简单的方式。

#### 原理

> [Clang 12 documentation](http://clang.llvm.org/docs/SanitizerCoverage.html#tracing-pcs-with-guards)中有这样一段描述：
>
> LLVM has a simple code coverage instrumentation built in (SanitizerCoverage). **It inserts calls to user-defined functions on function-, basic-block-, and edge- levels.**Default implementations of those callbacks are provided and implement simple coverage reporting and visualization

 Clang 编译器会在每个方法、函数、block 的调用起始处插入一个 `trace-pc-guard` 的 C flag，用户可以HOOK到每个函数的调用。

![](/images/WX20201218-183905@2x.png)

> 注：Clang 不仅会在每个函数的边缘插入”桩“，一个循环也会被插入，所以除了 `trace-pc-guard` 外，还需加入一个额外的 `func` 过滤掉循环。

这个C flag 就是所谓的“桩”.

这里会调用两个方法：

```objective-c
void __sanitizer_cov_trace_pc_guard_init(uint32_t *start, uint32_t *stop) {}
void __sanitizer_cov_trace_pc_guard(uint32_t *guard) {}
```

我们可以通过汇编看到 `Clang ` 实际是在每个函数调用的边缘位置插入了一个`bl`指令，跳转到 `__sanitizer_cov_trace_pc_guard` 函数，可以实现了一个全局的 AOP。

![](/images/WX20201217-173509@2x.png)

接下来就是一个体力活了。。。

##### 1. 获取函数地址

```objc
void *PC = __builtin_return_address(0);
```

##### 2. 获取函数符号

这里需要借助 Dl_info 结构体：

```objc
void *PC = __builtin_return_address(0);
Dl_info info;
dladdr(PC, &info);
```

我们可以打印一下 `info ` 值，会发现 `info.dli_sname` 即为我们梦寐以求的的符号字符串！

##### 3.生成 order 文件

```objc
// 1、拿到该函数返回到调用函数的函数地址
void *PC = __builtin_return_address(0);
Dl_info info;
dladdr(PC, &info);

// 2. 创建结构体保存 node
XYNode *node = malloc(sizeof(XYNode));
*node = (XYNode){PC, NULL};

// 3. 加入队列
OSAtomicEnqueue(&symbolQueue, node, offsetof(XYNode, next));
```

然后需要在 App 启动完成之后从队列中取出符号，以 `rootViewController `的`viewDidAppear` 方法调用完成作为启动结束的判断标志。

```objc
while (YES) {
    // 4. 取出 pc
    XYNode *node = OSAtomicDequeue(&symbolQueue, offsetof(XYNode, next));
    if (node == NULL) { break; }
    // 5. 赋值
    Dl_info info = {0};
    dladdr(node->pc, &info);
  	// 6. 拿到符号
    NSString *name = @(info.dli_sname);
    // 7. 处理函数名称(函数需要添加下划线)
    BOOL isObjcFunc = [name hasPrefix:@"-["] || [name hasPrefix:@"+["];
    NSString *symbolName = isObjcFunc ? name : [@"_" stringByAppendingString:name];
    [symbolNames addObject:symbolName];
}
```

```objc
  // 8. 数组取反
    symbolNames = (NSMutableArray<NSString *> *)[[symbolNames reverseObjectEnumerator] allObjects];
    // 9. 数组去重
    NSMutableArray *resFuncs = [NSMutableArray arrayWithCapacity:symbolNames.count];
    [symbolNames enumerateObjectsUsingBlock:^(NSString * _Nonnull obj, NSUInteger idx, BOOL * _Nonnull stop) {
        if (![resFuncs containsObject:obj]) {
            [resFuncs addObject:obj];
        }
    }];
    // 10. 写入文件
    NSString *symbolStr = [resFuncs componentsJoinedByString:@"\n"];
    NSData *data = [symbolStr dataUsingEncoding:NSUTF8StringEncoding];
    NSString *filePath = [NSTemporaryDirectory() stringByAppendingString:@"symbol.order"];
    [[NSFileManager defaultManager]createFileAtPath:filePath contents:data attributes:nil];
```

然后就可以让二进制文件根据生成的 order 文件进行重排了。

#### Swift 插桩

很简单，`Other Swift Flags` 添加两个 flag：

![](/images/WX20201221-150023@2x.png)

即可实现捕获 swift 函数。



**参考：**

[Improving App Performance with Order Files](https://eisel.me/order)

[Optimizing App Launch](https://developer.apple.com/videos/play/wwdc2019/423/ )

[抖音研发实践](https://mp.weixin.qq.com/s/Drmmx5JtjG3UtTFksL6Q8Q)

[Clang 12 documentation](http://clang.llvm.org/docs/SanitizerCoverage.html#tracing-pcs-with-guards)

