本文总结一下从 mac\win\模拟器等查看 crashLog的方式。

##1. 从 mac获取 crash report：
###方式一：（没有装 xcode 的 mac）
1) 将测试设备连接到 mac
通过 itunes 下载 crash log 到 mac。
2) 终端-输入 
open ~/Library/Logs/CrashReporter/MobileDevice
3) 以测试 app 命名的 log

###方式二：（装 xcode的 mac）
1) 连接测试设备
2) xcode-windows-devices-『view device log』-右键 export
![这里写图片描述](http://img.blog.csdn.net/20150430142404567)

##2. 从 windows获取 crash log
 路径：
C:\Users\<user_name>\AppData\Roaming\Apple computer\Logs\CrashReporter/MobileDevice

##3. 从模拟器获取 crash log
利用模拟器跑 monkey脚本时发生了 crash，如何查看相关信息呢？
步骤：
1）spotlight>输入『console』打开控制台 app;
2) 查看用户诊断报告下的 crash 信息
![这里写图片描述](http://img.blog.csdn.net/20150430142646466)


###参考文档
https://developer.apple.com/library/ios/documentation/IDEs/Conceptual/AppDistributionGuide/TestingYouriOSApp/TestingYouriOSApp.html#//apple_ref/doc/uid/TP40012582-CH8-SW7