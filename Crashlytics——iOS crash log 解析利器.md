# Crashlytics——iOS crash log 解析利器

给我一个不用Crashlytics来解析 iOS crash log的理由，反正我是无法拒绝这个好用的东东的。

Crashlytic 成立于2011年，是专门为移动应用开者发提供的保存和分析应用崩溃信息的工具。Crashlytics的使用者包括：支付工具Paypal, 点评应用Yelp, 照片分享应用Path, 团购应用GroupOn等移动应用。

2013年1月，Crashlytics被Twitter收购，成为又一个成功的创业产品。被收购之后，由于没有了创业公司的不稳定因素，我们更有理由使用它来分析应用崩溃信息。

使用Crashlytics的好处有：

1、Crashlytics不会漏掉任何应用崩溃信息。在发生崩溃后，用户再次进入 APP并联网情况下，日志自动上传。
2、Crashlytics可以象Bug管理工具那样，管理这些崩溃日志。例如：Crashlytics会根据每种类型的Crash的出现频率以及影响的用户量来自动设置优先级。对于每种类型的Crash，Crashlytics除了会像一般的工具提供Call Stack外，还会显示更多相关的有助于诊断的信息，例如：设备是否越狱，当时的内存量，当时的iOS版本等。对于修复掉的Crash日志，可以在Crashlytics的后台将其关掉。下图所示的是一个我的早期应用的崩溃记录，在我修复后，我将其更新为已修复状态。
3、Crashlytics可以每天和每周将崩溃信息汇总发到你的邮箱，所有信息一目了然。
4、拥有最新的系统表文件，不需要负担任何维护成本。
5、聚类清晰 ，可收起对分析无用的信息，机型、影响用户等信息完善。

##部署方法
###前提条件

 1. mac 系统
 2. 可以访问外国网站（用代理或公司内部代理）
 3. 下载 [fabric](https://ssl-download-crashlytics-com.s3.amazonaws.com/fabric/builds/Fabric-latest.zip)

###步骤
进入[Crashlytics官网](http://try.crashlytics.com/) ，注册一个账户。
![Crashlytics](http://bos.nj.bpc.baidu.com/v1/agroup/3563df279d976ef6e474f63a39df9b1db7d65cdd)
![注册账户](http://bos.nj.bpc.baidu.com/v1/agroup/55c0a600a7f4b0ba4341eaf3d0245b29116eab4b)

在使用Crashlytics前需要对原有的XCode工程进行配置，在这一点上，Crashlytics做得比其它任何我见过的SDK提供商都体贴。因为Crashlytics专门做了一个Mac端的App来帮助你进行配置，所以，在配置前你先需要去这里下载该应用。

应用下载后，运行该应用并登录帐号，添加项目：
![添加项目](http://bos.nj.bpc.baidu.com/v1/agroup/240f54979989e4983ddaea997c8083247d5d6441)
按照部署应用的步骤一步步执行。
双击需要集成的项目名：
![项目名](http://bos.nj.bpc.baidu.com/v1/agroup/bfd798b8bce349ffb2358c3c7065308b530b928e)
点击安装：
![图片](http://bos.nj.bpc.baidu.com/v1/agroup/f5bae83afa9a49047f0e76cec0cbdf1f99ed0d77)
按照应用的提示[给项目添加 run script](http://www.runscriptbuildphase.com/?utm_source=desktopapp&utm_medium=setup&utm_campaign=mac):
![run script](http://bos.nj.bpc.baidu.com/v1/agroup/3b57ac8eb4379a71420f0a7c55e6d838e6f5a38e)
按照提示把 framework拖到工程中：
![framework](http://bos.nj.bpc.baidu.com/v1/agroup/53aa3ea1b30236d6d7d6757538d201ff5480364b)

按照提示做完之后，就到了最后一步了，在AppDelegate

```objective-c
#import <Fabric/Fabric.h>
#import <Crashlytics/Crashlytics.h>
```
在didFinishLaunchingWithOptions方法中加入如下代码：
```objective-c
[Fabric with:@[[Crashlytics class]]];
[Crashlytics sharedInstance].debugMode = YES;
```
build setting里设置
![build setting](http://bos.nj.bpc.baidu.com/v1/agroup/4a56f2628272e4b07fc44351f399224b24661fa8)

手动触发一个 crash，可以在Crashlytics的dashboard里看到了
![图片](http://bos.nj.bpc.baidu.com/v1/agroup/96e674d9218ed0c82a1cb86f792205b6755a31c9)

##demo
放在我的 github 上，[地址这里](https://github.com/elisaxu/Example/tree/master/CrashlyticsDemo)。

##实现原理和使用体会

###实现原理

在原理上，Crashlytics的通过以下2步完成崩溃日志的上传和分析：

提供应用SDK，你需要在应用启动时调用其SDK来设置你的应用。SDK会集成到你的应用中，完成Crash信息的收集和上传。
修改工程的编译配置，加入一段代码，在你每次工程编译完成后，上传该工程对应的dSYM文件。研究过手工分析Crash日志的同学应该知道，只有通过该文件，才能将Crash日志还原成可读的Call Stack信息。
###使用体会

为了更加方便开发者设置相应的工程，Crashlytics提供了mac端的应用程序，帮助你检测相关工程是否正确设置并且提供相应的帮助信息。后来我还发现，该程序还会自动帮你升级Crashlytics的SDK文件。虽然这一点很体贴，但是我个人觉得还是不太友好。因为毕竟修改SDK会影响应用编译后的内部逻辑，在没有任何通知的情况下升级，我都无法确定Crashlytics有没有干坏事。不过国外的服务，特别是象Twitter这种相对较大知名度公司提供的服务要有节操得多，所以在这一点上我还是比较放心的。

使用Crashlytics可以让你摆脱管理应用崩溃记录的烦恼。

值得一提的是，Crashlytics本身的官方文档也非常健全，如果你在使用中遇到任何问题，也可以上去查看详细的文档。

#FAQ
[Crashlytics dSYM error](http://stackoverflow.com/questions/28614509/crashlytics-dsym-error)
[Crashlytics file not found](http://stackoverflow.com/questions/17754233/crashlytics-file-not-found)
[Crashlytics is not sending Crash report from iPhone](http://stackoverflow.com/questions/17818428/crashlytics-is-not-sending-crash-report-from-iphone)
[集成crashlystics](http://ezrohir.github.io/2014/12/19/%E9%9B%86%E6%88%90crashlytics.html)