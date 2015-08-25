===**欢迎转发，注明来源；版权所有，侵权必骂。**===

前阵子 facebook开源了其静态代码扫描工具，该工具通吃 JAVA\Android\iOS，不仅可以检查 Android 和 Java 代码中的 NullPointException 和 资源泄露，也可以发现 iOS 和 C 代码中的内存泄露问题。据介绍，它能像人类一样查看代码，并作出一些推测。但它的优势是，数分钟就能看完上千行代码。Facebook已经用它修复了八成的漏洞。Infer能将代码分解，小范围分析后再将结果整合在一起，兼顾分析的深度和速度。

**谁在使用 Infer？**
Infer 已经成为 Facebook 开发流程的一个环节，包括 Facebook Android 和 iOS 主客户端，Facebook Messenger， Instagram 在内的，以及其他影响亿万用户的手机应用，每次代码变更，都要经过 Infer 的检测。

本文介绍一下在MacOS X环境下安装 Infer 和如何在实际项目中应用这款静态扫描工具。

下载地址：
https://github.com/facebook/infer

##1. 前置条件
- 安装 python 2.7
由于 MAC自带，此处不再赘述。
- 安装 [opam](https://opam.ocaml.org/doc/Install.html#OSX)
```bash
brew install opam # Homebrew, OSX Mavericks or later
```
![下载 opam](http://img.blog.csdn.net/20150617110248065)

- XCode <= 6.3, >= 6.1
- clang (在XCode命令行工具中。 可通过以下命令安装 `xcode-select --install`) 

##2.安装指南
下载 infer
```bash
git clone https://github.com/facebook/infer.git
```
安装 OCaml :

```bash
opam init --comp=4.01.0  # (answer 'y' to the question)
eval `opam config env`
opam install sawja.1.5 atdgen.1.5.0 javalib.2.3 extlib.1.5.4
```
支持 JAVA\OC静态分析
```bash
cd infer #进入 infer的安装路径

./update-fcp.sh && ../facebook-clang-plugin/clang/setup.sh && ./compile-fcp.sh # 稍等片刻，喝杯咖啡吧 :)

make -C infer
export PATH=`pwd`/infer/bin:$PATH
```

##3.例子
..infer-master/example下的示例来演示下如何使用 Infer进行静态代码扫描。
![infer 下的 示例程序](http://img.blog.csdn.net/20150617133533740)

- 单独检查某个文件时（Hello.m）：
```bash
infer -- clang -c Hello.m
```
- 检查完整项目时（ios_hello）：
```bash
infer -- xcodebuild -target HelloWorldApp -configuration Debug -sdk iphonesimulator
```
**注意：**
只有将 infer设置为PATH环境变量时才可以直接执行上述命令，否则就需要在命令中指定 infer的安装路径，例如：
```bash
../infer/bin/infer -- clang -c Hello.m
```
Hello.m的源码如下：
```objective-c
#import <Foundation/Foundation.h>

@interface Hello: NSObject
@property NSString* s;
@end

@implementation Hello
NSString* m() {
    Hello* hello = nil;
    return hello->_s;
}
@end

```

通过静态扫描，我们可以看到其中有一处需要修改：
![Hello.m静态扫描结果](http://img.blog.csdn.net/20150617134401984)
NSString 类型的变量 s 没有指定assign\retain\copy属性。

##4.项目应用实战
下面以真实项目为例，介绍使用 infer的分析过程。
首先，cd命令进入到项目所在路径，该路径下有项目启动文件`XXX.xcodeproj`，命令行中执行以下命令：

```bash
infer -- xcodebuild -target `XXX` -configuration Debug -sdk iphonesimulator
```
编译结束后，终端会显示静态扫描的结果：
![项目实战扫描结果](http://img.blog.csdn.net/20150617135243687)

我们可以通过查看 warning，判断哪些代码需要优化。

## 5. 进阶
###5.1 Top-level 命令

*infer* : 执行 Infer的主命令，用python写的脚本文件。
*inferTest* : 执行 infer tests 的 shell 脚本。通过 Buck执行测试。
*inferTraceBugs* : 在 infer 报告中聚类错误日志的 python脚本。
###5.2 辅助命令
下面的这些在 infer/bin路径下的命令不是直接调用的，但是是被 上文的top-level命令所调用。

*InferJava* : 包含 java前端的二进制文件。

*InferClang* : 包含 clang前端的二进制文件。

*InferAnalyze* : 执行分析的 infer后端二进制文件。 

*InferPrint* : 打印关于方法和 Bug 列表的分析报告的二进制文件。
       
*inferJ* : 执行分析 java文件的命令。

*BuckAnalyze* : 执行分析用 buck编译的 java项目的命令。

*inferlib.py* : 为其他脚本提供支持的python库。

*utils.py* : 为其他脚本提供支持的python库。		

*jwlib.py* : 为其他脚本提供支持的python库。
