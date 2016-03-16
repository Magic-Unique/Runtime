# Runtime 的实践（二）

[上一篇: Runtime的实践（一）](https://github.com/Magic-Unique/Runtime/blob/master/E.1.Runtime的实践（一）.md)

[下一篇: Runtime的实践（三）](https://github.com/Magic-Unique/Runtime/blob/master/E.3.Runtime的实践（三）.md)

## 准备工作

我们知道了所有的类都是结构体指针，所有的对象也是结构体指针。我们还知道了OC的消息机制。现在我们试图去使用 objc/runtime.h 文件来进行编程。

创建一个**Mac OS X 命令行**（为了编译运行快速）工程，在`main.m`文件中引入 runtime 的头文件

```
#import <objc/runtime.h>
```

现在你就可以使用 runtime 里面的方法了。

我们要研究的是类和对象，自然需要一个类和一个对象了。

首先是需要一个类，我们自己定义一个类：

```
@interface TRPerson

@property (copy) NSString *name;

@end
```

这里我们需要一个obj，就取一个`NSArray`类型的对象

```
#import "TRPerson"

TRPerson *person = [[TRPerson alloc] init];
```

现在你可以进入`objc/runtime.h`里面看看有什么公开的函数。

对于那些一看就能看懂用途的函数我就不介绍了，接下来的三篇将会介绍三个比较常用的功能。

* 枚举属性
* 关联对象
* 方法交换


[上一篇: Runtime的实践（一）](https://github.com/Magic-Unique/Runtime/blob/master/E.1.Runtime的实践（一）.md)

[下一篇: Runtime的实践（三）](https://github.com/Magic-Unique/Runtime/blob/master/E.3.Runtime的实践（三）.md)
