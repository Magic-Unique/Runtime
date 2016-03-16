# Runtime 的实践（三）

[上一篇: Runtime的实践（二）](https://github.com/Magic-Unique/Runtime/blob/master/E.2.Runtime的实践（二）.md)

[下一篇: Runtime的实践（四）](https://github.com/Magic-Unique/Runtime/blob/master/E.4.Runtime的实践（四）.md)

## 枚举属性

在操作之前，我们必须要为这个类添加一些属性，以供我们枚举

```

@interface TRPerson : NSObject

@property (nonatomic, copy, readonly) NSString *name;

@property (nonatomic, strong) NSObject *obj;

@property (nonatomic, weak, setter=resignIdentifier:) id identifier;

@property (nonatomic, assign, getter=isRight) BOOL boolValue;

@property (nonatomic, assign) CGFloat cgFloat;

@property (atomic, assign) CGRect cgRect;

@end

```


真的是蛋疼，我把所有属性会有的情况都写了出来，目的就是为了比较。通过比较来学习这一方面的知识。

回到`main.m`文件，现在开始使用 runtime.h 来枚举这些东西了 （这个时候请你假装不知道这个类有什么属性）。

```
unsigned int outCount = 0;
objc_property_t *propertyList = class_copyPropertyList([TRPerson class], &outCount);
for (int i = 0; i < outCount; i++) {
	const char *name = property_getName(propertyList[i]);
	const char *attribute = property_getAttributes(propertyList[i]);
	NSLog(@"\nname = %s\nattribute = %s", name, attribute);
}
```

首先看这个函数，可以点进去看看定义：

```
objc_property_t *class_copyPropertyList(Class cls, unsigned int *outCount);

```

这个函数的作用是枚举出一个类的所有属性。

参数一：类

参数二：个数回传地址

返回值：`objc_property_t`的数组

所以我们给第一个参数传 `[TRPerson class]`

第二个参数是为了得到这个数组的长度，所以我们在前面声明了一个变量，并把这个变量的地址传到第二个参数中，函数会自动给他赋值属性个数。

获取到了数组和数组长度，我们就可以通过 for 循环遍历这个数组了。

在循环中，我们使用到了两个方法，第一个是

```
const char *property_getName(objc_property_t *property);
```

这个函数可以返回这个属性的名称。

另一个函数是

```
const char *property_getAttributes(objc_property_t *property);
```

这个函数可以返回这个属性的属性。也就是我们在 .h 文件中写“()”中的内容。

现在我们看看输出结果。


```
2015-11-23 22:09:30.483 temp[900:74089] 
name = name
attribute = T@"NSString",R,C,N,V_name

2015-11-23 22:09:30.483 temp[900:74089] 
name = obj
attribute = T@"NSObject",&,N,V_obj

2015-11-23 22:09:30.483 temp[900:74089] 
name = identifier
attribute = T@,W,N,SresignIdentifier:,V_identifier

2015-11-23 22:09:30.484 temp[900:74089] 
name = boolValue
attribute = Tc,N,GisRight,V_boolValue

2015-11-23 22:09:30.484 temp[900:74089] 
name = cgFloat
attribute = Td,N,V_cgFloat

2015-11-23 22:09:30.484 temp[900:74089] 
name = cgRect
attribute = T{CGRect={CGPoint=dd}{CGSize=dd}},V_cgRect
```

现在我们整理一下代码和输出

```
@property (nonatomic, copy, readonly) NSString *name;
name = name
attribute = T@"NSString",R,C,N,V_name

@property (nonatomic, strong) NSObject *obj;
name = obj
attribute = T@"NSObject",&,N,V_obj

@property (nonatomic, weak, setter=resignIdentifier:) id identifier;
name = identifier
attribute = T@,W,N,SresignIdentifier:,V_identifier

@property (nonatomic, assign, getter=isRight) BOOL boolValue;
name = boolValue
attribute = Tc,N,GisRight,V_boolValue

@property (nonatomic, assign) CGFloat cgFloat;
name = cgFloat
attribute = Td,N,V_cgFloat

@property (atomic, assign) CGRect cgRect;
name = cgRect
attribute = T{CGRect={CGPoint=dd}{CGSize=dd}},V_cgRect
```

好的，这样就很明显了，name是一个很好理解的字符串。

所以详细介绍一下 attribute 这个字符串

## attribute

### 以 T 开头
每一个 attribute 都是以 T 开头，可以理解为是规定。

### 可能包含有 @ 符号
有的 attribute 有 @ 符号，有的没有。仔细观察，你会发现含有 @ 符号的都是存放对象的属性

### 属性类型
@之后就是类型，如果是 id 就为空。如果不是就是双引号加类名

如果没有 @ 就是基本数据类型，基本数据类型，至于什么对应什么，可以自行研究

这里解释一下 CGFloat 和 CGRect

CGFoat 是 double 数据类型，所以是d

而 CGRect 有两个成员，一个是 CGPoint 一个是 CGSize

CGPoint 有两个成员，都是 CGFloat，所以有 CGPoint={dd}

同理，CGSize 也有两个 CGFloat，所以有 CGSize={dd}

以此类推，CGRect={CGPoint=dd}{CGSize=dd}

### 属性属性
通过不同属性的 attribute 值的比较和OC代码的参考

之后的字母可以总结为：

 字母 | OC             | 含义
------|---------------|---
  有R | readonly      | 只读
  无R | readwrite     | 读写
  有N | nonatomic     | 线程不安全
  无N | atomic        | 线程安全
  C   | copy          | 复制
  &   | strong/retain | 强引用
  W   | weak          | 弱引用
Sxxx: | setter        | set方法
 Gxxx | getter        | get方法
 Vxxx | var           | 实例变量
 
[上一篇: Runtime的实践（二）](https://github.com/Magic-Unique/Runtime/blob/master/E.2.Runtime的实践（二）.md)

[下一篇: Runtime的实践（四）](https://github.com/Magic-Unique/Runtime/blob/master/E.4.Runtime的实践（四）.md)

 