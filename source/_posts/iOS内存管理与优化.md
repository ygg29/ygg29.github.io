---
title: iOS内存管理与优化
tags:
  - iOS
  - 源码
categories:
  - iOS开发
date: 2019-4-13 23:17:01
---

## 内存布局

### 内存对齐



### 字节对齐

<!-- more -->

## Tagged Pointer

Apple 针对小对象做的一种**特殊优化**，iOS7 后引入，只支持64 位系统。

**指针对象将class和对象数据存储在对象指针中，指针实际上不指向任何东西**，存取性能极高。

主要存储如`NSNumber`、`NSDate`、`NSString`等。

引入 Tagged Pointer 后，相关逻辑能减少一半以上的内存占用，以及 3 倍的访问速度提升，100 倍的创建、销毁速度提升。

可通过设置环境变量 `OBJC_DISABLE_TAG_OBFUSCATION = YES` ， 来关闭 `Tagged Pointer` 的数据混淆, 一遍直接打印地址查看。

在 [源码](https://opensource.apple.com/source/objc4/objc4-781/runtime/objc-internal.h.auto.html) 中可以看到 Tagged Pointer 的表示：

```c++
enum objc_tag_index_t : uint16_t
{
    // 60-bit payloads
    OBJC_TAG_NSAtom            = 0, 
    OBJC_TAG_1                 = 1, 
    OBJC_TAG_NSString          = 2, 
    OBJC_TAG_NSNumber          = 3, 
    OBJC_TAG_NSIndexPath       = 4, 
    OBJC_TAG_NSManagedObjectID = 5, 
    OBJC_TAG_NSDate            = 6,

    // 60-bit reserved
    OBJC_TAG_RESERVED_7        = 7, 

    // 52-bit payloads
    OBJC_TAG_Photos_1          = 8,
    OBJC_TAG_Photos_2          = 9,
    OBJC_TAG_Photos_3          = 10,
    OBJC_TAG_Photos_4          = 11,
    OBJC_TAG_XPC_1             = 12,
    OBJC_TAG_XPC_2             = 13,
    OBJC_TAG_XPC_3             = 14,
    OBJC_TAG_XPC_4             = 15,
    OBJC_TAG_NSColor           = 16,
    OBJC_TAG_UIColor           = 17,
    OBJC_TAG_CGColor           = 18,
    OBJC_TAG_NSIndexSet        = 19,

    OBJC_TAG_First60BitPayload = 0, 
    OBJC_TAG_Last60BitPayload  = 6, 
    OBJC_TAG_First52BitPayload = 8, 
    OBJC_TAG_Last52BitPayload  = 263, 

    OBJC_TAG_RESERVED_264      = 264
};
```

Tagged Pointer 的规则如下：

> - (LSB)(macOS)64位分布如下：（Least Significant Bit，即**最低**有效位）
>   - 1 bit 标记是 Tagged Pointer
>   - 3 bits 标记类型
>   - 60 bits 负载数据容量，（存储对象数据）
> - (MSB)(iOS)64位分布如下：（Most Significant Bit，即**最高**有效位）
>   - tag index 表示对象所属的 class
>   - 负载格式由对象的 class 定义
>   - 如果 tag index 是 0b111(7)， tagged pointer 对象使用 “扩展” 表示形式
>   - 允许更多类，但 有效载荷 更小
> - (LSB)(macOS)(带有扩展内容)64位分布如下：
>   - 1 bit 标记是 Tagged Pointer
>   - 3 bits 是 0b111
>   - 8 bits 扩展标记格式
>   - 52 bits 负载数据容量，（存储对象数据）

**总结：**
- Tagged pointer 存储对象数据目前 分为60bits负载容量和52bits负载容量。
  - 当类标识为 0-6 时，负载数据容量为 60bits。
  - 当类标识为 7 时(对应二进制为 0b111)，负载数据容量为 52bits。
- 如果 tag index 是 0b111(7)， tagged pointer 对象使用 “扩展” 表示形式

## retainCount

我们通过一个例子来直观的感受一下 `retainCount`：

```objc
self.o = [NSObject alloc];
id o = [NSObject alloc];
NSLog(@"实例变量retainCount：%ld", (long)CFGetRetainCount((__bridge CFTypeRef)(self.o)));
NSLog(@"局部变量retainCount：%ld", (long)CFGetRetainCount((__bridge CFTypeRef)(o)));
```

上面的打印的结果的是什么？

答案是

> 实例变量retainCount：2
> 局部变量retainCount：1

根据源码：

![](/images/WX20210107-215218@2x.png)

在获取引用计数的时候实际上进行了 +1 之后才输出，所以真实的 `retainCount ` 其实分别为 1 和 0，说明 `alloc `并没有操作引用计数，这一点也可以通过查看 `alloc ` 的[源码](https://github.com/ygg29/SourceCode)实现证实，有兴趣的可以探究一下。

这里思考两个问题：

>1. 为什么 retainCount = 0 时对象并没有立即释放呢？
>2. 实例变量也只是做了 `alloc` 的初始化工作，为什么 retainCount 为 1？

通过分析我们可以得出结论：

- **alloc 方法没有与 `retainCount` 相关的操作**（init 实际是个工厂方法，为初始化提供便利）
- **局部变量交给 `autoReleasePool `管理，并不会在 retainCount = 0 时立即释放**
- **`retainCount` 强调的是一种引用的关系，即有几个对象引用，`retainCount`就为几，这里的实例变量被  `self ` 引用，所以 `retainCount` 为 1**

## nonpointer_isa

我们可以在 [objc4 的源码](https://opensource.apple.com/source/objc4/objc4-781/runtime/objc-private.h.auto.html)里面找到 isa 的实现, 是一个 isa_t 的类型：

```c++
union isa_t {
    isa_t() { }
    isa_t(uintptr_t value) : bits(value) { }

    Class cls;
    uintptr_t bits;
#if defined(ISA_BITFIELD)
    struct {
        uintptr_t nonpointer        : 1; // 是否开启指针优化                          
        uintptr_t has_assoc         : 1; // 是否有关联对象？ 没有则释放会更快
        uintptr_t has_cxx_dtor      : 1; // 是否有 C++ 析构函数，没有释放会更快
        uintptr_t shiftcls          : 44; /*MACH_VM_MAX_ADDRESS 0x7fffffe00000*/ //储存真正的类信息
        uintptr_t magic             : 6; // 用于在调试时分辨对象是否未完成初始化
        uintptr_t weakly_referenced : 1; // 是否有弱引用指针指向
        uintptr_t deallocating      : 1; // 是否正在释放
        uintptr_t has_sidetable_rc  : 1; // 引用计数是否过大以至于需要存在 SideTable 中
        uintptr_t extra_rc          : 8  // 引用计数
    };
#endif
};
```

所谓 `nonpointer `即指针不单单是一个地址，里面存储这一些有用的信息，`isa` 是一个**联合体**，**位域**中分别存储着不同的信息。

### retainCount 储存位置

我们可以查看retain 的[源码](https://opensource.apple.com/source/objc4/objc4-781/runtime/objc-object.h.auto.html)，在方法 `objc_object::rootRetain(bool tryRetain, bool handleOverflow)` 中

![](/images/WX20210107-224536@2x.png)



迁移中...



## 参考

[iOS内存管理之Tagged Pointer](https://www.infoq.cn/article/r5s0budukwyndafrivh4)

