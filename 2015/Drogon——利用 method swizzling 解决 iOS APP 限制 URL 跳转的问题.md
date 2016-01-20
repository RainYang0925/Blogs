===**欢迎转发，注明来源；版权所有，侵权必骂。**===

事情起因于，有位 QA同事在使用 monkey脚本对 iOS APP做稳定性测试时， 想对一些点击事件做黑名单限制，在运行monkey脚本过程中，当点击到某些控件时，使其不发生跳转。

这篇博客就记录一下我分析和解决这个问题的思路和最终的解决办法。


----------


##1. 问题分析
首先，产生跳转的链接可以分成以下两种类型：

* iOS 的系统控件
* H5页面

第一种情况的处理方法非常简单，那就是在工程源码中定位到具体的控件，把该控件 addSubview:方法注释掉或稍作修改即可。因此解决第一类问题的方式是 code review+修改源码。

第二种情况非常复杂，iOS 通过 UIWebview展示 H5页面，想限制 H5页面中子链接的跳转，就需要一个能获取子链接的方法。然后尝试 hook 这个方法，在 swizz 的方法中加入判断。限制跳转的 URL（后文称为『黑名单』）如果比较多，并且可能经常变动的话，可以把黑名单存放在 plist文件中，swizz 方法里从 Plist 读取这个黑名单 array一判断，万事大吉。解决第二类问题的方式是利用 objective-c的 runtime机制，hook 系统的代理方法，用自己的方法代替系统方法，达到偷梁换柱的效果。


----------


##2. 解决方案

###2.1 对于系统控件
例如，下图中右上角的分享按钮：
![hao123](http://img.blog.csdn.net/20150806142455644)
对于我这个对同事的项目完全不了解的人，如何快速定位到这个按钮的位置呢？让我们通过 code review一步步搞定它：

**Step1:**在Images.xcassets中找到这个『分享』按钮图片的名称。
![Step1](http://img.blog.csdn.net/20150806143117312)

**Step2:**根据『share』这个图片名，在 xcode中搜索
```objective-c
imageNamed:@"share"
```
![Step2](http://img.blog.csdn.net/20150806143321597)

找到该控件的创建语句之后，把它添加到 subview的语句改一改就好啦，不让这个我们不想点击的控件显示出来。
**Step3:**把截图中的 share删掉
![Step3](http://img.blog.csdn.net/20150806143746253)
再执行 Monkey就不用担心这个可恶的控件把我们带到不想去的地方了。

好了，说了这么多，下面这个小节才是这篇文章的重点，喝杯咖啡，我们继续。
###2.2 H5页面
看下面的这个页面，通过 UIWebView打开一个 H5页面后，我们想限制点击『电视剧』这个子链接时，不让页面发生跳转。怎么做呢？
![H5页面](http://img.blog.csdn.net/20150806144225502)

通过查阅 UIWebView的 API，和在 demo中试验，发现只有它的代理方法
```objective-c
- (BOOL)webView:(nonnull UIWebView *)webView shouldStartLoadWithRequest:(nonnull NSURLRequest *)request navigationType:(UIWebViewNavigationType)navigationType
```
可以获取到点击子链接时的 URL。了解 iOS运行时机制的同学知道 hook一般系统方法时，我们在category的+load:类方法中，在 dispatch_once里调用
```c
method_exchangeImplementations(originalMethod, swizzledMethod);
```
就可以了，但hook UIWebView的代理方法时，实现该方法的并不是 UIWebView本身，而是它的调用者。我们需要 hook调用者类的 webView:shouldStartLoadWithRequest:navigationType:方法。

我找到的实现方案是，建一个 UIViewController的分类，在分类的.h中定义一个宏
```objective-c
#define SWIZZ_IT [UIViewController swizzIt];
```
#### 2.2.1 开启\关闭(AppDelegate.m)
作为开关，当需要使用该分类对 H5页面的子链接进行黑名单限制时，就在 AppDelegate.m的
```objective-c
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
```
方法中调用该宏。如下：
![开启 swizz](http://img.blog.csdn.net/20150806150253014)
关闭的话把这行注释掉就好了。
####2.2 接口（UIViewController+XUXSwizz.h）
定义一个宏，供外界调用。
```objective-c
#import <UIKit/UIKit.h>
#define SWIZZ_IT [UIViewController swizzIt];

@interface UIViewController (XUXSwizz)

/**
 It will swizz the delegate method of UIWebview:
 webView:shouldStartLoadWithRequest:navigationType:
 @return void
 */
+ (void)swizzIt;

@end
``` 
####2.3 具体实现(UIViewController+XUXSwizz.m)
为了不把一大坨代码都堆在+load:中，我们设置一个 static的BOOL变量，用于标记是否对代理方法进行了 hook。
```objective-c
static BOOL isSwizzed;

+(void)load
{
    isSwizzed = NO;
}
```
如果已经被 swizz了，就直接返回啥也不干。如果没有被 swizz，就通过objc_getClassList获取 classList，遍历查看它和它的父类是否遵守特定的协议，如果遵守了特定的协议，就继续判断是否执行了协议的某个方法，若执行了，则进行 swizz。

```objective-c
+ (void)swizzIt
{
    // We won't do anything if it's already swizzed
    if (isSwizzed)
    {
        return;
    }
    
    SEL originalSelector = @selector(webView:shouldStartLoadWithRequest:navigationType:);
    SEL swizzledSelector = @selector(swizzwebView:shouldStartLoadWithRequest:navigationType:);
    
    Protocol *protocol = objc_getProtocol("UIWebViewDelegate"); // The protocol containing the method
    
    // Get the class list
    int classesCount = objc_getClassList ( NULL, 0 );
    Class *classes = malloc(sizeof(Class) * classesCount);
    objc_getClassList( classes, classesCount );
    
    // For every class
    for( int classIndex = 0; classIndex < classesCount; classIndex++ )
    {
        Class class = classes[classIndex];
        
        // Check, whether the class implements the protocol
        // The protocol confirmation can be found in a super class
        Class conformingClass = class;
        while(conformingClass)
        {
            if (class_conformsToProtocol( conformingClass, protocol ))
            {
                break;
            }
            conformingClass = class_getSuperclass( conformingClass );
        }
        
        // Check, whether the protocol is found in the class or a superclass
        if (conformingClass)
        {
            // Check, whether the protocol's method is implemented in the class,
            // but NOT the superclass, because you have to swizzle it there. Otherwise
            // it would be swizzled more than one time.
            unsigned int methodsCount;
            Method *methods = class_copyMethodList( class, &methodsCount );
            
            for( unsigned methodIndex = 0; methodIndex < methodsCount; methodIndex++ )
            {
                SEL checkSel = method_getName( methods[methodIndex]);
                if (originalSelector ==  checkSel)
                {
                    // Do the method swizzling
                    swizzInstance(conformingClass,originalSelector,swizzledSelector);
                    
                }
            }
        }
    }
    free(classes);
    
    isSwizzed = YES;
}
```

由于使用了 MRC，在 ARC的项目里还需要设置一下：
![ARC的项目](http://img.blog.csdn.net/20150806155830270)

需要添加的黑名单 URL填写到FilterURL.plist中。
![FilterURL](http://img.blog.csdn.net/20150806160135039)

##3. github 下载地址
[github 地址 在这里 ](https://github.com/elisaxu/Drogon)

上面有使用说明和一个小 demo，欢迎大家使用。

##4.闲扯工具命名缘由
因为最近在看《权力的游戏》，大爱龙女，因此把这个开源工具命名为她的那条会喷火的黑龙 Drogon，也寓意是使用了method swizzling这个黑魔法编写完成该工具。
![这里写图片描述](http://vignette3.wikia.nocookie.net/asoiaf/images/e/e3/Drogon_in_Daznak%27s_Pit.jpg/revision/latest?cb=20120810133314&path-prefix=zh)
由于该工具目前只在小范围使用，难以避免有不完善的地方，还希望大家多多交流，帮助我改进Drogon。