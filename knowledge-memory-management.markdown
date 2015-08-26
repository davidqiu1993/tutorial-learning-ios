# Knowledge: Memory Management

This page contains knowledge points about memory management for iOS development.


## ARC (Automatic Reference Counting)

iOS 5最显著的变化就是增加了Automatic Reference Counting（自动引用计数）。ARC是新LLVM 3.0编译器的特性，完全消除了手动内存管理的烦琐。在你的项目中使用ARC是非常简单的，所有的编程都和以前一样，除了你不再调用retain, release, autorelease。启用ARC之后，编译器会自动在适当的地方插入适当的retain, release, autorelease语句。你不再需要担心内存管理，因为编译器为你处理了一切。注意ARC是编译器特性，而不是iOS运行时特性（除了weak指针系统），它也不是其它语言中的垃圾收集器。因此ARC和手动内存管理性能是一样的，有些时候还能更加快速，因为编译器还可以执行某些优化。

该机能在 iOS 5/ Mac OS X 10.7 开始导入，利用 Xcode4.2 可以使用该机能。简单地理解ARC，就是通过指定的语法，让编译器(LLVM 3.0)在编译代码时，自动生成实例的引用计数管理部分代码。ARC并不是GC，它只是一种代码静态分析（Static Analyzer）工具。

### 指针保持对象的生命

ARC的规则非常简单：只要还有一个变量指向对象，对象就会保持在内存中。当指针指向新值，或者指针不再存在时，相关联的对象就会自动释放。这条规则对于实例变量、synthesize属性、本地变量都是适用的。

我们可以按所有权（ownership）来考虑ARC对象：

```
NSString *firstName = @"Ray";
```

此时 `firstName` 变量成为 `NSString` 对象的指针，也就是 `@"Ray"` 的拥有者，指向 `@"Ray"` 的引用计数加一。如下图所示：

![arc_01](http://gitlab.djicorp.com/uploads/david.qiu/learning-ios/c94c6d5793/arc_01.jpg)

当调用以下代码时：

```
self.textField.text = firstName
```

`self.textField.text` 成为了 `@"Ray"` 的另一个拥有者（一个对象的拥有者可以有多个），那么 `@"Ray"` 的引用再加一。如下图所示：

![arc_02](http://gitlab.djicorp.com/uploads/david.qiu/learning-ios/8b759e3f72/arc_02.jpg)

当以上所有指针指向新值，或者指针不再存在时，相关联的对象就会自动释放。


## Auto Release

retain, release, autorelease
