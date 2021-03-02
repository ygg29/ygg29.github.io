---
title: dealloc源码
tags:
  - iOS
  - runtime
  - 源码
categories:
  - iOS开发
date: 2020-12-02 16:17:41
---



基于[Objc4-781](https://github.com/ygg29/Objc4-781)

源码文件： `NSObject.mm`、`objc-object.h`、`objc-runtime-new.h`

<!-- more -->

```objective-c
inline void
objc_object::rootDealloc()
{
    // 判断 taggedPointer 对象
    if (isTaggedPointer()) return;  // fixme necessary?

    // 判断对象是否有额外工作
    // 弱引用、 关联对象、 c++析构函数、 sidetable_rc
    // 若没有上述对象，则直接释放
    if (fastpath(isa.nonpointer                     &&
                 !isa.weakly_referenced             &&
                 !isa.has_assoc                     &&
#if ISA_HAS_CXX_DTOR_BIT
                 !isa.has_cxx_dtor                  &&
#else
                 !isa.getClass(false)->hasCxxDtor() &&
#endif
                 !isa.has_sidetable_rc))
    {
        assert(!sidetable_present());
        free(this);
    } 
    else {
        object_dispose((id)this);
    }
}
```

```objective-c
void *objc_destructInstance(id obj) 
{
    if (obj) {
        // Read all of the flags at once for performance.
        bool cxx = obj->hasCxxDtor(); //是否有 c++ 析构函数
        bool assoc = obj->hasAssociatedObjects(); // 是否有关联对象

        // This order is important.
        if (cxx) object_cxxDestruct(obj);
        if (assoc) _object_remove_assocations(obj, /*deallocating*/true);
        obj->clearDeallocating();
    }

    return obj;
}
```



```objc
inline void 
objc_object::clearDeallocating()
{
    if (slowpath(!isa.nonpointer)) {
        // Slow path for raw pointer isa.
        sidetable_clearDeallocating(); // 清空 sidetable
    }
    else if (slowpath(isa.weakly_referenced  ||  isa.has_sidetable_rc)) {
        // Slow path for non-pointer isa with weak refs and/or side table data.
        clearDeallocating_slow();
    }

    assert(!sidetable_present());
}
```

```objc
NEVER_INLINE void
objc_object::clearDeallocating_slow()
{
    ASSERT(isa.nonpointer  &&  (isa.weakly_referenced || isa.has_sidetable_rc));

    SideTable& table = SideTables()[this];
    table.lock();
    if (isa.weakly_referenced) {
        // 清空 弱引用
        weak_clear_no_lock(&table.weak_table, (id)this);
    }
    if (isa.has_sidetable_rc) {
        // 清空 sidetable
        table.refcnts.erase(this);
    }
    table.unlock();
}
```



![dealloc流程](/Users/hexo_images/dealloc流程.png)

