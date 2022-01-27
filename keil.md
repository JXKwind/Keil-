# KEIL仿真调试

最好观看官方的使用手册

[手册](https://www.keil.com/support/man/docs/uv4/uv4_overview.htm)

## 基础按钮

![调试窗口](Picture\调试窗口.png)

## 变量的观察方式

### watch窗口

查看可变数据

可以将变量添加到窗口，实时观察变化

**注意 ： 如果添加的太多，会导致卡顿，此时建议使用memory窗口进行观察**

<cannot evaluate>，这是啥意思？这个是说明 KEIL 无法找到这个变量。就我所知，有两种情况会出现这种现象：

1)、这个变量不存在：有可能你之前声明过这个变量，后来发现没用到，删除了。

2)、使用 static 声明的变量。

第二种情况，可以运行到使用该变量的地方观察

可以切换进制显示

#### 通过watch窗口查看外设寄存器数据

![官方资料](Picture\官方资料.PNG)

```c
/** 
  * @brief General Purpose I/O
  */

/*
*	GPIO寄存器结构体
*	这是对照技术手册的地址进行对照的
*/
typedef struct
{
  __IO uint32_t CRL;
  __IO uint32_t CRH;
  __IO uint32_t IDR;
  __IO uint32_t ODR;
  __IO uint32_t BSRR;
  __IO uint32_t BRR;
  __IO uint32_t LCKR;
} GPIO_TypeDef;


#define PERIPH_BASE           ((uint32_t)0x40000000) /*!< Peripheral base address in the alias region */

#define APB2PERIPH_BASE       (PERIPH_BASE + 0x10000)

#define GPIOA_BASE            (APB2PERIPH_BASE + 0x0800)
#define GPIOB_BASE            (APB2PERIPH_BASE + 0x0C00)
#define GPIOC_BASE            (APB2PERIPH_BASE + 0x1000)


#define GPIOA               ((GPIO_TypeDef *) GPIOA_BASE)
#define GPIOB               ((GPIO_TypeDef *) GPIOB_BASE)
#define GPIOC               ((GPIO_TypeDef *) GPIOC_BASE)


```

KEIL 就会从这个地址里读出数据并按照你的指针结构体显示出来

watch窗口有个问题，刷新缓慢

### memory窗口

查看flash数据

按照地址查看，可以选择不同格式数据查看，可以自由修改

Memory 在数据显示上比 Watch 窗口更强大，它可以对单片机上的所有数据进行查看，缺点就是你不知道谁是谁了（没有变量名显示，只能靠地址分辨了）。

注意查看时要添加&



#### 通过memory窗口观察外设寄存器数据

已知外设地址，直接在memory窗口观察即可

可以选择不同宽度显示，方便辨认



### callstack窗口

临时变量可以通过callstack窗口进行查看，这个窗口也可以显示出来函数的调用关系

把断点设置在函数内部，程序停止到函数内部时，可以通过这个窗口查看

最新调用的函数在最下面（所谓的压栈），从下往上看就是

如果你使用操作系统，比如 uCOS，你是没办法在任务函数中观察到这个的，因为任务函数的调用由操作系统负责



### Register窗口

在单片机中，有一种及其特殊的变量，就是**寄存器变量**

这些寄存器没有所谓的地址，所以你没有办法通过取址符 & 获取一个申明为 register 的变量（寄存器的存取速度超快，所以如果一个变量的使用得非常频繁，那么申明为 register 是一个明智之举，但这只是建议编译器去这么做而已，编译器听不听就不知道了，所以即使你声明一个变量为 register，它还可能是内存变量）

可以通过Register窗口观察

## 逻辑分析仪

**很多时候我们并不满足于查看变量的值，可能还想看这个变量的历史变化，同时以波形的方式显示出来，这就需要了解 KEIL 另一个有趣的东西：逻辑分析仪**

![逻辑分析仪](Picture\逻辑分析仪.PNG)

从上面的按钮可以打开逻辑分析仪，通过setup按钮可以添加变量

只有在程序中定义过的，才可以被添加到逻辑分析仪内

![逻辑分析仪2](Picture\逻辑分析仪2.PNG)

可以添加变量，也可以添加对应的IO口

例如PA8

 1.（PORTA & 0x00000100）>> 8

2.   新建添加后，直接在编辑框中输入PORTA，然后底下显示类型中选位类型，下来在右移设置框里面填8，代表PA口的值右移8位，也就是要观察PA8的值
3. 在新建的时候直接输入 PORTA.8 代表PA8口，输入完之后按回车键，软件会自动变成位定义

显示类型用bit

## ini 文件的基础使用

如何记录调试过程中的历史数据？？
在keil里面：

[以下转自天使也有爱博文](https://blog.csdn.net/u014783785/article/details/92581102)

ITM 调试，硬件不支持就不能使用
ini文件可不用硬件支持也可以使用

 ini文件相当于一个额外的.c 文件，可以实现如单片机程序的绝大多数事情，比如读取 IO，读取寄存器，读取内存，操作寄存器，写入内存等等，更多详细的内容可参看官方的在线帮助文档。

[我认为的官方页面](https://www.keil.com/support/man/docs/uv4/uv4_df_createfunct.htm)

在keil的项目设置内添加ini文件

![](Picture\ini文件.PNG)

样例程序：

以下程序来自公众号：**鱼鹰谈单片机**

```
FUNC void TogglePower(void)
{
    unsigned int temp;

    temp = _RWORD(&GPIOA_ODR);
    temp ^= (0x01 << 4);       

    _WWORD(&GPIOA_ODR,temp);

    if(temp & (0x01 << 4))
    {
        printf("Power On!\n");
    }
    else
    {
        printf("Power Off!\n");
    }

}

DEFINE BUTTON "Power ON/OFF", "TogglePower()"

```

和写 C 程序一样，你也可以使用 // 来进行必要的注释，当然 /**/ 也是可以的

另外还要注意的是，指针的使用必然会报错，因为它毕竟不是真正的 C 语言代码，并不支持指针

进入调试界面，调出toolbox窗口

![](Picture\ini文件2.PNG)

单击按钮，可以command窗口出现对应log，观察对应端口也被置位



## Configuration Wizard窗口

[官方文档](http://www.keil.com/support/man/docs/uv4cl/uv4cl_ut_configwizard.htm)

在打开启动文件时，可以观察到keil自动打开一个窗口

![](Picture\启动文件配置.PNG)

点开这个选项，发现可以很方便的修改栈和堆的大小

![启动文件配置1](Picture\启动文件配置1.png)

不难发现，这些是对应的上的

keil如何生成这个文件的？

在前100行的注释下，添加

```
/* <<< Use Configuration Wizard in Context Menu >>>*/                     
```

就会出现这个页面

有些启动文件有出现<h>和</h>符号，这两个是成对出现的，表明中间是一个分组

<o>表示带选择或数字输入的选项，这个选项的名字叫 Heap Size (in Bytes)，后面的<0-16384:8>则表示这个值可输入的范围，即 0-16384，它是以 8 字节为单位的，即你的输入只能是 0x00，0x08，0x10……，当你输入其他数字时，它会自动进行修改成有效数字

注意：为了和源文件兼容，所有的语句都是在注释内，也就是说，即使将文件放到别的不支持配置向导的开发平台中，也不会影响原来的功能

## 准确获得代码运行时间

![运行时间](Picture\运行时间.PNG)

可以看出keil自带计时，并且时间是可以重置的，只要确认时间是准确的，就可以实现很多功能

使用硬件仿真器模拟时，需要设置一下

![运行时间1](Picture\运行时间1.PNG)

将Core Clock 设置为单片机主频，时间就是准确的了

## 通过keil保存数据到hex文件

有些时候，FLASH或 RAM保存了很多参数或者代码，如果通过串口助手之类的工具打印出来保存未免有些麻烦，事实上 KEIL 有命令可以帮助你快速将一块区域数据保存为 HEX 文件，比如鱼鹰想保存从地址 0x0800 0000 开始，大小为 0xC00 的数据，那么只要在 KEIL 命令行输入以下命令即可完成保存，方便快捷，你值得拥有

在调试界面的command窗口输入

```
SAVE data.hex (起始地址),(结束地址)
```

目前测试可以打印出来，数据是有的，但是比较难观察

举例 unsigned char Test_arr[100] = {0};

使用memory串口将第一个元素修改为0x11，最后一个元素修改为0x99

保存后如下所示

```
:020000042000DA
:0C062400110000000000000000000000B9
:1006300000000000000000000000000000000000BA
:1006400000000000000000000000000000000000AA
:10065000000000000000000000000000000000009A
:10066000000000000000000000000000000000008A
:10067000000000000000000000000000000000007A
:080680000000000000000099D9
:00000001FF
```

