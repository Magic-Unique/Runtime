# Runtime 的地位（二）

[上一篇: Runtime的地位（一）](https://github.com/Magic-Unique/Runtime/blob/master/B.1.Runtime的地位（一）.md)

[下一篇: Runtime的表现](https://github.com/Magic-Unique/Runtime/blob/master/C.Runtime的表现.md)

## 仅仅一个结构体
上一篇我们说到，Runtime 让面向对象特有的“类”以结构体的形式生存在面向过程中。那么这个结构体到底长什么样？我写不同的类，变成的结构体都不一样吗？

打开你的 Xcode，新建一个 **Mac OS X 命令行** 项目，在 main.m 中入头文件：

```
#import <objc/runtime.h>
```
然后利用`command`键和`鼠标左键`进入 Runtime 的头文件定义，你会发现一个如下的结构体：

```
struct objc_class {
    Class isa  OBJC_ISA_AVAILABILITY;
#if !__OBJC2__
    Class super_class                                        OBJC2_UNAVAILABLE;
    const char *name                                         OBJC2_UNAVAILABLE;
    long version                                             OBJC2_UNAVAILABLE;
    long info                                                OBJC2_UNAVAILABLE;
    long instance_size                                       OBJC2_UNAVAILABLE;
    struct objc_ivar_list *ivars                             OBJC2_UNAVAILABLE;
    struct objc_method_list **methodLists                    OBJC2_UNAVAILABLE;
    struct objc_cache *cache                                 OBJC2_UNAVAILABLE;
    struct objc_protocol_list *protocols                     OBJC2_UNAVAILABLE;
#endif

} OBJC2_UNAVAILABLE;
```
这个结构体的名称叫做`objc_class`，我相信你在开发中绝对不会用到这个结构体，你有可能看过这篇文章的时候才第一次见到，但是有一个东西你有可能用过，他叫`Class`

在 main.m 文件中再导入一个文件：

```
#import <objc/objc.h>
```

也点进去看看，会有这么几行：

```
typedef struct objc_class *Class;

struct objc_object {
    Class isa  OBJC_ISA_AVAILABILITY;
};

typedef struct objc_object *id;
```

我删除了中间的注释，为的是让你看的更清楚一些。

如果你有 C 语言基础，你一定知道`typedef`的语法以及含义，在这里我就不多说了。

苹果重新定义`objc_class`这个结构体，并取名为`*Class`，也就是说`Class`是一个 `objc_class`的一个指针。同样`id`也是`objc_class`的一个指针。

非常的显然了，平时我们这样编写代码：

```
id obj = [[NSObjcect alloc] init];
```

`obj`是一个`NSObject`的一个对象，而本质上`obj`是一个`id`，而`id`又是一个`objc_object`结构体的一个指针，所以你其实得到的只是一个**结构体的指针**而已。

那么同样，类其实也是有对象的，因为你可以这样打：

```
Class c = [obj class];
```

这样 c 就是 objc 的类型了，然后你可以直接通过这个 c 来创建对象：

```
id other_obj = [[c alloc] init];
```

如果你细心的话，你会发现 

`Class` 保存的一个类对象，但本质是`objc_class`的指针

`id` 保存一个对象，但本质是`objc_object`的指针

而我们所做的所有面向对象编程，对对象的操作，最终都会落实到这些结构体上。这些结构体保存着一个对象的所有信息。

[上一篇: Runtime的地位（一）](https://github.com/Magic-Unique/Runtime/blob/master/B.1.Runtime的地位（一）.md)

[下一篇: Runtime的表现](https://github.com/Magic-Unique/Runtime/blob/master/C.Runtime的表现.md)