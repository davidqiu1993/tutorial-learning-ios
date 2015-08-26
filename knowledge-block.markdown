# Knowledge: Block

块（Block）类似于 C#、Java 中的匿名函数，不用块而使用其他的方法一样可以达到相同的效果。和 C#、Java一 样，块的使用可以使代码更紧凑、便于阅读，显得高大上。


## 定义块

以下例子中定义了一个叫 `myBlock` 的块：

```
- (void) method
{
    NSInteger methodVar = 1;
    
    NSInteger (^myBlock)(NSInteger, NSInteger) = ^(NSInteger p1, NSInteger p2)
    {
        // codes inside the block named myBlock
        return methodVar + p1 + p2;
    };
}
```

如例子所示，块的定义是这样的格式：

```
^(参数类型 参数名称, 参数类型 参数名称...){
    代码
}；
```

声明一个块类型的变量是这样的格式：

```
返回值类型(^变量名称)(参数类型, 参数类型...)；

```

块类型的变量即可申明为方法内部的局部变量，也可以声明为类变量。

块代码体内能够使用包含块的方法内部的变量，如本例子中块中使用了局部变量 `methodVar`。需注意的是块中代码体只能使用 `methodVar`，而不能改变 `methodVar` 的值，要想改变局部变量的值，需要在声明局部变量的时候加上关键字 `__block`。


## 使用块

块可以作为方法的参数传递，以汽车为例，汽车在启动前和启动后要加入一些行为，这个行为由驾驶员决定，如下为代码实例：

_Car.h_:

```
#import <Foundation/Foundation.h>

@interface Car : NSObject

- (void) runWithAction:(void(^)(void))beforeRunBlock afterRun:(void(^)(void))afterRunBlock;

@end
```

_Car.m_:

```
#import "Car.h"

@implementation Car

- (void) runWithAction:(void(^)(void))beforeRunBlock afterRun:(void(^)(void))afterRunBlock
{
    beforeRunBlock();
    NSLog(@"run");
    afterRunBlock();
}

@end
```

块作为方法参数时格式和声明一个块类型的变量类似，只是少了变量名称。

___注意：如果参数传来的是空（`nil`)，执行 `beforeRunBlock()` 会导致程序崩溃。___

写一个Dirvier类如下:

_Driver.h_:

```
#import <Foundation/Foundation.h>

@interface Driver : NSObject

- (void) drive;

@end
```

_Driver.m_:

```
#import "Driver.h"
#import "Car.h"

@implementation Driver

- (void) drive
{
    Car *car = [[Car alloc]init];

    void (^beforeRun)(void) = ^{
        NSLog(@"check tire");
    };

    void (^afterRun)(void) = ^{
        NSLog(@"close door");
    };

    [car runWithAction:beforeRun afterRun:afterRun];
}

@end
```

_main.m_:

```
#import <Foundation/Foundation.h>
#import "Driver.h"

int main(int argc, const char * argv[])
{
    @autoreleasepool
    {
        Driver *driver = [[Driver alloc] init];
        [driver drive];
    }
    return 0;
}
```

运行结果如下：

```
2014-03-21 13:57:48.037 BlockTest[1211:303] check tire 
2014-03-21 13:57:48.039 BlockTest[1211:303] run 
2014-03-21 13:57:48.040 BlockTest[1211:303] close door
```

Driver类中可以不定义块变量，以一种类似于匿名方法的书写格式书写：

```
#import "Dirver.h"
#import "Car.h"

@implementation Dirver

- (void) drive
{
    Car *car = [[Car alloc] init];
    [car runWithAction:^{
        NSLog(@"check tire");
    } afterRun:^{
        NSLog(@"close door");
    }];
}

@end
```


