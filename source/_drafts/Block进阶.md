---
title: Block进阶
tags:
  - iOS
  - 源码
categories:
  - iOS开发
date: 2020-01-12 15:34:20
---



## Block 类型

![](/Users/hexo_images/Block.png)

Block 有三种类型：`__NSGlobalBlock__`、`__NSMallocBlock__` 和 `__NSStackBlock__`。

Block 在`默认存储在全局区`，当`捕获变量时`会发生 `_Block_copy` 将 Block 拷贝到`堆区`。

经过试验若没有指针强引用 Block，且 block 无参数返回值时，会存储在`栈区`。

<!-- more -->

## Block 结构

底层 Block 为 `Block_layout` 类型，源码为：

```c++
struct Block_layout {
    void * __ptrauth_objc_isa_pointer isa;
    volatile int32_t flags; // contains ref count
    int32_t reserved;
    BlockInvokeFunction invoke;
    struct Block_descriptor_1 *descriptor;
    // imported variables
    // descriptor_2 和 descriptor_3 会根据 flag 变化，可选值
    struct Block_descriptor_2 *descriptor_2;
    struct Block_descriptor_3 *descriptor_3;
};
```

flages 成员记录了当前 Block 的状态信息，如下：

```c++
// Values for Block_layout->flags to describe block objects
enum {
    BLOCK_DEALLOCATING =      (0x0001),  // runtime
    BLOCK_REFCOUNT_MASK =     (0xfffe),  // runtime
    BLOCK_INLINE_LAYOUT_STRING = (1 << 21), // compiler

#if BLOCK_SMALL_DESCRIPTOR_SUPPORTED
    BLOCK_SMALL_DESCRIPTOR =  (1 << 22), // compiler
#endif

    BLOCK_IS_NOESCAPE =       (1 << 23), // compiler
    BLOCK_NEEDS_FREE =        (1 << 24), // runtime
    BLOCK_HAS_COPY_DISPOSE =  (1 << 25), // compiler
    BLOCK_HAS_CTOR =          (1 << 26), // compiler: helpers have C++ code
    BLOCK_IS_GC =             (1 << 27), // runtime
    BLOCK_IS_GLOBAL =         (1 << 28), // compiler
    BLOCK_USE_STRET =         (1 << 29), // compiler: undefined if !BLOCK_HAS_SIGNATURE
    BLOCK_HAS_SIGNATURE  =    (1 << 30), // compiler
    BLOCK_HAS_EXTENDED_LAYOUT=(1 << 31)  // compiler
};
```

## Block循环引用检测





## 参考

[libclosure-78](https://opensource.apple.com/source/libclosure/libclosure-78/)

[循环引用检测](https://triplecc.github.io/2019/08/15/%E8%81%8A%E8%81%8A%E5%BE%AA%E7%8E%AF%E5%BC%95%E7%94%A8%E7%9A%84%E6%A3%80%E6%B5%8B/)

[clang](https://github.com/llvm-mirror/clang/blob/master/lib/CodeGen/CGObjCMac.cpp)

<a style="display:none;">https://mp.weixin.qq.com/s/q2UdQkgRyvksW6qnC1KF1w</a>

