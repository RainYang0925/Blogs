===**欢迎转发，注明来源；版权所有，侵权必骂。**===

Reveal作为分析APP UI的利器确实非常好用，用来查看任意UI布局也很方便：

##一.分析自己的 APP

　　1.打开Reveal（http://revealapp.com下载）

　　2.打开Xcode

　　3.Reveal——Help——Show Reveal Library in Finder
![Show](http://img.blog.csdn.net/20150820171003284)
　　　　　　

　　4.Xcode——导入Reveal.framework至当前项目中
![Xcode](http://img.blog.csdn.net/20150820171100768)
　　　　　　

　　5. 工程设置中，在Other Linker Flags项增加-ObjC -framework Reveal
![工程设置](http://img.blog.csdn.net/20150820171508549)
　　
　　6.运行当前项目后，打开Reveal，选择当前运行程序进行关联

![链接](http://img.blog.csdn.net/20150820171205845)　　　　　

　　7.连接成功后，app的UI层次元素就可以看到了。
![UI层次](http://img.blog.csdn.net/20150820171342751)
　　　　
##二、分析他人 APP

###1. 前提
* 越狱
* Cyndia安装OpenSSH,MobileSubstrate
* Mac上安装Reveal
* 越狱设备与安装Reveal的Mac在同一网段（**在此处被坑**）

###2. 步骤
* Reveal——Help——Show Reveal Library in Finder，获取libReveal.dylib
* 拷贝framework和dylib到越狱机
如果手机和 mac 在同一个网段可以用下面的命令在终端中执行：

``` shell
scp -r /Applications/Reveal.app/Contents/SharedSupport/iOS-Libraries/Reveal.framework root@172.22.X.X:/System/Library/Frameworks
scp /Applications/Reveal.app/Contents/SharedSupport/iOS-Libraries/libReveal.dylib root@172.22.X.X:/Library/MobileSubstrate/DynamicLibraries
```
或者，可以用 PP助手将Reveal.framework、libReveal.dylib直接导入到相应位置：
![PP助手](http://img.blog.csdn.net/20150820162758246)

* 新建一个文件命名为`libReveal.plist`，内容如下：
```
{   
    Filter = {  
         Bundles = ("com.apple.AppStore");   
    };   
}  
```
`Bundles`里填写要偷看的 app bundle id，把修改好的`libReveal.plist`通过 PP助手或 SCP放到/Library/MobileSubstrate/DynamicLibraries/下。

这里也可以指定多个BundleID的，同时监控任意多的app；如果不上传libReveal.plist，可以监控所有app，但会让设备很慢。

*重启越狱机，在 Reveal 中选择要分析的 APP，就可以查看 UI布局了。