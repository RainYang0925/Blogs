#什么时候用 @autoreleasepool
ARC 并不是舍弃了 @autoreleasepool，而是在编译阶段插入必要的 retain/release/autorelease 的代码调用。

所以，ARC 之下依然是延时释放的，依然是依赖于 NSAutoreleasePool，跟非 ARC 模式下手动调用那些函数本质上毫无差别，只是编译器来做会保证引用计数的正确性。

显式使用@autoreleasepool：
1）autorelease 机制基于 UI framework，因此写非UI framework的程序时，需要自己管理对象生存周期。

2）autorelease 触发时机发生在下一次runloop的时候。因此如何在一个大的循环里不断创建autorelease对象，那么这些对象在下一次runloop回来之前将没有机会被释放，可能会耗尽内存。这种情况下，可以在循环内部显式使用@autoreleasepool {}将autorelease 对象释放。
```objective-c
for (item in BigSet){    
    @autoreleasepool {        
        //create large mem objects    
     }
}

```

3)自己创建的线程。Cocoa的应用都会维护自己autoreleasepool。因此，代码里spawn的线程，需要显式添加autoreleasepool。

4) 很长的函数、很多中间变量时。
正常情况下，你创建的变量会在超出其作用域的时候被释放掉。
而如果你的函数写的很长，在你函数运行过程中出现很多中间变量，占据了大量的内存，怎么办？
用@autoreleasepool。
在@autoreleasepool中创建的变量，会在@autoreleasepool结束的时候执行一次release，进行释放。其实@autoreleasepool就相当于一层作用域。

##参考文档
http://clang.llvm.org/docs/AutomaticReferenceCounting.html#autoreleasepool

https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/MemoryMgmt/Articles/mmAutoreleasePools.html