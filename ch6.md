RISC-V是一种模块化的指令集体系结构，允许设计各种微处理器。这种灵活性允许行业参与者为具有不同需求的应用程序设计微处理器，包括用于嵌入式设备的超低功耗和紧凑的微处理器，以及用于运行在数据中心上的强大服务器的高性能微处理器。

为了实现这种灵活性，RISC-V的ISA依赖于4种基本ISA和几种扩展，以实现ISA的专用版本。表6.1给出了基本的ISA及其一些扩展。

**基本指令集**

| 名称   | 描述                             |
| ------ | -------------------------------- |
| RV32I  | 32位整数指令集                   |
| RV32E  | 32位整数指令集（嵌入式微处理器） |
| RV64I  | 64位整数指令集                   |
| RV128I | 128位整数指令集                  |



**扩展指令集**

| 名称 | 描述                                                     |
| ---- | -------------------------------------------------------- |
| M    | 乘法标准扩展                                             |
| A    | 原子指令标准扩展                                         |
| F    | 单精度浮点数标准扩展                                     |
| D    | 双精度浮点数标准扩展                                     |
| G    | 基本指令集 + 以上扩展，即基本指令集 + MAFD               |
| Q    | 四精度（quad-precision）浮点数标准扩展                   |
| L    | 十进制浮点数标准扩展                                     |
| C    | 压缩的指令（compressed instructions）标准扩展            |
| B    | 位操作标准扩展                                           |
| J    | 动态转换语言（dynamically translated languages）标准扩展 |
| T    | 事务性内存标准扩展                                       |
| P    | 单指令多数据流标准扩展                                   |
| V    | 向量操作标准扩展                                         |
| N    | 用户级中断标准扩展                                       |
| H    | hypervisor标准扩展                                       |

> 表6.1

在本书中，我们将专注于RV32IM，它包括RV32I基本指令以及支持乘除法的M扩展指令。RV32IM具有以下属性:

* 支持32位地址空间
* 包含33个32位寄存器
* 使用二进制补码表示有符号整数
* 含有基本指令，包括整数计算指令，整数加载/存储指令，流程控制指令
* 包含对寄存器中的值进行乘法、除法指令



# 6.1 数据类型和内存组织

ISA原生数据类型是ISA可以天生就能自然处理的数据类型，表6.2展示了RV32I原生数据类型的大小

| RV32I原生数据类型名称 | 大小（字节） |
| --------------------- | ------------ |
| `byte`                | 1            |
| `unsigned   byte`     | 1            |
| `halfword`            | 2            |
| `unsigned   halfword` | 2            |
| `word`                | 4            |
| `unsigned   word`     | 4            |

与其他现代其它的ISA类似，RISC-V是字节可寻址存的，即每个内存位置存储一个字节并有一个唯一的地址，如图6.1所示

![](./imgs/ch6/6.1.png)

> 图6.1

大于1字节的数据会存储在多个内存位置上。因此，在内存中存`halfword`类型的数据时，会将2个字节存储在2个连续的内存位置上。存`word`类型的数据时，会将4个字节存储在4个连续的内存位置上。

在将C语言的代码转换为RV32I汇编代码时，必须将C中的数据类型转换为RISC-V原生数据类型。表6.3给出了C原生数据类型到RV32I原生数据类型的映射。C语言中的所有指针（例如`int*`、`char*`和`void*`）都表示内存地址，并映射到`unsigned word`数据类型。

| C语言数据类型     | RV32I原生数据类型   | 大小 |
| ----------------- | ------------------- | ---- |
| `bool`            | `byte`              | 1    |
| `char`            | `byte`              | 1    |
| `unsigned char`   | `unsigned byte`     | 1    |
| `short`           | `halfword`          | 2    |
| `unsigned  short` | `unsigned halfword` | 2    |
| `int`             | `word`              | 4    |
| `unsigned int`    | `unsigned word`     | 4    |
| `long`            | `word`              | 4    |
| `unsigned long`   | `unsigned word`     | 4    |
| `void*`           | `unsigned word`     | 4    |

> 表6.3

# 6.2 RV32I 寄存器

RV32I的非特权ISA包含33个32位寄存器，也称为非特权寄存器。

寄存器`x0`是一个硬连线到0（hard-wired to zero）的特殊寄存器，也就是说，读取它的值时总是返回值0。寄存器`pc`保存程序计数器，即下一条要执行的指令的地址。每次执行一条指令时，它的内容都会自动更新，而且可能会被称为 *流程控制（control-flow）* 的特殊指令更新。

其余的寄存器（`x1`~`x31`）是通用寄存器，可以互换使用。尽管如此，遵循寄存器使用标准通常是很重要的，这将在后面讨论。例如在调用函数时，总是使用同一组寄存器来传递参数。为便于编程，这些寄存器被赋予了别名，可以在编写汇编代码时使用。这样做的目的是让程序员在编程时使用更有意义的寄存器名。例如，写`a0`而不是`x10`来引用保存函数第一个参数的寄存器。表6.4给出了非特权寄存器的列表、别名及其描述。

> 译注：hard-wired这个单词的解释如下：
>
> *make (a function) a permanent feature in a  computer by means of permanently connected circuits, so that it cannot  be altered by software*

| 寄存器  | 别名                                                  | 描述                | 调用者保存 | 被调用者保存 |
| ------- | ----------------------------------------------------- | ------------------- | ---------- | ------------ |
| x0      | zero                                                  | 硬连线到0           |            |              |
| x1      | ra（**r**eturn **a**ddress）                          | 保存返回地址        | ✔️          |              |
| x2      | sp（**s**tack **p**ointer）                           | 栈指针              |            | ✔️            |
| x3      | gp（**g**lobal **p**ointer）                          | 全局指针            |            |              |
| x4      | tp（**t**hread **p**ointer）                          | 线程指针            |            |              |
| x5-x7   | t0-t2（**t**emporary register **0-2**）               | 临时寄存器0-2       | ✔️          |              |
| x8      | s0/fp（**s**aved register 0 / **f**rame **p**ointer） | 保存寄存器0或帧指针 |            | ✔️            |
| x9      | s1（**s**aved register 1 ）                           | 保存寄存器1         |            | ✔️            |
| x10-x17 | a0-a7（**f**unction arguments 0 to 7）                | 函数参数0-7         | ✔️          |              |
| x18-x27 | s2-s11（**s**aved register **2-11** ）                | 保存寄存器2-11      |            | ✔️            |
| x28-x31 | t3-t6（**t**emporary register **3-6**）               | 临时寄存器3-6       | ✔️          |              |
| pc      | pc（**p**rogram **c**ounter）                         | 程序计数器          |            |              |

> 表6.4 RV32I非特权寄存器

# 6.3 Load/Store 架构

**load/store体系结构**是一种指令集体系结构，它要求在操作值之前，必须显式地从内存中读取值，或将值写入内存。换句话说，要从内存中读/写一个值，软件必须执行一个load/store指令。

> 译注：根据《计算机体系结构》，指令系统可以分为：
>
> * 堆栈型
>
> * 累加型
>
> * register型
>
>   * register-register型（即load/store）
>   * register-memory型
>
>   

RISC-V是一种load/store体系结构，因此，为了对存储在内存中的数据执行操作（例如算术操作），它要求先执行load指令，将数据从内存加载到寄存器。让我们考虑以下汇编代码，它从内存中加载一个值，将其乘以2，并将结果回存到内存中

```assembly
lw a5, 0(a0)
add a6, a5, a5
sw a6, 0(a0)
```

第一个指令叫load word，用助记符`lw`表示，是一个加载指令。它从内存中检索一个`word`的值，并将其存储在寄存器`a5`中。表达式`0(a0)`表示被加载的值的内存地址。在本例中，地址的值是寄存器`a0`的值和常数0的和。换句话说，在执行该加载指令时，如果寄存器`a0`包含值8000，硬件将从地址8000加载数据。

第二个指令由助记符`add`表示，它将两个值相加并将结果存在寄存器中。在本例中，它将来自寄存器`a5`的值相加，并将结果存储在寄存器`a6`上。注意，由于两个源操作数相同，即`a5`，结果相当于将`a5`的值乘以2。

最后，第三条指令称为store word，由助记符`sw`表示，将寄存器`a6`中的值回存到内存中。同样，表达式`0(a0)`表示需要写入数据的内存地址。



# 6.4 伪指令

在对汇编程序进行汇编时，汇编器将每条汇编指令（纯文本格式）转换为相应的机器指令（二进制格式）。例如，汇编指令`add x10, x11, x12`被转换为一条4个字节的机器指令，它的编码是0x00c58533。

伪指令是一种汇编指令，它在ISA中没有对应的机器指令，但可以被汇编器自动翻译成一条或多条替代的机器指令，以达到相同的效果。例如，`no operation`指令或`nop`就是一个RV32I伪指令，它被汇编器转换为`addi x0, x0, 0`指令。另一个例子是`mv`指令，它将一个寄存器的内容复制到另一个寄存器中。例如伪指令`mv a5, a7`的作用是将`a7`的内容复制到`a5`中，这条伪指令会被转换为指令`addi a5, a7, 0`，将`a7`中的值加0，并将结果存储在寄存器`a5`中。

由于本书的重点是汇编编程，所以其余部分不会区分伪指令和真正的RV32I机器指令。请读者参考[RISC-V指令集手册](https://riscv.org/technical/specifications/)，以获得真实的RV32I机器指令和伪指令的完整列表。



# 6.5 逻辑、位移和算术指令

逻辑、移位和算术指令是在数据上执行逻辑、移位和算术操作的指令。



## 6.5.1 指令的语法和操作数

在RV32I中，所有的逻辑、移位和算术指令所操作的数据，都由指令中的操作数表示。这些指令包含三个操作数，一个目标操作数和两个源操作数。

* 第一个操作数表示目标寄存器（`rd`），即存储操作结果的寄存器。
* 第二个操作数表示一个寄存器（`rs1`），它包含第一个源操作数。
* 第三个操作数表示第二个源操作数，它可以是另一个寄存器（`rs2`）或一个立即数 （`imm`）。

因此，逻辑、移位和算术指令的语法可以是以下两种之一:

```assembly
MNM rd, rs1, rs2

MNM rd, rs1, imm
```

其中`MNM`是指令的助记符，`rd`表示目标寄存器，`rs1`表示第一个源操作数，`rs2`（或`imm`）表示第二个操作数。下面的汇编代码展示了RV32I汇编指令的逻辑、移位和算术的例子:

```assembly
and a0, a2, a6 # a0 <= a2 & a6
slli a1, a3, 2 # a1 <= a3 << 2
sub a4, a5, a6 # a4 <= a5 - a6
```

* 第一条指令使用`a2`和`a6`中的值执行按位“与”操作，并将结果存储在`a0`上。

* 第二条指令将值从`a3`左移两位，并将结果存储在`a1`上。在本例中，第二个源操作数（即2）是立即数(`imm`)。

* 最后，第三条指令用`a5`的值减去`a6`的值，并将结果存储在寄存器`a4`上。

任何通用寄存器（`x0`-`x31`）都可以用作`rd`、`rs1`或`rs2`。然而，值得注意的是，如果将`x0`指定为目标操作数（`rd`），则结果将被丢弃。这是因为`x0`硬连线到0（值恒为零）。





## 6.5.2 处理值较大的立即数

立即数是一个编码到指令本身中的常量。除了这个值，指令还必须编码其他信息，如操作码和其他操作数。因为所有的RV32I指令都是32位的，所以指令中可用来编码立即数的bit的长度小于32位。事实上，RV32I的算术、逻辑和移位指令，只能对可以表示为12位的二进制补码的立即数进行编码。换句话说，**这些指令的立即数必须大于或等于 $-2048(−2^{11})$ ，小于或等于$ 2047(2^{11}−1)$**。

例如以下汇编代码都是非法的：

```assembly
add a0, a5, 2048
add a0, a5, 10000
add a0, a5, -3000
```

因为汇编器无法将立即数编码到指令中（即不能编码为12位的二进制补码）。此时汇编器将无法汇编代码，并可能显示错误消息。当试图用GNU汇编器（**as**）汇编这些代码的话，会显示以下信息：

```bash
prog.s: Assembler messages:
prog.s:1: Error: illegal operands ‘add a0,a5,2048’
prog.s:2: Error: illegal operands ‘add a0,a5,10000’
prog.s:3: Error: illegal operands ‘add a0,a5,-3000’
```

指令中要使用小于-2048或大于2047的立即数，程序员可以使用多条指令来合成一个值，将其存储到寄存器中，然后使用一条指令从寄存器中读取它。有几种方法可以使用RV32I指令组合这些值。例如，我们可以将一个小常数（例如1000）装入寄存器，将其值左移以乘以2的幂，然后再加上另一个小常数以得到所需的值。下面的汇编代码通过将值1000加载到`a5`中，将其向左移动两位，并将其加5来生成值4005。

```assembly
ADD a5, x0, 1000
SLLI a5, a5, 2
ADD a5, a5, 5
```

在RISC-V中，将立即数加载到寄存器的推荐方法是使用 *load immediate* 伪指令(`li`)。该伪指令被汇编器自动转换为最佳的机器指令序列，以生成所需的值。*load immediate* 指令的语法是：

```assembly
li rd, imm
```

这里`rd`表示目标寄存器，`imm`表示立即数的值

> 注：这里非常重要，RISCV中加载一个32位的值到寄存器`r`通常会分为2步
>
> 1. 将立即数的**高20位加载到**`r`
> 2. 在1的基础上，将立即数的**低12位累加到**`r`
>
> 例如伪指令：
>
> `	li x3, 0x87654321`
>
> 会被翻译为：
> ```assembly
> 0x80000000 <+0>:     lui     gp,0x87654
> 0x80000004 <+4>:     addi    gp,gp,801 # 0x87654321
> ```
>
> * 0x87654321的高20位是0x87654，通过`lui`加载到`gp`寄存器（即`x3`寄存器）
> * 0x87654321的低12位是801，即16进制的0x321，通过`addi`累加到`gp`
> * 因此`x3`最终的值为 (0x87654<< 12) + 0x321= 0x87654321



> 需要注意的是，伪指令`li x3, 0x87654321`在用gdb debug的时候，需要用`disassemble/m`命令才能展示出其背后真正实现的指令，用`list`命令只会展示源码中的汇编，例如：
>
> ```bash
> 0x00001000 in ?? ()
> (gdb) list
> 1       .global _start
> 2       _start:
> 3               li x3, 0x87654321
> 4               j _start
> 5
> (gdb) disassemble/m _start
> Dump of assembler code for function _start:
> 3               li x3, 0x87654321
>    0x80000000 <+0>:     lui     gp,0x87654
>    0x80000004 <+4>:     addi    gp,gp,801 # 0x87654321
> 
> 4               j _start
>    0x80000008 <+8>:     j       0x80000000 <_start>
> 
> End of assembler dump.
> (gdb) 
> ```
>
> 

## 6.5.3 逻辑指令

表6.5给出了RV32I逻辑指令。`and`/`or`/`xor`指令对存储在寄存器`rs1`和`rs2`中的值执行按位进行"与"、"或"、"异或"操作，并将结果存储在寄存器`rd`中。而`andi`/`ori`/`xori`指令则使用存储在寄存器`rs1`中的值和一个立即数执行操作。

| 指令                  | 描述                                                         |
| --------------------- | ------------------------------------------------------------ |
| `and rd, rs1, rs2  `  | 对寄存器`rs1`和`rs2`中的值按位执行"与"操作，结果存在寄存器`rd`中 |
| `or rd, rs1, rs2  `   | 对寄存器`rs1`和`rs2`中的值按位执行"或"操作，结果存在寄存器`rd`中 |
| `xor rd, rs1, rs2  `  | 对寄存器`rs1`和`rs2`中的值按位执行"亦或"操作，结果存在寄存器`rd`中 |
| `andi rd, rs1, imm  ` | 对寄存器`rs1`和立即数`imm`按位执行"与"操作，结果存在寄存器`rd`中 |
| `ori rd, rs1, imm  `  | 对寄存器`rs1`和立即数`imm`按位执行"或"操作，结果存在寄存器`rd`中 |
| `xori rd, rs1, imm  ` | 对寄存器`rs1`和立即数`imm`按位执行"亦或"操作，结果存在寄存器`rd`中 |

> 表6.5

下面的汇编代码展示了有效的逻辑指令

```assembly
and a0, a2, s2 # a0 <= a2 & s2
or a1, a3, s2 # a1 <= a3 | s2
xor a2, a2, a1 # a2 <= a2 ^ a1
andi a0, a2, 3 # a0 <= a2 & 3
ori a1, a3, 4 # a1 <= a3 | 4
xori a2, a2, 1 # a2 <= a2 ^ 1
```

下面的代码将两个立即数加载到寄存器`a1`和`a2`，并执行一个“与”操作。这个操作的结果（0x0000AB00）存储在寄存器`a0`中

```assembly
li a1, 0xFE01AB23 # a1 <= 0xFE01AB23
li a2, 0x0000FF00 # a2 <= 0x0000FF00
and a0, a1, a2 # a0 <= a1 & a2
```



## 6.5.4 移位指令

移位指令用于将二进制值向左或向右移位。这些指令可用于将bits转换为words（译注：即通过移位将bit扩展为word，或者反过来操作），或执行算术乘法和除法运算。表6.6给出了RV32I的移位指令。

| 指令                  | 描述                                                         |
| --------------------- | ------------------------------------------------------------ |
| `sll rd, rs1, rs2  `  | 对寄存器`rs1`中的值进行逻辑左移，左移的位数存在`rs2`中，结果存在寄存器`rd`中 |
| `srl rd, rs1, rs2  `  | 对寄存器`rs1`中的值进行逻辑右移，右移的位数存在`rs2`中，结果存在寄存器`rd`中 |
| `sra rd, rs1, rs2  `  | 对寄存器`rs1`中的值进行算术右移，右移的位数存在`rs2`中，结果存在寄存器`rd`中 |
| `slli rd, rs1, imm  ` | 对寄存器`rs1`中的值进行逻辑左移`imm`位，结果存在寄存器`rd`中 |
| `srli rd, rs1, imm  ` | 对寄存器`rs1`中的值进行逻辑右移`imm`位，结果存在寄存器`rd`中 |
| `srai rd, rs1, imm  ` | 对寄存器`rs1`中的值进行算术右移`imm`位，结果存在寄存器`rd`中 |

> 表6.6

逻辑左移指令（`sll`或`slli`）对存储在寄存器中的值执行逻辑左移。移动的位数是指令的一个操作数，可以是寄存器中的值，也可以是立即数。下面的代码展示了逻辑左移指令的示例。第一个移位指令（`slli`）将`a2`中的值左移两位，结果存储在`a0`中。第二个（`sll`）执行类似的操作，但左移的位数由`a3`的值决定。

```assembly
li a2, 24      # a2 <= 24
slli a0, a2, 2 # a0 <= a2 << 2
sll a1, a2, a3 # a0 <= a2 << a3
```

上述代码中的前两条指令将立即数24加载到寄存器`a2`中，左移两位，并将结果存储在`a0`上。立即数24用二进制表示是：

```
00000000 00000000 00000000 00011000
```

逻辑左移操作将位向左移动，舍弃最左边的位，并在右边补0。因此，向左移动两位后，结果是二进制数：

```
00000000 00000000 00000000 01100000
```

这个二进制数对应于十进制值96，相当于24 × 4。实际上，**逻辑左移操作可以用来将数字乘以2的幂**。在这种情况下，将一个值向左平移 $N$ 次等价于将该值乘以  $2^N$ 。下面的汇编代码展示了使用逻辑左移指令分别将寄存器`a3`中的值乘以2、4和8

```assembly
slli a0, a3, 1 # a0 <= a2 * 2
slli a1, a3, 2 # a0 <= a2 * 4
slli a2, a3, 3 # a0 <= a2 * 8
```

移位操作比乘法操作更容易在硬件中实现，执行时间和更短/或耗能更少。因此，只要有可能，编译器就会尝试生成这些指令来执行乘法。

逻辑右移指令（`srl`或`srli`）对存储在寄存器中的值执行逻辑右移。与逻辑左移指令类似，移位的数量被指令的操作数表示，可以是寄存器中的值或立即数。下面的代码展示了逻辑右移指令的示例。第1条移位指令（`srli`）将`a5`中的值向右移动两位，并将结果存储在`a0`中。第2条（`srl`）执行类似的操作，但向右移动的位数存在`a7`中。

```assembly
li   a5, 24     # a5 <= 24
srli a0, a5, 2 	# a0 <= a5 >> 2
srl  a1, a5, a7 # a0 <= a5 >> a7
```

上述代码中的前两条指令将立即数24加载到寄存器`a5`中，然后右移两位，并将结果存储在`a0`上。立即数24用二进制数表示是：

```
00000000 00000000 00000000 00011000
```

逻辑右移操作将所有位向右移动，舍弃最右边的位并在左边补0。因此，向右平移两位后，结果为：

```
00000000 00000000 00000000 00000110
```

这个二进制数对应于十进制数6，相当于24/4。事实上，逻辑右移运算相当于整数除以2的幂。

在上面的例子中，我们对值24右移两位得到值6，即24/4。但这对于负数是无效的。以−24为例。该值在RISC-V中由以下二进制数表示

```
11111111 11111111 11111111 11101000
```

逻辑右移操作将所有位向右移动，舍弃最右边的位并向左补0。因此，向右平移两位后，结果为：

```
00111111 11111111 11111111 11111010
```

注意，这个数不是负数，也不等于24除以4。实际上这是一个非常大的正数（1073741818）。

如果将前面的二进制数当作无符号数的话，它的值4294967272而不是- 24。此时逻辑右移两位的结果是这个数被4除，即1073741818。

总而言之，逻辑右移操作只能用于无符号数的除法。此时使用逻辑右移操作将无符号数右移 $N$ 位，等价于将无符号数除以 $2^N$。

**算术右移**指令（`sra`或`srai`）对存储在寄存器中的值执行算术右移。与逻辑右移类似，向右移动的位数是指令上的一个操作数，可以是寄存器中的值，也可以是立即数。下面的代码展示了算术右移指令的示例。第一条算术右移指令（`srai`）将`a5`中的值向右移动两位，并将结果存储在`a0`中。第二个（`sra`）执行类似的操作，但向右移动的位数的值存在`a7`中。

```assembly
li a5, -24     # a5 <= -24
srai a0, a5, 2 # a0 <= a5 >> 2
sra a1, a5, a7 # a0 <= a5 >> a7
```

上述代码中的前两条指令将立即数−24加载到寄存器`a5`中，向右移动两位，并将结果存储在`a0`上。如前所述，立即数- 24的二进制表示为：

```
11111111 11111111 11111111 11101000
```

算术右移操作将所有的bit向右移动，并丢弃最右边的bit。对于最左边（多出来）的位，它不是简单地补0，而是用（原来的）最左边的bit填充，也就是说，如果最左边的bit等于1，则在左边插入1。如果最左边的位等于0，则在左边插入0。因此，在前一个上执行两次右移运算后，结果将是二进制数：

```
11111111 11111111 11111111 11111010
```

这个二进制数对应于十进制−6，相当于24/4。事实上，算术右移运算可以用于**有符号整数**除以2的幂。注意，即这条指令也可以用于**正整数**除以2的幂的整数。这是因为正数最左边的位是零。因此，算术右移操作会在移位时在最左边补0。

总之，算术右移运算只能用于有符号数的整数除法。在本例中，使用算术右移操作将有符号数向右平移 $N$ 位，等价于有符号数除以 $2^N$

> 译注：有符号数算术右移的时候，分两种情况讨论：
>
> * 正数：则最左边的符号位是0，则右移时需要不断往左边填充0，此时并不会改变数字的算术性质（即保持正负，但值被除以2）
> * 负数：则最左边的符号位是1，则右移时需要不断往左边填充1，此时也不会改变数字的算术性质，读者可以尝试证明这一点。

## 6.5.5 算术运算指令

表6.7 展示了RV32I和M扩展中的算术指令

| 指令                    | 描述                                                         |
| ----------------------- | ------------------------------------------------------------ |
| `add rd, rs1, rs2  `    | 将寄存器`rs1`和`rs2`中的值相加，结果存在寄存器`rd`中         |
| `sub rd, rs1, rs2  `    | 将寄存器`rs1`的值减去寄存器`rs2`的值，结果存在寄存器`rd`中   |
| `addi rd, rs1, imm  `   | 将寄存器`rs1`的值和立即数`imm`相加，结果存在寄存器`rd`中     |
| `mul rd, rs1, rs2  `    | 将寄存器`rs1`和`rs2`中的值相乘，结果存在寄存器`rd`中         |
| `div{u} rd, rs1, rs2  ` | 寄存器`rs1`的值除以`rs2`的值，结果存在寄存器`rd`中。指令中`u`后缀是可选的，用于表明`rs1`和`rs2`中的是无符号数 |
| `rem{u} rd, rs1, rs2  ` | 计算`rs1`除以`rs2`后的余数，结果存在`rd`中。指令中`u`后缀是可选的，用于表明`rs1`和`rs2`中的是无符号数 |

> 表6.7

加法指令（`add`和`addi`）将两个数字相加，并将结果存储在寄存器（`rd`）中。在这条指令中，第一个数字都从寄存器`rs1`中获取。对于第二个数字，`add`指令从寄存器`rs2`中获取，而`addi`指令则使用立即数`imm`。

减法指令（`sub`）将`rs2`中的值与`rs1`中的值相减，并将结果存储在`rd`中。RV32I`subi`指令，即从一个寄存器的内容中减去一个立即数并将结果存储在另一个寄存器中的指令。但值得注意的是，程序员可以很容易地通过使用`addi`指令添加一个负的立即数来实现这种效果。下面的代码是一个指令示例，它从寄存器`a2`的内容中减去直接值`10`，然后使用`addi`指令将结果存储在`a0`上。

```assembly
addi a0, a2, -10 # a0 <= a2 - 10
```

乘法指令（`mul`）将`rs1`和`rs2`中的值相乘，并将结果存储在`rd`中。

除法指令（`div`和`divu`）将`rs1`中的值除以`rs2`中的值，并将结果存储在`rd`中。指令`div`除有符号数，而`divu`除无符号数

余数指令（`rem`和`remu`）计算`rs1`中的值除以`rs2`中的值的余数，并将结果存储在`rd`中。`rem`指令计算有符号数除法的余数，而`remu`计算无符号数除法的余数。

下面的汇编代码展示了RV32IM算术指令的例子

```assembly
add a0, a2, t2 # a0 <= a2 + t2
addi a0, a2, 10 # a0 <= a2 + 10
sub a1, t3, a0 # a1 <= t3 - a0
mul a0, a1, a2 # a0 <= a1 * a2
div a1, a3, a5 # a1 <= a3 / a5
rem a1, a3, a5 # a1 <= a3 % a5
remu a1, a3, a5 # a1 <= a3 % a5
```

你知道吗？

如果你正在为缺少M扩展的RV32I编程，即它不包含乘法和除法指令，则可以将算术和移位指令结合起来执行乘法和除法。下列汇编代码展示了如何使用`slli`和`addi`指令将`a2`的值分别乘以5和10:

```assembly
slli a0, a2, 2 # a0 <= a2 * 4
add a0, a0, a2 # a0 <= a0 + a2, i.e., a2 * 5
slli a1, a0, 1 # a1 <= a0 * 2, i.e., a2 * 10
```



# 6.6 数据移动指令

RV32I的数据移动指令可用于：

* 将数据从内存加载到寄存器中
* 将寄存器数据存储到内存中
* 将数据从一个寄存器复制到另一个寄存器，或
* 将立即数或标签的地址值加载到寄存器中

表6.8给出了RV32I数据移动指令，表6.9给出了RV32I数据移动**伪指令**。

| 指令                 | 描述                                                         |
| -------------------- | ------------------------------------------------------------ |
| `lw rd, imm(rs1)  `  | 从内存中加载一个32位有符号或无符号word到寄存器`rd`。内存地址是通过将立即数`imm`与`rs1`中的值相加来计算的 |
| `lh rd, imm(rs1)`    | 从内存中加载一个16位**有符号halfword**到寄存器`rd`。内存地址是通过将立即数`imm`与`rs1`中的值相加来计算的 |
| `lhu rd, imm(rs1)  ` | 从内存中加载一个16位**无符号halfword**到寄存器`rd`。内存地址是通过将立即数`imm`与`rs1`中的值相加来计算的 |
| `lb rd, imm(rs1)  `  | 从内存中加载一个8位**有符号byte**到寄存器`rd`。内存地址是通过将立即数`imm`与`rs1`中的值相加来计算的 |
| `lbu rd, imm(rs1)  ` | 从内存中加载一个8位**无符号byte**到寄存器`rd`。内存地址是通过将立即数`imm`与`rs1`中的值相加来计算的 |
| `sw rs1, imm(rs2)  ` | 将寄存器`rs1`中32位的值写入内存中。内存地址是通过将立即数`imm`与`rs2`中的值相加来计算的 |
| `sh rs1, imm(rs2)  ` | 将寄存器`rs1`中低16位的值写入内存中。内存地址是通过将立即数`imm`与`rs2`中的值相加来计算的 |
| `sb rs1, imm(rs2)  ` | 将寄存器`rs1`中低8位的值写入内存中。内存地址是通过将立即数`imm`与`rs2`中的值相加来计算的 |

> 表6.8



| 指令                     | 描述                                                         |
| ------------------------ | ------------------------------------------------------------ |
| `mv rd, rs`              | 将寄存器`rs`中的值复制寄存器`rd`                             |
| `li rd, imm`             | 将立即数`imm`加载到寄存器`rd`                                |
| `la rd, rot`             | 将标签`rot`的地址加载到寄存器`rd`                            |
| `L{W|H|HU|B|BU} rd, lab` | `lw`、`lh`、`lhu`、`lb`和`lbu`指令对应的伪指令，伪指令执行和机器指令一样的操作，但内存地址是根据标签`lab`计算的（**即此时传入的时候标签的名称**）。 |
| `S{W|H|B} rd, lab`       | `sw`、`sh`和`sb`指令对应的伪指令，伪指令执行和机器指令一样的操作，但内存地址是根据标签`lab`计算的 |

> 表6.9

## 6.6.1 load类指令

所有RV32I的load类指令（`lw`、`lh`、`lhu`、`lb`和`lbu`）将值从内存加载到寄存器。这些指令的汇编语法如下：

```assembly
MNM rd, imm(rs1)
```

`MNM`是指令助记符。第一个操作数（`rd`）表示目标寄存器，即内存中的值最终被加载到的地方。第二个（`imm`）和第三个（`rs1`）操作数分别表示一个立即数和一个寄存器。将这两个操作数的值相加，得到内存地址（即把这个内存中这个地址的值加载寄存器`rd`中）。

### **load word**（`lw`）

**load word**指令（`lw`）将一个32位的word从内存加载到寄存器中。由于word类型占4个字节，所以该指令从4个连续的内存地址中加载4个字节，并将这4个字节存储到目标寄存器中。RV32I使用的是小端格式，内存中位于最低地址的值会加载到寄存器的 *最低有效字节*（least significant byte）中。图6.2展示了把一个值从内存加载到寄存器`a0`的过程。在这个例子中，数据（一个4字节的word）从内存中地址为8000的地方开始存储，占据4个连续的内存位置，然后被读取。内存的起始地址是通过将立即数0与寄存器`a2`（ $8000_{10}$ ）中的值相加来计算的。

> 译注：和下面的指令不一样，该指令不分有符号和无符号，因为word的长度和寄存器的长度一致，无需考虑寄存器中空余字节的填充问题。

![](./imgs/ch6/6.2.jpg)

> 图6.2

**load word**指令用于从内存中加载（C语言中的）这些类型的数据：`int`、`unsigned int`、`long`、`unsigned long`和指针。



### **load unsigned byte**（ `lbu`）

**load unsigned byte**指令（ `lbu`）将一个8位unsigned byte从内存加载到寄存器中。寄存器有32位（4字节），从内存加载的unsigned byte存储在寄存器的*最低有效字节*上，寄存器上的其他3个字节设置为0。图6.3展示了把一个unsigned byte从内存加载到寄存器`a0`的过程。在这个例子中，数据（unsigned byte）存储在内存地址值为8000的地方，该地址是通过将立即数0与寄存器`a2`中的值（ $8000_{10}$ ）相加而来的。





![](./imgs/ch6/6.3.jpg)

> 图6.3

**load unsigned  byte**指令用于从内存中加载（C语言中的）`unsigned char`类型的数据。



### **load byte**（`lb`）

**load byte**指令（`lb`）将一个8位signed byte从内存加载到寄存器中。同样，寄存器是32位的，从内存中加载的signed byte存储在 *最低有效字节*上。如果该值非负，则寄存器其他3个byte都设为0。如果是负数，则其他3个字节的所有位都设为1。图6.4说明了把一个非负的signed byte（0x08 = $8_{10}$）从内存加载到寄存器`a0`的过程。在这个例子中，数据是一个非负的unsigned byte，存储在内存地址值为8000的地方，该地址是通过将立即数0与寄存器`a2`中的值（ $8000_{10}$ ）相加而来的。注意，寄存器高位的byte被设为0。

![](./imgs/ch6/6.4.jpg)

>  图6.4

图6.5说明了把一个负的signed byte（0xFE = $−2_{10}$）从内存加载到寄存器`a0`的过程。同样，数据存储在内存地址值为8000的地方，该地址是通过将立即数0与寄存器`a2`中的值（ $8000_{10}$ ）相加而来的。但请注意，寄存器高位的3个byte的所有bit都被设为1，因此最终值是为0xFFFFFFFE，即 $−2_{10}$。

![](./imgs/ch6/6.5.jpg)

> 图6.5

**load byte**指令用于从内存中加载（C语言中的）`char`类型的数据。

---



### load unsigned halfword（`lhu`）

**load unsigned halfword**（`lhu`）指令, 将一个16位的unsigned halfword从内存加载到寄存器中。unsigned halfword这种类型占据两个字节，所以该指令从两个连续的内存位置加载两个byte，并将这两个byte存储到目标寄存器中。同样，由于RV32I遵循小端格式，位于内存中最低地址的byte被加载到寄存器的*最低有效字节*中，而内存中的第二个byte则被加载到寄存器的*第二低有效字节* （second-least significant byte）中。寄存器剩下两个byte被设为0。图6.6说明了把一个unsigned halfword值（0x0108）从内存加载到寄存器`a0`的过程。在这个例子中，数据是一个unsigned halfword，存储在内存地址值为8000的地方，该地址是通过将立即数0与寄存器`a2`中的值（ $8000_{10}$ ）相加而来的。注意，寄存器高位的两个byte都被设置为0。

![](./imgs/ch6/6.6.jpg)

>  图6.6

**load unsigned halfword**指令用于从内存中加载（C语言中的）`unsigned short`类型的数据。



### load halfword（`lh`）

**load halfword**（`lh`）指令将一个16位的signed halfword从内存加载到寄存器中。由于halfword这种类型占据两个字节，所以该指令从两个连续的内存位置加载两个byte，并将这两个byte存储到目标寄存器中。同样，由于RV32I遵循小端格式，位于内存中最低地址的byte被加载到寄存器的*最低有效字节*中。如果值非负，则寄存器剩余两个字节的所有bit设置为0，如果值为负，则所有bit设置为1。图6.7展示了把一个非负的halfword值（0x0102 = $258_{10}$）从内存加载到寄存器`a0`的过程。在这个例子中，数据是一个非负的halfword，存储在内存地址值为8000的地方，该地址是通过将立即数0与寄存器`a2`中的值（ $8000_{10}$ ）相加而来的。注意，寄存器高位的两个byte都被设置为0，最终结果是0x00000102，即 $258_{10}$。

![](./imgs/ch6/6.7.jpg)

>  图6.7

图6.8说明了把一个负的halfword（0xFFFE = $−2_{10}$）从内存加载到寄存器`a0`的过程。同样，数据存储在内存地址值为8000的地方，该地址是通过将立即数0与寄存器`a2`中的值（ $8000_{10}$ ）相加而来的。但请注意，寄存器高位的两个byte的全部bit都被设置1，最终值正确设置为0xFFFFFFFE，即 $−2_{10}$。

![](./imgs/ch6/6.8.jpg)

>  图6.8

**load halfword**指令用于从内存中加载（C语言中的）`short`类型的数据。

## 6.6.2 store类指令

所有RV32I存储指令（`sw`、`sh`和`sb`）都将寄存器中的值存储到内存中。这些指令的汇编语法如下：

```assembly
MNM rs1, imm(rs2)
```

其中`MNM`是指令助记符。第一个操作数（`rs1`）表示源寄存器，它包含了要存到内存中的值。第二个（`imm`）和第三个（`rs2`）操作数分别表示一个立即数和一个寄存器。将这两个操作数的值相加，得到内存地址。

**store word  **（`sw`）指令将寄存器`rs1`中的一个32位word存储到内存中。由于word类型有4个字节，该指令将这4个字节存储到4个连续的内存位置中。RV32I遵循小端序格式，因此最低有效字节存储在内存的最低地址上，以此类推。图6.9说明了通过`sw`指令将`a0`中的值存储到内存中的过程。在这个例子中，数据（一个4字节的word）从地址8000开始，存储在4个连续的内存位置上。起始地址是通过将立即数0与寄存器`a2`（ $8000_{10}$ ）的值相加而来的。

![](./imgs/ch6/6.9.jpg)

>  图6.9

**store word **指令用于把（C语言中的）`int`, `unsigned int`, `long`, `unsigned long` 类型的数据写到内存中。

---

**store half word **指令（`sh`）将寄存器`rs1`中低16 bit的half word存储到内存。由于half word类型有两个字节，该指令将寄存器`rs1`的低位的两个字节存储到2个连续的内存位置中续。RV32I遵循小端序格式，因此最低有效字节存储在内存的最低地址上，以此类推。图6.10展示了通过`sh`指令将`a0`的值存储到内存中的过程。在这个例子中，数据（一个2字节的half word）从地址8000开始，存储在2个连续的内存位置上。起始地址是通过将立即数0与寄存器`a2`（ $8000_{10}$ ）的值相加而来的。

![](./imgs/ch6/6.10.jpg)

>  图6.10

**store half word **指令用于把（C语言中的）`short`, `unsigned short`类型的数据写到内存中。

---

**store byte**指令（`sb`）将寄存器`rs1`中低8 bit的字节存储到内存。图6.11展示了通过`sb`指令将`a0`的值存储到内存中的过程。在这个例子中，数据（一个字节）存储在地址为8000内存位置上。该地址是通过将立即数0与寄存器`a2`（ $8000_{10}$ ）的值相加而来的。

![](./imgs/ch6/6.11.jpg)

>  图6.11

**store byte**指令用于把（C语言中的）`char`, `unsigned char`类型的数据写到内存中。

## 6.6.3 数据移动类伪指令

**copy register**指令（`mv`）是一个伪指令，它将值从一个寄存器复制到另一个寄存器。该指令的汇编语法是：

```assembly
mv rd, rs
```

其中`rd`的目标寄存器，`rs`是源寄存器。

> 译注：`mv`指令会翻译为`addi rd, rs, 0`来实现

---

**load immediate**指令（`li`）是一个伪指令，它将一个立即数加载到寄存器中。在6.5.2节讨论过，根据立即数的不同，汇编器可以将该伪指令转换为一条或多条机器指令。该指令的汇编语法是：

```assembly
li rd, imm
```

其中`rd`的目标寄存器，`imm`是需要被加载的立即数。

---

**load address**指令（`la`）是一个伪指令，它将一个**由标签表示的32位地址（注意是地址）**加载到寄存器中。该指令的汇编语法是

```assembly
la rd, symbol
```

其中`rd`的目标寄存器，`symbol`是标签。

> 译注：如果标签在编译期就能确定（是一个常量），那么此时`la`可以被翻译为（假设`rd`是`x5`）：
>
> ```assembly
> auipc x5, 0       #注意是auipc，跟pc有关
> addi  x5, x5, 0
> ```

---

**load global**指令是一组伪指令，便于从由标签指向的内存位置加载值。这些指令的汇编语法是：

```assembly
l{w|h|hu|b|bu} rd, symbol
```

其中`rd`的目标寄存器，`symbol`是标签。以下面伪指令为例：

```assembly
lh a0, var_x
```

它从标签`var_x`指向的内存位置加载一个half word到寄存器`a0`中。由于标签表示的32位地址可能无法被编码到令中，因此汇编器可能会生成多个RV32I机器指令来执行加载操作。此时它首先生成指令将标签的地址加载到`rd`中。然后，生成另外一条指令将内存中的值加载到`rd`中。

---

**store global**指令是一组伪指令，用于将值存储到由标签指向的内存位置中。这些指令的汇编语法是：

```assembly
s{w|h|b} rs, symbol, rt
```

其中`rs`表示源寄存器，`symbol`表示标签，`rt`表示用于地址计算的临时寄存器。例如：

```assembly
sw a0, var_x, a5
```

该指令将`a0`中的一个word加载到标签`var_x`指向的内存中。与load global指令类似，标签的地址可能无法编码到指令中，因此汇编器可能生成多个RV32I机器指令，以便将标签指向的地址加载到寄存器中。此时，汇编程序不能使用`rs`（即`var_x`）作为临时寄存器，因为这样会改变`rs`中的内容。为此，用户需要显式指定一个通用寄存器，以便汇编器在为伪指令生成代码时将其用作临时寄存器。

# 6.7 流程控制指令

在RISC-V和大多数通用处理器中，**正常执行流是按照指令在内存上的组织顺序执行指令**。换句话说，一旦一条指令被执行，处理器就会获取位于下一个内存位置的指令来执行。RISC-V指令长度为4字节，因此，在执行位于地址0x8000的一条指令后，处理器将从地址0x8004获取下一条指令。

流程控制指令是能够改变正常执行流的指令。在这种情况下，要执行的下一条指令取决于流程控制指令的含义。

> 注意：不同的作者可能会使用不同的术语来指代流程控制指令。例如，Waterman和Asanovi’c[4]使用术语**控制转移指令（control transfer instructions  ）**。其他开发者也可以使用术语**跳转（jump）**或**分支（branch）**来指代流程控制指令。

流程控制指令可以分为条件流程控制和无条件流程控制两类，或者分为直接或间接流程控制两类。下面几节将讨论这些属性。

## 6.7.1 条件流程控制

条件流程控制指令在某些条件下改变正常执行流。换句话说，是否改变正常执行流，取决于给定的条件是否被满足。例如分支相等指令`beq`比较两个寄存器的值，如果它们的值相等则**跳转到目标地址**（改变正常执行流）。

> 注意：
>
> * 跳转（to jump）一词通常用于表明流程控制指令改变了正常执行流
> * 目标地址是，当跳转发生时，下一条需要被读取（并执行）的指令的地址

在下面的代码中，如果寄存器`a0`和`a1`的值相等，指令`beq`会跳转到标签`L`（表示目标地址）开始执行。在这种情况下，下一条要执行的指令将是指令`sub`。

```assembly
beq a0, a1, L
add a0, a0, a1
L:
sub a2, a2, a3
```

如果寄存器`a0`和`a1`的值不同，那么指令`beq`不会跳转，执行流程继续往下执行，即执行内存中的下一条指令（`add`）。

RV32I中有几个条件流程控制指令。表6.10给出了这些指令和伪指令。

| 指令                   | 描述                                                         |
| ---------------------- | ------------------------------------------------------------ |
| `beq rs1, rs2, lab  `  | 如果寄存器`rs1`和`rs2`的值相等，就跳转到标签`lab`            |
| `bne rs1, rs2, lab  `  | 如果寄存器`rs1`和`rs2`的值不相等，就跳转到标签`lab`          |
| `beqz rs1, lab  `      | 如果寄存器`rs1`的值为0，就跳转到标签`lab`（伪指令）          |
| `bnez rs1, lab  `      | 如果寄存器`rs1`的值不为0，就跳转到标签`lab`（伪指令）        |
| `blt rs1, rs2, lab  `  | 如果寄存器`rs1`的值小于寄存器`rs2`的值，就跳转到标签`lab`    |
| `bltu rs1, rs2, lab  ` | 如果寄存器`rs1`的无符号数值小于寄存器`rs2`的无符号数值，就跳转到标签`lab` |
| `bge rs1, rs2, lab  `  | 如果寄存器`rs1`的值大于等于寄存器`rs2`的值，就跳转到标签`lab` |
| `bgeu rs1, rs2, lab  ` | 如果寄存器`rs1`的无符号数值大于等于寄存器`rs2`的无符号数值，就跳转到标签`lab` |

> 表6.1

如果寄存器`rs1`的值小于寄存器`rs2`的值，则指令`blt rs1, rs2, lab`跳转到标签`lab`。在这种情况下，处理器假定`rs1`和`rs2`中的值是有符号值（用补码表示），因此该指令会认为0xFFFFFFFF小于0x00000000。要比较无符号数的值，必须使用`bltu rs1, rs2, lab`指令。在这种情况下，处理器假定`rs1`和`rs2`中的值是无符号的，因此该指令认为0xFFFFFFFF大于0x00000000。

> 注：0xFFFFFFFF作为有符号数时候值为-1，作为无符号数时候值为4294967295 

下面的汇编代码展示了RV32I中的一些流程控制指令：

```assembly
beq a0, a2, THEN # 如果a0 = a2，跳转到标签THEN
bne a1, a3, ELSE # 如果a1 != a3，跳转到标签ELSE 
blt a2, a3, NEXT # 如果a2 < a3，跳转到标签NEXT（比较有符号数）
bge a4, a1, LOOP # 如果a4 >= a1 ，跳转到标签LOOP（比较无符号数）
bltu a0, a2, L # 如果a0 < a2跳转到标签L（比较无符号数）
```

条件流程控制指令通常用于实现if和循环语句。下列代码使用`bne`指令实现了一个迭代10次的循环。在本例中，寄存器`a0`被用作循环计数器，并初始化为10（第2行）。在每次循环迭代后，用`sub`指令减少它的值（第7行）。再用`bne`指令比较计数器与0的大小（第8行）。如果计数器不为零，执行流将重定向到标签`LOOP`并再次执行循环。否则执行流继续往下执行，执行循环之后的`add`指令（第9行）。

```assembly
# 初始化计数器
mov a0, 10
LOOP:
# 做一些事情
# ...
# 减少计数器的值，比较后根据情况继续循环或跳出循环
sub a0, a0, 1
bne a0, zero, LOOP
add a1, a1, a1 # 跳出循环后执行的指令
```

第7章将讨论如何使用流程控制指令来实现高级语言中的条件语句和循环语句，例如C和C++中的`if-then`、`if-then-else`语句以及`while`和`for`循环。

> 注：在RISC-V ISA手册中，Waterman和Asanovi’c使用术语*条件分支*（conditional branch  ）来指代条件流程控制指令。请注意，术语*分支*（branch）意味着执行流可能会分叉到两个不同的路径，而这正是RISC-V条件流程控制指令（的功能）。作者使用术语*无条件跳转*（unconditional jumps）来指代无条件流程控制指令。

## 6.7.2 直接流程控制VS间接流程控制

**直接流程控制指令是一种将目标地址直接编码到指令本身中的指令**。例如，指令`beq a0, a1, L`是一个直接流程控制指令，因为目标地址，即标签`L`定义的地址，被编码到指令本身中。

**间接流程控制指令是一种通过内存或寄存器，间接指定目标地址的流程控制指令**。例如，`jr rs1`是一个间接流程控制指令，它跳转到的地址由寄存器`rs1`的值间接定义。因此改变`rs1`的值，可以很容易地改变指令的目标地址。

> 注：间接流程控制指令通常称为间接跳转指令。我们将在7.3节看到，这些指令可用于从函数返回。

**将目标地址编码到RISC-V流程控制指令中**

RV32I将内存地址编码为32位无符号数，因此**目标地址自然是32位地址**。然而RV32I指令都是32位的，因此，不可能在单个32位长的指令中编码32位目标地址、操作码和其他参数（如寄存器）。为了克服这一限制，目标地址被编码为一个（相对于PC）偏移量，在指令执行时通过PC的值+目标地址计算真实的地址值。在RV32I中，条件流程控制指令的目标地址被编码为12位有符号偏移量。使用12位的偏移量对目标进行编码，可以防止指令跳转到离当前指令太远的地方。

在这种情况下，程序员将目标地址加载到寄存器中，然后使用间接跳转指令跳转到目标地址。

> 译注：最后一句有点奇怪，从上下文推断，作者想表达的应该是如果地址太大无法编码到指令中，就要改为使用间接流程控制，将地址加载到寄存器中然后再跳转。

## 6.7.3 无条件流程控制  

无条件流程控制指令是一种总是跳转到给定地址的指令。例如，`jl`指令是一条流程控制指令，它总是跳转到标签`L`。在下面的代码中，`j`指令（第2行）跳转到标签`FOO`，因此，在执行`j`指令之后处理器执行`sub`指令（第7行）。

```assembly
div a0, a1, a2
j FOO
add a0, a0, a1
mul a0, a1, a2

FOO:
sub a0, a1, a2
# ...
```

> 注意：在RISC-V ISA手册中，Waterman和Asanovi’c使用术语*无条件跳转*（unconditional jumps）来指无条件流程控制指令

在RV32I中有几个无条件流程控制指令。表6.11给出了这些指令和伪指令。

| 指令                | 描述                                                         |
| ------------------- | ------------------------------------------------------------ |
| `j lab`             | 伪指令，跳转到标签`lab`指定的地址                            |
| `jr rs1`            | 伪指令，跳转到寄存器`rs1`指定的地址                          |
| `jal lab`           | 伪指令，将返回地址（PC+4）存到寄存器`ra`中，然后跳转到标签`lab`指定的地址 |
| `jal rd, lab`       | 将返回地址（PC+4）存到寄存器`rd`中，然后跳转到标签`lab`指定的地址 |
| `jarl rd, rs1, imm` | 将返回地址（PC+4）存到寄存器`rd`中，然后跳转到某个地址，该地址的计算方式是：`imm`的值 + 寄存器`rs1`的值 |
| `ret`               | 伪指令，跳转到寄存器`ra`指定的地址                           |

> 表6.11

指令`j lab`跳转到标签`lab`，而指令`jr rs1`跳转到存储在寄存器`rs1`上的地址。

**跳转与链接**

跳转和链接指令（`jal rd, lab`）将后续指令（即PC+4）的地址存储在寄存器`rd`中，然后跳转到标签`lab`。这个过程称为*跳转和链接*（jump and link），在调用例程时特别有用。下面用标记了地址的代码来说明这个过程：
* 首先，`jal`指令（位于内存地址0x8000）用于调用（跳转到）例程`FOO`（第1行）。在执行时，这条指令将后续指令的地址（PC+4 = 0x8004）存储到寄存器`ra`中并跳转到`FOO`。
* 然后，一旦`FOO`被执行，它通过间接跳转（`jr`）返回到寄存器`ra`所指向的地址（第8行）。此时，由于`ra`包含值0x8004，执行流跳转到`sub`指令（第2行）。
* 接下来，执行另一条跳转和链接指令来调用例程`FOO`。同样，跳转和链接指令会将后续指令的地址（PC+4 = 0x800C）存储到寄存器`ra`中，并跳转到`FOO`。但请注意，此时后续的指令是`mul`指令，返回地址是0x800C。
* 最后，`FOO`执行后，例程通过间接跳转（`jr`）返回寄存器`ra`所指向的地址（第8行），在本例中包含值0x800C，这会导致执行流跳转到`mul`指令。

```assembly
0x8000: jal ra, FOO # 调用FOO
0x8004: sub a0, a1, a2
0x8008: jal ra, FOO # 再次调用FOO
0x800C: mul a0, a1, a2

FOO: # FOO例程
0x8080: add a0, a0, a1 # 执行一些计算
0x8084: jr ra # 从例程中返回
```

`jarl rd, rs1, imm`指令也将后续指令的地址（即PC+4）存储在寄存器`rd`中，但在这里，目标地址是通过将`rs1`的值加上立即数`imm`计算的。因此这是间接跳转。

> 注意：
>
> 指令`jr`（jump register）和指令`ret`（return）都是伪指令，由汇编器转换为`jarl`指令。例如：
>
> * `jr rs1`被转换为`jarl zero, rs1, zero`。 
> * `ret`被转换为`jarl zero, ra, zero`。
>
> 指令`j lab`也是伪指令，会被汇编器转换为`jal zero, lab`。
>
> 在RV32I中，存储在寄存器`zero`中的任何值都会被丢弃。

## 6.7.4 系统调用

用户程序（user programs）通常需要执行输入和输出操作。例如在执行一些计算后，程序可能需要将结果打印在屏幕上，或将其写入文件。

输入输出操作通常由输入输出设备执行，如键盘、指向设备（pointing  devices）、打印机、显示器、网络设备、硬盘驱动器等。这些设备由操作系统管理，因此输入和输出操作也由操作系统执行。为了软件开发的可移植性和灵活性，操作系统通常实现一组抽象（如和文件夹、文件等），并提供一组服务例程（service routines）以允许用户程序执行输入和输出操作，用户程序通过调用这些例程来执行输入和输出操作。

系统调用是一种特殊的调用操作，用于调用操作系统服务例程。在RISC-V中，它是由一个叫做`ecall`的特殊指令执行的。

例如，假设用户想调用Linux中的write系统调用，在屏幕上显示一些信息。该系统调用接受三个参数：

1. 文件描述符、
2. 写入信息的缓冲区地址
3. 写入的字节数

文件描述符是一个整数，用于标识一个文件或一个设备。在这种情况下，它表示信息写入的文件或设备。在许多Linux发行版中，文件描述符1用于表示标准输出（即`stdout`），通常是终端屏幕。在下面的例子中，缓冲区`msg`的内容被写入文件描述符1，也就是屏幕。

* 首先，代码设置系统调用所需的参数（第6行到第8行）

* 然后设置寄存器`a7`的值为某个整数，该数字指明了被调用的服务例程，在本例中是write系统调用（第9行）

* 最后，它通过执行`ecall`指令进行调用

```assembly
.data
msg: .asciz "Assembly rocks" # 在内存上分配一个字符串

.text
start:
    li a0, 1 # a0: 文件描述符 = 1 (stdout)
    la a1, msg # a1: 需要写入的消息的地址
    li a2, 14 # a2:  需要写入的消息的长度(14个字节)
    li a7, 64 # 系统调用的类型 (write = 64)
    ecall # 发起系统调用
```

> 注意：每个操作系统可能有一组不同的服务例程。本书的重点不是讨论特定操作系统提供的服务例程。然而为了说明这些概念，在必要时本文将使用Linux操作系统上可用的服务例程



# 6.8 条件设置（conditional set）指令

与条件流程控制指令类似，条件设置指令比较两个值，但它不是跳转到标签，而是将目标寄存器设置为1或0，表示条件是否为真。表6.12给出了RV32I条件设置指令和伪指令。

| 指令                   | 描述                                                         |
| ---------------------- | ------------------------------------------------------------ |
| `slt rd, rs1, rs2  `   | 如果`rs1`的有符号数的值小于`rs2`的，则将`rd`设置为1，否则设置为0。 |
| `slti rd, rs1, imm  `  | 如果`rs1`的有符号数的值小于符号扩展后的立即数`imm`，则将`rd`设置为1，否则设置为0。 |
| `sltu rd, rs1, rs2  `  | 如果`rs1`的无符号数的值小于`rs2`的，则将`rd`设置为1，否则设置为0。 |
| `sltui rd, rs1, imm  ` | 如果`rs1`的无符号数的值小于立即数`imm`，则将`rd`设置为1，否则设置为0。 |
| `seqz rd, rs1  `       | （伪指令）如果`rs1`中的值等于0，则将`rd`设置为1，否则设置为0 |
| `snez rd, rs1  `       | （伪指令）如果`rs1`中的值不等于0，则将`rd`设置为1，否则设置为0 |
| `sltz rd, rs1  `       | （伪指令）如果`rs1`中的值小于0，则将`rd`设置为1，否则设置为0 |
| `sgtz rd, rs1  `       | （伪指令）如果`rs1`中的大小于0，则将`rd`设置为1，否则设置为0 |

如果寄存器`rs1`中的有符号数的值小于寄存器`rs2`的，则`slt`指令将寄存器`rd`设为1，否则为0。注意在这种情况下，两个值都被视为有符号数。sltu`指令与此类似，但它将值作为无符号数比较。

如果寄存器`rs1`中的有符号数的值小于符号扩展后的立即数，则`slti`指令将寄存器`rd`设为1，否则为0。注意在这种情况下，两个值都被视为有符号数字。`sltiu`指令与此类似，但它将值作为无符号数比较。

# 6.9 溢出检测

在执行算术运算时，RISC-V没有提供专门的硬件支持来检测溢出。然而，常规的条件跳转指令和条件设置指令可用于确定是否发生了溢出。下面的代码展示了如何检测两个无符号整数相加时是否发生了溢出。在这种情况下，万一发生溢出，`bltu`指令会跳转到`handle ov  `标签

```assembly
add a0, a1 , a2 # 两数相加
bltu a0, a1, handle_ov # 如果溢出跳转到标签handle_ov
...
handle_ov: # 处理溢出的代码
...
```

> 注：如果两个无符号整数相加发生了溢出，则溢出后的结果小于任意一个加数。这是因为：
>
> 假设 $a1$、 $a2$是两个无符号整数，令
>
> *  $s = a1 + a2$， $s$ 是数学运算意义上的和，不考虑溢出。
>
> *  $s_o = a1 + a2$， $s_o$ 是计算机运算意义上的和，考虑溢出。
>
> 如果没有溢出，则 $s=s_o$ ；当溢出发生，意味着截断像高位溢出的值，则有
> $$
> s_o = s - 2^{32} = a1 + a2 -2^{32}
> $$
> 即有：
> $$
> s_o -a1 = a2-2^{32} < 0\\
> or\\
> s_o -a2 = a1-2^{32} <0
> $$
> 

溢出发生时除了使用条件跳转，还可以使用下列代码来设置寄存器（在本例中是`t1`）以表示是否发生了溢出。注意，如果`a0`(包含`a1`+`a2`)的内容小于`a1`的内容，那么`sltu`指令将寄存器`t1`设为1。

```assembly
add a0, a1, a2 # 两数相加
sltu t1, a0, a1 #  如果溢出发生，即 (a1+a2) < a1时 ，t1 = 1
# 否则, t2 = 0 (没有溢出)
```

以下代码展示了两个有符号整数相加时，如何检测溢出

```assembly
add a0, a1, a2 # 两数相加
slti t1, a2, 0 # t1 = (a2 < 0)
slt t2, a0, a1 # t2 = (a1+a2 < a1)
bne t1, t2, handle_ov # 如果 (a2<0) && (a1+a2>=a1) 或者  (a2>=0) && (a1+a2<a1)时，加法产生了溢出
...
handle_ov: # 处理溢出的代码
...
```



# 6.10 多字节变量（multi-word variables）的算术运算

RV32I对32位整数的运算有硬件原生的支持。例如`add`指令可以用来对两个（有符号或无符号）32位数字相加。但在某些情况下，可能需要对一些由多个word组成的数字（大于32位）求和。此时用户可以分别对数字的高位和低位进行求和，然后再加上进位或者借位。

下面的代码展示了如何将两个64位的数相加。第一个数存储在寄存器`a1`和`a0`中，而第二个数存储在寄存器`a3`和`a2`中，结果存在寄存器`a5`和`a4`中。

* 第一步是将64位数字的低位相加（第1行）。
* 然后为了检查进位，代码检查无符号加法的结果是否小于其中一个操作数（在本例中是`a2`）。如果是，这意味着有一个进位，并且寄存器`t1`被设置`为1`，否则意味着没有进位，寄存器`t1`设置为0（第2行）。
* 接下来，将64位数高32位的部分加在一起（第4行）
* 最后，将高位的和与进位相加

```assembly
add a4, a0, a2 # 低位相加，
sltu t1, a4, a2 # 如果结果（a4）小于a2，说明发生溢出，此时t1=1，相当于t1存储了进位
               # 否则t1=0
add a5, a1, a3 # 高位相加
add a5, t1, a5 # 高位相加的结果再加上进位
```

