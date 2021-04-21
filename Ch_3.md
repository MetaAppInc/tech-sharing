汇编是机器代码的文本表示，对于程序中的每一条指令。GCC调用汇编器和链接器，根据汇编代码生成可执行的机器代码。

使用高级语言编程相比于使用汇编更加高效，也更可靠。但学习机器代码有利于我们理解编译器的优化能力，分析代码中隐含的低效率。

# 程序编码

## 机器级代码

计算机使用了不同形式的抽象

- 指令集结构或指令集架构(ISA)来定义机器级程序的格式和行为
- 机器级程序使用的内存是虚拟地址，看上去是一个非常大的数组

程序内存包括：程序的可执行机器代码，操作系统需要的一些信息，用来管理过程调用和返回的运行时栈，以及用户分配的内存块。

一个机器指令只执行一个非常简单的操作。例如，将存放在寄存器中的数字相加，在存储器和寄存器之间传送数据，或是条件分支转移到新的指令地址。编译器必须产生这些指令的序列，从而实现程序结构。

机器程序执行的只是一个字节序列，机器对产生指令的源代码一无所知。

- x86-64的指令长度从1到15个字节不等
- 指令格式：从某个位置开始，将机器解码为唯一的指令
- GCC正向编译和反编译生成的GCC代码有细微的差别，很多指令结尾的`q` , 是大小指示符

## 汇编格式的注解

以 `.` 开头的是指导汇编器和链接器工作的伪指令, 我们通常可以忽略这些。

## 数据格式

Intel 用字表示16位数据类型。称32为双字，64为四字

![](https://cdn.jsdelivr.net/gh/Jerrywang959/mypicchuang@master/img/1618984481915-1618984481904.png)
# 访问信息

一个 x86 中央处理单元包括一组16个储存64位值的通用目的寄存器。都以 %r 开头

大多数的指令都有一个或多个操作数(operand)，指出执行一个操作中要使用的数据源数据，以及放置结果的目的位置。一般有三种

- 立即数。用来表示常数。
- 寄存器。寄存器的地位1,2,4,8字节中的一个作为操作数，分别对应8,16,32,64位。用 $r_a$ 表示任意寄存器 $a$，用引用 $R[r_a]$ 来表示它的数值。
- 内存引用。根据计算的地址访问某个有效位置。用$M_b[Addr]$表示对储存在内存从地址 $Addr$ 开始的$b$个字节的引用。为了简便，我们通常允许省去下标$b$。

![](https://cdn.jsdelivr.net/gh/Jerrywang959/mypicchuang@master/img/1618984504090-1618984504080.png)

## 数据传送的数据

mov 类，把数据从源位置复制到目的位置

movb, movw, movl, movq 主要区别在于操作的大小不同，分别是  1、2、4、8

ret 返回函数被调用点的指令

## 压入和弹出栈数据

pushq 将四字压入栈

popq 将四字弹出栈

# 算数和逻辑操作

除了 leaq 外，都有分大小的变种

![](https://cdn.jsdelivr.net/gh/Jerrywang959/mypicchuang@master/img/1618984520305-1618984520298.png)

## 加载有效地址

 `leaq` 是 `movq` 指令的变形，指令形式上是从内存读取数据到寄存器

## 一元和二元操作

第二组是一元操作，只有一个操作数。`incq (%rsp)` 使得栈顶的8字节元素加1

第三组是二元操作，第二个操作数既是源又是目的。`subq %rax,%rdx` 使寄存器 `%rdx`的值减去 `%rax` 中的值。

## 移位操作

先给出移位量，后给出要移位的数。

## 特殊操作运算符

![](https://cdn.jsdelivr.net/gh/Jerrywang959/mypicchuang@master/img/1618984534404-1618984534397.png)
# 控制

## 条件码

除了整数寄存器，CPU还维护着单个位的条件码寄存器，用来描述最近的算数或逻辑操作的属性。

CF：进位标志符号。最近的操作使得最高位产生进位，检查无符号操作的溢出。

ZF：零标志符号。最近的操作得出的结果为0。

SF：符号标志。最近的操作得到的结果为负数。

OF：溢出标志。最近的操作导致一个补码溢出—正溢出或负溢出。

## 访问条件码

条件码通常不会读取，常用的使用方式有三种。

- 根据条件码的某种组合，将一个字节设置为0或者1（这一类指令称为SET指令
- 条件转跳到程序的某个其他部分
- 有条件地传送数据

![](https://cdn.jsdelivr.net/gh/Jerrywang959/mypicchuang@master/img/1618984548360-1618984548352.png)

## 跳转指令

jmp 执行切换到程序中的一个全新的位置

## 条件控制来实现条件分支

cmpq 进行比较

jge, je 根据条件跳转