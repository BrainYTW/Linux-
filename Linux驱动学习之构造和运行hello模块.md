# Linux驱动学习之构造和运行hello模块

摸索了几天，终于调试出第一个Linux设备驱动程序——hello模块。之所以会摸索几天，是因为自己配置和构造内核树，题外话，首先要准备好一个内核源代码树(可以是来自kernel.org网站，也可以是发行版的内核源代码包)，构造一个新内核，然后安装到自己的系统中。这里由于种种原因，没有构造成功，所以下一篇我将用树莓派的Linux系统构造一个新的内核。



* 1.首先，新建一个hello.c文件

```c
#include <linux/init.h>
#include <linux/module.h>
MODULE_LICENSE("Dual BSD/GPL");
static int hello_init(void)
{
	printk(KERN_EMERG "Hello, World\n");
	return 0;
}
static void hello_exit(void)
{
	printk(KERN_EMERG "Goodbye, World\n");
}
module_init(hello_init);
module_exit(hello_exit); 
```

这个模块定义了两个函数，一个在模块加载到内核时被调用hello_init，一个在模块去除时被调用hello_exit。module_init和module_exit这两行使用了特别的内核宏来指出这两个函数的角色，另一个宏MODULE_LICENSE是用来告知内核，该模块带有一个自由的许可证，没有这样的说明，在模块加载时内核会报错。

printk函数在Linux内核中定义并且对模块可用，它与标准C库函数printf的行为相似。内核需要它自己的打印函数，因为没有C库的支持。字串KERN_ALERT是消息的优先级。在此模块中指定了一个高优先级，因为使用默认优先级的消息可能不会直接显示，这依赖于运行的内核版本、klogd守护进程的版本以及配置。



* 2.为了编译模块文件，在同一目录下创建一个Makefile文件

```makefile
ifneq ($(KERNELRELEASE),)
obj-m := hello.o    # obj-m指出将要编译成的内核模块列表。*.o格式文件会自动地由相应的*.c文件生成（不需要显式地罗列所有源代码文件）
else
KDIR := /lib/modules/$(shell uname -r)/build
all:
        make -C $(KDIR) M=$(PWD) modules  # 把上述程序编译为一个运行时加载和删除的模块。
clean:
        rm -f *.ko *.o *.mod.o *.mod.c *.symvers *.order
endif
```

make -C $(KDIR) M=$(PWD) modules这个命令首先是改变目录到用 -C 选项指定的位置（即内核源代码目录，这个参数要根据自己的情况而定）。这个 M= 选项使Makefile在构造modules目标前，返回到模块源码目录。然后，modules目标指向obj-m变量中设定的模块。这里的编译规则的意思是：在包含内核源代码位置的地方进行make，然后再编译 $PWD （当前）目录下的modules。这里允许我们使用所有定义在内核源代码树下的所有规则来编译我们的内核模块。



* 3.编译完毕之后，就会在源代码目录下生成很多文件，其中有一个hello.ko文件，这就是内核驱动模块了。我们使用下面的命令来加载hello模块。

```bash
### 装载
yuantianwen@VM-0-17-ubuntu:~/driver_test$ sudo insmod hello.ko
### 卸载
yuantianwen@VM-0-17-ubuntu:~/driver_test$ sudo rmmod hello.ko
```

这时，你会发现终端里什么输出也没有，不用急，因为printk是内核输出函数，要查看的话，还要执行下列指令。

```bash
yuantianwen@VM-0-17-ubuntu:~/driver_test$ dmesg | tail
[3970428.579400] Hello, World
[3970440.776704] Goodbye, World
```

至此，一个最简单的内核模块驱动程序就完成了。^_^ 



也可参考[其他链接](https://www.jianshu.com/p/3aa1de768b63)