---
title: alloc源码
tags:
  - iOS
  - runtime
  - 源码
categories:
  - iOS开发
date: 2020-12-03 10:17:41
---

基于[Objc4-781](https://github.com/ygg29/Objc4-781)

源码文件：`NSObject.mm`、`objc-runtime-new.h`

<!-- more -->

### alloc 流程说明

1. 调用 `+alloc` -> `_objc_rootAlloc `
2. 调用 `callAlloc `-> `_objc_rootAllocWithZone `-> `_class_createInstanceFromZone`
3. 调用 `_class_createInstanceFromZone` 
   1. `cls->instanceSize`计算实际需要申请的空间大小，并对结果进行 **16 字节**的对齐
   2. `calloc` 向操作系统申请内存空间
   3. `initInstanceIsa` 将创建出的对象的 isa 指针与 class 关联

核心方法源码如下图：

![](/Users/hexo_images/WX20210304-162621@2x.png)

![](/Users/hexo_images/WX20210304-163910@2x.png)

### 其他构造方法

#### +new

``` objc
+ (id)new {
    return [callAlloc(self, false/*checkNil*/) init];
}
```

#### -init

`+new` 方法调用了 `callAlloc init` 和 `alloc` 中的步骤 2 相同。

同时，`+new` 调用了默认的实例构造器 `-init`, 所以，如果 `class `中**没有实现其他的实例构造方法**时， 使用 `alloc init` 和使用 `new` 是一样的效果。

#### init

源码中有两个 init 方法，一个实例方法，一个类方法：

```objc
+ (id)init {
    return (id)self;
}

- (id)init {
    return _objc_rootInit(self);
}
```

关于类方法， 就返回自身，尚不知有什么用处。

由源码可见`-init` 是个初始化方法，原样返回 `obj`， 便于程序员自定义做一些初始化的工作。

```c++
id
_objc_rootInit(id obj)
{
    // In practice, it will be hard to rely on this function.
    // Many classes do not properly chain -init calls.
    return obj;
}

```

#### +allocWithZone

进入 `+allocWithZone` 方法， 发现调用了 `_objc_rootAllocWithZone ` 并且 `zone` 参数也并未使用，所以流程其实和 `alloc `是一样的， 注释也说明该方法被 `alloc `替换了。

![](/Users/hexo_images/WX20210304-155841@2x.png)

