---
title: Module、xcconfig 与多环境配置
tags:
  - iOS
  - 组件化

categories:
  - iOS开发

date: 2019-04-02 15:10:16
---

## 什么是 xcconfig 

[.xcconfig](https://help.apple.com/xcode/mac/8.3/#/dev745c5c974) 文件是对单个 `Scheme` 的配置，相比于Build Setting 中数百个选项，xcconfig 文件可以用单个文件管理环境配置，让项目更易于理解和维护。

如 Pod 中的配置项管理就是使用的 `xcconfig `文件:

<img src="/Users/hexo_images/WX20210203-150349@2x.png" style="zoom:50%;" />

<!-- more -->

### xcconfig 文件的配置

1. **新建 xcconfig 文件：**

   文件命名遵循一定规则，一般为： <配置文件夹 name>-\<product name>.<配置环境>.xcconfig。

   如 `Pods-rx.debug.xcconfig`

<img src="/Users/hexo_images/xcconfig-new-file--light-2196fa7cbd39f743d406ba6de095699bd6bc8965a9f92d0d5fd98d59d54b89cd949dc82ca4fd4735b40dd55a423d462ce94085077a5b5cdfa78edffdce5e6537.png" style="zoom:40%;" />

2. **添加进对应 Scheme：**

   ![](/Users/hexo_images/WX20210203-151326@2x.png)

### xcconfig的使用

创建好了 `xcconfig ` 文件后就可以很容易的修改项目的构建配置信息，比如 App 名称、App 版本号、bundle identifier等。

#### build 环境配置

xcconfig 文件里面可以这样写：

**Config-rx-debug：**

```ruby
MARKETING_VERSION=2.0
PRODUCT_NAME = App开发版
```

**Config-rx-test：**

```ruby
MARKETING_VERSION=1.0
PRODUCT_NAME = App测试版
```

`MARKETING_VERSION `与 `PRODUCT_NAME` 分别对应项目版本号与项目名称。

![](/Users/hexo_images/WX20210203-154912@2x.png)



**详细的配置项对应名称可以在 [xcodebuildsettings](https://xcodebuildsettings.com/) 网站找到，打不开的话 [Build setting reference](https://help.apple.com/xcode/mac/8.3/#/itcaec37c2a6) 上面的内容一样。 **

这样在 `xcconfig` 文件中更改的配置，就会自动更新到具体的项目中，我们在管理一些配置方面就方便容易的多了。

#### App 内使用

可以通过 `xcconfig ` 文件配置一些环境变量，可以避免在代码中使用一些预编译指令，例如`DEBUG`宏等。

**Config-rx-debug：**

```ruby
BIAS = /
BASE_URL=http:${BIAS}/google.com
```

**Config-rx-test：**

```ruby
BIAS = /
BASE_URL=http:${BIAS}/baidu.com
```

在 info.plist 中添加对应的 key-value：

![](/Users/hexo_images/WX20210203-160723@2x.png)

在代码中使用：

```swift
        if let info = Bundle.main.infoDictionary {
            let baseurl = info["BASE_URL"] as! String
			// do something
        }
```



## Module

### Module 是什么

大多数软件都是有大量的库构成的，无论是系统库、自建库还是第三方库。

`Module`提供了一种可替代的、更简单的使用软件库的方法，它提供了更灵活的编译期特性，并消除了使用C预处理器访问库API所固有的许多问题。

### Module需要解决的问题

传统的访问库的方式（`#include`） ，是由C预处理器提供的，这是一种很糟糕的访问库 API 的方式，面临如下问题：

**编译时扩展性：**每次包含一个头文件时，编译器必须对该头文件及其包含的每个头文件进行预处理和解析。这个过程对于应用中的每个翻译单元都**必须重复**，这涉及到大量的冗余工作。在一个包含N个翻译单元和M个翻译单元的项目中，即使M个翻译单元共享了大部分的头文件，编译器仍要执行M x N项工作。c++尤其糟糕，因为模板的编译模型迫使大量代码进入头文件。

**脆弱性：**`#include`指令被预处理程序视为**文本包含**，因此在包含时受任何主动宏定义的约束。如果任何激活的宏定义碰巧与库中的名称发生冲突，就会破坏库API或导致库头本身的编译失败。举一个极端的例子，`#define std "The c++ Standard"`，然后包含一个标准库头文件:结果是c++标准库实现中一连串可怕的失败。当两个不同库的头文件由于宏冲突而交互时，会出现更微妙的现实问题，用户被迫重新排序#include指令或引入#undef指令来打破(无意的)依赖关系。

**传统解决方法：**C程序员已经采用了许多约定来解决C预处理器模型的脆弱性。例如，绝大多数头文件都需要包含保护，以确保多个包含不会破坏编译。宏的名字用`LONG_PREFIXED_UPPERCASE_IDENTIFIERS`来写，以避免冲突，一些库/框架开发人员甚至在头文件中使用___下划线的名字来避免与“正常”的名字发生冲突，(按照惯例)这些名字甚至不应该是宏。对于非c语言的开发人员来说，这些约定是进入的障碍，对于更有经验的开发人员来说，这些约定是样板，并且使我们的头文件比它们应该的要难看得多

**工具混乱：**在基于c的语言中，很难构建与软件库协同工作的工具，因为这些库的边界并不清晰。哪些头文件属于特定的库，这些头文件应该以什么样的顺序包含以保证它们能够正确编译?头文件是C, c++， objective - c++，还是这些语言的变体之一?这些头文件中的哪些声明实际上是API的一部分，哪些声明只是因为必须作为头文件的一部分而出现的?



当使用 `Module` 表示时，编译器会直接将 module 编译为一个个独立的二进制模块，并让它的API直接对应用程序可用。这种导入方式解决了很多问题：

**编译时扩展性：**`module` **只编译一次**，将模块导入翻译单元是一个恒时操作(独立于模块系统)。因此，每个软件库的API只解析一次，将M x N编译问题减少为M + N问题。

**脆弱性：**每个模块都被解析为一个独立的实体，因此它有一个一致的预处理器环境。这完全消除了对___下划线名称和类似的防御技巧的需要。此外，当遇到导入声明时，当前的预处理器定义会被忽略，因此一个软件库不会影响另一个软件库的编译方式，从而消除了包含顺序的依赖关系。

**工具混乱：**模块描述了软件库的API，而工具可以推理并将模块作为API的表示形式呈现出来。因为`module`只能独立构建，所以工具可以依赖模块定义来确保它们获得库的完整API。此外，模块可以指定它们与哪些语言一起工作，因此，例如，一个人不能意外地试图将c++模块加载到C程序中

>  iOS7.0 之后引入 Module，可以通过 `@import AFNetworking` 引入 `Module`，在支持 Module 的项目中，#import 也会被编译为 @import 的引入方式。

## Module应用

使用 .modulemap 文件，描述 module 与 header 之间的关系。

**module.modulemap ：**

```objc
framework module OCFramework {
    umbrella header "OCFramework.h" // 指定 header 文件
    export * // 导出 module
    module * { // 导出子 module
        export *
    }
}
```

> `umbrella header "OCFramework.h"` 也可以写为 `umbrella "Headers"` , Headers 实际就是头文件所在目录，两种想过一样。

**注意：**编译器会自动查找命名为`module.modulemap` 的文件，如果为自定义 modulemap 文件，需要手动设置 modulemap file：

![](/Users/hexo_images/WX20210203-175828@2x.png)

#### framework 中 OC 与 swift 混编 

framework 中没有 bridging 文件，需要使用 module 进行 OC 和 swift 的混编。

modulemap 文件写法与上面相同，OC 使用 swift 代码与常规用法相同，需要引入 \<PRODUCT_NAME>-swift.h文件,；swift 使用 OC 代码，需要将 OC 头文件导入到 header 文件里才能使用。



## 参考

[Xcode Build Configuration Files](https://nshipster.com/xcconfig/)

[Using Xcode Configuration (.xcconfig) to Manage Different Build Settings](https://www.appcoda.com/xcconfig-guide/)

[Apple configuration setting file](https://help.apple.com/xcode/mac/11.4/#/dev745c5c974)

[Clang modules](https://clang.llvm.org/docs/Modules.html)

