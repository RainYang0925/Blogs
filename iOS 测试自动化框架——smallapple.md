继上面几篇博客所讲的 iOS 专项测试，本篇给大家介绍一款好用的自动化测试框架——smallapple。

该框架由百度移动云测试部的同事开源在 github上，下面我先来讲一下这个框架的用法，跟官网上有些不同，因为在使用过程中遇到一些钉子，在此文中也给大家一一指出，防止重复踩坑。


----------


##1. smallapple简介
###1）下载地址
github：[https://github.com/hyxbiao/smallapple](https://github.com/hyxbiao/smallapple)
###2）介绍
Smallapple是一个开源的IOS自动化测试工具，旨在提供一套完整的IOS自动化测试解决方案，提供针对IOS App的功能和性能测试，同时提供类似Android adb、重签名、instruments结果解析、录制回放等工具集。

Smallapple通过一键式的执行方式，自动完成App重签名、安装、测试、性能采集（包括CPU、内存、流量等）、Crash检测和结果报告等工作。

###3）特性
Smallapple致力于以最简单的方式，最小的代价提供给用户使用。

>* 支持非越狱设备
>* 不需要依赖源码
>* 支持Appstore或者第三方下载的App安装测试
>* 完全的命令行模式

Smallapple是基于苹果官方的instruments工具来开发，测试脚本支持原生的UI Automation，同时扩展了很多功能使得自动化测试更加便捷、易用。

##2.如何部署
首先，硬件方面需要：
>* 一台 MAC（ macbook pro、air、iMac均可）
>* 要求OSX 10.9+
>* 安装 XCode 5.1+
>* 确保已安装Apple Developer Tools (command line tools)

其次，在苹果开发者网站上$99（688RMB）买一个开发者帐号，购买步骤请自行百度，本文不再赘述。
然后按照下图的步骤，添加 development证书，认证 App ID,添加你的测试设备，生成 develop profiles.

![开发者帐号步骤](http://img.blog.csdn.net/20150512220408011)

IOS Developer开发者证书(Certificates)，可通过以下命令在终端查看。
```
security find-identity -v -p codesigning
```

好，关于硬件和开发者帐号搞完之后，可以从 github上下载最新的 smallapple了。解压之后，把用上面的开发者帐号打好的包放到 smallapple根目录下，如下图。（什么？你还不会打包？请百度之）
![把测试包放在根目录下](http://img.blog.csdn.net/20150512221227976)

##3.快速开始
cd到 smallapple根目录下，在终端里输入：
```
sh smallapple.sh automation -p mobileprovision的绝对路径 -i "开发证书，通过security find-identity -v -p codesigning命令获得"  -o result yourapp.ipa
```
运行完之后，如果没啥报错，在根目录下会生成一个 result文件夹，打开该文件夹下的 result.html可查看测试报告，华丽丽有木有。点击橙色的小点可以查看该时间点的截图。
![测试报告](http://img.blog.csdn.net/20150512222338484)

##4.扩展
大多数情况下，只需要提供一个编译好的App(.ipa or .app)即可进行自动化的测试。

建议每次新跑测试时将原来的result目录删除或者-o指定其它目录。

man一下：
```
usage: smallapple automation [options] <.ipa/.app path | bundle id>
options:
    -s <device id>                 : specify device id. default the first found device
    -b                             : use bundle id instead of app path
    -o <result dir>                : result direcotry. default $PWD/result

script options:
    -t <template>                  : instruments template. default SMALLAPPLE/templates/Automation_Monitor_CoreAnimation_Energy_Network.tracetemplate
    -c <script>                    : instruments automation js. default SMALLAPPLE/scripts/UIAutoMonkey.js

resign options:
    -p <.mobileprovision path>     : .mobileprovision path
      or  -e <entitlement path>    : entitlement path
    -i <developer identity>        : ios developer identity

crash options:
    -d <dsym path>                 : dSYM path for crash analyze

example:
    smallapple automation test.ipa
    smallapple automation -b com.baidu.BaiduMobile
    smallapple automation -c <testcase> -b com.baidu.BaiduMobile
    smallapple automation -s <device> -p <provision> -i <identity> -c <testcase> test.ipa
```
好，知道了-c -o的含义后，我们来做点更有意思的配置。
###4.1 Monkey测试

在没有指定测试脚本情况下，smallapple会对test.ipa进行默认Monkey测试，crash、性能数据和报告都会保存在result目录下。
```
$ sh smallapple.sh automation -o result test.ipa
$ open result/report.html
```
###4.2 指定run.js脚本测试
```
$ sh smallapple.sh automation -c run.js -o result test.ipa
```
###4.3 对已安装在设备上的开发版App测试
需要Bundle Id来指定App，如果不记得可通过以下两种方式获取：

**通过App对应的安装包**
```
$ sh smallapple.sh appinfo CFBundleIdentifier test.ipa
```
或者**通过iosutil**查看已安装App，找到对应的Bundle Id
```
$ bin/iosutil listapp
```
对**Bundle Id**为com.smallapple.abc的App进行测试
```
$ sh smallapple.sh automation -c run.js -o result -b com.smallapple.abc
```
###4.4 对第三方App重签名测试

使用-p指定描述文件(Provisoning Profiles)，-i指定开发者证书(Certificates)
```
$ sh smallapple.sh automation -p <provision path> -i "iPhone Developer: smallapple" -c run.js -o result other.ipa
```
###4.5 可视化脚本录制回放

smallapple record或者直接open bin/iOSRecorder.app即可打开录制回放工具
```
$ sh smallapple.sh record
```
##5.更多介绍
###5.1 覆盖安装测试

bin/iosutil可以查看指定App的数据文件，并且可以将App的Documents和Library目录下文件拷贝到Mac机进行旧版本数据的备份，或者将Mac机文件还原上传到这2个目录下。

查看com.baidu.BaiduMobile的Documents目录
```
$ bin/iosutil ls -b com.baidu.BaiduMobile /Documents
```
备份com.baidu.BaiduMobile的Documents目录下data文件
```
$ bin/iosutil pull -b com.baidu.BaiduMobile /Documents/data ./backup
```
完成新版本安装后，还原com.baidu.BaiduMobile的Documents目录下data文件
```
$ bin/iosutil push -b com.baidu.BaiduMobile ./backup/data /Documents
```

待续未完。。