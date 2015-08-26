# Knowledge: Memory Management

This page contains knowledge points about memory management for iOS development.


## 1. ARC (Automatic Reference Counting)

iOS 5最显著的变化就是增加了Automatic Reference Counting（自动引用计数）。ARC是新LLVM 3.0编译器的特性，完全消除了手动内存管理的烦琐。在你的项目中使用ARC是非常简单的，所有的编程都和以前一样，除了你不再调用retain, release, autorelease。启用ARC之后，编译器会自动在适当的地方插入适当的retain, release, autorelease语句。你不再需要担心内存管理，因为编译器为你处理了一切。注意ARC是编译器特性，而不是iOS运行时特性（除了weak指针系统），它也不是其它语言中的垃圾收集器。因此ARC和手动内存管理性能是一样的，有些时候还能更加快速，因为编译器还可以执行某些优化。

该机能在 iOS 5/ Mac OS X 10.7 开始导入，利用 Xcode4.2 可以使用该机能。简单地理解ARC，就是通过指定的语法，让编译器(LLVM 3.0)在编译代码时，自动生成实例的引用计数管理部分代码。ARC并不是GC，它只是一种代码静态分析（Static Analyzer）工具。


### 1.1 变化对比

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


### 1.2 指针保持对象的生命

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


### 1.3 ARC基本规则

* retain, release, autorelease, dealloc 由编译器自动插入，不能在代码中调用
* dealloc虽然可以被重载，但是不能调用 `[super dealloc]`


### 1.4 ARC的修饰符

ARC主要提供了4种修饰符，他们分别是: `__strong`, `__weak`, `__autoreleasing`, `__unsafe_unretained`。

#### `__strong`

表示引用为强引用。对应在定义 `@property` 时的 `strong`。所有对象只有当没有任何一个强引用指向时，才会被释放。

注意：___如果在声明引用时不加修饰符，那么引用将默认是强引用。___当需要释放强引用指向的对象时，需要将强引用置nil。

#### `__weak`

表示引用为弱引用。对应在定义 `@property` 时用的 `weak`。弱引用不会影响对象的释放，即只要对象没有任何强引用指向，即使有100个弱引用对象指向也没用，该对象依然会被释放。不过好在，___对象在被释放的同时，指向它的弱引用会自动被置`nil`，这个技术叫zeroing weak pointer___。这样有效得防止无效指针、野指针的产生。`__weak` 一般用在 delegate 关系中防止循环引用或者用来修饰指向由 Interface Builder 编辑与生成的 UI 控件。



## 2. 手动内存管理（`retain`, `release`, `autorelease`）

旧版本的 Objective-C 在管理内存时，遵循一套简单的规则。每一个对象都有一个名为 `retainCount` 的变量，它表示该对象有多少个引用。任何继承了NSObject 的对象都可以使用这个特性，对基本数据类型无效。


### 2.1 原理

* 每个对象内部都保存了一个与之相关联的整数，称为引用计数器（`retainCount`）
* 每当使用 `alloc`, `new` 或者 `copy` 创建一个对象时，对象的引用计数器被设置为 1
* 给对象发送一条 retain 消息（即调用 `retain` 方法），可以使引用计数器值 +1
* 给对象发送一条 release 消息，可以使引用计数器值 -1
* 当一个对象的引用计数器值为 0 时，那么它将被销毁，其占用的内存被系统回收，OC 也会自动向对象发送一条 dealloc 消息。一般会重写 `dealloc` 方法，在* 这里释放相关资源。一定不要直接调用 `dealloc` 方法。
* 可以给对象发送 `retainCount` 消息获得当前的引用计数器值。


### 2.2 内存管理原则

* 谁创建，谁释放（“谁污染，谁治理”）。如果你通过 alloc、new 或者 (mutable) copy 来创建一个对象，那么你必须调用 release 或 autorelease。或句话说，不是你创建的，就不用你去释放
* 一般来说，除了 alloc、new 或 copy 之外的方法创建的对象都被声明了 autorelease（autorelease 是延迟释放内存，不用你自己去手动释放，系统会知道在什么时候该去释放掉它。）
* 谁 retain，谁 release。只要你调用了 retain，无论这个对象是如何生成的，你都要调用 release


### 2.3 关于 `retain` 与 `release`

retain 之后 count 加一。alloc 之后 count 就是1，release 就会调用 dealloc 销毁这个对象。

如果手动调用过 retain 一次，需要 release 两次。通常在 method 中把参数赋给成员变量时需要retain。

例如，ClassA有setName这个方法：

```
-(void)setName:(ClassName*) inputName
{
    name = inputName;
    [name retain];      // 此处 retian，等同于 [inputName retain], count 等于 2
}
```

调用时：

```
ClassName *myName = [[ClassName alloc] init];
[classA setName:myName];  // retainCount == 2
[myName release];         // retainCount == 1，在 ClassA 的 dealloc 中 release，name 才能真正释放内存。
```

### 2.4 关于 `autorelease`

autorelease 更加 tricky，而且很容易被它的名字迷惑。我在这里要强调一下：autorelease 不是 garbage collection，完全不同于 Java 或者 .Net 中的 GC。

autorelease 和作用域没有任何关系！

autorelease 原理：

1. 先建立一个 autorelease pool
2. 对象从这个 autorelease pool 里面生成。
3. 对象生成之后调用 autorelease 函数，这个函数的作用仅仅是在 autorelease pool 中做个标记，让 pool 记得将来 release 一下这个对象。
4. 程序结束时，pool 本身也需要 rerlease, 此时 pool 会把每一个标记为 autorelease 的对象 release 一次。___如果某个对象此时 retainCount 大于 1，这个对象还是没有被销毁。___

上面这个例子应该这样写：

```
ClassName *myName = [[[ClassName alloc] init] autorelease]; // 标记为 autorelease
[classA setName:myName]; // retainCount == 2
[myName release];        // retainCount == 1
```

注意：在 ClassA 的 dealloc 中不能 release name，否则 release pool 时会 release 这个retainCount 为 0 的对象，这是不对的。

记住一点：如果这个对象是你 alloc 或者 new 出来的，你就需要调用 release。如果使用 autorelease，那么仅在发生过 retain 的时候 release 一次（让retainCount 始终为1）。


## 3. Resources

Below are the related resources about memory management.

* [iOS开发ARC内存管理技术要点](http://www.cnblogs.com/flyFreeZn/p/4264220.html)
* Automatic Reference Counting on iOS \([English Version](http://www.drdobbs.com/mobile/automatic-reference-counting-on-ios/240000820) | [Chinese Version, Translation](http://www.oschina.net/translate/automatic-reference-counting-on-ios)\)


