---
title: load()与initialize()
tags:
  - iOS
  - runtime
  - 源码

categories:
  - iOS开发

date: 2021-03-02 19:41:28
---

基于[Objc4-781](https://github.com/ygg29/Objc4-781)

源码文件： `objc-os.mm`、`objc-runtime-new.h`

<!-- more -->

## 问题

- `+load` 和 `+initialize` 的调用有先后顺序么？
- `+load` 调用其他对象的方法时怎样的？
- 分类中的 `+load` 方法调用顺序？

## +load的调用流程



## +initialize 的调用流程



