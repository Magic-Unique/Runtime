# Runtime的表现

[上一篇: Runtime的地位(二)](https://github.com/Magic-Unique/Runtime/blob/master/B.2.Runtime的地位(二).md)

[下一篇: Runtime的使用后果](https://github.com/Magic-Unique/Runtime/blob/master/D.Runtime的使用后果.md)


## 结构体解析

这一篇是对 runtime 涉及的两个结构体做一些解析。首先我整理了一下 `objc/objc.h`和`objc/runtime.h`两个头文件的内容，让大家好看一些：

```
typedef struct objc_class {
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

} *Class OBJC2_UNAVAILABLE;
//以上是一个类的结构体

typedef struct objc_object {
    Class isa  OBJC_ISA_AVAILABILITY;
} *id ;
//以上是一个对象的结构体

```
我们从对象结构体开始介绍(objc_object)。

当我们获取到一个对象的时候，说白了就是一个对象的引用，也就是一个`id`。上面那个结构体被重定义为`*id`，那么`id`就是一个`objc_object`的指针。

这个结构体里有一个成员，是类型为`Class`的一个`isa`。我们可以这么理解，我们实例化出来的对象，它实际上只保存一个`类的结构体`的指针，也就为了说明这个对象是由什么类实例化出来的。

现在来看看类的结构体(objc_class)。

对象被实例化出来后，他就保存了那个创造自己的类的信息，这个信息也就是一个`objc_class`结构体，那么我们来看看，这个结构体到底保存了那些关于一个类的信息

* `Class isa` 
	* 保存一个`objc_class`的一个指针，叫做`isa`
	* 这个`isa`和`objc_object`中的重名，具体内容和区别之后分析
* `Class super_class`
	* 保存一个`objc_class`的一个指针，叫做`super_class`
	* 显而易见，这里就是一个指向父类的指针，为了保存自己是从哪里集成来的
* `char *name`
	* 保存这个类名的字符串指针
	* 这个也不用多解释，NSObject类里这里的值就是"NSObject"
* `long version`
* `long info`
* `long instance_size`
	* 以上三个成员不多解释，用不到
* `struct objc_ivar_list *ivars` 
	* 从定义可以看出，是一个结构体指针，保存着这个类的实例变量列表
* `struct objc_method_list **methodLists`
	* 从定义可以看出，是一个结构体指针，保存着这个类的方法列表
* `struct objc_cache *cache` 
	* 从定义可以看出，是一个结构体指针，保存缓存
	* 什么东西可以保存到缓存，这个下文会解释
* `struct objc_protocol_list *protocols`
	* 从定义可以看出，是一个结构体指针，保存协议列表

我们写的类，实际上也就是这些东西，一些实例变量、一些方法、实现一些协议，还给类取了个名字，拥有父类的所有特性。

现在我们终于知道我们写的类最终会变成了一个什么样的结构体。

你一定会问，这些结构体对我们有什么用？它仅仅是编译时候的产物，和开发有什么关系。


## 结构体的作用

结构体，就是一块内存，用于统一保存大小不同且不变的集合。

**在 C 语言中，允许我们去创建与修改结构体**

对，你没听错，就是**修改**。我们可以动态的修改这些结构体内的内容，甚至你在一个对象创建后，你可以真真切切地修改他的类型和父类。你甚至可以为一个类动态的增加方法，而不是在 .m 文件中去写方法。同样，你甚至可以去**修改苹果写的类**。

[上一篇: Runtime的地位(二)](https://github.com/Magic-Unique/Runtime/blob/master/B.2.Runtime的地位(二).md)

[下一篇: Runtime的使用后果](https://github.com/Magic-Unique/Runtime/blob/master/D.Runtime的使用后果.md)
