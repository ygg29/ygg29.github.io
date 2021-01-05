---
title: swift协议与函数派发的几点问题
tags:
  - iOS
  - swift
  - 函数派发
  - 泛型
  - 源码
categories:
  - iOS开发
date: 2019-10-24 19:12:19
---

### 1、
问题先从一个协议说起

<!-- more -->

``` swift
protocol DefaultProtocol: NSObjectProtocol{
    func defaultImpl()
}
extension DefaultProtocol{
    func defaultImpl(){
        print("默认实现")
    }
}
```
>这是一个方法带有默认实现的协议，实现它：

``` swift
//1
class SuperClass: NSObject, DefaultProtocol{}
class SubClass: SuperClass {
    func defaultImpl() {
        print("子类实现")
    }
}
```
>使用：
``` swift
let object: DefaultProtocol = SubClass()
object.defaultImpl()
```
>输出是什么呢？

可以简单分析一下代码：
	   ` object` 指定为 `DefaultProtocol` 类型，其指针指向的是`SubClass`对象， 当调用```object.defaultImpl()``` 时，系统首先拿到```object```的 ```isa```指针找到 ```SubClass```  然后找到```SubClass```对应的虚函数表，根据偏移量找到```defaultImpl()```方法调用，输出 `子类实现`
		
大概就是这么个流程对吧
>ok 
>现在来看输出

![](/images/1240.png)


纳尼？！！不科学！
这太不唯物主义了！

难道swift实现了什么黑魔法？
>我们把代码稍微调整一下：
``` swift 
//2
class SuperClass: NSObject, DefaultProtocol{
    func defaultImpl() {
        print("父类实现")
    }
}
class SubClass: SuperClass {
    override func defaultImpl() {
        print("子类实现")
    }
}
```
>再来看输出：

![](/images/1240-20201228191236613.png)

ok， 这才是我们想要的结果。
>由1、2结合可以推测出来，当`SuperClass`没有实现协议方法时，`SubClass`对协议方法的实现并不会写到自己的`method_list`中, 所以才会调用方法的默认实现，在 2中`SuperClass`实现了协议方法，`SubClass`重写覆盖了父类在`method_list`中的方法，才能正常调用。

通过google（[这里](https://forums.swift.org/t/default-protocol-implementation-inheritance-behaviour-the-current-situation-and-what-if-anything-should-be-done-about-it/28049)）后发现这玩意很可能是个bug，并且尝试过`Java`和`kotlin`的相同实现方法都没有问题。
>那就暂且当作一个待修复的 bug 处理吧。

### 2、

>那自己写的协议与系统提供的协议会不会有不一样的表现呢？

拿个我们最熟悉的协议 `UIScrollViewDelegate`举个简单的🌰。

``` swift
class SuperVC: UIViewController, UIScrollViewDelegate {
    override func viewDidLoad() {
        super.viewDidLoad()
        self.view.addSubview(scrollView)
    }
    lazy var scrollView: UIScrollView = {
        let o = UIScrollView.init(frame: self.view.bounds)
        o.contentSize = CGSize(width: ScreenW*2, height: ScreenH*2)
        o.backgroundColor = .magenta
        o.delegate = self
        return o }()
}

class SubVC: SuperVC {
    func scrollViewDidScroll(_ scrollView: UIScrollView) {
        print("滚动\(scrollView.contentOffset)")
    }
}
```
>输出

![](/images/1240-20201228191236806.png)

**父类没有实现的协议方法在子类正常调用！！！**
？？？？？？？
![](/images/1240-20201228191236711.png)
![](/images/1240-20201228191236805.png)


如果问题到这里，就按照bug解决， 爱谁谁，以后多注意就完事了
>**但偏偏我又多做了一个试验**

### 3、

>如果实现协议的类带了泛型呢？

**Let's coding happily**

``` swift
class SuperVC<T>: UIViewController, UIScrollViewDelegate {

    var text: T?
    override func viewDidLoad() {
        super.viewDidLoad()
        self.view.addSubview(scrollView)
    }
    lazy var scrollView: UIScrollView = {
        let o = UIScrollView.init(frame: self.view.bounds)
        o.contentSize = CGSize(width: ScreenW*2, height: ScreenH*2)
        o.backgroundColor = .magenta
        o.delegate = self
        return o }()
}

class SubVC: SuperVC<NSString> {
    override var text: NSString? {get{return "范型～～～"} set{}}
    func scrollViewDidScroll(_ scrollView: UIScrollView) {
        print("滚动\(scrollView.contentOffset)")
        print(text!)
    }
}
```
>输出：
>
>**并没有任何输出！**

**同问题1一样子类跳过父类实现的协议方法并没有被调用**
而现在我只是多加了个泛型而已

试验到这我已经词穷了， 一万头草泥马，万脸懵逼。

### 问题

**1、协议中的`optional`方法, 实现的时候是通过何种方式派发的？**
**2、为什么自己实现的协议与系统表现的不一样，系统提供的协议做了什么？**
**3、swift在编译的过程中，对有范型的类除了做了泛型擦除还做了什么，以至于影响到了函数的派发？**
	
    