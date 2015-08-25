
目前的商业 APP基本都需要进行网络请求，用户携带手机处于各种网络环境下，我们的 APP在这些环境下能否依然提供良好的用户体验？
这里不讲具体的代码实现和优化方法，只讲一下如果把 APP当做黑盒，如何模拟弱网络情况，如何测试电量和流量。嗯，这篇文章是写给 QA的。

##1.网络环境模拟
###1）network link conditioner
有两种方式，首先介绍一个简单方式，只需要操作你的 iphone。
工具：Iphone>设置>developer>network link conditioner
步骤：
a) 打开 iPhone>设置>开发者>network link conditioner status打开；
![network link conditioner](http://img.blog.csdn.net/20150430150456504)
b) 选择你要的网络情况，或者自定义，增加一些丢包率使环境更接近真实，一些参数我列在3）中；

###2）Charlse
另一种方式是使用网络代理工具，不仅可以模拟网络，更可以截获请求，做更多有意思的事。
工具:charlse

下载请去http://www.charlesproxy.com/，trail 版基本够用。

步骤：
1)截取iPhone上的网络封包:“Proxy”->“Proxy Settings”，填入代理端口8888，并且勾上”Enable transparent HTTP proxying” ;
![选择 proxy settings](http://img.blog.csdn.net/20150430144039802)
![设置 proxy](http://img.blog.csdn.net/20150430144056837)

2)Terminal，输入ifconfig en0, 即可获得该电脑的IP;
![获得 IP](http://img.blog.csdn.net/20150430144102986)

3)在iPhone的 “设置”->“无线局域网“中，可以看到当前连接的wifi名,最底部有“HTTP代理”一项，我们将其切换成手动，然后填上Charles运行所在的电脑的IP，以及端口号8888;
![设置手机代理](http://img.blog.csdn.net/20150430144201205)

4)模拟慢速网络:”Proxy”–>“Throttle Setting”项，勾选上“Enable Throttling”。
5) 模拟2G 可选『Throttle preset』-『56 kbps』;
6)模拟3G 选『Throttle preset』-『3G』;
![配置网络参数](http://img.blog.csdn.net/20150430144408609)
*注意：不测弱网络时，记得把手动代理关闭~ 否则影响上网速度哈！*
###3）一些参数
可根据以下情况，设置 charlse、手机上的参数，模拟各种网络环境：
> 1) GMS 上传:14.4K/s ,下载：14.4k/s
> 2)GPRS:40         80
> 3)EDGE:118       237
> 4)3G:128      1920
> 5)HSDPA:   348      14400

测试过程再加丢包率，模拟更真实的网络环境。



##2.上网流量监控
工具：Instruments Blank> net activity

![选择 net activity](http://img.blog.csdn.net/20150430144833388)

步骤：
a)profile>blank>net activity
b)运行 app，查看监控窗口
需要关注哪些情况呢？
首先，查看网络从 WIFI切换到其他情况时，是否会终止对用户产生大量付费流量的操作，并做提醒。用户发现自己的电话账单由于你的 APP偷偷使用付费流量多出几百块钱时，想想他的怒火。。
其次，通过监控视图，可以从黑盒的角度发现 app 进行网络请求的时机，如果持续不断，耗电也会增加；

![监控流量](http://img.blog.csdn.net/20150430144838382)
