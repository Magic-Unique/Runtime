# Runtime 的实践(一)

[上一篇: Runtime的表现](https://github.com/Magic-Unique/Runtime/blob/master/C.Runtime的表现.md)

[下一篇: Runtime的实践(二)](https://github.com/Magic-Unique/Runtime/blob/master/E.2.Runtime的实践(二).md)

## 检验 OC 的消息机制
说了长篇大论，似乎遗漏了一个知识点，就是 OC 的消息机制。

OC 的消息机制是指，在外部需要执行某个对象的方法时，使用的方式是“发送消息”而不是“调用”。

在学 Runtime 之前你绝对不理解为什么是发送消息而不是调用。

其实很简单，最终结果都是执行一段代码，但是 OC 要执行哪段代码是不会确定的，而“调用”是确定的。

> 不确定的原因
> 
> Runtime 允许我们去修改一个方法的实现，也就是说，外部告诉一个对象，执行“xxx”方法，但是那个对象是否执行和执行什么内容是不能确定的。
> 
> 比如我写了一个方法
> 
> ```
> in Class MyClass:
> - (void)aMethodCreatedByMyself {
>     NSLog(@"%s", "aMethodCreatedByMyself");
> }
> ```
> 
> 然后我在外部调用
> 
> ```
> MyClass *myObj = [[MyClass alloc] init];
> [myObj aMethodCreatedByMyself];
> ```
> 
> 它正常输出了 aMethodCreatedByMyself
> 
> 看似确定的代码，其实是不确定的。因为你没有写任何的 Runtime 代码，所以这个方法没有做过任何的“运行时修改”。所以你觉得它很确定。但是如果我在别的地方，对这个方法做修改，修改他的实现内容，那么输出结果就会大不一样。
> 
> 如果你现在不理解也没事，之后学到方法修改的时候，你就会恍然大悟了。
> 


现在来上手操作一下
新建一个**Mac OS X 命令行**工程

然后你得到了一个 main.m 文件

在 main.m 文件中写一句最简单的实例化对象的代码

```
id obj = [[NSObject alloc] init];
//或许你喜欢上面这种写法，但是这里我希望你用下面的写法
//这样我们会更好地看出编译后到底长什么样。
id obj = [NSObject alloc];
obj = [obj init];
```

保存一下后，用终端 cd 到这个文件的目录(我的项目名叫 Runtime，存放在桌面)

```
cd ~/Desktop/Runtime/Runtime
```

然后执行编译命令，编译成 C++

```
clang -rewrite-objc main.m
```

然后你就可以得到一个 main.cpp 文件，这就是 C++ 文件了。

打开这个文件，到文件的最后，你可以找到你写的 main 函数，看看它有什么内容

```
id obj = ((NSObject *(*)(id, SEL))(void *)objc_msgSend)((id)objc_getClass("NSObject"), sel_registerName("alloc"));
obj = ((id (*)(id, SEL))(void *)objc_msgSend)((id)obj, sel_registerName("init"));
```

似乎看不懂，我可以帮他稍微简略一下。

首先我们写的第一句话

```
id obj = [NSObject alloc];
//下面是翻译C++后的结果
Class classForAlloc = objc_getClass("NSObject");//get class
SEL selForAlloc = sel_registerName("alloc");//get selector
id obj = objc_msgSend(classForAlloc, selForAlloc);//do selector and get obj

```

1. 通过一个字符串创建一个类的结构体指针
2. 通过一个字符串创建一个类方法的结构体指针
3. 向类结构体发送一个消息，得到一个结果。

之后我们写了另一句话

```
obj = [obj init];
//翻译后
SEL selForInit = sel_registerName("init");
obj = objc_msgSend(obj, selForInit);
```
这就是一个很简单的消息机制，把面向对象变为面向过程。

[上一篇: Runtime的表现](https://github.com/Magic-Unique/Runtime/blob/master/C.Runtime的表现.md)

[下一篇: Runtime的实践(二)](https://github.com/Magic-Unique/Runtime/blob/master/E.2.Runtime的实践(二).md)

