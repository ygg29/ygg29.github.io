---
title: OC中"__attribute__ "关键字的应用
tags:
  - iOS
  - runtime
categories:
  - iOS开发
date: 2020-06-25 12:29:25
---

关于__attribute__，你可能用的不多，但是一定经常见到，在系统的Foundation框架中，__attribute__的使用非常频繁。首先，__attribute__用于在函数，变量或类型声明时进行特殊属性设置的编译器指令。需要注意，它是一种编译器指令，这也就表明了使用它我们可以做更高级的检查与优化功能。

__attribute__根据其修饰的场景不同可以分为3种类型，分别为函数属性，变量属性和类型属性。__attribute__的使用格式如下：

```javascript
__attribute__((属性列表))
```

 下面，我们来介绍几种常用的__attribute__属性。

<!-- more -->

#### 1.format

   format用来对格式化字符串的参数使用情况进行检查，例如在使用NSLog函数进行输出时，如果我们传入的可变参数没有在格式化字符串中使用，编译器会提示警告，如下：

![](/Users/hexo_images/f8zx6926jg.png)

其实这个提示警告的功能就是借助__attribute__的format属性实现的，例如我们自定义一个LOG方法使其拥有相同的功能，如下：

```javascript
void MyLog(NSString *format, ...) __attribute__((format(__NSString__, 1, 2)));
```

format属性有3个参数可以设置，其中第一个参数指定要检查的格式化风格，这里设置为NSString的格式化风格，第2个参数为格式化字符串参数的位置，从1开始，第3个参数为对应的格式化可变参数的位置。

#### 2.deprecated

   deprecated可以指定被修饰的函数在编译时产生警告，表示这个函数已经过期，提示开发者谨慎使用，例如：

![img](/Users/hexo_images/etnvkeaw97.png)

deprecated属性也可以添加一个参数来指定要输出的警告信息，例如：

![img](/Users/hexo_images/q8y03as0sg.png)

#### 3. availability

   这个属性用来指定接口的可用版本，例如指定可用的平台，接口引入的版本，废弃的版本，不可用的版本以及提示信息等，示例如下：

![img](/Users/hexo_images/1fhcaj6629.png)

其中，第1个参数指定API可用的平台，可选则为macos或ios，introduced参数设置API引入的系统版本，deprecated参数设置API废弃的版本，obsoleted参数用来设置API不可使用的版本。与availability属性相似，还有一个unavailabel属性可用，这个属性直接将某个API标记为不可用，例如：

![img](/Users/hexo_images/8953osd6hl.png)

#### 4. nonnull

   nonnull属性用来指定某些指针参数不能为空，其内可以传入一组下标(从1开始)，被下标指定的参数不可为空，例如：

![img](/Users/hexo_images/vp93vp5zbd.png)

#### 5. const

   const属性是一种编译优化功能，其可以将某个函数指定为const类型的，这时会将函数的返回值当做const变量来使用，这时如果没有使用到返回值，则函数本身会被优化，避免执行造成额外的性能消耗，例如：

![img](/Users/hexo_images/xb83r1n76m.png)

#### 5. cleanup

   cleanup属性可以实现一个非常强大的功能，它有这样的一种特点，可以为某个变量指定一个清理函数，当变量离开其作用域的时候，会调用这个清理函数，示例如下：

```javascript
void clean(int *p) {
    NSLog(@"变量%d被清理", *p);
}

int main(int argc, const char * argv[]) {
    @autoreleasepool {
        int a __attribute__((cleanup(clean))) = 10;
        a  = 20;
    }
    // 离开作用域后 将打印 变量20被清理
    return 0;
}
```

需要注意，指定的清理函数会将当前变量的指针作为参数传入。

#### 6. constructor与destructor

   constructor属性可以指定函数在main函数执行之前进行调用，与之对应destructor可以指定某个函数在main函数执行结束之后再执行。这是一种非常强大的机制，在实际应用中也非常频繁，例如对以一个拥有模块化和路由功能的应用程序，可以通过这种方式来自动化的进行路由注册(无需手动调用)，需要注意，constructor与destructor属性都可以设置一个优先级参数，优先级高的函数会先执行(0-100的优先级为系统保留)，示例如下：

```javascript
void __attribute__((constructor(101))) func1() {
    NSLog(@"Func1");
}

void __attribute__((constructor(102))) func2() {
    NSLog(@"Func2");
}

void __attribute__((destructor(101))) func3() {
    NSLog(@"Func3");
}

void __attribute__((destructor(102))) func4() {
    NSLog(@"Func4");
}

// 会依次打印
/*
 2020-04-27 16:05:57.270522+0800 TestDemo[33430:13458931] Func1
 2020-04-27 16:05:57.271017+0800 TestDemo[33430:13458931] Func2
 2020-04-27 16:05:57.271193+0800 TestDemo[33430:13458931] main函数执行
 2020-04-27 16:05:57.271273+0800 TestDemo[33430:13458931] Func4
 2020-04-27 16:05:57.271316+0800 TestDemo[33430:13458931] Func3
 */

int main(int argc, const char * argv[]) {
    @autoreleasepool {
        NSLog(@"main函数执行");
    }
    return 0;
}
```

#### 7. objc_subclassing_restricted

   这是Objective-C中常用的一个属性，有时候，我们定义了一个类，但是不希望再有其他的类继承于它，即我们要定义的类本身就是一个最终类，不能再被继承，这是就可以使用这个属性来修饰，如果有类继承它会报编译错误，例如：

![img](/Users/hexo_images/wropr69pz3.png)

#### 8. objc_requires_super

   这个属性用来修饰Objective-C中父类的方法，如果子类进行了重写，在重写的方法中没有调用父类方法，则会进行编译器提示。在实际编程中，很多时候，都是由于子类重写了父类的方法造成不可预知的问题，通过使用这个属性可以有效的对开发者进行提示，例如：

![img](/Users/hexo_images/ub68u6s6x2.png)

#### 9. enable_if

   enable_if提供了一种方式对函数的参数进行校验，不满足校验规则的参数传递将在编译时报错，使得函数的使用更加安全，例如：

![img](/Users/hexo_images/o0jp40hmv8.png)

这种编译时即可对函数参数进行检查的机制可以避免写很多运行时的代码，并且比运行时更高效的规避错误。

#### 10. overloadable

   在C语言中，对于相同的函数名，哪怕参数不同，也不能够重复定义。overliadable属性可以指定某个函数为可重载，这样既可定义名字相关参数不同的多个C函数，在调用时，编译器会根据传入的参数类型自行判断具体调用哪个函数，如下：

![img](/Users/hexo_images/fkktp9gnzh.png)

#### 11. objc_runtime_name

   这是一个很有趣的属性，其可以运行时改变Objective-C类的类名，但是不会影响其行为。这个属性通常可以用来做代码混淆加密，例如：

```javascript
__attribute__((objc_runtime_name("Some")))
@interface MyObject : NSObject

- (void)myObject:(int)param;

@end

@implementation MyObject

- (void)myObject:(int)param {
    NSLog(@"myObject");
}

@end

int main(int argc, const char * argv[]) {
    @autoreleasepool {
        MyObject *object = [[MyObject alloc] init];
        [object myObject:1];
        NSLog(@"%@", [object class]); // Some
    }
    return 0;
}
```

上面的代码可以很好的运行，但是在打印其类型时却会打印出一个莫名其妙的Some，需要注意，这个属性要谨慎使用，其有时候也会非常危险，例如代码中有做这样的逻辑就会产生未知的异常，并且很难定位：

```javascript
 [[object className] isEqualToString:@"MyObject"]
```

除了上面介绍的11中常用的属性外，可用的属性还有很多，例如对内存的分配进行管理的属性，对初始化方法进行修饰的属性等，如果有兴趣，可以参考如下文档：

https://gcc.gnu.org/onlinedocs/gcc-4.0.0/gcc/Function-Attributes.html