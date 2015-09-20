title: 'Design Patterns, Decorator/AOP in Python and C/C++'
date: 2015-09-20 21:14:45
tags:
category:
---
### 设计模式之装饰器/面向切面编程在Python和C/C++中的体现
最近接触到python中的装饰器，顿感奇妙，函数前面声明一句话，即可改变函数的部分行为。不同的声明，可以从不同方面改变函数的行为，就像一栋房子重新装修了下，就有了不同的风格。
比如
```python
def hello(fn):
    def wrapper():
        print "hello, %s" % fn.__name__
        fn()
        print "goodby, %s" % fn.__name__
    return wrapper

@hello
def foo():
    print "i am foo"

foo()
```
运行后
```
$ python hello.py
hello, foo
i am foo
goodby, foo
```
追究其本质，首先是因为在python中一切皆对象，函数也不例外，所以有了动态修改函数指向对象的机会，那么经过修饰器修饰后，foo其实已不是原来的foo，而是被包装过的：`foo = hello(foo)`，再次调用时，新的foo指向了包装后的新函数，从而包括了原来的foo功能和包装器的新功能。
> 装饰器作为一种设计模式，在插入日志，性能测试，事务处理方面有着广泛应用。

作为一个C起家的程序员，看完这些突然意识到，这样的模式其实在C实践中也有很多应用，比如上面插入日志的例子，在C中就是：
```c
void foo(int x)
{
	print("in foo: %d\n", x);
}
```
修饰下，插入日志。
```c
void _foo(int x)
{
	print("in foo: %d\n", x);
}

#define foo(x)		\
	do {			\
		printf("hello\n");	\
		_foo(x);	\
		printf("goodbye\n");	\
	} while (0)
```

更进一步，C语言中有种特殊的调试技巧：某个函数被广泛调用，我们想追踪这个函数在动态执行时，被哪些函数调用，以及传入的参数是什么名称。
最简单粗暴的方法就是找出所有调用的地方，前面插入跟踪代码，打印调用函数和参数。
```c
#define ARG_1	50
#define ARG_2	100

void foo1()
{
	printf("caller %s, argument ARG_1\n", __func__);
	foo(ARG_1);
}

void foo2()
{
	printf("caller %s, argument ARG_2\n", __func__);
	foo(ARG_2);
}
```
显然这样做既费时痛苦又无扩展性，如果想跟踪其它函数，还得再这样改一遍。
改进的做法就是用宏定义同名覆盖函数，然后加上调试信息。
```c
void _foo(int x)
{
	print("in foo: %d\n", x);
}

#define foo(x)		\
	do {			\
		printf("caller %s, argument %s\n", __func__, ##x);	\
		_foo(x);	\
	} while (0)
```
只需如此修改，其它所有调用foo的地方无需修改即可达到前述跟踪目的。这里面蕴含的思想其实跟修饰器是类似的：`通过修改原函数的指向，在原函数的基础上添加新功能`。

当然这里有三点区别：
1. C实现的修饰是编译时修改，而python是运行时动态修改；
2. C函数的修饰器就是宏定义，而且需要修改原函数原型的名称，以防命名冲突；Python的修饰器是另一个函数，修饰后原函数被覆盖。
3. C函数被修饰后，并没有生成新的函数，而是在原来的函数调用处添加了一些语句，而python就容易多了，重新生成新的函数对象。


参考文章：
[coolshell的Python修饰器的函数式编程](http://coolshell.cn/articles/11265.html)
[Python装饰器与面向切面编程](http://www.cnblogs.com/huxi/archive/2011/03/01/1967600.html)