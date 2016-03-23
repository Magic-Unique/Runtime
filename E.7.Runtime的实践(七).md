# Runtime的实践七

[上一篇: Runtime的实践(六)](https://github.com/Magic-Unique/Runtime/blob/master/E.6.Runtime的实践(六).md)

上篇中使用了方法交换实现修改一个不是自己写的类，通过这个方法可以快速的将某个类全局插入代码。但是这个方法依旧有一些需要注意的地方，在这里特地指出来。

## 方法交换最好只交换“第一实现类”的方法

何为“第一实现类”呢？

比如我们常用的几个类就是`UIView`,`UIButton`，你会发现所有的UI控件都有一个方法叫做`addSubview:`。其实这个方法的实现是`UIView`，而`UIButton`只是继承了`UIView`这个类(`UIButton`直接继承`UIControl`，`UIControl`直接继承`UIView`)，所以`UIButton`也有这个`addSubview:`方法。但是`UIButton`里并没有对`addSubview:`进行重写，所以当我们调用这个方法的时候，实际只是调用他的父类的`addSubview:`。

那么问题来了，如果我们利用上一篇的方法对`UIButton`的`addSubview:`方法做了插入代码的处理，会变成什么样呢？可以这样尝试：

```objc
#import <objc/runtime.h>

@implementation UIButton (Modify)

+ (void)load {
	Method addSubview = class_getInstanceMethod([self class], @selector(addSubview:));
	Method myAddSubview = class_getInstanceMethod([self class], @selector(my_addSubview:));
	method_exchangeImplementations(addSubview, myAddSubview);
}

- (void)my_addSubview:(UIView *)subview {
	[self setTitle:@"has a new subview" forState:UIControlStateNormal];
	[self my_addSubview:subview];
}

@end
```

运行后奔溃如下:

```
*** Terminating app due to uncaught exception 'NSInvalidArgumentException', reason: '-[UIStatusBarForegroundView setTitle:forState:]: unrecognized selector sent to instance 0x7fcbb2d11a30'
```

这里说了"向`UIStatusBarForegroundView`发送`setTitle:forState:`消息时候没有响应". 也就是说我们对`UIButton`的`addSubview`做了修改, 却影响到了`UIStatusBarForegroundView`, 我们可以断定我们修改的不是`UIButton`的方法, 而是`UIView`的方法, 并且影响到了所有`UIView`的子类. 所以我们在`UIButton`中新的方法里调用`UIButton`特有的方法的时候, 在别的类中就会出现崩溃的情况.

那如果实际当中必须要用到这个方法去修改原有类要怎么办呢?

可以在新的方法中利用`if`关键字来判断自己是不是`UIButton`, 再决定是否要执行心加入的方法:

```objc
#import <objc/runtime.h>

@implementation UIButton (Modify)

+ (void)load {
	Method addSubview = class_getInstanceMethod([self class], @selector(addSubview:));
	Method myAddSubview = class_getInstanceMethod([self class], @selector(my_addSubview:));
	method_exchangeImplementations(addSubview, myAddSubview);
}

- (void)my_addSubview:(UIView *)subview {
	if ([self isKindOfClass:[UIButton class]]) {
		[self setTitle:@"Title" forState:UIControlStateNormal];
	}
	[self my_addSubview:subview];
}

@end
```

当我们再次运行后又会出现新的崩溃:

```
*** Terminating app due to uncaught exception 'NSInvalidArgumentException', reason: '-[UIStatusBarForegroundView my_addSubview:]: unrecognized selector sent to instance 0x7fd332416c40'
```

说`UIStatusBarForegroundView`这个类没有`my_addSubview:`方法.

换位思考, 如果我们把扩展写成`UIView`的扩展不就好了吗?

```objc

#import <objc/runtime.h>

@implementation UIView (Modify)

+ (void)load {
	Method addSubview = class_getInstanceMethod([self class], @selector(addSubview:));
	Method myAddSubview = class_getInstanceMethod([self class], @selector(my_addSubview:));
	method_exchangeImplementations(addSubview, myAddSubview);
}

- (void)my_addSubview:(UIView *)subview {
	if ([self isKindOfClass:[UIButton class]]) {
		UIButton *button = (UIButton *)self;
		[button setTitle:@"Title" forState:UIControlStateNormal];
	}
	[self my_addSubview:subview];
}

@end
```

综上:

1. 如果要修改A类的b方法, 则b方法不能是A类的父类中的方法
2. 如果要修改某一方法c, 则必须到第一个实现这个类的扩展里写.


[上一篇: Runtime的实践(六)](https://github.com/Magic-Unique/Runtime/blob/master/E.6.Runtime的实践(六).md)