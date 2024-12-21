本文主要介绍嵌入式实时操作系统（RTOS），并且以uc/OS为例，将其移植到stm32F103C8T6上，构建3个任务：其中两个task分别以1s和3s周期对LED灯进行点亮-熄灭的控制；另外一个task以2s周期通过串口发送“hello uC/OS! 欢迎来到RTOS多任务环境！”。

#### 目录 

 *  [一、任务要求][Link 1]
 *  [二、嵌入式实时操作系统（RTOS）——uC/OS简介][RTOS_uC_OS]
 *  *  [2.1、RTOS的定义][2.1_RTOS]
    *  [2.2、uC/OS-III简介][2.2_uC_OS-III]
    *  [2.3、组成部分][2.3]
    *  [2.4、相关概念][2.4]
    *  [2.5、特点][2.5]
    *  [2.6、基本功能][2.6]
    *  [2.7、 正常运行平台要求][2.7_]
    *  [2.8、数据类型][2.8]
    *  [2.9、移植用到的部分函数][2.9]
 *  [三、工程创建][Link 2]
 *  *  [3.1、使用STM32CubeMX创建工程][3.1_STM32CubeMX]
    *  [3.2、获取ucOS][3.2_ucOS]
    *  [3.3、准备工作][3.3]
    *  [3.4、uCOS移植][3.4_uCOS]
 *  [四、代码编写][Link 3]
 *  [五、编译烧录][Link 4]
 *  *  [5.1、设置编译环境][5.1]
    *  [5.2、编译][5.2]
    *  [5.3、硬件连接][5.3]
    *  [5.4、烧录][5.4]
    *  [5.5、实现效果][5.5]
 *  [六、总结][Link 5]

## 一、任务要求 

学习 嵌入式实时操作系统（RTOS） ，以 uc/OS为例，将其移植到 stm32F103 上，构建 至少3个任务（task）：其中两个task分别以1s和3s周期对LED灯进行点亮-熄灭的控制；另外一个task以2s周期通过串口发送“hello uC/OS! 欢迎来到RTOS多任务环境！”。记录详细的移植过程。

## 二、嵌入式实时操作系统（RTOS）——uC/OS简介 

### 2.1、RTOS的定义 

Embedded Real-time Operation System ，简写为（RTOS).

当外界事件或数据产生时，能够接受并以足够快的速度予以处理，其处理的结果又能在规定的时间之内来控制生产过程或对处理系统作出快速响应，并 控制所有实时任务协调一致运行 的嵌入式操作系统。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/2bd3be3d55874637bfe86906201adc2f.png)



RTOS（Real Time OS）即 实时操作系统，根据各个任务的要求，进行资源(包括存储器、外设等)管理、消息管理、任务调度、异常处理等工作。在RTOS支持的系统中，每个任务均有一个优先级，RTOS根据各个任务的优先级，动态地切换各个任务，保证对实时性的要求。

实时多任务操作系统，以 分时方式 运行多个任务，任务之间的切换以 优先级 为根据。只有优先服务方式的RTOS才是真正的实时操作系统。

在工业控制、 军事设备、 航空航天等领域对系统的响应时间有苛刻的要求，这就需要使用实时系统。我们常常说的嵌入式操作系统都是嵌入式实时操作系统。比如 μC/OS-II、eCOS 和 Linux、HOPEN OS。

### 2.2、uC/OS-III简介 

uC/OS 是 Micrium 公司出品的RTOS类实时操作系统， UCOS目前有两个版本：UCOS-II 和 UCOS-III。

uC/OS-III (Micro C OS Three 微型的C 语言编写的操作系统第3版) 是一个 可升级的，可固化的，基于优先级的实时内核。特别适合于微处理器和控制器，适合很多商业操作系统的实时操作系统(RTOS)，它 对任务的个数无限制。

uC/OS-III 是一个第3代的系统内核，支持现代的实时内核所期待的大部分功能。例如资源管理，同步，任务间的通信等等。然而，uC/OS-III 提供的特色功能在其它的实时内核中是找不到的，比如说完备的运行时间测量性能，直接地发送信号或者消息到任务，任务可以同时等待多个内核对象 等。

uC/OS-III 是一个 可扩展的，可固化的，抢占式的实时内核 ，它管理的任务个数不受限制。它是第三代内核，提供了现代实时内核所期望的所有功能包括资源管理、同步、内部任务交流等。uC/OS-III 也提供了很多特性是在其他实时内核中所没有的。比如能在运行时测量运行性能，直接得发送信号或消息给任务，任务能同时等待多个信号量和消息队列。

### 2.3、组成部分 

μC/OS 可以大致分成 核心、任务处理、时间处理、任务同步与通信，CPU的移植 等5个部分

 *  核心部分(OSCore.c)：是操作系统的处理核心，包括操作系统初始化、操作系统运行、中断进出的前导、时钟节拍、任务调度、事件处理等多部分；能够 维持系统基本工作 的部分都在这里
 *  任务处理部分(OSTask.c)：任务处理部分中的内容都是与任务的操作密切相关的；包括任务的建立、删除、挂起、恢复等等
 *  时钟部分(OSTime.c)：μC/OS-III中的最小时钟单位是timetick（时钟节拍）；任务延时等操作是在这里完成的
 *  任务同步和通信部分：为事件处理部分，包括信号量、邮箱、消息队列、事件标志等部分；主要用于任务间的互相联系和对临界资源的访问
 *  CPU的移植部分：这部分内容由于牵涉到SP等系统指针，所以通常用汇编语言编写；主要包括中断级任务切换的底层实现、任务级任务切换的底层实现、时钟节拍的产生和处理、中断的相关处理部分等内容

### 2.4、相关概念 

 *  任务（线程） 是简单的程序。单CPU 中，在 任何时刻只能是一个任务被执行。

任务看起来像C 函数。在大多数嵌入式系统中，任务通常是 无限循环 的。任务不能像C函数那样，它是不能返回值的。

在 uC/OS-III 中，任务就是 程序实体，uC/OS-III 能够管理和调度这些小任务（程序）。uC/OS-III中的任务由三部分组成：任务堆栈、任务控制块和任务函数。

 *  任务堆栈：上下文切换的时候用来保存任务的工作环境，就是STM32的内部寄存器值。

任务堆栈是任务的重要部分，堆栈是在RAM中按照“先进先出（FIFO）”的原则组织的一块连续的存储空间。为了满足任务切换和响应中断时保存CPU寄存器中的内容及任务调用其它函数时的需要，每个任务都应该有自己的堆栈。

> \#define START\_STK\_SIZE 512 //堆栈大小  
> CPU\_STK START\_TASK\_STK\[START\_STK\_SIZE\]; //定义一个数组来作为任务堆栈  
> 任务堆栈初始化

任务如果想要切换回上一个任务并且还能接着从上次被中断的地方开始运行，恢复现场即可，现场就是CPU的内部各个寄存器。因此在创建一个新任务时，必须把系统启动这个任务时所需的CPU各个寄存器初始值事先存放在任务堆栈中。这样当任务获得CPU使用权时，就把任务堆栈的内容复制到CPU的各个寄存器，从而可以任务顺利地启动并运行。

把任务初始数据存放到任务堆栈的工作就叫做任务堆栈的初始化，uC/OS-III 提供了完成堆栈初始化的函数：OSTaskStkInit()。

当然，用户一般不会直接操作堆栈初始化函数，任务堆栈初始化函数由任务创建函数 OSTaskCreate() 调用。不同的CPU对于的寄存器和对堆栈的操作方式不同，因此在移植uC/OS-III 的时候需要用户根据各自所选的CPU来编写任务堆栈初始化函数。

 *  任务控制块：任务控制块用来记录任务的各个属性。

任务控制块是用来 记录与任务相关的信息的数据结构，每个任务都要有自己的任务控制块。我们使用OSTaskCreate()函数来创建任务的时候就会给任务分配一个任务控制块。任务控制块由用户自行创建。

> OS\_TCB StartTaskTCB; //创建一个任务控制块

\*\*USOCIII提供了用于任务控制块初始化的函数：OS\_TaskInitTCB()。\*\*但是，用户不需要自行初始化任务控制块。因为和任务堆栈初始化函数一样，函数 OSTaskCreate() 在创建任务的时候会对任务的任务控制块进行初始化。

 *  任务函数：由用户编写的任务处理代码，任务函数通常是一个 无限循环，也可以是一个只执行一次的任务。任务的参数是一个void类型的，可以可以传递不同类型的数据甚至是函数。

任务函数其实就是一个C语言的函数，但是在使用 uC/OS-III 的情况下这个函数不能有用户自行调用，任务函数何时执行执行，何时停止完全有操作系统来控制。

uC/OS-III 支持 时间片轮转调度，因此在一个优先级下会有多个任务，那么我们就要对这些任务做一个管理，这里使用 OSRdyList\[\] 数组管理这些任务。

OSRdyList\[\] 数组中的每个元素对应一个优先级，比如 OSRdyList\[0\] 就用来管理优先级0下的所有任务。OSRdyList\[0\] 为 OS\_RDY\_LIST 类型，从上面 的OS\_RDY\_LIST 结构体可以看到成员变量： HeadPtr 和 TailPtr 分别指向OS\_TCB，我们知道 OS\_TCB 是可以用来 构造链表 的，因此 同一个优先级下的所有任务是通过链表来管理的 ，HeadPtr 和 TailPtr 分别指向这个链表的头和尾，NbrEntries 用来记录此优先级下的任务数量。

同一优先级下如果有多个任务的话最先运行的永远是HeadPtr所指向的任务

### 2.5、特点 

 *  必要性 ---- 嵌入式系统软硬件愈加庞大复杂。
 *  微型化、可裁减 ---- 软、硬件小而精，够用即可。
 *  实时性 ---- 抢占式管理策略，满足时间正确性。
 *  可靠性 ---- 无人值守、自动化设备的使用要求。
 *  易移植 ---- 便于应用到多种的硬件平台。
 *  微内核 ---- 完成OS主要功能的代码很小（附加功能需另挂）。

### 2.6、基本功能 

 *  多任务管理 \-> 丰富的多任务管理函数供目标系统设计者容易完成多任务应用设计。
 *  内存管理 \-> 动态内存管理充分利用硬件资源。
 *  外设管理 \-> 例如I2C、UART、Timer、SPI等设备的驱动。

### 2.7、 正常运行平台要求 

1.  处理器的C编译器能产生可重入代码
2.  用C语言就可以打开和关闭中断
3.  处理器支持中断，并且能产生定时中断(通常在10至100Hz之间)
4.  处理器支持能够容纳一定量数据(可能是几千字节)的硬件堆栈
5.  处理器有将堆栈指针和其它CPU寄存器读出和存储到堆栈或内存中的指令

### 2.8、数据类型 

```java
typedef unsigned char BOOLEAN；
typedef unsigned char INT8U；/无符号8位/
typedef signed char INT8S；/带符号8位/
typedef unsigned int INT16U；/无符号16位/
typedef signed int INT16S；/带符号16位/
typedef unsigned long INT32U；/无符号32位数/
typedef signed long INT32S；/带符号32位数/
typedef float FP32；/* 单精度浮点数*/
typedef double FP64；/* 双精度浮点数*/
typedef unsigned int OS_STK；/堆栈入口宽度/
typedef unsigned int OS_CPU_SR；/寄存器宽度/
```

### 2.9、移植用到的部分函数 

OSStartHighRdy() ：该函数在 OSStart() 多任务启动之后，负责从最高优先级任务的TCB控制块中获得该任务的堆栈指针sp，通过sp依次将CPU现场恢复，此时系统就将控制权交给用户创建的该任务的进程，直到该任务被阻塞或者被其他更高优先级的任务抢占了CPU；该函数仅仅在多任务启动时被执行一次，用来启动第一个，也就是最高优先级的任务执行  
OSCtxSw() ：该函数是任务级的上下文切换函数，在任务因为被阻塞而主动请求与CPU调度时执行，主要工作是先将当前任务的CPU现场保存到该任务堆栈中，然后获得最高优先级任务的堆栈指针，从该堆栈中恢复此任务的CPU现场，使之继续执行，从而完成一次任务切换  
OSIntExit() ：该函数是中断级的任务切换函数，在时钟中断ISR中发现有高优先级任务在等待时，需要在中断退出后不返回被中断的任务，而是直接调度就绪的高优先级任务执行；其目的在于能够尽快让高优先级的任务得到响应，保证系统的实时性能  
OSTickISR() ：该函数是时钟中断处理函数，主要任务是负责处理时钟中断，调用系统实现的 OSTimeTick 函数，如果有等待 时钟信号的高优先级任务，则需要在中断级别上调度其执行；另外两个相关函数是 OSIntEnter() 和 OSIntExit()，都需要在ISR中执行

下面我们来看看如何具体实现RTOS系统下的多任务操作

## 三、工程创建 

### 3.1、使用STM32CubeMX创建工程 

 * 打开 STM32CubeMX，双击 ACCESS TO MCU SELECTOR  
   ![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/91c981ab84a94e9dace924f061718ee2.png)

 * 在搜索框输入 STM32F103C8，双击 STM32F103C8Tx 或者选中之后点击右上角的 Start Project 进入配置界面  
   ![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/93494db1530c4fd2aef725fbe6e62e68.png)

 * 点击 Pinout & Configuration，选择 RCC，将 HSE 设置为 Crystal/Ceramic Resonator  
   ![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/36b705269c6a41c8a352b3df5972d474.png)

 * 点击 SYS，将 Debug 设置为 Serial Wire  
   ![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/9a00f8c647b4490987c86543049f86a0.png)

 * 配置串口 USART1，点击 Connectivity，点击 USART1，将 Mode 设置为 Asynchronous  
   ![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/81389564a9b4451eb776bbd376757ff9.png)

 * 设置 PA5、 PC13 作为两个 LED 的输出端口，将 PA5 和 PC13 设置为 GPIO-Output  
   ![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/a257ee78e32a487cb669cb5dc7979ed6.png)

 * 设置好工程名称、存储路径以及编译环境后生成工程  
   ![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/6fdfb5039bbd45aa94177ad096282fe2.png)

   ![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/26abb1ab447848b9ade3f78735e7ca9a.png)

 * 点击 Open Project 打开工程  
   ![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/4b36fea73eb84e4fa8fd5b86505a7eb6.png)


### 3.2、获取ucOS 

链接：[https://pan.baidu.com/s/1P2NXc64Q\_2c1tXRAvWY3xQ?pwd=2022 ][https_pan.baidu.com_s_1P2NXc64Q_2c1tXRAvWY3xQ_pwd_2022]  
提取码：2022  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/4f4f732c345f600d8bad4b6b5caa60cc.png)

### 3.3、准备工作 

 *  在 uC-BSP 文件夹中新建 bsp.c 和 bsp.h 文件  
    ![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/3f72bc2ab34d4ae7959d779288e81137.png)

 *  将 app\_cfg.h 、 cpu\_cfg.h 、 includes.h 、 lib\_cfg.h 、 os\_app\_hooks.c 、os\_app\_hook.h、os\_cfg.h、os\_cfg\_app.h 复制到文件夹 uC-CONFIG 中  
    ![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/aa40e1bea27a40bda4eb9b5c921ef54b.png)

 *  将 Software 的 uC-BSP、uC-CONFIG、uC-CPU、uC-LIB、uCOS-III 文件复制到 CubeMX\_RTOS 工程的 MDK-ARM 文件夹下  
    ![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/48a7199219a84475aaacbc9a16e51032.png)


    ![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/4accc67bddd74d23a74552e4fc2b3aa0.png)


### 3.4、uCOS移植 

 * 打开使用 CubeMX 生成的工程文件，右键 CubeMX\_RTOS （工程）文件，点击 Manage Project Items 或者直接点击工具栏上面的 Manage Project Items  
   ![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/6a28751d51e9475f9148bde0eb729c3d.png)

 * 在 Groups 下添加如下的文件夹  
   ![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/1968622dc180420f82611a7b291b9827.png)

 * 点击 cpu–>Add Files…，在 MDK-ARM\\uC-CPU 路径下，文件类型选择 All files，选中以下文件，点击 Add 添加  
   ![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/3986537a5e5d43baa13144979057ecfc.png)

   ![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/d3c79c34662e45648b1e14883533f9e9.png)

 * 再在 MDK-ARM\\uC-CPU\\ARM-Cortex-M3\\RealView 路径下选中以下文件，点击 Add 进行添加  
   ![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/550f9df93cca4e54ab9414c7e2470d1d.png)

   ![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/1df71c36cef24007a94742ea8ed62e4e.png)

 * 点击 lib–>Add Files… ，在 MDK-ARM\\uC-LIB 路径下选中下图文件，点击 Add 添加  
   ![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/afc7f51b4c1047038d6aa370bf9d256f.png)

   ![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/e631c50304954c219b1330eef7dd015f.png)

 * 再在 MDK-ARM\\uC-LIB\\Ports\\ARM-Cortex-M3\\RealView 路径下，选中以下文件，点击Add 添加  
   ![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/d333b1985cc347e49867b8bab555e44f.png)

   ![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/b2f37195958b4138b804882da9386ee9.png)

 * 点击 port–>Add Files…，在 MDK-ARM\\uCOS-III\\Ports\\ARM-Cortex-M3\\Generic\\RealView 路径，选中以下文件，点击 Add 添加  
   ![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/a6127fe7fa92406aa31f4483281ff2da.png)

   ![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/d0f5a74af7594f908635a61380be5f01.png)

 * 点击 source–>Add Files…，在 MDK-ARM\\uCOS-III\\Source 路径下选中以下全部 .c .h 文件，点击 Add 添加  
   ![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/1528d5ecace84d5da8be09ce4b961c04.png)

   ![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/d0303da6c85347b482c1b1f86c92188f.png)

 * 点击 config–>Add Files…，在 MDK-ARM\\uC-CONFIG 路径下选中全部文件，点击Add 添加  
   ![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/883227df357343dab6e228a108c38cf6.png)

   ![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/8c8d886894d94428b5df3b4d3c569b3b.png)

 * 点击 bsp–>Add Files…，选中 LMDK-ARM\\uC-BSP 路径下的全部文件，点击 Add 添加  
   ![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/88a8a22f139441259293c8506ce252ba.png)

   ![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/2620f31a4e3d4beb83c2e2b07a94f0aa.png)

 * 导入文件路径，点击 魔法棒->C/C+±>（Include Paths后面的）…->添加 ，然后添加下面的路径  
   ![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/9172db6abe0a4bf4b52b66b8eb301e60.png)


## 四、代码编写 

 *  在 bsp.c 中添加如下代码

```java
#include "includes.h"

#define  DWT_CR      *(CPU_REG32 *)0xE0001000
#define  DWT_CYCCNT  *(CPU_REG32 *)0xE0001004
#define  DEM_CR      *(CPU_REG32 *)0xE000EDFC
#define  DBGMCU_CR   *(CPU_REG32 *)0xE0042004

#define  DEM_CR_TRCENA                   (1 << 24)
#define  DWT_CR_CYCCNTENA                (1 <<  0)

CPU_INT32U  BSP_CPU_ClkFreq (void)
{
            
   
     
     
    return HAL_RCC_GetHCLKFreq();
}

void BSP_Tick_Init(void)
{
            
   
     
     
	CPU_INT32U cpu_clk_freq;
	CPU_INT32U cnts;
	cpu_clk_freq = BSP_CPU_ClkFreq();
	
	#if(OS_VERSION>=3000u)
		cnts = cpu_clk_freq/(CPU_INT32U)OSCfg_TickRate_Hz;
	#else
		cnts = cpu_clk_freq/(CPU_INT32U)OS_TICKS_PER_SEC;
	#endif
	OS_CPU_SysTickInit(cnts);
}
void BSP_Init(void)
{
            
   
     
     
	BSP_Tick_Init();
	MX_GPIO_Init();
}

#if (CPU_CFG_TS_TMR_EN == DEF_ENABLED)
void  CPU_TS_TmrInit (void)
{
            
   
     
     
    CPU_INT32U  cpu_clk_freq_hz;


    DEM_CR         |= (CPU_INT32U)DEM_CR_TRCENA;                /* Enable Cortex-M3's DWT CYCCNT reg.                   */
    DWT_CYCCNT      = (CPU_INT32U)0u;
    DWT_CR         |= (CPU_INT32U)DWT_CR_CYCCNTENA;

    cpu_clk_freq_hz = BSP_CPU_ClkFreq();
    CPU_TS_TmrFreqSet(cpu_clk_freq_hz);
}
#endif

#if (CPU_CFG_TS_TMR_EN == DEF_ENABLED)
CPU_TS_TMR  CPU_TS_TmrRd (void)
{
            
   
     
     
    return ((CPU_TS_TMR)DWT_CYCCNT);
}
#endif

#if (CPU_CFG_TS_32_EN == DEF_ENABLED)
CPU_INT64U  CPU_TS32_to_uSec (CPU_TS32  ts_cnts)
{
            
   
     
     
  CPU_INT64U  ts_us;
  CPU_INT64U  fclk_freq;

  fclk_freq = BSP_CPU_ClkFreq();
  ts_us     = ts_cnts / (fclk_freq / DEF_TIME_NBR_uS_PER_SEC);

  return (ts_us);
}
#endif
 
#if (CPU_CFG_TS_64_EN == DEF_ENABLED)
CPU_INT64U  CPU_TS64_to_uSec (CPU_TS64  ts_cnts)
{
            
   
     
     
	CPU_INT64U  ts_us;
	CPU_INT64U  fclk_freq;

  fclk_freq = BSP_CPU_ClkFreq();
  ts_us     = ts_cnts / (fclk_freq / DEF_TIME_NBR_uS_PER_SEC);
	
  return (ts_us);
}
#endif
```

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/887662831f13439ca115bf59fdab07d1.png)


 *  在 bsp.h 中添加如下代码

```java
#ifndef  __BSP_H__
#define  __BSP_H__

#include "stm32f1xx_hal.h"

void BSP_Init(void);

#endif
```

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/420c6e2d0c7c41b5a649d76493859521.png)


 * 在 startup\_stm32f103xb.s （在Application/MDK-ARM下）文件的以下两个位置（75、76行和174-179行），  
   将 `PendSV_Handler` 改为 `OS_CPU_PendSVHandler`，`SysTick_Handler` 改为 `OS_CPU_SysTickHandler`  
   ![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/cdbbed713408427a8df3c42b36ee4758.png)

   ![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/ca91bd99c7ed4e7e936dbac38becaafc.png)

 * 修改 app\_cfg.h （在config下）文件的代码：  
   `DEF_ENABLED` 改为 `DEF_DISABLED`  
   `#define APP_TRACE BSP_Ser_Printf` 改为 `#define APP_TRACE(void)`  
   ![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/52027f3bc4e04cb88e7b14b6c0a4b227.png)

   ![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/fa3075fbd126482f9000490d04e803d5.png)

 * 修改 includes.h 文件代码，在 `#include <bsp.h>`下面添加 `#include "gpio.h"` 和 `#include "app_cfg.h"`  
   将 `#include <stm32f10x_lib.h>` 改为 `#include "stm32f1xx_hal.h"`  
   ![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/1bb31ef83639419abc4014235455065b.png)

 * 修改 lib\_cfg.h 文件代码，将 27u 修改为 5u  
   ![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/7836e34312314e57a7b351559c42f786.png)

 * 修改 usart.c 文件代码，添加代码完成 printf 重定向

```java
int fputc(int ch,FILE *f){
            
   
     
     
	HAL_UART_Transmit(&huart1,(uint8_t *)&ch,1,0xffff);
	return ch;
}
```

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/81241e43e2684d49916e4ca81d692773.png)


 *  上滑到 `#include "usart.h"`，在 .h 文件处右键，点击，修改 usart.h 文件代码，添加一句定义代码：  
    ![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/b6d8de044ad74ae6954f3ed2faa43721.png)


```java
typedef struct __FILE FILE;
```

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/ea8edcb07a1d4e08a563cfe21adec0ea.png)


 *  初始化管脚，在 gpio.c 文件中修改代码：

```java
void MX_GPIO_Init(void)
{
            
   
     
     

  GPIO_InitTypeDef GPIO_InitStruct = {
            
   
     
     0};

  /* GPIO Ports Clock Enable */
  __HAL_RCC_GPIOC_CLK_ENABLE();
  __HAL_RCC_GPIOD_CLK_ENABLE();
  __HAL_RCC_GPIOA_CLK_ENABLE();

  /*Configure GPIO pin Output Level */
  HAL_GPIO_WritePin(GPIOC, GPIO_PIN_13, GPIO_PIN_RESET);
	HAL_GPIO_WritePin(GPIOA, GPIO_PIN_5, GPIO_PIN_RESET);


  /*Configure GPIO pin : PC13|PA5 */
  GPIO_InitStruct.Pin = GPIO_PIN_13|GPIO_PIN_5;
  GPIO_InitStruct.Mode = GPIO_MODE_OUTPUT_PP;
  GPIO_InitStruct.Pull = GPIO_NOPULL;
  GPIO_InitStruct.Speed = GPIO_SPEED_FREQ_LOW;
  HAL_GPIO_Init(GPIOC, &GPIO_InitStruct);
	HAL_GPIO_Init(GPIOA, &GPIO_InitStruct);
	
}
```

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/763bf3d8fdb842f59b04b1f4137689fd.png)


 *  编写 main.c 的代码

```java
/* Includes ------------------------------------------------------------------*/
#include "main.h"
#include "gpio.h"
#include "usart.h"
/* Private includes ----------------------------------------------------------*/
/* USER CODE BEGIN Includes */
#include <includes.h>
#include "stm32f1xx_hal.h"
/* USER CODE END Includes */

/* Private typedef -----------------------------------------------------------*/
/* USER CODE BEGIN PTD */

/* USER CODE END PTD */

/* Private define ------------------------------------------------------------*/
/* USER CODE BEGIN PD */
/* 任务优先级 */
#define START_TASK_PRIO		3
#define LED0_TASK_PRIO		4
#define MSG_TASK_PRIO		5
#define LED1_TASK_PRIO		6

/* 任务堆栈大小	*/
#define START_STK_SIZE 		96
#define LED0_STK_SIZE 		64
#define MSG_STK_SIZE 		64
#define LED1_STK_SIZE 		64

/* 任务栈 */	
CPU_STK START_TASK_STK[START_STK_SIZE];
CPU_STK LED0_TASK_STK[LED0_STK_SIZE];
CPU_STK MSG_TASK_STK[MSG_STK_SIZE];
CPU_STK LED1_TASK_STK[LED1_STK_SIZE];

/* 任务控制块 */
OS_TCB StartTaskTCB;
OS_TCB Led0TaskTCB;
OS_TCB MsgTaskTCB;
OS_TCB Led1TaskTCB;

/* USER CODE END PD */

/* Private macro -------------------------------------------------------------*/
/* USER CODE BEGIN PM */

/* USER CODE END PM */

/* Private variables ---------------------------------------------------------*/

/* USER CODE BEGIN PV */

/* 任务函数定义 */
void start_task(void *p_arg);
static  void  AppTaskCreate(void);
static  void  AppObjCreate(void);
static  void  led_pc13(void *p_arg);
static  void  send_msg(void *p_arg);
static  void  led_pa5(void *p_arg);
/* USER CODE END PV */

/* Private function prototypes -----------------------------------------------*/
void SystemClock_Config(void);
/* USER CODE BEGIN PFP */

/* USER CODE END PFP */

/* Private user code ---------------------------------------------------------*/
/* USER CODE BEGIN 0 */
/**
  * @brief System Clock Configuration
  * @retval None
  */
void SystemClock_Config(void)
{
            
   
     
     
  RCC_OscInitTypeDef RCC_OscInitStruct = {
            
   
     
     0};
  RCC_ClkInitTypeDef RCC_ClkInitStruct = {
            
   
     
     0};

  /**Initializes the CPU, AHB and APB busses clocks 
  */
  RCC_OscInitStruct.OscillatorType = RCC_OSCILLATORTYPE_HSE;
  RCC_OscInitStruct.HSEState = RCC_HSE_ON;
  RCC_OscInitStruct.HSEPredivValue = RCC_HSE_PREDIV_DIV1;
  RCC_OscInitStruct.HSIState = RCC_HSI_ON;
  RCC_OscInitStruct.PLL.PLLState = RCC_PLL_ON;
  RCC_OscInitStruct.PLL.PLLSource = RCC_PLLSOURCE_HSE;
  RCC_OscInitStruct.PLL.PLLMUL = RCC_PLL_MUL9;
  if (HAL_RCC_OscConfig(&RCC_OscInitStruct) != HAL_OK)
  {
            
   
     
     
    Error_Handler();
  }
  /**Initializes the CPU, AHB and APB busses clocks 
  */
  RCC_ClkInitStruct.ClockType = RCC_CLOCKTYPE_HCLK|RCC_CLOCKTYPE_SYSCLK
                              |RCC_CLOCKTYPE_PCLK1|RCC_CLOCKTYPE_PCLK2;
  RCC_ClkInitStruct.SYSCLKSource = RCC_SYSCLKSOURCE_PLLCLK;
  RCC_ClkInitStruct.AHBCLKDivider = RCC_SYSCLK_DIV1;
  RCC_ClkInitStruct.APB1CLKDivider = RCC_HCLK_DIV2;
  RCC_ClkInitStruct.APB2CLKDivider = RCC_HCLK_DIV1;

  if (HAL_RCC_ClockConfig(&RCC_ClkInitStruct, FLASH_LATENCY_2) != HAL_OK)
  {
            
   
     
     
    Error_Handler();
  }
}

/* USER CODE END 0 */

/**
  * @brief  The application entry point.
  * @retval int
  */
int main(void)
{
            
   
     
     
	OS_ERR  err;
	OSInit(&err);
  HAL_Init();
	SystemClock_Config();
	//MX_GPIO_Init(); 这个在BSP的初始化里也会初始化
  MX_USART1_UART_Init();	
	/* 创建任务 */
	OSTaskCreate((OS_TCB     *)&StartTaskTCB,                /* Create the start task                                */
				 (CPU_CHAR   *)"start task",
				 (OS_TASK_PTR ) start_task,
				 (void       *) 0,
				 (OS_PRIO     ) START_TASK_PRIO,
				 (CPU_STK    *)&START_TASK_STK[0],
				 (CPU_STK_SIZE) START_STK_SIZE/10,
				 (CPU_STK_SIZE) START_STK_SIZE,
				 (OS_MSG_QTY  ) 0,
				 (OS_TICK     ) 0,
				 (void       *) 0,
				 (OS_OPT      )(OS_OPT_TASK_STK_CHK | OS_OPT_TASK_STK_CLR),
				 (OS_ERR     *)&err);
	/* 启动多任务系统，控制权交给uC/OS-III */
	OSStart(&err);            /* Start multitasking (i.e. give control to uC/OS-III). */
               
}


void start_task(void *p_arg)
{
            
   
     
     
	OS_ERR err;
	CPU_SR_ALLOC();
	p_arg = p_arg;
	
	/* YangJie add 2021.05.20*/
  BSP_Init();                                                   /* Initialize BSP functions */
  //CPU_Init();
  //Mem_Init();                                                 /* Initialize Memory Management Module */

#if OS_CFG_STAT_TASK_EN > 0u
   OSStatTaskCPUUsageInit(&err);  		//统计任务                
#endif
	
#ifdef CPU_CFG_INT_DIS_MEAS_EN			//如果使能了测量中断关闭时间
    CPU_IntDisMeasMaxCurReset();	
#endif

#if	OS_CFG_SCHED_ROUND_ROBIN_EN  		//当使用时间片轮转的时候
	 //使能时间片轮转调度功能,时间片长度为1个系统时钟节拍，既1*5=5ms
	OSSchedRoundRobinCfg(DEF_ENABLED,1,&err);  
#endif		
	
	OS_CRITICAL_ENTER();	//进入临界区
	/* 创建LED0任务 */
	OSTaskCreate((OS_TCB 	* )&Led0TaskTCB,		
				 (CPU_CHAR	* )"led_pc13", 		
                 (OS_TASK_PTR )led_pc13, 			
                 (void		* )0,					
                 (OS_PRIO	  )LED0_TASK_PRIO,     
                 (CPU_STK   * )&LED0_TASK_STK[0],	
                 (CPU_STK_SIZE)LED0_STK_SIZE/10,	
                 (CPU_STK_SIZE)LED0_STK_SIZE,		
                 (OS_MSG_QTY  )0,					
                 (OS_TICK	  )0,					
                 (void   	* )0,					
                 (OS_OPT      )OS_OPT_TASK_STK_CHK|OS_OPT_TASK_STK_CLR,
                 (OS_ERR 	* )&err);		

/* 创建LED1任务 */
	OSTaskCreate((OS_TCB 	* )&Led1TaskTCB,		
				 (CPU_CHAR	* )"led_pa5", 		
                 (OS_TASK_PTR )led_pa5, 			
                 (void		* )0,					
                 (OS_PRIO	  )LED1_TASK_PRIO,     
                 (CPU_STK   * )&LED1_TASK_STK[0],	
                 (CPU_STK_SIZE)LED1_STK_SIZE/10,	
                 (CPU_STK_SIZE)LED1_STK_SIZE,		
                 (OS_MSG_QTY  )0,					
                 (OS_TICK	  )0,					
                 (void   	* )0,					
                 (OS_OPT      )OS_OPT_TASK_STK_CHK|OS_OPT_TASK_STK_CLR,
                 (OS_ERR 	* )&err);										 
				 
	/* 创建MSG任务 */
	OSTaskCreate((OS_TCB 	* )&MsgTaskTCB,		
				 (CPU_CHAR	* )"send_msg", 		
                 (OS_TASK_PTR )send_msg, 			
                 (void		* )0,					
                 (OS_PRIO	  )MSG_TASK_PRIO,     	
                 (CPU_STK   * )&MSG_TASK_STK[0],	
                 (CPU_STK_SIZE)MSG_STK_SIZE/10,	
                 (CPU_STK_SIZE)MSG_STK_SIZE,		
                 (OS_MSG_QTY  )0,					
                 (OS_TICK	  )0,					
                 (void   	* )0,				
                 (OS_OPT      )OS_OPT_TASK_STK_CHK|OS_OPT_TASK_STK_CLR, 
                 (OS_ERR 	* )&err);
				 
	OS_TaskSuspend((OS_TCB*)&StartTaskTCB,&err);		//挂起开始任务			 
	OS_CRITICAL_EXIT();	//进入临界区
}
/**
  * 函数功能: 启动任务函数体。
  * 输入参数: p_arg 是在创建该任务时传递的形参
  * 返 回 值: 无
  * 说    明：无
  */
static  void  led_pc13 (void *p_arg)
{
            
   
     
     
  OS_ERR      err;

  (void)p_arg;

  BSP_Init();                                                 /* Initialize BSP functions                             */
  CPU_Init();

  Mem_Init();                                                 /* Initialize Memory Management Module                  */

#if OS_CFG_STAT_TASK_EN > 0u
  OSStatTaskCPUUsageInit(&err);                               /* Compute CPU capacity with no task running            */
#endif

  CPU_IntDisMeasMaxCurReset();

  AppTaskCreate();                                            /* Create Application Tasks                             */

  AppObjCreate();                                             /* Create Application Objects                           */

  while (DEF_TRUE)
  {
            
   
     
     
		HAL_GPIO_WritePin(GPIOC,GPIO_PIN_13,GPIO_PIN_RESET);
		OSTimeDlyHMSM(0, 0, 1, 0,OS_OPT_TIME_HMSM_STRICT,&err);
		HAL_GPIO_WritePin(GPIOC,GPIO_PIN_13,GPIO_PIN_SET);
		OSTimeDlyHMSM(0, 0, 1, 0,OS_OPT_TIME_HMSM_STRICT,&err);
    /* USER CODE END WHILE */

    /* USER CODE BEGIN 3 */
  }
  /* USER CODE END 3 */
}

static  void  led_pa5 (void *p_arg)
{
            
   
     
     
  OS_ERR      err;

  (void)p_arg;

  BSP_Init();                                                 /* Initialize BSP functions                             */
  CPU_Init();

  Mem_Init();                                                 /* Initialize Memory Management Module                  */

#if OS_CFG_STAT_TASK_EN > 0u
  OSStatTaskCPUUsageInit(&err);                               /* Compute CPU capacity with no task running            */
#endif

  CPU_IntDisMeasMaxCurReset();

  AppTaskCreate();                                            /* Create Application Tasks                             */

  AppObjCreate();                                             /* Create Application Objects                           */

  while (DEF_TRUE)
  {
            
   
     
     
		HAL_GPIO_WritePin(GPIOA,GPIO_PIN_5,GPIO_PIN_RESET);
		OSTimeDlyHMSM(0, 0, 3, 0,OS_OPT_TIME_HMSM_STRICT,&err);
		HAL_GPIO_WritePin(GPIOA,GPIO_PIN_5,GPIO_PIN_SET);
		OSTimeDlyHMSM(0, 0, 3, 0,OS_OPT_TIME_HMSM_STRICT,&err);
    /* USER CODE END WHILE */

    /* USER CODE BEGIN 3 */
  }
  /* USER CODE END 3 */
}

static  void  send_msg (void *p_arg)
{
            
   
     
     
  OS_ERR      err;

  (void)p_arg;

  BSP_Init();                                                 /* Initialize BSP functions                             */
  CPU_Init();

  Mem_Init();                                                 /* Initialize Memory Management Module                  */

#if OS_CFG_STAT_TASK_EN > 0u
  OSStatTaskCPUUsageInit(&err);                               /* Compute CPU capacity with no task running            */
#endif

  CPU_IntDisMeasMaxCurReset();

  AppTaskCreate();                                            /* Create Application Tasks                             */

  AppObjCreate();                                             /* Create Application Objects                           */

  while (DEF_TRUE)
  {
            
   
     
     
		printf("hello uc/OS！欢迎来到RTOS多任务环境！ \r\n");
		OSTimeDlyHMSM(0, 0, 2, 0,OS_OPT_TIME_HMSM_STRICT,&err);
    /* USER CODE END WHILE */

    /* USER CODE BEGIN 3 */
  }
  /* USER CODE END 3 */
}


/* USER CODE BEGIN 4 */
/**
  * 函数功能: 创建应用任务
  * 输入参数: p_arg 是在创建该任务时传递的形参
  * 返 回 值: 无
  * 说    明：无
  */
static  void  AppTaskCreate (void)
{
            
   
     
     
  
}


/**
  * 函数功能: uCOSIII内核对象创建
  * 输入参数: 无
  * 返 回 值: 无
  * 说    明：无
  */
static  void  AppObjCreate (void)
{
            
   
     
     

}

/* USER CODE END 4 */

/**
  * @brief  This function is executed in case of error occurrence.
  * @retval None
  */
void Error_Handler(void)
{
            
   
     
     
  /* USER CODE BEGIN Error_Handler_Debug */
  /* User can add his own implementation to report the HAL error return state */

  /* USER CODE END Error_Handler_Debug */
}

#ifdef  USE_FULL_ASSERT
/**
  * @brief  Reports the name of the source file and the source line number
  *         where the assert_param error has occurred.
  * @param  file: pointer to the source file name
  * @param  line: assert_param error line source number
  * @retval None
  */
void assert_failed(uint8_t *file, uint32_t line)
{
            
   
     
      
  /* USER CODE BEGIN 6 */
  /* User can add his own implementation to report the file name and line number,
     tex: printf("Wrong parameters value: file %s on line %d\r\n", file, line) */
  /* USER CODE END 6 */
}
#endif /* USE_FULL_ASSERT */

/************************ (C) COPYRIGHT STMicroelectronics *****END OF FILE****/
```

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/9b39f3e9734f4c31920314ca89e86ad7.png)


OSTimeDlyHMSM (CPU\_INT16U hours,CPU\_INT16U minutes,CPU\_INT16U seconds,CPU\_INT32U milli,OS\_OPT opt,OS\_ERR \*p\_err)函数是用来延时，控制LED亮灭周期和串口通信周期的，具体参数如下：![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/ca89e006cb02428a80a8561dbfa16125.png)


## 五、编译烧录 

### 5.1、设置编译环境 

点击魔法棒，在 Target 下的 Code Generation 处勾选 Use MicroLIB，将 IRAM1 的 Size 修改成 0x8000  
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/283b33c6a0a245b7b91db979af06375c.png)


 *  在 Output 下勾选 Create HEX File  
    ![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/200d0d297918439fb1641fd466bee08a.png)


### 5.2、编译 

 *  点击编译，生成 HEX 文件  
    ![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/011a311fa3024354af15eb938cf72f97.png)


### 5.3、硬件连接 

<table> 
 <thead> 
  <tr> 
   <th>USB转TTL</th> 
   <th>STM32F103C8T6</th> 
  </tr> 
 </thead> 
 <tbody> 
  <tr> 
   <td>GND</td> 
   <td>G</td> 
  </tr> 
  <tr> 
   <td>3V3</td> 
   <td>3V3</td> 
  </tr> 
  <tr> 
   <td>RXD</td> 
   <td>PA9</td> 
  </tr> 
  <tr> 
   <td>TXD</td> 
   <td>PA10</td> 
  </tr> 
 </tbody> 
</table>



注意将核心板上的BOOT0设置为1，BOOT1设置为0

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/08cb26989be949a995b187bf66859b96.png)



LED模块连接方法：

<table> 
 <thead> 
  <tr> 
   <th>输出端口</th> 
   <th>目的端口</th> 
  </tr> 
 </thead> 
 <tbody> 
  <tr> 
   <td>3.3V</td> 
   <td>LED长脚</td> 
  </tr> 
  <tr> 
   <td>PA5</td> 
   <td>LED短脚</td> 
  </tr> 
 </tbody> 
</table>



![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/668a11361f754ab69e53c1c6105525ca.png)


PC13连接着板子上的LED，从而驱动板子上的LED实现亮灭灯的操作

### 5.4、烧录 

将USB转TTL连接到电脑的USB端口，打开 FlyMcu 烧录助手进行烧录  
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/8ca9d40b53834c54b19e90c48e361ac9.png)


### 5.5、实现效果 

[video(video-0WfOFiXK-1734767153897)(type-csdn)(url-https://live.csdn.net/v/embed/439971)(image-https://img-home.csdnimg.cn/images/20230724024159.png?origin_url=https%3A%2F%2Fv-blog.csdnimg.cn%2Fasset%2F6f4554ab373c752b2aa208bbde485e2c%2Fcover%2FCover0.jpg&pos_id=img-yr4Y5SSn-1734767282601))(title-串口)]

[video(video-OkcjLwyh-1734767162960)(type-csdn)(url-https://live.csdn.net/v/embed/439970)(image-https://img-home.csdnimg.cn/images/20230724024159.png?origin_url=https%3A%2F%2Fv-blog.csdnimg.cn%2Fasset%2F844dca6136af92f6b1ce7a8eccd7da3c%2Fcover%2FCover0.jpg&pos_id=img-D6SysPVX-1734767282602))(title-LED)]



 * 连接PA5的LED（蓝色）以三秒为周期亮灭，连接在PC13上的板子上的LED（绿色）以一秒为周期亮灭  


   蓝灯三秒改变一次，绿灯一秒改变一次

 * STM32F103C8T6每两秒通过串口向电脑发送一句“hello uc/OS! 欢迎来到RTOS多任务环境！”

   STM32通过串口每两秒向电脑发送一次信息

## 六、总结 

本次实验主要介绍嵌入式实时操作系统（RTOS）,并且在理论知识讲解的基础上，以 uc/OS 为例，将其移植到 stm32F103C8T6 上，同时实现3个任务：其中两个分别以 1s 和 3s 为周期对LED灯进行点亮-熄灭的控制；另外一个以 2s 为周期通过串口发送“hello uc/OS! 欢迎来到RTOS多任务环境！”。

本次实验不仅加深了我对 实时操作系统 的理解，还加深了我对多任务知识的理解。本次实验完成过程中也遇到了不少问题，实现的过程也比较繁琐，需要花费大量时间。并且在实现的过程中，需要添加很多文件，添加的时候需要主要不能重复添加，也不能少添加，否则会导致编译出错或者运行不了。

本次实验中用到了 OSTimeDlyHMSM 这个函数来控制时间，比起之前的定时简单不少，但是生成的HEX文件明显比之前的文件大上不少，编译和烧录的时间也比之前长一些。

通过本次实践操作，提升了我发现问题、分析问题、解决问题的能力。最后就是感谢大家的阅读，欢迎大家指出本文存在的问题！

参考列表：  
1.[STM32F103C8T6移植uCOS基于HAL库][STM32F103C8T6_uCOS_HAL]  
2.[STM32F103C8T6移植uC/OS-III基于HAL库超完整详细过程][STM32F103C8T6_uC_OS-III_HAL]  
3.[STM32F103C8移植uCOSIII（HAL库）][STM32F103C8_uCOSIII_HAL]  
4.[STM32F103C8T6基于HAL库移植uC/OS-III及逻辑分析仪波形观测][STM32F103C8T6_HAL_uC_OS-III]


[Link 1]: #_5
[RTOS_uC_OS]: #RTOSuCOS_9
[2.1_RTOS]: #21RTOS_10
[2.2_uC_OS-III]: #22uCOSIII_22
[2.3]: #23_31
[2.4]: #24_45
[2.5]: #25_84
[2.6]: #26_92
[2.7_]: #27__98
[2.8]: #28_105
[2.9]: #29_121
[Link 2]: #_130
[3.1_STM32CubeMX]: #31STM32CubeMX_131
[3.2_ucOS]: #32ucOS_157
[3.3]: #33_162
[3.4_uCOS]: #34uCOS_173
[Link 3]: #_216
[Link 4]: #_770
[5.1]: #51_771
[5.2]: #52_778
[5.3]: #53_782
[5.4]: #54_807
[5.5]: #55_811
[Link 5]: #_818
[https_pan.baidu.com_s_1P2NXc64Q_2c1tXRAvWY3xQ_pwd_2022]: https://pan.baidu.com/s/1P2NXc64Q_2c1tXRAvWY3xQ?pwd=2022
[STM32F103C8T6_uCOS_HAL]: https://blog.csdn.net/qq_45659777/article/details/121570886
[STM32F103C8T6_uC_OS-III_HAL]: https://blog.csdn.net/weixin_43116606/article/details/105532222
[STM32F103C8_uCOSIII_HAL]: https://blog.csdn.net/junseven164/article/details/121534916
[STM32F103C8T6_HAL_uC_OS-III]: https://blog.csdn.net/qq_46467126/article/details/121441622?spm=1001.2014.3001.5502

