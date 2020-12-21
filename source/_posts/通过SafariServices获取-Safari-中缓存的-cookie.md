---
title: 通过SafariServices获取Safari中缓存的cookie
date: 2017-05-12 09:51:18
category: 软件开发
tags: iOS
---

需求场景：

>通过推广人分享获得的新用户要确认与推广人产生直接联系（即与推广人绑定），iOS 因为系统封闭无法取得其他应用的信息，iOS9后苹果推出的`SafariServices`可以在应用中打开一个 Safari 页面，这个页面与 safari 共享 cookie，解决了这个问题

基本流程与思路：
>1、推广人A分享链接
>2、新用户B打开该链接（**必须用 Safari 打开，否则在 app中无法读取 cookie**）
> 3、写入信息到 cookie（这里写的是A的id，cookie 的有效时间可以自己设定）
> 4、B打开 app
>5、 **在 app中通过`SafariViewController`使服务器获取到 Safari 保存的 cookie 并通过 openurl 传给 app**
>6、B获取到 cookie

 **上面的流程都是隐形过程，用户感受不到。**

 **不过 iOS10.0之后，iOS可以通过 js 读写剪切板，即新用户打开某个连接后将所需要的内容保存在剪切板中，在需要的时候再从剪切板中取出（目前Android 多是采用这种做法）。因为使用SafariServices这种获取 cookie 的方式，`SafariViewController`始终是隐藏的，而这是苹果命令禁止的：**
>SafariViewContoller must be used to visibly present information to users; the controller may not be hidden or obscured by other views or layers. Additionally, an app may not use SafariViewController to track users without their knowledge and consent.

所以如果你仅支持 iOS10的话，使用剪切板是最安全的，

iOS 部分代码
需要添加库`SafariServices.framwork `
并引入头文件` #import <SafariServices/SafariServices.h>`
遵循协议`SFSafariViewControllerDelegate `
```objectivec
- (void)getCookieFromSafari{
    //获取 safari cookie推广人 id
    SFSafariViewController *safari = [[SFSafariViewController alloc] initWithURL:[NSURL URLWithString:@"your url"]];
          safari.view.frame = CGRectMake(0, 0, 1, 1);
    safari.delegate = self;
    UIViewController *C = [UIApplication sharedApplication].keyWindow.rootViewController;
//    [C presentViewController:safari animated:NO completion:nil];
        [C addChildViewController:safari];
        [self.window addSubview:safari.view];
}
#pragma mark ------SFSafariViewControllerDelegate{
- (void)safariViewController:(SFSafariViewController *)controller didCompleteInitialLoad:(BOOL)didLoadSuccessfully{
    
    if (didLoadSuccessfully) {
//        [controller dismissViewControllerAnimated:NO completion:nil];
                [controller removeFromParentViewController];
                [controller.view removeFromSuperview];
        }
}
```
**在 AppDelegate 中会通过方法 **
```objectivec
- (BOOL)application:(UIApplication *)application openURL:(NSURL *)url sourceApplication:(NSString *)sourceApplication annotation:(id)annotation
```
将你需要的信息通过 ``` url ```传过来