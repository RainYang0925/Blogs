#iOS 代码格式化管理
转自http://blog.csdn.net/zhangao0086/article/details/42872263

虽然在项目创建和团队组建的初期，我们就把公共约定以及一些规范定下来了，并且由于我们的代码是通过Git来做版本控制的，web上直接就支持Markdown格式的readme文件，可以随时看到最新的版本，但是这种规范只能依靠个人的意识，或者通过代码Review来解决，而且做代码Review的时候，你也不好意思总是写上一堆诸如“这里要加个空格”、“那里要加上换行”的评论吧？如果不管，久而久之，会因为每个人的习惯不同，代码呈现出多种风格，看起来也不像一个成熟团队做出来的产品。
为了弥补Xcode代码格式化的短板，我们选择了引入一个第三方的插件：CLangFormat。

具体流程：
1. 先安装Package Manager（也可以跳过，看第2步）
官网地址：https://github.com/supermarin/Alcatraz
安装方法：在终端输入：curl -fsSL https://raw.github.com/supermarin/Alcatraz/master/Scripts/install.sh | sh
安装成功后在Xcode的Window里能看到“Package Manager”


2. 安装CLangFormat
GitHub地址：https://github.com/travisjeffery/ClangFormat-Xcode
安装方法：直接在Package Manager里搜索并安装，如果不想安装Package Manager的话，就直接把上面那个GitHub中的代码Clone下来，在Xcode中编译、运行，然后重启Xcode即可。


3.  配置CLangFormat
虽然CLangFormat本身就内置了一些标准化的代码格式化方案，但是同样可以自定义，我们就采用了自定义的方法。
具体的，在工程目录或者workspace目录下创建一个".clang-format"文件，添加类似于以下内容的参数：
```
# 基础样式  
BasedOnStyle: LLVM  
  
# 缩进宽度  
IndentWidth: 4  
  
# 圆括号的换行方式  
BreakBeforeBraces: Attach  
  
# 支持一行的if  
AllowShortIfStatementsOnASingleLine: true  
  
# switch的case缩进  
IndentCaseLabels: true  
  
# 针对OC的block的缩进宽度  
ObjCBlockIndentWidth: 4  
  
# 针对OC，属性名后加空格  
ObjCSpaceAfterProperty: true  
  
# 每行字符的长度  
ColumnLimit: 0  
  
# 注释对齐  
AlignTrailingComments: true  
  
# 括号后加空格  
SpaceAfterCStyleCast: true 
 
```

然后在Xcode的“Edit”->“CLang Format”中选中“File”，并让倒数第二行显示“Disable Format On Save”。

后面这个看实际情况，需不需要在文件随时保存的时候格式化，如果喜欢用快捷键的话，在“系统偏好设置”里能对所有的Menu选项设置快捷键，设置一个“Format File in Focus”的快捷键也很好用。

附上CLangFormat的所有可用参数文档：http://clang.llvm.org/docs/ClangFormatStyleOptions.html