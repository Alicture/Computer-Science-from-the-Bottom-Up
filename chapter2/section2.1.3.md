# 实际影响

### 代码中二进制的使用

虽然二进制是每个计算机的基础语言，但是使用高级语言对计算机编程时不知道这些最初的东西是完全现实的。不管怎样，对于低级代码，我们的兴趣在于那一些反复使用的基本二进制原则。

### 掩码与旗标

#### 掩码

在低层代码中，保持结构和变量尽可能的空间高效是很重要的。在某些情况下，这需要高效地把两个（大致相关的）变量合并成一个。

记住每个二进制位代表着两种状态，所以如果我们知道一个变量只有，假设，16种可能的状态，那么它可以用4个二进制位来表示（即 $$2^4=16$$ 不同数值）。但是C语言中我们能够声明的最小类型是8个二进制位（一个Char类型），所以我们可以浪费4个比特，或者找到一种方法使用那些多余的二进制位。

我们可以通过***掩码***的过程轻松地做到这些。还记得逻辑运算的规则吗，数值是如何被提取的应该很清晰。

过程如下面的图标所说明。我们对低4位感兴趣，所以把我们的掩码在那些位置设为1。因为***逻辑与***运算将在比特位都为1时设置，所以那些掩码设为0的比特位有效地屏蔽了我们不感兴趣的比特位。

### 图2.1 掩码

![](http://ww1.sinaimg.cn/large/76731d17gy1fcdnc6wk1qj209x05bjr5)

要得到前4个比特（蓝色部分），我们需要倒置掩码。你会注意到这样给出的结果是0x90然而我们真正想要的值是0x09。要使这些比特在正确的位置，我们使用***右移***运算。

设置二进制位需要***逻辑或***运算。我们使用0作为掩码，而不使用1。你应该画一张和上面类似的图表，尝试使用逻辑或运算设置二进制位。

#### 旗标

通常一个程序会有大量的变量只有当旗标在某些状态时才存在。例如，状态机，它是一种算法，用来传输一些不同的状态，但是每次只能有一种状态。假设它有8个不同状态；我们可以简单地声明8个不同变量，每个状态一个变量。但许多情况下声明一个8比特的变量更好，然后为每个比特指派一个***旗标***表示一个特定的状态。

旗标是掩码的一种特殊情况，但是每个比特代表一个特定的布尔状态（开启或关闭）。一个n比特的变量可以存储n个不同的旗标。下面的示例展示了使用旗标的一种典型案例——你将会经常看到基于这个代码的各种变化。

### 例2.1 使用旗标

```c
  1 #include <stdio.h>
    
    
    /*
  5  *  define all 8 possible flags for an 8 bit variable
     *      name  hex     binary
     */
    #define FLAG1 0x01 /* 00000001 */
    #define FLAG2 0x02 /* 00000010 */
 10 #define FLAG3 0x04 /* 00000100 */
    #define FLAG4 0x08 /* 00001000 */
    /* ... and so on */
    #define FLAG8 0x80 /* 10000000 */
    
 15 int main(int argc, char *argv[])
    {
    	char flags = 0; /* an 8 bit variable */
    
    	/* set flags with a logical or */
 20 	flags = flags | FLAG1; /* set flag 1 */
    	flags = flags | FLAG3; /* set flag 3
    
    	/* check flags with a logical and.  If the flag is set (1)
    	 * then the logical and will return 1, causing the if
 25 	 * condition to be true. */
    	if (flags & FLAG1)
    		printf("FLAG1 set!\n");
    
    	/* this of course will be untrue. */
 30 	if (flags & FLAG8)
    		printf("FLAG8 set!\n");
    
    	/* check multiple flags by using a logical or
    	 * this will pass as FLAG1 is set */
 35 	if (flags & (FLAG1|FLAG4))
    		printf("FLAG1 or FLAG4 set!\n");
    
    	return 0;
    }
 40 
```

