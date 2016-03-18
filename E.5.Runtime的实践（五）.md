# Runtime 的实践（五）

# 此文由 @bo 撰写[冷秋稍作修改]

[上一篇: Runtime的实践（四）](https://github.com/Magic-Unique/Runtime/blob/master/E.4.Runtime的实践（四）.md)

[下一篇: Runtime的实践（六）](https://github.com/Magic-Unique/Runtime/blob/master/E.6.Runtime的实践（六）.md)

## 方法交换
我们回看`Runtime 的地位(二)`可以发现，Object-C的对象 是由一个结构体指针所构成的。结构体指针如下

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
在这里，我们可以找到  

实例变量`struct objc_ivar_list *ivars`，  
方法列表`struct objc_method_list **methodLists `，  
缓存方法列表`struct objc_cache *cache`。  

当我们向对象发送消息的时候，OC会到缓存方法列表中开始找方法的指针，如果缓存列表中找不到,就会到方法列表中找，如果本类的方法列表中找不到，就会到父类里面找，直到找到方法的指针或者最终的父类`NSObject`也找不到方法的指针为止。当找不到方法指针的时候，编译器会发出`[XXXX 某方法]unrecognized selector sent to instance 0x100400d90`的警告。
当找到方法指针的时候，OC会将会在内存中找到方法指针所指向的那个代码块，并运行它。

我们知道，程序之所以能运行，是因为方法和变量都是存在程序的内存中。所以如果我们改变了方法指针指针所指向的内存地址的内容或者直接改变了方法指针指向的地址，我们就可以改变了方法的实现。

### Runtime中的方法交换
Runtime给了我们一个函数来实现方法交换，你只需要导入`objc/Runtime.h`文件即可使用这个函数。  
这个函数是

```objc
/** 
 * Exchanges the implementations of two methods.
 * 交换两个方法的实现
 * 
 * @param m1 Method to exchange with second method.
 * @param m2 Method to exchange with first method.
 * 
 * @note This is an atomic version of the following:
 * 这个函数的实现如下:
 *  \code 
 *  IMP imp1 = method_getImplementation(m1);
 *  IMP imp2 = method_getImplementation(m2);
 *  method_setImplementation(m1, imp2);
 *  method_setImplementation(m2, imp1);
 *  \endcode
 */
void method_exchangeImplementations(Method m1, Method m2);
```
这个方法的注释官方已给出, 我们只要关注他的参数以及返回值(由于返回值为void, 所以在此没有多余的解释, 主要解释两个参数).

`Method`是Objective-C语言中的一个结构体, 在`runtime.h`头文件中有定义. 在这个函数中, `Method`顾名思义就是要交换的方法. 我们可以通过下面这个函数来获取一个类的`Method`.

```objc
Method class_getInstanceMethod(Class cls, SEL name);
```
现在这两个参数是我们平时看的见的参数. 综上所述, 我们只要将**两组**要交换的方法的`SEL`和该方法所在的`Class`传入进去即可实现方法交换.

由此最终的代码会变成:

```objc
Method m1 = class_getInstanceMethod([M1 class], @selector(method1name));
Method m2 = class_getInstanceMethod([M2 class], @selector(method2name));
method_exchangeImplementations(m1, m2);
```
如果你不知道方法交换的最终效果, 现在我们用一个很简单的例子来说明这个问题.

这是两个类的文件, 每个类都有自己的方法以及对应的实现部分

(这里的实现部分比较单纯, 只是输出一个方法名用作标记).

```objc
@interface classOne : NSObject
@end

@implementation classOne（）
- (void)methodOne {
    NSLog(@"one");
}
@end
```

```objc
@interface classTwo : NSObject
@end

@implementation classTwo（）
- (void)methodTwo {
    NSLog(@"two");
}
@end
```

按常理来说, 如果我们调用`[[classOne new] methodOne]`则会输出`one`.

同理如果调用`[[classTwo new] methodTwo]`则会输出`two`.

但是如果我们在某一个时刻执行了`一次`下面的代码

```objc
Method method1 = class_getInstanceMethod([classTwo class], @selector(methodTwo));
Method method2 = class_getInstanceMethod([classOne class], @selector(methodOne));
method_exchangeImplementations(method1, method2);

```
在此之后(直到程序结束前)，我们运行`[[classOne new] methodOne]`的时候，打印的是`two`。

# 这个是runtime的黑科技，慎用。例如千万别在一个控制器里写8个通知和写goto。
# 这个是runtime的黑科技，慎用。例如千万别在一个控制器里写8个通知和写goto。
# 这个是runtime的黑科技，慎用。例如千万别在一个控制器里写8个通知和写goto。

重要的事说三遍


[上一篇: Runtime的实践（四）](https://github.com/Magic-Unique/Runtime/blob/master/E.4.Runtime的实践（四）.md)

[下一篇: Runtime的实践（六）](https://github.com/Magic-Unique/Runtime/blob/master/E.6.Runtime的实践（六）.md)
