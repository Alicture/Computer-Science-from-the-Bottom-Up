# 实现抽象

总的来说，抽象是被一个总称为***应用程序接口***（API）的东西实现。API是一个有点模糊的术语，它在多种编程尝试的环境中意味着不同的东西。基本上，程序员设计函数集然后为他们的接口和设计功能写文档并遵守一个原则：提供API的确切实现是不透明的。

例如，许多大型网络应用提供可以通过HTTP访问的API。通过这种方式访问数据其实触发了许多一系列复杂的远程过程（子程序）调用，数据库查询指令和数据转存，所有的这些对于只是简单的收到约定数据的最终用户是不透明的。

那些熟悉***面向对象***语言例如Java，Python或C++的人将会对***类***提供的抽象很熟悉。方法提供了通向类的接口，但是抽象了实现。

## 使用C语言实现抽象

***指针函数***，一个被用于缺乏内建面向对象概念的Linux和其它大型C代码库中的常见方法。学习阅读这个格式是了解大型C代码库的关键。通过理解如何阅读代码中提供的抽象能够建立对内部API设计的理解。

### 例1.1.具有指针函数的抽象

```c
#include <stdio.h>

/* The API to implement */
struct greet_api
{
	int (*say_hello)(char *name);
	int (*say_goodbye)(void);
};

/* Our implementation of the hello function */
int say_hello_fn(char *name)
{
	printf("Hello %s\n", name);
	return 0;
}

/* Our implementation of the goodbye function */
int say_goodbye_fn(void)
{
	printf("Goodbye\n");
	return 0;
}

/* A struct implementing the API */
struct greet_api greet_api =
{
	.say_hello = say_hello_fn,
	.say_goodbye = say_goodbye_fn
};

/* main() doesn't need to know anything about how the
 * say_hello/goodbye works, it just knows that it does */
int main(int argc, char *argv[])
{
	greet_api.say_hello(argv[1]);
	greet_api.say_goodbye();

	printf("%p, %p, %p\n", greet_api.say_hello, say_hello_fn, &say_hello_fn);

	exit(0);
}
```

上面的代码是在整个Linux内核以及其他C程序中反复使用的结构最简单的示例。让我们看看一些特定的元素。

我们以定义API的结构体（``struct greet_api``）开始。名称被圆括号包住且具有指针标记的函数被描述为***指针函数***[^1]。这个指针函数描述了它必须指向的函数的***原型***；在函数中指向它却没有正确的返回类型或参数会最终产生一个编译警告；如果把它留在代码中也许会导致运行错误或崩溃。

接着我们看一下API的实现。通常对于更加复杂的功能你将会看到一种风格：API实现函数只是包装着其他通常有一个或两个下划线[^2]前缀的函数（就是说：``say_hello_fn()``将会调用另一个函数`` _say_hello_function()``）。这有几种用处；总的来说它与API从更复杂的功能中分离更简单以及更小的部分（比如组织或检查参数）有关，这通常化简了当确保API保持不变时通向内部运行中重要改变的路径。不管怎样，我们的实现非常简单，甚至不需要它自己的支持函数。在许多项目中，单-，双-，甚至三下划线函数前缀将有不同含义，但整体上它是一种视觉警告，警告这个函数不应该在API之外直接调用。

倒数第二点，我们在 ``struct greet_api greet_api`` 中填写指针函数。这个函数的名字是个指针；所以不需要对这个函数取地址（即：``&say_hello_fn``）。

最终我们可以在main中的结构体调用API函数。

查看源码时你会不断的看到这种风格。下面给出一个来自于 Linux内核源码中``include/linux/virtio.h``的小小实例用来说明：

### 例1.2 include/linux/virtio.h中的抽象

```c
  /**
 * virtio_driver - operations for a virtio I/O driver
 * @driver: underlying device driver (populate name and owner).
 * @id_table: the ids serviced by this driver.
 * @feature_table: an array of feature numbers supported by this driver.
 * @feature_table_size: number of entries in the feature table array.
 * @probe: the function to call when a device is found.  Returns 0 or -errno.
 * @remove: the function to call when a device is removed.
 * @config_changed: optional function to call when the device configuration
 *    changes; may be called in interrupt context.
 */
struct virtio_driver {
        struct device_driver driver;
        const struct virtio_device_id *id_table;
        const unsigned int *feature_table;
        unsigned int feature_table_size;
        int (*probe)(struct virtio_device *dev);
        void (*scan)(struct virtio_device *dev);
        void (*remove)(struct virtio_device *dev);
        void (*config_changed)(struct virtio_device *dev);
#ifdef CONFIG_PM
        int (*freeze)(struct virtio_device *dev);
        int (*restore)(struct virtio_device *dev);
#endif
};
```

只要稍微的理解这个结构体是一个虚拟I/O设备的描述。我们可以看到这个API的用户（设备驱动作者）需要提供许多在操作系统运行的不同情况下被调用的函数（当探测到新硬件，当硬件被撤除等等）。它也包含了一系列数据；结构体应该包含相关数据。

以类似这样的描述符开始了解内核代码的重多层次通常是最简单的方法。

### 库

库有两个说明抽象的作用。

* 允许程序员重复使用经常调用的代码
* 作为一个***暗箱***为程序员实现功能。

例如，一个实现存取JPEG文件原数据的库有两个优势，许多希望存取图片文件的程序能够都使用同一个库；编写这些程序的程序员不需要去担心JEPG文件格式的具体细节，而是全神贯注在他们的程序想对图像做什么。

UNIX平台的标准库被统称为***libc***。它提供了系统的基本接口：基本调用例如``read()``,``write()``以及``printf()``。这个API在它的完整版中被一个称为POSIX的规范所描述。它可以免费在线获取并且描述了组成标准UNIX API的许多调用。

大多数UNIX平台大体上遵循了POSIX标准，虽然经常在一些细枝末节处有差异，但是有时在重要的地方（因此各种GNU autotools 的错综复杂经常尝试为你将这些差别抽象走）。Linux有许多没有被POSIX说明的接口；仅仅用它们写应用程序会使得你的应用程序可移植性降低。

库是具有众多细节的基本抽象。后面的章节将会以更加详细的方式讲述库是如何工作的。





_____

[^1]: 你经常会发现参数的名称被省略，但只有参数的类型被具体声明。这允许了实现者去声明他们自己参数名称避免编译器发出警告。
[^2]: 双下划线函数 __foo 也许会在交谈中被称为 "dunder foo"。

 

