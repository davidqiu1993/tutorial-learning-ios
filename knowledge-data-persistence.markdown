# Knowledge: Data Persistence

This page contains knowledge points about data persistence in iOS development.


## 1. 文件系统

不管是Mac OS X 还是iOS的文件系统都是建立在UNIX文件系统基础之上的。


### 1.1 沙盒模型

在iOS中，一个App的读写权限只局限于自己的沙盒目录中。

__沙盒模型到底有哪些好处呢?__

* 安全：别的 App 无法修改你的程序或数据
* 保护隐私：别的 App 无法读取你的程序和数据
* 方便删除：因为一个 App 所有产生的内容都在自己的沙盒中，所以删除 App 只需要将沙盒删除就可以彻底删除程序了

__iOS App 沙盒中的目录__:

* _App Bundle_: 如 `xxx.app` 其实是一个目录，里面有 app 本身的二进制数据以及资源文件
* _`Documents`_: 存放程序产生的文档数据
* _`Library`_: 默认包含两个目录，分别是 `Caches` 和 `Preferences`
* _`tmp`_: 临时文件目录

如果我们想在程序中获取上面某个目录的路径，应该如何实现呢？

下面就讲讲路径的获取，通过 `NSPathUtilities.h` 中的 `NSSearchPathForDirectoriesInDomains` 函数，我们便可以获取我们想要的路径。此函数具体声明如下：

```
NSArray* NSSearchPathForDirectoriesInDomains(NSSearchPathDirectory directory, NSSearchPathDomainMask domainMask, BOOL expandTilde); 
```

这个函数声明有如下参数：

* `directory`: 目录类型，比如 `Documents` 目录就是 `NSDocumentDirectory`
* `domainMask`: 在 iOS 的程序中，这个参数取 `NSUserDomainMask`
* `expandTilde`: 表示是否将 `~` 展开成完整路径，一般取 `YES` 即可

注意：函数返回的类型为数组，___在 iOS 中一般这个数组中只包含一个元素，所以直接取 `lastObject` 即可。___




## Resources

Below are the resources about data persistence issues.

* [iOS持久化](http://geeklu.com/2012/01/ios-persistence/)