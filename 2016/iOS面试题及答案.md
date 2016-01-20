#基础知识

##1.什么情况使用 weak 关键字，相比 assign 有什么不同？
weak 关键词一般用于 以下场景：
1）IBOutlet 控件属性。
2）可能导致循环引用的变量。例如，delegate 代理属性。
不同：
assign 简单赋值，不更改索引计数,一般用来声明基本类型，可以用于非 OC 对象；
weak 修饰的变量是弱引用，如果没有强引用指向该变量，则被清空（nil），必须用于 OC 对象。

##2.怎么用 copy 关键字？
应用场景：
1）`带有可变类型的数据类型`，例如 NSString\NSMutableString NSArray\NAMutableArray NSDictionary\NSMutableDictionary等，因为可变非可变类型间可进行赋值，为保证非可变类型的对象不会无意间改变，所以应该在赋新值时 copy 一份，示例如下：
```objective-c
#import "ViewController.h"

@interface ViewController ()
@property(nonatomic, copy) NSString *copyedStr;
@property(nonatomic, strong) NSString *strongStr;
@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];

    NSMutableString *mStr = [NSMutableString stringWithString:@"mstr"];
    self.copyedStr = mStr;
    self.strongStr = mStr; // 若把可变类型的字符串赋值给 strong 修饰的不可变字符串，指向的同一块地址
    [mStr appendString:@"xxx"]; // 可变类型字符串改变后，不可变字符串的值也会改变

    NSLog(@"mstr = %@",mStr);
    NSLog(@"copyedStr = %@",self.copyedStr);
    NSLog(@"strongStr = %@",self.strongStr);

}
@end
```

打印结果：
```
2016-01-20 21:50:56.163 test[37555:7384657] mstr = mstrxxx
2016-01-20 21:50:56.164 test[37555:7384657] copyedStr = mstr
2016-01-20 21:50:56.164 test[37555:7384657] strongStr = mstrxxx
```
2）block，在 ARC 中只做为提示作用，编译器自动对 block 进行 copy。与用 strong 修饰一样。
[NSString属性什么时候用copy，什么时候用strong?](http://www.cocoachina.com/ios/20150512/11805.html)

##3.这个写法会出什么问题： @property (copy) NSMutableArray *array;
没有写 nonatomic,默认会使用 atomic，影响性能。
可变类型用 copy 修饰时深拷贝，产生一个不可变类型副本，当对其进行增删改时会崩溃，错误如下：
```
-[__NSArray0 addObject:]: unrecognized selector sent to instance 0x7d074380
```
示例程序：
```objective-c
#import "ViewController.h"

@interface ViewController ()
@property(nonatomic, copy) NSMutableArray *array;
@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];

    self.array = [NSMutableArray array];
    [self.array addObject:@"first"];
    NSLog(@"mArray = %@",self.array);

}
@end
```

##4.如何让自己的类用 copy 修饰符？如何重写带 copy 关键字的 setter？
遵守<NSCopy,NSMutableCopy>,并实现协议方法
```objective-c
- (id)copyWithZone:(nullable NSZone *)zone;
- (id)mutableCopyWithZone:(nullable NSZone *)zone;
```

##5.@property 的本质是什么？ivar、getter、setter 是如何生成并添加到这个类中的
本质就是创建属性指针，并生成 getter\setter

##6.@protocol 和 category 中如何使用 @property
都只会生成 getter\setter，不会生成成员变量。若要添加成员变量，需要借助运行时：
objc_setAssociatedObject
objc_getAssociatedObject

##7.runtime 如何实现 weak 属性

##8.@property中有哪些属性关键字？/ @property 后面可以有哪些修饰符？
nonatomic\atomic
readonly\readwrite
assign\strong\weak\copy
getter\setter

##9.weak属性需要在dealloc中置nil么？
不用，ARC 中若 weak 属性不再被强引用，则被置为 nil.

##10.@synthesize和@dynamic分别有什么作用？

##11.ARC下，不显式指定任何属性关键字时，默认的关键字都有哪些？
对于基本数据类型：atomic\readwrite\assign
对于普通 OC 对象：atomic\readwrite\strong

##12.@synthesize合成实例变量的规则是什么？假如property名为foo，存在一个名为_foo的实例变量，那么还会自动合成新变量么？

##13.在有了自动合成属性实例变量之后，@synthesize还有哪些使用场景？

##14.objc中向一个nil对象发送消息将会发生什么？

##15.objc中向一个对象发送消息[obj foo]和objc_msgSend()函数之间有什么关系？

##16.什么时候会报unrecognized selector的异常？

##17.一个objc对象如何进行内存布局？（考虑有父类的情况）

##18.一个objc对象的isa的指针指向什么？有什么作用？

##19.下面的代码输出什么？

##20.runtime如何通过selector找到对应的IMP地址？（分别考虑类方法和实例方法）

##21.使用runtime Associate方法关联的对象，需要在主对象dealloc的时候释放么？

##22.objc中的类方法和实例方法有什么本质区别和联系？
