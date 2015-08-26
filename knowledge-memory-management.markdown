# Knowledge: Memory Management

This page contains knowledge points about memory management for iOS development.


## ARC (Automatic Reference Counting)

iOS 5最显著的变化就是增加了Automatic Reference Counting（自动引用计数）。ARC是新LLVM 3.0编译器的特性，完全消除了手动内存管理的烦琐。在你的项目中使用ARC是非常简单的，所有的编程都和以前一样，除了你不再调用retain, release, autorelease。启用ARC之后，编译器会自动在适当的地方插入适当的retain, release, autorelease语句。你不再需要担心内存管理，因为编译器为你处理了一切。注意ARC是编译器特性，而不是iOS运行时特性（除了weak指针系统），它也不是其它语言中的垃圾收集器。因此ARC和手动内存管理性能是一样的，有些时候还能更加快速，因为编译器还可以执行某些优化。

该机能在 iOS 5/ Mac OS X 10.7 开始导入，利用 Xcode4.2 可以使用该机能。简单地理解ARC，就是通过指定的语法，让编译器(LLVM 3.0)在编译代码时，自动生成实例的引用计数管理部分代码。ARC并不是GC，它只是一种代码静态分析（Static Analyzer）工具。


### 变化对比

通过一小段代码，我们看看使用ARC前后的变化点。

没有使用 ARC 的代码：

```
@interface NonARCObject : NSObject {  
    NSString *name;  
}  
-(id)initWithName:(NSString *)name;  
@end  
 
@implementation NonARCObject  
-(id)initWithName:(NSString *)newName {  
    self = [super init];  
    if (self) {  
        name = [newName retain];  
    }  
    return self;  
}  
 
-(void)dealloc {  
    [name release];  
    [Super dealloc];  
}  
@end
```

使用 ARC 后的代码：

```
@interface ARCObject : NSObject {  
    NSString *name;  
}  
-(id)initWithName:(NSString *)name;  
@end  
 
@implementation ARCObject  
-(id)initWithName:(NSString *)newName {  
    self = [super init];  
    if (self) {  
        name = newName;  
    }  
    return self;  
}  
@end
```

我们之前使用Objective-C中内存管理规则时，往往采用下面的准则：

* 生成对象时，使用autorelease
* 对象代入时，先autorelease后再retain
* 对象在函数中返回时，使用return [[object retain] autorelease];

而使用ARC后，我们可以不需要这样做了，甚至连最基础的release都不需要了。

__使用ARC的好处__：

* 看到上面的例子，大家就知道了，以后写Objective-C的代码变得简单多了，因为我们不需要担心烦人的内存管理，担心内存泄露了
* 代码的总量变少了，看上去清爽了不少，也节省了劳动力
* 代码高速化，由于使用编译器管理引用计数，减少了低效代码的可能性

__不好的地方__：

* 记住一堆新的ARC规则 — 关键字及特性等需要一定的学习周期
* 一些旧的代码，第三方代码使用的时候比较麻烦；修改代码需要工数，要么修改编译开关

由于 XCode4.2 中缺省ARC就是 ON 的状态，所以编译旧代码的时候往往有"Automatic Reference Counting Issue"的错误信息。这个时候：

* 可以将项目编译设置中的“Objectice-C Auto Reference Counteting”设为NO。
* 如果只想对某个.m文件不适应ARC，可以只针对该类文件加上 -fno-objc-arc 编译FLAGS。


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


### ARC基本规则

* retain, release, autorelease, dealloc 由编译器自动插入，不能在代码中调用
* dealloc虽然可以被重载，但是不能调用 `[super dealloc]`


### ARC的修饰符

ARC主要提供了4种修饰符，他们分别是: `__strong`, `__weak`, `__autoreleasing`, `__unsafe_unretained`。

#### `__strong`

表示引用为强引用。对应在定义 `@property` 时的 `strong`。所有对象只有当没有任何一个强引用指向时，才会被释放。

注意：___如果在声明引用时不加修饰符，那么引用将默认是强引用。___当需要释放强引用指向的对象时，需要将强引用置nil。

#### `__weak`

表示引用为弱引用。对应在定义 `@property` 时用的 `weak`。弱引用不会影响对象的释放，即只要对象没有任何强引用指向，即使有100个弱引用对象指向也没用，该对象依然会被释放。不过好在，___对象在被释放的同时，指向它的弱引用会自动被置`nil`，这个技术叫zeroing weak pointer___。这样有效得防止无效指针、野指针的产生。`__weak` 一般用在 delegate 关系中防止循环引用或者用来修饰指向由 Interface Builder 编辑与生成的 UI 控件。



## Auto Release

retain, release, autorelease