# Runtime 的实践（四）
## 关联对象
关联对象目前比较常用的地方就是给已有的类添加属性和对应的成员变量。

### 利用 Category 给现有的类添加属性

比如我们要给一个 NSArray 添加一个属性叫做`TRPerson *person`。

在之前的工程中，我们新建一个**Objective-C文件**。

File: `Person`

File Type:`Category`

Class:`NSArray`

然后我们在`NSArray+Person.h`里写

```
#import "TRPerson.h"

@interface NSArray (Person)

@property (nonatomic, strong) TRPerson *person;

@end
```

于是我们就给`NSArray`这个类添加了一个`TRPerson`类型的`person`

进入`main.m`文件实验一下：

```
NSArray *array = @[@"Magic", @"Unique"];
TRPerson *person = [TRPerson new];
array.person = person;
NSLog(@"%@", array.person);
```
我们新建了一个`NSArray`和一个`TRPerson`，并把`person`赋值给了`array`的属性中。

然后输出看看这个属性是否成功赋值。

于是你很开心地按下了`command+R`，发现这家伙奔溃了，奔溃原因：

```
[__NSArrayI setPerson:]unrecognized selector sent to instance 0x100400d90
```

`NSArray`并没有这个`setPerson：`这个方法，仔细一看 Xcode 还给我们两个警告。

于是我们进入`NSArray+Person.m`文件写上 get 方法和 set 方法：

```
#import "NSArray+Person.h"

@implementation NSArray (Person)

- (void)setPerson:(TRPerson *)person {
    _person = person;
}

- (TRPerson *)person {
    return _person;
}

@end
```

很好，两个**错误**，因为并没有实例变量。或许你觉得可以添加一个扩展。于是你写了一个扩展并且给他添加了一个实例变量 `TRPerson *_persion`，然后这次连编译都不通过。

于是你哭了ಥ_ಥ。。。

**Category只能给已有的类添加方法，不能添加实例变量**

## 利用 Runtime 和 Category 给现有的类添加属性

### 关联对象

关联对象是只将一个对象**a**利用一个**key**绑定一个对象**b**，之后**a**就可以利用这个**key**获取到这个对象**b**

同时，**a**可以二次绑定同一个**key**，并且会替换之前被绑定的对象。之后再通过**key**获取对象，就是一个新绑定的值了。

是不是感觉和普通的属性很像？可以赋值，可以取值，可以重复赋值然后覆盖。

的确，只是调用一个函数而已。

### 函数

Runtime 给了我们两个函数来实现**关联对象**，你只需要`#import <objc/runtime.h>`就可以使用这两个函数了。

这两个函数分别是：

```
//set function
void objc_setAssociatedObject(id object, const void *key, id value, objc_AssociationPolicy policy);

//get function
id objc_getAssociatedObject(id object, const void *key);
```

#### 赋值函数
来看看 set 函数里的东西

* `object` 要持有“别的对象”的对象，这里也就是指**a**
* `key` 关联关键字，是一个字符串常亮，是一个地址（这里注意，地址必须是不变的，地址不同但是内容相同的也不算同一个key）
* `value` 也就是值，你可以猜的出应该是值**b**了
* `policy` 这是一个枚举，你可以点进去看看这个枚举是什么：
	* `OBJC_ASSOCIATION_ASSIGN` 
	* `OBJC_ASSOCIATION_RETAIN_NONATOMIC` 
	* `OBJC_ASSOCIATION_COPY_NONATOMIC` 
	* `OBJC_ASSOCIATION_RETAIN`
	* `OBJC_ASSOCIATION_COPY`

如果你了解 Objective-C，那你一定知道上面这些枚举的作用了。

所以，我们就可以在前面的`NSArray`的分类里这么写

```
- (void)setPerson:(TRPerson *)person {
    objc_setAssociatedObject(self, "person", person, OBJC_ASSOCIATION_RETAIN_NONATOMIC);
}
```

#### 取值函数

现在来看看 get 函数里的东西

* object 持有“别的对象”的对象，这里指**a**
* key 关联关键字

看完 set 函数之后，get 函数就显而易见了！这里不用多解释，立马就可以写代码

```
- (TRPerson *)person {
    return objc_getAssociatedObject(self, "person");
}
```

### 验证结果

运行demo，成功输出如下内容：

```
2015-11-25 23:18:11.625 temp[745:79654] <TRPerson: 0x100305b60>
```

说明我们给 array 赋值的结果成功保留下来，下一次去取值可以成功取到上一次保存的值了。