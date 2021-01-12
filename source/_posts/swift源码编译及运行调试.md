---
title: swift源码编译及运行调试
date: 2020-10-03 17:15:32
tags: 
	- iOS
	- 源码
categories:
  - iOS开发
---

#### 预期：

编译出可运行的 swift 源码，并可在 vscode 中进行 debug 调试运行。

<!-- more -->

#### 准备：

- **环境**
  - Xcode 12.3
  - Python 2.x
  - brew install cmake ninja

- **硬盘空间**。swift 编译出文件大小大约为 50G，准备一个空间充足的电脑或者移动硬盘。
- **耐心**。 编译过程较慢，足足编译了一天，当然这条和设备有关，也有的不到一个小时就OK的。

我编译出的文件夹大小为:![](/images/swift源码大小.png)

#### 编译：

编译流程在 [Git](https://github.com/apple/swift/blob/main/docs/HowToGuides/GettingStarted.md#incremental-builds-with-ninja) 上面描述的很清楚，有两种方式：通过 **XCode** 和 **ninja**，不过查过的资料有大佬反映通过 Xcode编译的兼容性不是很好，我采用的是 ninja 编译。懒得看官方的可直接往下看：

以 **swift5.3.1**为例：

- clone 项目

  ```shell
  git clone --branch swift-5.3.1-RELEASE https://github.com/apple/swift.git
  utils/update-checkout --clone
  ```

- 通过工具链安装 swift 的依赖库

  `./swift/utils/update-checkout --tag swift-5.3.1-RELEASE --clone`

- 编译

  `./swift/utils/build-script -r --debug-swift-stdlib --lldb`



#### 调试

1. 安装 vscode，并安装CodeLLDB 插件。

![](/images/swift_vscode配置.png)

``` json
{
	"version": "0.2.0", 
  "configurations": [
		{
      "type": "lldb",
      "request": "launch",
      "name": "Debug",
      "program": "${workspaceFolder}/build/Ninja-RelWithDebInfoAssert+stdlib-DebugAssert/swift-macosx-x86_64/bin/swift",
      "args": [],
    	"cwd": "${workspaceFolder}"
    } 
	]
}
```

注意：program 的文件和编译的文件路径要相同

2. LLDB配置

   为了进行 debug，需要对 LLDB 进行一些配置：

   - 找到􏲝􏰿􏱀 `CodeLLDB` 􏰧􏲠􏲡􏱗􏱟􏰞安装目录：--（1）

     ![](/images/image-20201207152145435.png)

   - 找到编译后的 `LLDB` 文件夹：--（2）

     ![](/images/image-20201207152353407.png)

将（2）中的文件全部拷贝到（1）中，同时将 （2）中`bin` 文件夹中的 lldb 文件 拷贝到 （1）中`lldb` 目录下的 lib 目录中，并将名称修改为 `liblldb.dylib`



然后可以愉快的调试了。

Enjoy it！