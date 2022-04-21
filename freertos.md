## FreeRTOS 用户手册阅读笔记

### 前言

**关于FREEROTS**

FreeRTOS是由实时工程师有限公司独家拥有、开发和维护的。实时工程师有限公司一直与世界领先的芯片公司紧密合作，为您提供优秀的商业级完全免费的高质量软件。

FreeRTOS非常适合于使用微控制器或小型微处理器的深度嵌入式实时应用程序。这种类型的应用程序既包括硬件实时需求又包括软实时需求。

软实时需求是指那些声明时间期限的需求，但违反时间期限并不会使系统变得无用。例如，对键盘的响应太慢可能会让系统看起来失去响应，而实际上不会让它无法使用。

硬实时需求是那些规定时间最后期限的需求，违反最后期限将导致系统的绝对故障。例如，如果驾驶员安全气囊对碰撞传感器输入反应太慢会造成非常严重的后果。

FreeRTOS是一个实时内核（或实时调度器），在此之上可以构建嵌入式应用程序来满足它们的硬实时需求。它允许将应用程序组织为一个独立的执行线程的集合。

在只有一个核心的处理器上，任何时候都只能执行一个线程。内核通过检查应用程序设计器分配给每个线程的优先级来决定应该执行哪个线程。在最简单的情况下，应用程序设计器可以为实现硬实时需求的线程分配更高的优先级，而为实现软实时需求的线程分配更低的优先级。这将确保硬实时线程总是在软实时线程之前执行，但是优先级分配决策并不总是那么简单。

**关于术语的注释**

在FreeRTOS中，每个执行的线程都被称为“任务”。在嵌入式社区中，对于术语还没有达成共识，但我更喜欢“任务”而不是“线程”，因为线程在某些应用领域中可以有更具体的含义。

**为什么要使用实时内核？**

有许多成熟的技术可以编写不使用内核的好的嵌入式软件，而且，如果正在开发的系统很简单，那么这些技术可能提供最合适的解决方案。在更复杂的情况下，使用内核可能更好，但二者发生切换的时间点总是主观的。如前所述，任务优先级可以帮助确保应用程序满足其处理期限，但内核也可以带来其他不那么明显的好处。下面非常简要地列出了其中的一些内容。

+ 抽象化后的时间信息

​	内核负责执行时间，并为应用程序提供与时间相关的API。这使得应用程序代码的结构更简单，整体代码的大小也更小。

+ 可维护性/可扩展性

抽象化时间细节会导致模块之间的相互依赖关系减少，并允许软件以一种受控的和可预测的方式发展。此外，内核负责计时，因此应用程序性能不太容易受到底层硬件变化的影响。

+ 模块化

任务是独立的模块，每个模块都应该有一个明确定义的目的。

+ 团队协作

任务还应该有定义良好的接口，允许团队更容易地进行开发

+ 便于测试

如果任务是定义良好的具有干净接口的独立模块，则可以对它们进行隔离测试。

+ 代码重用

更大的模块化和更少的相互依赖关系导致可以用更少的精力重用代码。

+ 提高效率

使用内核允许软件完全由事件驱动，因此轮询未发生的事件不会浪费处理时间。代码只有在有一些必须执行的事情时才会执行，与效率节约相反的是需要处理RTOS滴答中断，并将执行从一个任务切换到另一个任务。然而，不使用RTOS的应用程序通常会包含某种形式的滴答中断。

+ 中断时间

在调度程序启动时，将自动创建空闲任务。它在没有希望执行的应用程序任务时执行。空闲任务可用于测量备用处理容量，执行后台检查，或者简单地将处理器置于低功耗模式。

+ 电池管理

通过使用RTOS而获得的效率提高允许处理器在低功耗模式下花费更多的时间。每次空闲任务运行时，将处理器置于低功耗状态，可以显著降低功耗。FreeRTOS也有一个特殊的无蜱虫模式。使用无标记模式允许处理器进入一个比其他方式更低功耗的模式，并在低功率模式下保持更长的功耗时间。

+ 灵活的中断处理

中断处理程序可以通过延迟处理到应用程序编写器创建的任务或FreeRTOS守护进程任务，从而保持非常短。

+ 混合处理需求

简单的设计模式可以在应用程序中实现周期性、连续和事件驱动的混合处理。此外，通过选择适当的任务和中断优先级，可以满足硬实时和软实时需求。

**FreeRTOS特点：**

FreeRTOS具有以下标准特性：

+ 抢先和合作操作

+ 灵活的任务优先级分配
+ 灵活、快速、轻便的任务通知机制
+ 队列
+ 二进制信号量
+ 计数信号量
+ 互斥量
+ 递归互斥量
+ 软件定时器
+ 事件组
+ 时间片钩子功能
+ 中断钩子功能   
+ 堆栈溢出检查
+ 跟踪记录
+ 任务运行时统计信息收集
+ 可选的商业许可和支持
+ 全中断嵌套模型（对于某些体系结构）
+ 极端低功耗应用无滴答能力
+ 在适当时，软件管理的中断堆栈(这可以帮助保存RAM)

**包括的源文件和项目**

获取伴随这本书而来的例子

本书中提供的所有示例的源代码、预配置的项目文件和完整的构建说明都在一个附带的zip文件中提供。您可以从 http://www.FreeRTOS.org/Documentation/code下载zip文件，如果你没有收到与这本书一起出现的副本。zip文件可能不包括最新版本的FreeRTOS。

### 1.FreeRTOS分布

#### 1.1章节简介及范围

FreeRTOS作为一个单一的zip文件存档文件分发，其中包含所有官方的FreeRTOS端口和大量预配置的演示应用程序。

本章旨在帮助用户使用以下FreeRTOS文件和目录：

+ 提供了FreeRTOS目录结构的顶级视图。
+ 描述任何特定的FReeRTOS项目实际上需要哪些文件。
+ 介绍演示应用程序。
+ 提供有关如何创建新项目的信息。

这里的描述仅涉及到官方的FreeRTOS发行版。这本书附带的例子使用了一个稍微不同的组织。

#### 1.2了解FreeRTOS分发版

定义：FreeRTOS端口

FreeRTOS可以使用大约20种不同的编译器来构建，并且可以在30多种不同的处理器架构上运行。每个受支持的编译器和处理器的组合都被认为是一个单独的FreeRTOS端口。

编译FreeRTOS

FreeRTOS可以被认为是一个库，它为裸机应用程序提供了多任务功能。FreeRTOS是作为一组C源文件提供的。有些源文件对所有端口都是通用的，而其他文件是特定于端口的。将源文件作为项目的一部分来构建，以便使应用程序可以使用FreeRTOSAPI。为了方便您做到这一点，每个官方的FreeRTOS端口都提供了一个演示应用程序。演示应用程序被预配置为构建正确的源文件，并包含正确的头文件。

演示应用程序应该“开箱即用”构建，尽管一些演示比其他的更老，有时自演示发布后进行的构建工具的更改可能会导致问题。第1.3节描述了演示应用程序。

**FreeRTOSConfig.h**

FreeRTOS是由一个名为FreeRTOSConfig.h的头文件配置的。FreeRTOSConfig.h用于定制FreeRTOS。例如，FreeRTOSConfig.h包含像configUSE_PREEMPTION这样的常量，它的设置定义了是使用合作调度算法还是优先级调度算法。由于FreeRTOSConfig.h包含特定于应用程序的定义，因此它应该位于正在构建的应用程序的一部分的目录中，而不是位于包含FreeRTOS源代码的目录中。

为每个FreeRTOS端口提供了一个演示应用程序，并且每个演示应用程序都包含一个FreeRTOSConfig.h文件。因此，没有必要创建FreeRTOSConfig.h文件从头做起，相反，建议从为使用的FreeRTOS端口提供的演示应用程序使用的FreeRTOSTOConfig.h开始。

**官方免费提供服务和操作系统发行版**

FreeRTOS分布在单个zip文件中。zip文件包含所有FreeRTOS端口的源代码，以及所有FreeRTOS演示应用程序的项目文件。它还包含了FreeRTOS+生态系统组件，以及FreeRTOS+生态系统演示应用程序。不要被FreeRTOS发行版中的文件数量所吓到！在任何一个应用程序中，只需要非常少量的文件。

FreeRTOS发行版中的顶级目录

> FreeRTOS
>
> > Source   包含FreeRTOS源文件的目录
> >
> > Demo    包含预配置和端口特定的FreeRTOS演示项目的目录
>
> Freertos-plus
>
> > Source 包含一些FreeRTOS+生态系统组件的源代码的目录
> >
> > Demo 包含针对FreeRTOS+生态系统组件的演示项目的目录

zip文件只包含FreeRTOS源文件的一个副本；所有FreeRTOS演示项目和所有FreeRTOS+演示项目都希望在FreeRTOS/源目录中找到FreeRTOS源文件，如果目录结构发生更改，可能不会构建。

**所有端口共有的免费RTOS源文件**

核心的FreeRTOS源代码仅包含在两个C文件中，这是所有FReeRTOS端口通用的。这些任务被称为tasks.c和list.c，它们直接位于FreeRTOS/Source目录中，如图2所示。除了这两个文件外，以下源文件还位于同一目录中

+ queuue.c   提供队列和信号量服务，如本书后面所述。Queuue.c几乎总是必需的

+ timers.c    提供了软件计时器功能，如本书后面所述。只有要使用软件计时器，它才需要包含在构建中。

+ event_groups.c   提供事件组功能，如本书后面所述。只有在实际上要使用事件组时，才需要将其包含在构建中。

+ croutine.c  实现了FreeRTOS的协同例程功能。只有在实际上要使用协同例程时，它才只需要包含在构建中。协同例程原本打算用于非常小的微控制器，现在很少使用，因此没有维护到与其他FreeRTOS特性相同的级别。在这本书中没有描述共同例程。

> FreeRTOS
>
> > > Source
> > >
> > > > task.c 
> > > >
> > > > list.c
> > > >
> > > > queue.c
> > > >
> > > > timer.c
> > > >
> > > > event_groups.c
> > > >
> > > > croutine.c

上述的文件名可能会导致名称空间冲突，因为许多项目将已经包含了具有相同名称的文件。但是FreeRTOS官方认为现在更改文件的名称不妥，因为这样做会破坏与已经使用FreeRTOS、自动化工具和IDE插件的项目的兼容性。

**特定于端口的FreeRTOS源文件**

特定于FreeRTOS端口的源文件包含在 FreeRTOS/Source/portable目录中。可移植目录被安排为一个层次结构，首先由编译器，然后由处理器架构。该层次结构如图3所示。

如果在使用编译器“*compiler*”的具有架构“*architecture*”的处理器上运行FreeRTOS，那么除了核心的FreeRTOS源文件外，您还必须构建位于FreeRTOS/Source/portable/[compiler]/[architecture]目录中的文件。

正如将在第2章，堆内存管理中描述的，FreeRTOS还将堆内存分配视为可移植层的一部分。使用更早于V9.0.0的FreeRTOS版本的项目必须包含一个堆内存管理器。在FreeRTOSV9.0.0中，只有当configSUPPORT_DYNAMIC_ALLOCATION在FreeRTOSConfig.h中设置为1，或者当configSUPPORT_DYNAMIC_ALLOCATION未定义时，才需要堆内存管理器。

FreeRTOS提供了5个堆分配方案示例。这五种方案分别命名为heap_1到heap_5，并分别由源文件heap_1.c到heap_5.c实现。堆分配方案包含在FreeRTOS/Source/portable/MemMang 目录中。如果您已将FreeRTOS配置为使用动态内存分配，那么就必须在项目中构建这五个源文件中的一个，除非您的应用程序提供了一个替代实现。

**包括路径**

FreeRTOS要求在编译器的包含路径中包含三个目录。这些是：

1.核心FreeRTOS头文件的路径，它是FFreeRTOS/Source/include。

2.特定于正在使用的FreeRTOS端口的源文件的路径。如上所述，FreeRTOS/Source/portable/[*compiler*]/[*architecture*].

3.指向FreeRTOSConfig.h头文件的路径

**头文件**

使用FreeRTOSAPI的源文件必须包含“FreeRTOS.h”，后面是包含正在使用的API函数的原型的头文件  ‘task.h’, ‘queue.h’, 

‘semphr.h’, ‘timers.h’ or ‘event_groups.h’

#### 1.3演示应用程序

每个FreeRTOS端口都至少有一个演示应用程序，这些应用程序应该不会生成错误或警告，尽管一些演示比其他应用程序旧，有时演示发布后的构建工具的更改可能会导致问题。

Linux用户注意：FreeRTOS是在Windows主机上开发和测试的。当演示项目构建在Linux主机上时，这偶尔会导致构建错误。构建错误几乎总是与引用文件名时使用的字母或文件路径中使用的斜杠字符的方向有关。

演示应用程序有几个目的：

+ 提供一个工作的和预配置的项目示例，包含正确的文件和设置正确的编译器选项。
+ 允许用最少的设置或先验知识进行“开箱即用”的实验。
+ 作为如何使用FreeRTOSAPI的演示。
+ 作为可以创建真实应用程序的基础。

每个演示项目都位于FreeRTOS/Demo directory下的一个唯一的子目录中。子目录的名称表示演示项目所关联的端口。

每个演示应用程序也会通过FreeRTOS.org网站上的一个网页来描述。本网页包括以下资料：

+ 如何在FreeRTOS目录结构中找到演示版本的项目文件。
+ 项目被配置为要使用的硬件。
+ 如何设置运行演示的硬件。
+ 如何构建演示版本。
+ 预计演示将如何运行。

所有的演示项目都创建了一个公共演示任务的子集，其实现包含在 FreeRTOS/Demo/Common/Minimal directory。常见的演示任务纯粹是为了演示如何使用FreeRTOSAPI——它们没有实现任何特定有用的功能。

最近的演示项目也可以构建一个初学者的“blinky”项目。blinky的项目是非常基本的。通常，他们将只创建两个任务和一个队列。每个演示项目都包含一个名为main.c的文件。这包含main() 函数，从这里创建所有演示应用程序任务。有关特定于该演示的信息，请参阅单个main.c文件中的注释。

#### 1.4创建FreeRTOS项目

**从所提供的项目上进行更改**

每个FreeRTOS端口都至少有一个预配置的演示应用程序，该应用程序的构建应该没有错误或警告。建议通过调整这些现有项目中的一个来创建新项目；这将允许项目包含正确的文件，安装正确的中断处理程序，以及设置正确的编译器选项

要从现有演示项目启动新应用程序：

1.打开所提供的演示项目，并确保它按照预期进行构建和执行。

2.删除用来定义演示任务的源文件。位于演示/公共目录中的任何文件都可以从项目中删除。

3.删除main()中的所有函数调用，除了prvSetupHardware()和vTaskStartScheduler()，如清单1所示。

4.检查项目是否仍然可以构建

**从头开始创建一个新的项目**

如前所述，建议从现有的演示项目中创建新的项目。如果不这样做，则可以使用以下过程创建一个新项目：

1.使用您所选择的工具链，创建一个还不包含任何FreeRTOS源文件的新项目

2.确保可以构建新项目、下载到目标硬件并执行。

3.只有当您确定您已经有了一个正在工作的项目时，才能将表1中详细说明的FreeRTOS源文件添加到该项目中。

4.将为正在使用的端口提供的演示项目所使用的FreeRTOSConfig.h头文件复制到项目目录中。

5.将以下目录添加到项目将搜索的路径中，以查找头文件：

+ FreeRTOS/Source/include 
+ FreeRTOS/Source/portable/[*compiler*]/[*architecture*]（其中compiler和architecture适合您选择的端口）
+ 包含FreeRTOSConfig.h头文件的目录
+ 包含FreeRTOSConfig.h头文件的目录

6.从相关的演示项目中复制编译器设置。

7.安装可能是必要的任何FreeRTOS中断处理程序。使用描述正在使用的端口的网页，以及为正在使用的端口提供的演示项目作为参考。

### 2.堆内存管理

#### 2.1本章简介及适用范围

**前提**

FreeRTOS是作为一组C源文件提供的，因此成为一个称职的C程序员是使用FreeRTOS的先决条件，因此本章假设读者熟悉以下概念：

+ 如何构建一个C项目，包括不同的编译和链接阶段
+ 堆和栈是什么。
+ 标准的C库malloc()和免费的()函数

**动态内存分配及其与freertos的相关性**

从FreeRTOSV9.0.0中，内核对象可以在编译时静态分配，或在运行时动态分配：

本书的以下章节将介绍内核对象，如任务、队列、信号量和事件组。为了使FreeRTOS尽可能容易使用，这些内核对象不是在编译时静态分配，而是在运行时动态分配；FreeRTOS在每次创建内核对象时都分配RAM，并在每次删除内核对象时都释放RAM。此策略减少了设计和规划工作，简化了API，并最小化了RAM占用。

本章讨论了动态内存分配。动态内存分配是一个C语言编程的概念，而不是一个特定于FreeRTOS或多任务处理的概念。它与FreeRTOS相关，因为内核对象是动态分配的，而且由通用编译器提供的动态内存分配方案并不总是适合于实时应用程序。

内存可以使用标准的C库malloc()和free()函数进行分配，但由于以下原因，它们可能不适合：

+ 它们并不总是在小型嵌入式系统上可用。
+ 它们的实现可能相对较大，占用了有价值的代码空间。
+ 它们很少是线程安全的。
+ 它们不是确定性的；执行函数所花费的时间与调用不同。
+ 他们可能会导致内存碎片。
+ 它们可能会使连接器的配置复杂化。
+ 如果允许堆空间增长到其他变量使用的内存中，从而成为难以调试错误的来源。

**用于动态内存分配的选项**

FreeRTOS的早期版本使用了一种内存池分配方案，即在编译时预先分配不同大小的内存块的池，然后由内存分配函数返回。尽管这是在实时系统中使用的一种常见方案，但它被证明是许多支持请求的来源，主要是因为它不能足够有效地使用RAM，以使其适用于非常小的嵌入式系统——因此该方案被放弃了。

FreeRTOS现在将内存分配视为可移植层的一部分（而不是核心代码库的一部分）。这是为了认识到不同的嵌入式系统具有不同的动态内存分配和时间要求，因此一个单一的动态内存分配算法将永远只适用于应用程序的一个子集。此外，从核心代码库中删除动态内存分配可以使应用程序编写器能够在适当的时候提供它们自己的特定实现。

当FreeRTOS需要RAM时，它不是调用Malloc()，而是调用pvportMalloc()。当释放RAM时，内核不是调用free()，而是调用vPortFree()。pvPortMalloc() 与标准C库malloc()函数具有相同的原型，而vPortFree()与标准C库free())函数具有相同的原型。

pvPortMalloc() 和vPortFree()是公共功能，所以也可以从应用程序代码中调用。

FreeRTOS提供了pvPortMalloc()和vPortFree()的五个实现示例，所有这些都在本章中记录。FreeRTOS应用程序可以使用其中一个示例实现，或者提供它们自己的实现。

这五个例子分别定义在heap_1.c、heap_2.c、heap_3.c、heap_4.c和heap_5.c源文件中，所有这些文件都位于FreeRTOS/Source/portable/MemMang 目录中

**总览**

本章旨在让读者能够很好地理解：

什么时候FreeRTOS分配RAM。

FreeRTOS提供的五个内存分配方案。

选择哪种内存分配方案。

#### 2.2内存分配方案的示例

**Heap_1**

对于小型专用嵌入式系统，通常在调度程序启动之前只创建任务和其他内核对象。在这种情况下，只有在应用程序开始执行任何实时功能之前，内核才能动态地分配内存，并且在应用程序的生命周期中仍然分配内存。这意味着所选择的分配方案不必考虑任何更复杂的内存分配问题，如裁决和碎片化，而可以只考虑代码大小和简单等属性。

Heap_1.c 实现了一个非常基本的pvPortMalloc()，并且没有实现vPortFree()。从未删除任务或其他内核对象的应用程序适合使用heap_1。

一些原本会禁止使用动态内存分配的商业关键和安全系统也有可能适合使用heap_1。关键系统通常禁止动态内存分配，因为与不确定性，内存碎片和失败的分配相关的不确定性——但Heap_1总是确定性的，不能将内存碎片分割。

heap_1分配方案将一个简单的数组细分为更小的块，作为对pvPortMalloc()的调用。该数组被称为FreeRTOS堆。

数组的总大小（以字节为单位）由FreeRTOSConfig.h中的定义configTOTAL_HEAP_SIZE设置。以这种方式定义一个大数组可以使应用程序似乎消耗大量RAM——甚至在从数组分配任何内存之前。

每个已创建的任务都需要从堆中分配一个任务控制块(TCB)和一个堆栈。图5演示了heap_1如何在创建任务时细分简单数组。

参见图5：

+ A显示了在创建任何任务之前的数组——整个数组都是空闲的。

+ B显示了创建一个任务后的数组。

+ C表示在创建了三个任务后的数组。

   ![image-20220418210521280](C:\Users\86178\Desktop\RTOS\${pic}\image-20220418210521280.png)

**Heap_2**

为了向后兼容，Heap_2保留在FReeRTOS发行版中进行向后兼容，但不建议在新设计中使用它。请考虑使用heap_4而不是heap_2，因为heap_4提供了增强的功能。

Heap_2.c还通过细分一个由configTOTAL_HEAP_SIZE标注的数组来工作。它使用最佳匹配算法来分配内存，与heap_1不同，它允许释放内存。同样，数组是静态声明的，因此将使应用程序似乎消耗大量RAM，甚至在数组的任何内存被分配之前。

最佳拟合算法确保pvPortMalloc()使用大小最接近请求字节数的空闲内存块。例如，考虑以下场景：

+ 堆包含三个可用内存块，分别为5字节、25字节和100字节。

+ pvPortMalloc()请求20字节的RAM

  

请求的字节数将适合的最小空闲内存块是25字节块，因此pvPortMalloc()将25字节的块分成一个20字节的块和一个5字节的块

，在返回一个指向20字节块的指针之前。新的5字节块仍然可用于未来调用pvPortMalloc()。

与heap_4不同，Heap_2不将相邻的自由块合并成一个更大的块，因此它更容易产生碎片化 。但是，如果分配和随后释放的块总是相同的大小，碎片不是问题。Heap_2适用于重复创建和删除任务的应用程序，前提是分配给已创建任务的堆栈的大小不会改变。

![image-20220418210946886](C:\Users\86178\Desktop\RTOS\${pic}\image-20220418210946886.png)

图6演示了在创建、删除任务，然后再次创建任务时，最佳匹配算法如何工作。参见图6：

1.A显示了在创建了三个任务后的数组。一个很大的自由块仍然保留在数组的顶部。

2.B显示了删除其中一个任务后的数组。位于数组顶部的较大的自由块保留了下来。现在还有两个较小的空闲块以前分配给TCB和删除任务的堆栈

3.C表示在创建另一个任务后的情况。创建任务导致了对pvportMalloc()的两个调用，一个用于分配一个新的TCB，另一个用于分配任务堆栈。任务是使用xTaskCreate()API函数创建的所述，详见第3.4节。对创建()的调用发生在xTask创建()的内部。

每个TCB的大小完全相同，因此最佳拟合算法确保先前分配给已删除任务的TCB的RAM块被重用，以分配新任务的TCB。

分配给新创建的任务的堆栈大小与分配给之前删除的任务的堆栈大小相同，因此最佳匹配算法可以确保重用之前分配给已删除任务的堆栈的内存块来分配新任务的堆栈。

位于数组顶部的较大的未分配块保持不变。

Heap_2不是确定性的，但比大多数malloc()和free()的标准库实现都要快。

**Heap_3**

Heap_3.c使用了标准的库malloc()和free()函数，因此堆的大小由链接器配置定义，而configTOTAL_HEAP_SIZE设置没有影响。

Heap_3通过暂时暂停FreeRTOS调度程序，使malloc()和free()线程安全。线程安全和调度程序暂停都是第7章，资源管理中讨论的主题。

**Heap_4**

与heap_1和heap_2一样，heap_4的工作原理是将一个数组细分成更小的块。与前面一样，数组是静态声明的，并由configTOTAL_HEAP_SIZE标注，因此将使应用程序似乎消耗大量RAM，甚至在实际从数组分配任何内存之前。

Heap_4使用第一个拟合算法来分配内存。与heap_2不同，heap_4将（合并）相邻的空闲内存块合并成一个更大的内存块，从而将内存碎片化的风险降到最低。

第一个拟合算法确保pvPortMalloc()使用第一个空闲内存块来保存请求的字节数。例如，考虑以下场景：

+ 堆包含三个可用内存块，按照它们在数组中出现的顺序，分别为5字节、200字节和100字节。
+ pvPortMalloc()以请求20字节的RAM。

请求字节数的第一个自由的RAM块是200字节块，因此pvPortMalloc()将200字节块分成一个20字节的块和一个180字节1的块，然后返回一个指向20字节块的指针。新的180字节块仍然可用于对pvPortMalloc()的未来调用Heap_4将（合并）相邻的自由块合并成一个更大的块，最大限度地减少了碎片化的风险，并使其适合于重复分配和释放不同大小的RAM块的应用程序.

![image-20220418212022628](C:\Users\86178\Desktop\RTOS\${pic}\image-20220418212022628.png)

图7展示了在分配和释放内存时，heap_4如何首先配合内存合并的算法工作。参见图7：

1.A显示了在创建了三个任务后的数组。一个很大的自由块仍然保留在阵列的顶部

2.B显示了删除其中一个任务后的数组。位于数组顶部的较大的自由块保留了下来。还有一个自由块，其中的TCB和堆栈,以前已分配了已删除的任务。请注意，与演示heap_2不同的是，删除TCB时释放的内存，以及删除Stack时释放的内存，并不作为两个独立的空闲块，而是组合创建一个更大的单个空闲块。

3.C显示了创建FreeRTOS队列后的情况。队列使用x创建()API函数创建，详见第4.3节。x创建创建()调用pvPortMalloc()来分配队列使用的RAM。由于heap_4使用第一个拟合算法，pvportMalloc()将从第一个空闲RAM块分配RAM，该块大到足以保存队列，图7中是删除任务时释放的RAM。但是，该队列并不消耗空闲块中的所有RAM，因此该块被分成两个，并且未使用的部分仍然可用于将来调用pvPortMalloc()。

4.D显示了在pvPortMalloc()直接从应用程序代码调用后的情况，而不是通过调用FreeRTOSAPI函数间接的。用户分配的块足够小，可以容纳第一个自由块，这是分配给队列的内存和分配给以下TCB的内存之间的块。当任务被删除时释放的内存现在已经被分成三个单独的块；第一个块保存队列，第二个块保存用户分配的内存，第三个块保持空闲。

5.E显示队列被删除后的情况，这会自动释放已分配给已删除队列的内存。现在在用户分配的块的两边都有空闲内存。

6.F显示了用户分配的内存也被释放后的情况。用户分配的块所使用的内存与两边的空闲内存相结合，以创建一个更大的单个空闲块。

Heap_4不是确定性的，但比大多数malloc()和免费()的标准库实现都要快。

**设置Heap_4所使用的数组的起始地址**

本节包含高级级别的信息。使用Heap_4并不需要阅读或理解本节

有时，应用程序编写器需要将heap_4使用的数组放在一个特定的内存地址上。例如，FreeRTOS任务使用的堆栈是从堆中分配的，因此可能有必要确保堆位于快速的内部内存中，而不是缓慢的外部内存中。

默认情况下，heap_4使用的数组将在heap_4.c源文件中声明，其起始地址将由链接器自动设置。但是，如果在FreeRTOSConfig.h中将configAPPLICATION_ALLOCATED_HEAP编译时配置常量设置为1，那么该数组必须由使用FreeRTOS的应用程序声明。如果该数组被声明为应用程序的一部分，那么应用程序的写入器就可以设置其起始地址。

如果configAPPLICATION_ALLOCATED_HEAP在FreeRTOSConfig.h中被设置为1，那么必须在应用程序的一个源文件中声明一个名为configAPPLICATION_ALLOCATED_HEAP的uint8_t数组，并由configTOTAL_HEAP_SIZE设置进行标注。

将变量放置在特定内存地址所需的语法取决于正在使用的编译器，因此请参阅编译器的文档。下面是两个编译器的示例：

+ 清单2显示了GCC编译器声明数组并将数组放在名为.my_heap的内存部分。
+ 清单3显示了IAR编译器声明该数组所需的语法，并将该数组放在绝对内存地址0x20000000处。

![image-20220418213935469](C:\Users\86178\Desktop\RTOS\${pic}\image-20220418213935469.png)

**Heap_5**

heap_5用于分配和空闲内存的算法与heap_4使用的算法相同。与heap_4不同，heap_5并不局限于从单个静态声明的数组中分配内存；heap_5可以从多个独立的内存空间中分配内存。Heap_5在运行FreeRTOS的系统提供的RAM在系统的内存映射中不作为单个连续（无空）块出现时，很有用

在编写时，当heap_5是唯一提供的内存分配方案时，必须显式初始化才能调用 pvPortMalloc() 。Heap_5使用vPortDefineHeapRegions()  API函数初始化。当使用heap_5时，必须在任何内核对象（任务、队列、信号量等）创建之前调用vPortDefineHeapRegions() 。

 **vPortDefineHeapRegions() API函数**

vPortDefineHeapRegions()用于指定每个单独内存区域的起始地址和大小，这些内存区域共同构成了heap_5所使用的总内存。

![image-20220418214519341](C:\Users\86178\Desktop\RTOS\${pic}\image-20220418214519341.png)

每个单独的存储区域都由HeapRegion_t类型的结构来描述。所有可用内存区域的描述作为HeapRegion_t结构数组传递到 vPortDefineHeapRegions() ()。

![image-20220418215205961](C:\Users\86178\Desktop\RTOS\${pic}\image-20220418215205961.png)



**vPortDefineHeapRegions() **参数解释

pxHeapRegions指向HeapRegion_t结构数组开头的指针。数组中的每个结构都描述了在使用heap_5时将成为堆的一部分的内存区域的起始地址和长度。数组中的HeapRegion_t结构必须按起始地址排序；描述具有最低起始地址的内存区域的HeapRegion_t结构必须是数组中的第一个结构，而描述具有最高起始地址的内存区域的HeapRegion_t结构必须是数组中的最后一个结构。数组的末端由一个HeapRegion_t结构标记，该结构的启动起始地址成员设置为NULL。

举例来说，考虑图8A中所示的假设内存映射，它包含三个独立的RAM块：RAM1、RAM2和RAM3。假设可执行代码被放置在只读内存中，但没有显示。

![image-20220418224735682](C:\Users\86178\Desktop\RTOS\${pic}\image-20220418224735682.png)

Listing 6显示了一个HeapRegion_t结构阵列，它们一起描述了三个RAM块。

![image-20220419093533748](C:\Users\86178\Desktop\RTOS\${pic}\image-20220419093533748.png)

虽然Listing 6正确地描述了RAM，但它没有演示一个可用的示例，因为它将所有RAM分配给堆，没有留下任何RAM供其他变量使用。

当构建项目时，构建过程的链接阶段将为每个变量分配一个RAM地址。链接器可供使用的RAM通常由链接器配置文件来描述，例如链接器脚本。

在图8B中，假设链接器脚本包含了关于RAM1的信息，但不包含关于RAM2或RAM3的信息。因此，链接器在RAM1中放置了变量，只留下RAM1中地址0x0001nnnn以上的部分可供heap_5使用。0x0001nnnn的实际值将取决于被链接的应用程序中所包含的所有变量的组合大小。链接器使所有RAM2和所有RAM3都未使用，使得整个RAM2和整个RAM3可供heap_5使用。

如果使用了清单6中所示的代码，那么在地址0x0001nnnn下面分配给heap_5的RAM将与用于保存变量的RAM重叠。为了避免这种情况，xHeap区域[]数组中的第一个HeapRegion_t结构可以使用起始地址0x0001nnnn，而不是起始地址0x00010000。然而，这并不是一个推荐的解决方案，因为：

1.始地址可能不容易确定

2.链接器使用的RAM数量可能会在未来的构建中发生变化，因此需要更新到 HeapRegion_t structure中使用的起始地址。

3.如果链接器使用的RAM和heap_5使用的RAM重叠，构建工具将不知道，因此不能警告应用程序编写器。

Listing 7展示了一个更方便和可维护的示例。它声明了一个名为ucHeap的数组。ucHeap是一个普通变量，因此它成为了链接器分配给RAM1的数据的一部分。xHeap区域数组中的第一个HeapRegion_t结构描述了ucHeap的起始地址和大小，因此ucHeap成为由heap_5管理的内存的一部分。ucHeap的大小可以增加，直到连接器使用的RAM消耗掉了所有的RAM1，如图8C所示。

![image-20220419095522022](C:\Users\86178\Desktop\RTOS\${pic}\image-20220419095522022.png)

清单7中展示的方法的优点包括：

1.不需要使用硬编码的起始地址。

2.在HeapRegion_t结构中使用的地址将由链接器自动设置，因此将始终是正确的，即使链接器使用的RAM数量在未来的构建中发生变化。

3.分配给heap_5的RAM不可能将由链接器放置到RAM1中的数据重叠。

4.如果ucHeap的大小太大，则该应用程序将不会进行链接。

#### 2.3Heap相关实用程序功能

**xPortGetFreeHeapSize()  API功能**

函数返回在调用该函数时堆中的空闲字节数。它可以用于优化堆的大小。例如，如果xPortGet免费大小()在创建了所有内核对象后返回2000，那么configTOTAL_HEAP_SIZE的值可以减少到2000。

当使用heap_3时，xPortGetFreeHeapSize()不可用。

**xPortGetMinimumEverFreeHeapSize() API功能**

函数返回自FreeRTOS应用程序开始执行以来堆中存在的最小未分配的最小字节数。

xPortGetMinimumEverFreeHeapSize（）返回的值表明应用程序离堆空间耗尽还有多远。例如，如果xPortGetMinimumEverFreeHeapSize()返回200，那么，在应用程序开始执行后的某个时候，它距离堆空间耗尽不到200字节。

xPortGetMinimumEverFreeHeapSize（）只有在使用heap_4或heap_5时才可用。

**Malloc失败钩函数**

pvPortMalloc()可以直接从应用程序代码中调用()。每次创建一个内核对象时，在FreeRTOS源文件中也会调用它。内核对象的示例包括任务、队列、信号量和事件组——所有这些都将在本书的后面章节中进行描述。

就像标准库Malloc()函数一样，如果pvPortMalloc()不能返回一个RAM块，因为请求的大小的块不存在，那么它将返回NULL。如果因为应用程序编写器正在创建一个内核对象而执行()，并且对pvPortMalloc()的调用返回NULL，则不会创建内核对象。

如果对pvPortMalloc()的调用返回为空，则可以将所有示例堆分配方案配置为调用一个钩子（或回调）函数。

如果在FreeRTOSConfig.h中将configUSE_MALLOC_FAILED_HOOK设置为1，则应用程序必须提供一个malloc失败的钩子函数，它具有Listing 10所示的名称和原型。该函数可以以任何适合于该应用程序的方式来实现。

![image-20220419111145205](C:\Users\86178\Desktop\RTOS\${pic}\image-20220419111145205.png)

### 3. 任务管理

#### 3.1章节简介及范围

**概览**

本章旨在让读者能够很好地理解：

+ FreeRTOS如何为应用程序中的每个任务分配处理时间。
+ FreeRTOS如何选择在任何给定的时间应该执行哪个任务。
+ 每个任务的相对优先级如何影响系统行为。
+ 任务可以存在的状态。

读者还应该获得一个很好的理解：

+ 如何实现任务。

+ 如何创建一个或多个任务的实例。

+ 如何使用任务参数。
+ 如何更改已经创建的任务的优先级。
+ 如何删除一个任务。
+ 如何使用任务实现周期性处理（软件计时器将在后面的一章中讨论）。
+ 空闲任务何时执行以及如何使用它。

本章中介绍的概念对于理解如何使用FreeRTOS以及FreeRTOS应用程序的行为非常重要。因此，这是书中最详细的一章。

#### 3.2任务函数

任务作为C函数实现。它们唯一的特别之处是它们的原型，它必须返回void并接受一个void指针参数。Listing11展示了这个原型。

![image-20220419113347263](C:\Users\86178\Desktop\RTOS\${pic}\image-20220419113347263.png)

每个任务本身都是一个小程序。它有一个入口点，通常会在一个无限的循环中永远运行，并且不会退出。一个典型任务的结构如清单12所示。

不允许FreeRTOS任务以任何方式从其实现函数返回——它们不能包含‘return”语句，也不允许在函数结束后执行。如果不再需要一个任务，则应该显式地删除它。在 Listing12中也说明了这一点。

单个任务函数定义可用于创建任意数量的任务——每个创建的任务都是一个单独的执行实例，具有自己的堆栈和在任务内部定义的任何自动（堆栈）变量的副本。

![image-20220419113858935](C:\Users\86178\Desktop\RTOS\${pic}\image-20220419113858935.png)

#### 3.3顶级任务状态

 

一个应用程序可以包含许多任务。如果运行应用程序的处理器包含单个核心，那么在任何给定的时间只能执行一个任务。这意味着任务可以处于两种状态之一，运行和不运行。首先考虑这个简单化的模型，但请记住，它是一个过于简化的模型。本章后面显示“未运行”状态实际上包含许多子状态。

当一个任务处于“正在运行”的状态时，处理器正在执行该任务的代码。当任务处于“不运行”状态时，该任务处于休眠状态，其状态已保存，以便下次调度程序决定进入运行状态时恢复执行。当任务恢复执行时，它从上次离开运行状态之前将要执行的指令执行。

![image-20220419125441902](C:\Users\86178\Desktop\RTOS\${pic}\image-20220419125441902.png)



从“未正在运行”状态转换为“正在运行”状态的任务据说已“切换”或“交换”。相反，从“运行”状态转换到“不运行”状态的任务被称为“关闭”或“交换关闭”。FreeRTOS调度程序是唯一可以切换输入和输出任务的实体。

#### **3.4创建任务**

 **xTaskCreate()**API函数

注：FreeRTOSV9.0.0还包括xTask创建静态()函数，该函数可以分配在编译时静态创建任务所需的内存

使用FreeRTOSxTaskCreate()API函数创建任务。这可能是所有API函数中最复杂的，所以不幸的是，它是第一次遇到的，但必须首先掌握任务，因为它们是多任务处理系统中最基本的组件。本书附带的所有示例都使用了xTaskCreate()函数，所以有许多示例需要参考。

第1.5节，数据类型和编码样式指南，描述了所使用的数据类型和命名约定。

![image-20220419125643109](C:\Users\86178\Desktop\RTOS\${pic}\image-20220419125643109.png)

| 参数名/返回值  | 描述                                                         |
| -------------- | ------------------------------------------------------------ |
| pvTaskCode     | 任务只是从不退出的C函数，因此，通常被实现为一个无限循环。pvTaskCode参数只是一个指向实现该任务的函数的指针（实际上，它只是该函数的名称）。 |
| pcName         | 该任务的描述性名称。FreeRTOS没有以任何方式使用它。它纯粹作为调试辅助工具。通过一个人类可读的名称来识别一个任务比试图通过其句柄来识别它要简单得多。应用程序定义的常量configMAX_TASK_NAME_LEN定义了任务名称最大长度——包括NULL终止符。提供超过此最大值的字符串将导致该字符串被静默地截断。 |
| usStackDepth   | 每个任务都有自己的唯一堆栈，在创建任务时由内核分配给任务。usstack深度值告诉内核使堆栈有多大。该值指定堆栈可以保存的字数，而不是字节数。例如，如果堆栈是32位宽，并且sstackDepth为100，那么将分配400字节的堆栈空间（100*4字节）。堆栈深度乘以堆栈宽度不能超过uint16_t类型的变量中可以包含的最大值。空闲任务所使用的堆栈的大小由应用程序定义的常数configMINIMAL_STACK_SIZE1来定义。在FreeRTOS演示应用程序中，为正在使用的处理器体系结构分配给这个常量的值是为任何任务推荐的最小值。如果您的任务使用了大量的堆栈空间，则必须分配一个更大的值。 |
| pvParameters   | 任务函数接受指向void（void*）的参数。分配给pv参数的值是传递到任务中的值。本书中的一些例子演示了如何使用该参数。 |
| uxPriority     | 定义要执行的任务的优先级。优先级可以从最低优先级0分配到最高优先级（configMAX_PRIORITIES-1）。configMAX_PRIORITIES是在第3.5节中描述的一个用户定义的常量。传递一个高在上面的ux优先级值(configMAX_PRIORITIES-1)将导致分配给任务的优先级被静默地限制为最大的合法值。 |
| pxCreatedTask  | pxCreatedTask任务可以用来将句柄传递给正在创建的任务的句柄。这个句柄可以用于引用API调用中的任务，例如，更改任务优先级或删除任务。 |
| Returned value | 有两个可能的返回值：pdPASS 这表明该任务已成功创建。pdFAIL这表明还没有创建任务，因为没有足够的可供FreeRTOS使用的堆内存来分配足够的RAM来保存任务数据结构和堆栈。 |

**Example 1. Creating tasks**

本示例演示了创建两个简单任务，然后启动正在执行的任务所需的步骤。这些任务只是定期打印出一个字符串，使用一个=空循环来创建延时，这两个任务都以相同的优先级创建，除了它们打印出的字符串之外，它们都是相同的——它们各自的实现请参见Listing 14和Listing 15。

main()函数在启动调度程序之前创建任务——其实现请参见Listing 16。

```c
/****Listing 14 : Implementation of the first task used in Example 1 *****/
void vTask1( void *pvParameters )
{
const char *pcTaskName = "Task 1 is running\r\n";
volatile uint32_t ul; /* volatile to ensure ul is not optimized away. */
 /* As per most tasks, this task is implemented in an infinite loop. */
 for( ;; )
 {
 /* Print out the name of this task. */
 vPrintString( pcTaskName );
 /* Delay for a period. */
 for( ul = 0; ul < mainDELAY_LOOP_COUNT; ul++ )
 {
 /* This loop is just a very crude delay implementation. There is
 nothing to do in here. Later examples will replace this crude
 loop with a proper delay/sleep function. */
 }
 } 
}
```

```c
/***Listing 15 :Implementation of the second task used in Example 1***/
void vTask2( void *pvParameters )
{
const char *pcTaskName = "Task 2 is running\r\n";
volatile uint32_t ul; /* volatile to ensure ul is not optimized away. */
 /* As per most tasks, this task is implemented in an infinite loop. */
 for( ;; )
 {
 /* Print out the name of this task. */
 vPrintString( pcTaskName );
 /* Delay for a period. */
 for( ul = 0; ul < mainDELAY_LOOP_COUNT; ul++ )
 {
 /* This loop is just a very crude delay implementation. There is
 nothing to do in here. Later examples will replace this crude
 loop with a proper delay/sleep function. */
 }
 }
}
```

主()函数在启动调度程序之前创建任

```c
/***Listing 16 : Starting the Example 1 tasks***/
int main( void )
{
 /* Create one of the two tasks. Note that a real application should check
 the return value of the xTaskCreate() call to ensure the task was created
 successfully. */
 xTaskCreate( vTask1, /* Pointer to the function that implements the task. */
 "Task 1",/* Text name for the task. This is to facilitate 
 debugging only. */
 1000, /* Stack depth - small microcontrollers will use much
 less stack than this. */
 NULL, /* This example does not use the task parameter. */
 1, /* This task will run at priority 1. */
 NULL ); /* This example does not use the task handle. */
 /* Create the other task in exactly the same way and at the same priority. */
 xTaskCreate( vTask2, "Task 2", 1000, NULL, 1, NULL );
 /* Start the scheduler so the tasks start executing. */
 vTaskStartScheduler(); 
 
 /* If all is well then main() will never reach here as the scheduler will 
 now be running the tasks. If main() does reach here then it is likely that 
 there was insufficient heap memory available for the idle task to be created. 
 Chapter 2 provides more information on heap memory management. */
 for( ;; );
}
```

实验结果如下图：

![image-20220419132514257](C:\Users\86178\Desktop\RTOS\${pic}\image-20220419132514257.png)屏幕截图显示每个任务在下一个任务执行之前只打印一次消息。这是一个通过使用FreeRTOSWindows模拟器而产生的人工场景。Windows模拟器并不是真正实时的。此外，写入Windows控制台需要较长时间，会导致Windows系统调用。使用快速和非阻塞打印函数在真正的嵌入目标上执行相同的代码可能会导致每个任务在切换之前多次打印其字符串。

图10显示了似乎同时执行的两个任务；但是，由于这两个任务都在同一个处理器核心上执行，因此情况并非如此。实际上，这两个任务都在快速进入和退出运行状态。这两个任务以相同的优先级运行，因此在同一处理器核心上共享时间。它们的实际执行模式如图11所示。

图11底部的箭头显示了从时间t1开始的时间流逝。彩色的线表示在每个时间点上正在执行哪个任务——例如，任务1在时间t1和时间t2之间正在执行。任何时候只能有一个任务处于运行状态。因此，当一个任务进入“运行”状态（任务切换）时，另一个任务进入“不运行”状态（任务切换）。

![image-20220419132633428](C:\Users\86178\Desktop\RTOS\${pic}\image-20220419132633428.png)

示例1在启动调度程序之前，从主()中创建了这两个任务。也可以从另一个任务中创建一个任务。例如，任务2可以从任务1中创建，如Listing17所示。

```c
/**Listing 17. Creating a task from within another task after the scheduler has started**/
void vTask1( void *pvParameters )
{
const char *pcTaskName = "Task 1 is running\r\n";
volatile uint32_t ul; /* volatile to ensure ul is not optimized away. */
 /* If this task code is executing then the scheduler must already have
 been started. Create the other task before entering the infinite loop. */
 xTaskCreate( vTask2, "Task 2", 1000, NULL, 1, NULL );
 for( ;; )
 {
 /* Print out the name of this task. */
 vPrintString( pcTaskName );
 /* Delay for a period. */
 for( ul = 0; ul < mainDELAY_LOOP_COUNT; ul++ )
 {
 /* This loop is just a very crude delay implementation. There is
 nothing to do in here. Later examples will replace this crude
 loop with a proper delay/sleep function. */
 }
 } 
}

```

**例子2：使用任务参数**

在示例1中创建的两个任务几乎是相同的，它们之间唯一的区别是它们打印出的文本字符串。可以通过创建单个任务实现的两个实例来删除此重复。然后，可以使用任务参数将它应该打印出来的字符串传递给每个任务。

Listing 18包含示例2所使用的单个任务函数(vtask函数)的代码。这个单一函数取代了示例1中使用的两个任务函数(vTask1和vTask2)。注意如何将任务参数转换为char*以获得任务应该打印出的字符串。

```c
/****Listing 18. The single task function used to create two tasks in Example 2***/
void vTaskFunction( void *pvParameters )
{
char *pcTaskName;
volatile uint32_t ul; /* volatile to ensure ul is not optimized away. */
 /* The string to print out is passed in via the parameter. Cast this to a
 character pointer. */
 pcTaskName = ( char * ) pvParameters;
 /* As per most tasks, this task is implemented in an infinite loop. */
 for( ;; )
 {
 /* Print out the name of this task. */
 vPrintString( pcTaskName );
 /* Delay for a period. */
 for( ul = 0; ul < mainDELAY_LOOP_COUNT; ul++ )
 {
 /* This loop is just a very crude delay implementation. There is
 nothing to do in here. Later exercises will replace this crude
 loop with a proper delay/sleep function. */
 }
 } }
```

尽管现在只有一个任务实现(vTask函数)，但也可以创建已定义任务的多个实例。每个创建的实例都将在FreeRTOS调度程序的控制下独立执行。清单19显示了如何使用xTaskCreate()函数的pv参数参数来将文本字符串传递到任务中。

```c
/***Listing 19. The main() function for Example 2.***/
/* Define the strings that will be passed in as the task parameters. These are
defined const and not on the stack to ensure they remain valid when the tasks are
executing. */
static const char *pcTextForTask1 = "Task 1 is running\r\n";
static const char *pcTextForTask2 = "Task 2 is running\r\n";
int main( void )
{
 /* Create one of the two tasks. */
 xTaskCreate( vTaskFunction, /* Pointer to the function that 
 implements the task. */
 "Task 1", /* Text name for the task. This is to 
 facilitate debugging only. */
 1000, /* Stack depth - small microcontrollers
 will use much less stack than this. */
 (void*)pcTextForTask1, /* Pass the text to be printed into the 
 task using the task parameter. */
 1, /* This task will run at priority 1. */
 NULL ); /* The task handle is not used in this 
 example. */
 /* Create the other task in exactly the same way. Note this time that multiple
 tasks are being created from the SAME task implementation (vTaskFunction). Only 
 the value passed in the parameter is different. Two instances of the same 
 task are being created. */
 xTaskCreate( vTaskFunction, "Task 2", 1000, (void*)pcTextForTask2, 1, NULL );
 /* Start the scheduler so the tasks start executing. */
 vTaskStartScheduler(); 
 
 /* If all is well then main() will never reach here as the scheduler will 
 now be running the tasks. If main() does reach here then it is likely that 
 there was insufficient heap memory available for the idle task to be created. 
 Chapter 2 provides more information on heap memory management. */
 for( ;; );
}

```

示例2的输出与图10中的示例1完全相同。

#### 3.5任务优先级

xTaskCreate() API 函数的ux优先级参数为正在创建的任务分配一个初始优先级。可以在调度程序启动后使用vTask优先集()API函数更改优先级。可以在调度程序启动后，通过使用vTaskPrioritySet() 来更改优先级.

可用的最大优先级数由应用程序定义的对象设置configMAX_PRIORITIES 在FreeRTOSConfig.h中编译时间配置常数。低数字优先级值表示低优先级任务，优先级0是可能的最低优先级。因此，可用优先级的范围为0到（configMAX_PRIORITIES-1）。任意数量的任务都可以共享相同的优先级——确保最大限度的设计灵活性。

FreeRTOS调度程序可以使用两种方法之一来决定哪个任务将处于运行状态。configMAX_PRIORITIES可以设置到的最大值取决于所使用的方法：

1. 通用方法

   通用方法用C语言实现，可以与所有FreeRTOS体系结构端口一起使用。当使用泛型方法时，FreeRTOS不限制configMAX_PRIORITIES可以设置到的最大值。但是，最好将configMAX_PRIORITIES值保持在必要的最小值，因为其值越高，消耗的RAM就越多，最坏情况的执行时间就越长。如果在FreeRTOSConfig.h中configUSE_PORT_OPTIMISED_TASK_SELECTION设置为0，或者如果未定义configUSE_PORT_OPTIMISED_TASK_SELECTION，或者泛型方法是为使用的FreeRTOS端口提供的唯一方法，则将使用通用方法。

 2.体系结构优化方法

体系结构优化方法使用了少量的汇编代码，并且比通用方法更快。configMAX_PRIORITIES设置不会影响最坏情况下的执行时间。如果使用了体系结构优化的方法，则configMAX_PRIORITIES不能大于32。与通用方法一样，建议将configMAX_PRIORITIES保持在必要的最小值，因为它的值越高，所消耗的RAM就越多。如果在FreeRTOSConfig.h中将configUSE_PORT_OPTIMISED_TASK_SELECTION设置为1，则将使用体系结构优化方法。

并非所有的FreeRTOS端口都提供了架构优化的方法。

FreeRTOS调度程序将始终确保能够运行的最高优先级任务是选择要进入运行状态的任务。当能够运行多个具有相同优先级的任务时，调度程序将依次将每个任务转换为和退出“正在运行”状态。

#### **3.6时间测量和计时器中断**

第3.12节，调度算法描述了一个被称为“时间切片”的可选特性。到目前为止的例子中使用了时间切片，是在它们产生的输出中观察到的行为。在示例中，两个任务都以相同的优先级创建，并且两个任务总是能够运行。因此，为“时间片”执行的每个任务，在时间片开始时输入“运行”状态，在时间片结束时退出“运行”状态。在图11中，t1和t2之间的时间等于单个时间片。

为了能够选择下一个要运行的任务，调度程序本身必须在每个时间片1的末尾执行。一个周期性的中断，称为“滴答中断”，被用于这个目的。时间片的长度有效地由提示中断频率设置，提示中断频率由FreeRTOSConfig.h中的应用程序定义的configTICK_RATE_HZ编译时配置常数进行配置。例如，如果configTICK_RATE_HZ被设置为100(Hz)，那么时间片将为10毫秒。两次滴答中断之间的时间被称为“滴答周期”。一个时间切片等于一个滴答周期。

图11可以扩展以显示调度程序本身在执行顺序中的执行。如图12所示，顶部行显示调度程序的执行时间，细箭头显示从任务到提示中断的执行顺序，然后从提示中断到另一个任务的执行顺序。configTICK_RATE_HZ的最佳值取决于正在开发的应用程序，尽管100的值是典型的。

需要注意的是，时间片的结束并不是调度程序可以选择要运行的新任务的唯一位置；正如本书中所演示的那样，在当前执行的任务进入阻塞状态后，或当中断将更高优先级的任务移动到就绪状态时，调度程序还将选择要立即运行的新任务。

![image-20220419141220410](C:\Users\86178\Desktop\RTOS\${pic}\image-20220419141220410.png)

FreeRTOSAPI调用总是以多个滴答周期来指定时间，这通常被简单地称为“滴答”。pdMS_TO_TICKS()宏将以毫秒为单位指定的时间转换为以刻度数指定的时间。可用的分辨率取决于所定义的滴答频率，如果滴答频率高于1KHz(如果configTICK_RATE_HZ大于1000)，则不能使用pdMS_TO_TICKS()。可用的分辨率取决于所定义的滴答频率，如果滴答频率高于1KHz(如果configTICK_RATE_HZ大于1000)，则不能使用pdMS_TO_TICKS()。Listing 20 显示了如何使用pdMS_TO_TICKS()将指定为200毫秒的时间转换为在刻度中指定的等效时间。

![image-20220419141607710](C:\Users\86178\Desktop\RTOS\${pic}\image-20220419141607710.png)

注意：不建议直接在应用程序中指定时间，而是使用pdMS_TO_TICKS()宏以毫秒为单位指定时间，这样做，确保在改变指定时间频率时，应用程序中指定的时间不会改变。

“滴答计数”值是自调度程序启动以来发生的滴答中断的总数，假设滴答计数没有溢出。用户应用程序在指定延迟周期时不必考虑溢出，因为时间一致性是FreeRTOS在内部管理的。

 第3.12节，调度算法，描述了影响调度程序何时选择要运行的新任务，以及何时执行提示中断的配置常量

**例子3 优先级实验**

调度程序将始终确保能够运行的最高优先级任务是选择要进入运行状态的任务。在我们到目前为止的示例中，有两个任务以相同的优先级创建了，因此它们都依次进入和退出了运行状态。本示例查看了当示例2中创建的两个任务之一的优先级被更改时，会发生什么。这一次，第一个任务将在优先级1处创建，第二个任务将在优先级2处创建。创建这些任务的代码如清单21所示。实现这两个任务的单个函数没有改变；它仍然只是定期周期性地打印出一个字符串，使用一个空循环来创建一个延迟。

```c
/**Listing 21. Creating two tasks at different priorities**/
/* Define the strings that will be passed in as the task parameters. These are
defined const and not on the stack to ensure they remain valid when the tasks are
executing. */
static const char *pcTextForTask1 = "Task 1 is running\r\n";
static const char *pcTextForTask2 = "Task 2 is running\r\n";
int main( void )
{
 /* Create the first task at priority 1. The priority is the second to last 
 parameter. */
 xTaskCreate( vTaskFunction, "Task 1", 1000, (void*)pcTextForTask1, 1, NULL );
 /* Create the second task at priority 2, which is higher than a priority of 1.
 The priority is the second to last parameter. */
 xTaskCreate( vTaskFunction, "Task 2", 1000, (void*)pcTextForTask2, 2, NULL );
 /* Start the scheduler so the tasks start executing. */
 vTaskStartScheduler(); 
 
 /* Will not reach here. */
 return 0;
}
```

示例3所产生的输出如图13所示。调度程序将始终选择能够运行的最高优先级的任务。任务2的优先级高于任务1，并且始终能够运行；因此，任务2是唯一一个进入“正在运行”状态的任务。由于任务1从未进入“正在运行”状态，因此它从未打印出其字符串。任务1被任务2称为“缺乏”处理时间。

![image-20220419143729070](C:\Users\86178\Desktop\RTOS\${pic}\image-20220419143729070.png)

任务2总是能够运行，因为它不需要等待任何事情——它要么执行空循环，要么打印到终端。

图14显示了示例3的执行序列

![image-20220419143836341](C:\Users\86178\Desktop\RTOS\${pic}\image-20220419143836341.png)

#### 3.7扩展“未运行”状态

到目前为止，创建的任务总是有处理来执行，从来没有需要等待任何事情——因为它们不必等待任何事情，它们总是能够进入运行状态。这种类型的“连续处理”任务的用途有限，因为它们只能在非常最低的优先级下创建。如果它们以任何其他优先级运行，它们将完全阻止优先级较低的任务运行。

为了使任务有用，必须将它们重写为事件驱动的。事件驱动的任务只有在触发它的事件发生后才可以执行工作（处理），并且不能在该事件发生之前进入运行状态。调度程序总是选择能够运行的最高优先级的任务。高优先级任务无法运行意味着调度程序无法选择它们，而必须选择一个能够运行的低优先级任务。因此，使用事件驱动的任务意味着可以在不同的优先级上创建任务，而不会使最高优先级的任务占用处理时间中的所有较低优先级的任务。

**阻塞状态**

等待事件的任务据说处于“已阻止”状态，这是“未运行”状态的子状态。

任务可以进入阻塞状态，以等待两种不同类型的事件：

1. 时间（与时间相关）事件—事件是延迟期间或达到的绝对时间。例如，一个任务可能会进入阻塞状态，等待10毫秒通过
2. 同步事件——事件来自另一个任务或中断。例如，任务可能进入阻止状态，等待数据到达队列。同步事件涵盖了广泛的事件类型。

FreeRTOS队列、二进制信号量、计数信号量、互斥、递归互斥、事件组和直接到任务通知都可以用于创建同步事件。所有这些特点都将在这本书的未来章节中介绍。

任务可以用超时阻塞同步事件，有效地同时阻塞两种类型的事件。例如，一个任务可能会选择等待一个数据到达队列的最长时间为10毫秒。如果任何一个数据在10毫秒内到达，或者在10毫秒内通过而没有数据到达，则该任务将离开阻塞状态。

**挂起状态**

“挂起”也是不运行的一个子状态。处于“已挂起”状态下的任务对调度程序不可用。进入挂起状态的唯一方法是通过调用vTaskSuspend() API函数，唯一解除的方法是通过调用vTaskResume() 或xTaskResumeFromISR() API函数。大多数应用程序都不使用“已挂起”状态。

 **就绪状态**

处于“未运行”状态但未被阻止或挂起的任务称为处于“就绪”状态。它们能够运行，因此“准备好”运行，但目前没有处于运行状态。

**正在完成状态转换图**

图15扩展了前面的过度简化的状态图，以包括本节中描述的所有未运行的子状态。示例中创建的任务迄今尚未使用阻塞或挂起状态；它们只在就绪状态和运行状态之间转换——由图15中的粗体线突出显示。

![image-20220419144625947](C:\Users\86178\Desktop\RTOS\${pic}\image-20220419144625947.png)

**例4：使用阻止状态创建延迟**

到目前为止，在示例中创建的所有任务都是“周期性的”——它们延迟了一段时间，并打印出字符串，然后再次延迟，以此类推。延迟已经使用一个零循环非常粗略地生成——任务有效地轮询一个递增的循环计数器，直到它达到一个固定的值。示例3清楚地展示了该方法的缺点。较高优先级的任务在执行空循环时仍然处于运行状态，“抢占”任何处理时间的较低优先级的任务。

任何形式的轮询都有其他几个缺点，尤其是它的效率低下。在轮询期间，该任务确实没有任何工作要做，但它仍然使用最大的处理时间，因此浪费了处理器周期。示例4通过用对vTaskDelay()API函数的调用替换轮询空循环来纠正这种行为，它的原型如清单22所示。新的任务定义如清单23所示。

注意，只有在FreeRTOSConfig.h中将INCLUDE_vTaskDelay设置为1时，vTaskDelay()API函数才可用。

vTaskDelay() 将调用任务置于阻塞状态，以获得固定数量的滴答中断。任务处于“阻止”状态时不使用任何处理时间，因此该任务仅在实际工作要完成时使用处理时间。

**vTaskDelay() parameters**

| Parameter Name | Description                                                  |
| -------------- | ------------------------------------------------------------ |
| xTicksToDelay  | 在切换回已就绪状态之前，调用任务将处于“阻止”状态的中断数。例如，当滴答计数为10,000时，调用vTaskDelay（100）的函数，它将立即进入阻塞状态，并保持在阻塞状态，直到滴答计数达到10,100。宏pdMS_TO_TICKS()可用于将以毫秒为单位指定的时间转换为在刻度中指定的时间。例如，调用vTaskDelay(pdMS_TO_TICKS（100）)将导致调用任务保持在阻塞状态100毫秒。 |

```c
/**Listing 23. The source code for the example task after the null loop delay has been 
replaced by a call to vTaskDelay()**/
void vTaskFunction( void *pvParameters )
{
char *pcTaskName;
const TickType_t xDelay250ms = pdMS_TO_TICKS( 250 );
 /* The string to print out is passed in via the parameter. Cast this to a
 character pointer. */
 pcTaskName = ( char * ) pvParameters;
 /* As per most tasks, this task is implemented in an infinite loop. */
 for( ;; )
 {
 /* Print out the name of this task. */
 vPrintString( pcTaskName );
 /* Delay for a period. This time a call to vTaskDelay() is used which places
 the task into the Blocked state until the delay period has expired. The 
 parameter takes a time specified in ‘ticks’, and the pdMS_TO_TICKS() macro
 is used (where the xDelay250ms constant is declared) to convert 250 
 milliseconds into an equivalent time in ticks. */
 vTaskDelay( xDelay250ms );
 } 
}
```

尽管这两个任务仍然在不同的优先级上创建，但它们现在都将运行。如图16所示的示例4的输出确认了预期的行为。

![image-20220419145934993](C:\Users\86178\Desktop\RTOS\${pic}\image-20220419145934993.png)

图17中所示的执行序列解释了为什么这两个任务都会运行，即使它们是在不同的优先级下创建的。为了简单起见，我们省略了调度程序本身的执行。

空闲任务将在调度程序启动时自动创建，以确保始终至少有一个任务能够运行（至少有一个任务处于就绪状态）。第3.8节，空闲任务和空闲任务钩，更详细地描述了空闲任务。

![image-20220419150018053](C:\Users\86178\Desktop\RTOS\${pic}\image-20220419150018053.png)

只有这两个任务的实现发生了变化，而不是它们的功能。将图17和图12进行比较，可以清楚地表明，该功能正在以一种更有效的方式实现。

图12显示了任务使用空循环来创建延迟时的执行模式——因此总是能够运行，并因此在它们之间使用100%的可用处理器时间。图17显示了任务在整个延迟期间进入阻塞状态时的执行模式，因此只使用处理器时间，当它们实际有需要执行的工作（在这种情况下只是打印一条消息），因此只使用可用处理时间的一小部分。

在图17的场景中，每次任务离开阻塞状态时，它们在重新进入阻塞状态之前只执行少量的阻塞状态。大多数情况下，没有能够运行的应用程序任务（没有处于就绪状态的应用程序任务），因此，也没有可以选择来进入运行状态的应用程序任务。在这种情况下，空闲的任务将会运行。分配给空闲时间的处理时间是系统中备用处理能力的度量。仅仅通过允许应用程序完全被事件驱动，使用RTOS就可以显著增加备用处理容量。

图18中的粗体线显示了示例4中任务执行的转换，每个任务在返回到就绪状态之前通过阻塞状态转换。

![image-20220419151810284](C:\Users\86178\Desktop\RTOS\${pic}\image-20220419151810284.png)

 **vTaskDelayUntil() API函数**

vTaskDelayUntil() 类似于 vTaskDelay(). 正如刚才所展示的，vTaskDelay()参数指定了调用vTaskDelay()的任务与再次退出阻塞状态之间应该发生的滴答中断次数。任务保持在阻塞状态的时间长度由vTaskDelay()参数指定，但是任务离开阻塞状态的时间相对于调用vTaskDelay()的时间。

vTaskDelayUntil()指定调用任务从阻塞状态移动到就绪状态的确切勾选计数值。直到()API函数时应该使用一个固定的执行期间（你希望你的任务定期执行与固定频率），调用任务的时间是绝对的，而不是相对于函数被调用(与vTaskDelay())，因为调用任务被解除阻塞的时间是绝对的，而不是相对于调用函数的时间(就像vTaskDelay()的情况一样)。

```c
void vTaskDelayUntil( TickType_t * pxPreviousWakeTime, TickType_t xTimeIncrement );
```

Table 10.vTaskDelayUntil()参数

| Parameter Name     | Description                                                  |
| ------------------ | ------------------------------------------------------------ |
| pxPreviousWakeTime | 这个参数的命名是基于这样一个假设，vTaskDelayUntil()被用于执行一个以固定频率定期执行的任务。在这种情况下，pxPreviousWakeTime保持任务最后一次离开阻塞状态（被“唤醒”）的时间。此时间被用作参考点，以计算任务下一次离开阻塞状态的时间。vTaskDelayUntil()函数中自动更新；它通常不会被应用程序代码修改，但必须初始化为当前的滴答计数，然后第一次使用。Listing 25演示了如何执行初始化。 |
| xTimeIncrement     | 这个参数的命名也是基于以下假设，vTaskDelayUntil()正被用于执行一个定期且以固定频率执行的任务——该频率由x时间增量值设置。xTimeIncrement()用于指定滴答数，宏pdMS_TO_TICKS()可用于将以毫秒为单位指定的时间转换为在刻度中指定的时间 |

例程5：正在将示例任务转换为要使用的任务 vTaskDelayUntil()

示例4中创建的两个任务是周期性任务，但是使用vTaskDelay()并不能保证它们运行的频率是固定的，因为任务离开阻塞状态的时间相对于它们调用vTaskDelay()的时间。将任务转换为使用vTaskDelayUntil()而不是vTaskDelay()解决了这个潜在的问题。

```c
/**Listing 25. The implementation of the example task using vTaskDelayUntil()**/
void vTaskFunction( void *pvParameters )
{
char *pcTaskName;
TickType_t xLastWakeTime;
 /* The string to print out is passed in via the parameter. Cast this to a
 character pointer. */
 pcTaskName = ( char * ) pvParameters;
 /* The xLastWakeTime variable needs to be initialized with the current tick
 count. Note that this is the only time the variable is written to explicitly.
 After this xLastWakeTime is automatically updated within vTaskDelayUntil(). */
 xLastWakeTime = xTaskGetTickCount();
 /* As per most tasks, this task is implemented in an infinite loop. */
 for( ;; )
 {
 /* Print out the name of this task. */
 vPrintString( pcTaskName );
 /* This task should execute every 250 milliseconds exactly. As per
 the vTaskDelay() function, time is measured in ticks, and the
 pdMS_TO_TICKS() macro is used to convert milliseconds into ticks.
 xLastWakeTime is automatically updated within vTaskDelayUntil(), so is not
 explicitly updated by the task. */
 vTaskDelayUntil( &xLastWakeTime, pdMS_TO_TICKS( 250 ) );
 } }
```

示例5的输出与图16中示例4完全相同。

例6：结合阻塞和非阻塞任务

以前的例子已经单独检查了轮询和阻塞任务的行为。这个示例通过演示两种方案组合时的执行序列来加强预期系统行为，如下所示。

1. 在优先级为1的地方创建了两个任务。这些东西只是不断地打印出一个字符串。这些任务永远不会进行任何可能导致它们进入阻塞状态的API函数调用，因此总是处于准备状态或运行状态。这种性质的任务被称为“连续处理”任务，因为它们总是有工作要做（尽管在这种情况下是相当琐碎的工作）。连续处理任务的源代码如清单26所示。
2. 然后在优先级2处创建第三个任务，因此高于其他两个任务的优先级。第三个任务也只是打印出一个字符串，但这次是周期性的，所以使用vTaskDelayUntil() API函数将自己放置在每次打印迭代之间的阻止状态。定期任务的源代码如清单27所示。

```c
/**Listing 26. The continuous processing task used in Example 6**/
void vContinuousProcessingTask( void *pvParameters )
{
char *pcTaskName;
 /* The string to print out is passed in via the parameter. Cast this to a
 character pointer. */
 pcTaskName = ( char * ) pvParameters;
 /* As per most tasks, this task is implemented in an infinite loop. */
 for( ;; )
 {
 /* Print out the name of this task. This task just does this repeatedly
 without ever blocking or delaying. */
 vPrintString( pcTaskName );
 } }
/** Listing 27. The periodic task used in Example 6 **/
void vPeriodicTask( void *pvParameters )
{
TickType_t xLastWakeTime;
const TickType_t xDelay3ms = pdMS_TO_TICKS( 3 );
 /* The xLastWakeTime variable needs to be initialized with the current tick
 count. Note that this is the only time the variable is explicitly written to.
 After this xLastWakeTime is managed automatically by the vTaskDelayUntil()
 API function. */
 xLastWakeTime = xTaskGetTickCount();
 /* As per most tasks, this task is implemented in an infinite loop. */
 for( ;; )
 {
 /* Print out the name of this task. */
 vPrintString( "Periodic task is running\r\n" );
 /* The task should execute every 3 milliseconds exactly – see the
 declaration of xDelay3ms in this function. */
 vTaskDelayUntil( &xLastWakeTime, xDelay3ms );
 } }
```

图19显示了示例6所产生的输出，并解释了图20中所示的执行序列所给出的观察到的行为

![image-20220419161736832](C:\Users\86178\Desktop\RTOS\${pic}\image-20220419161736832.png)

![image-20220419161751231](C:\Users\86178\Desktop\RTOS\${pic}\image-20220419161751231.png)

#### 3.8 空闲任务和空闲任务回调

示例4中创建的任务的大部分时间都处于阻塞状态。在此状态下，它们无法运行，因此无法由调度程序选择。

必须始终有一个任务可以进入运行状态1。为了确保这种情况，当调用 vTaskStartScheduler() 时，调度程序会自动创建一个空闲任务。空闲的任务只需要设计在一个循环中——因此，就像最初的第一个示例中的任务一样，它总是能够运行。

空闲任务具有最低的优先级（优先级为零），以确保它永远不会阻止更高优先级的应用程序任务进入运行状态——尽管没有什么可以阻止应用程序设计人员在空闲任务优先级上创建任务，从而共享空闲任务优先级。FreeRTOSConfig.h中的configIDLE_SHOULD_YIELD编译时配置常数可用于防止空闲任务消耗处理时间，而这些处理时间将更有效地分配给应用程序任务。configIDLE_SHOULD_YIELD在第3.12节，调度算法中被描述。

以最低优先级运行可以确保一 旦更高优先级的任务进入已就绪状态，空闲任务就会从“正在运行”状态脱离。这可以在图17中的时间tn中看到，在那里空闲任务被立即交换出来，以允许任务2在即时任务2离开阻塞状态时执行。任务2被认为已经抢占了空闲的任务。抢占自动发生，并且不知道任务被抢占。

**Idle Task Hook Functions**

通过使用空闲钩子（或空闲回调）函数，可以将应用程序特定的功能直接添加到空闲任务中—该函数是空闲任务循环的每次迭代自动调用的函数。

空闲任务钩子的常用用途包括：

+ 执行低优先级、后台处理或连续处理功能。
+ 测量备用处理能力的量。（只有在所有高优先级应用程序任务没有工作可执行时，空闲任务才会运行；因此，测量分配给空闲任务的处理时间可以清楚显示空闲的处理时间。）
+ 将处理器置于低功率模式，在没有执行应用处理时提供简单和自动的省电方法（尽管使用此方法可以实现的省电小于使用第10章低功率支持中所述的无滴答空闲模式可以实现）。

**空闲任务钩功能实现的限制**

空闲任务钩子函数必须遵守以下规则

1.一个空闲的任务挂起功能永远不能尝试阻止或挂起。

注意：以任何方式阻止空闲任务都可能导致没有任务进入运行状态的场景。

2.如果应用程序使用了vTaskDelete() API函数，那么空闲任务钩子必须始终在合理的时间段内返回给其调用者。这是因为空闲任务负责在删除任务后清理内核资源。如果空闲任务永久保持在空闲钩子功能中，则无法进行此清理。

空闲任务钩子函数必须具有Listing28所示的名称和原型。

![image-20220419201805226](C:\Users\86178\Desktop\RTOS\${pic}\image-20220419201805226.png)

例7：定义空闲任务钩子功能

示例4中使用阻塞vTaskDelay()API调用在执行空闲任务时产生了大量的空闲时间，因为这两个应用程序任务都处于“阻塞”状态。 Example 7通过添加一个空闲钩子函数来利用这个空闲时间，其源代码如清单29所示。

![image-20220419201949797](C:\Users\86178\Desktop\RTOS\${pic}\image-20220419201949797.png)

configUSE_IDLE_HOOK必须在FreeRTOSConfig.h中设置为1，才能调用空闲钩子函数。

实现创建任务的函数将轻微修改，以打印出ulIdleCycleCount值，如listing30所示。



![image-20220419202443280](C:\Users\86178\Desktop\RTOS\${pic}\image-20220419202443280.png)

示例7所产生的输出如图21所示。它显示了空闲任务钩子函数在应用程序任务的每次迭代之间被调用了大约400万次（迭代的次数取决于执行演示的硬件的速度）。

![image-20220419202521219](C:\Users\86178\Desktop\RTOS\${pic}\image-20220419202521219.png)



#### 3.9更改任务的优先级

**vTaskPrioritySet()API函数**

 vTaskPrioritySet() API函数可以用于在调度程序启动后更改任何任务的优先级。注意，只有在FreeRTOSConfig.h中将INCLUDE_vTaskPrioritySet设置为1时，e vTaskPrioritySet()API函数才可用。

![image-20220419203012186](C:\Users\86178\Desktop\RTOS\${pic}\image-20220419203012186.png)

| **Parameter Name** | Description                                                  |
| ------------------ | ------------------------------------------------------------ |
| pxTask             | 其优先级正在被修改的任务的句柄（主题任务）—请参见x任务创建()API功能的px创建任务参数，以了解有关获取任务句柄的信息。一个任务可以通过传递NULL来代替一个有效的任务句柄来更改它自己的优先级。 |
| uxNewPriority      | 要设置的主题任务的优先级。这将被自动限制为(configMAX_PRIORITIES-1)的最大可用优先级，其中configMAX_PRIORITIES是在FreeRTOSConfig.h头文件中设置的编译时间常数。 |

**uxTaskPriorityGet () API Function**

uxTaskPriorityGe()API函数可用于查询任务的优先级。请注意，只有在FreeRTOSConfig.h中将INCLUDE_uxTaskPriorityGet设置为1时，INCLUDE_uxTaskPriorityGet()API函数才可用。

| **Parameter Name** | Description                                                  |
| ------------------ | ------------------------------------------------------------ |
| pxTask             | 其优先级正在被修改的任务的句柄（主题任务）—请参见x任务创建()API功能的px创建任务参数，以了解有关获取任务句柄的信息。一个任务可以通过传递NULL来代替一个有效的任务句柄来更改它自己的优先级。 |
| Returned value     | 当前分配给正在被查询的任务的优先级。                         |

**Example 8. 更换任务优先级**

调度程序将始终选择最高的就绪状态任务作为进入正在运行状态的任务。示例8通过使用vTaskPrioritySet()API函数来更改两个任务的优先级来演示这一点。

示例8以两个不同的优先级创建了两个任务。这两个任务都没有做出任何可能导致其进入阻塞状态的API函数调用，因此两者都始终处于“已就绪”状态或“正在运行”状态。因此，具有最高相对优先级的任务将始终是调度程序选择的处于运行状态的任务。

示例8的行为如下：

1。任务1（Listing33）是以最高优先级创建的，因此保证要先运行。在将Task2的优先级（Listing34）提高到其自身的优先级以上之前，任务1将打印出几个字符串。 2.任务2具有最高相对优先级，即开始运行（进入已运行状态）。任意一次只能有一个任务处于“运行”状态，因此当任务2处于“运行”状态时，任务1处于“就绪”状态。

 3.任务2在将自己的优先级设置为任务1的优先级以下之前打印出一条消息。

 4.任务2将其优先级降低意味着任务1再次成为最高优先级的任务，因此任务1重新进入“运行”状态，迫使任务2回到“就绪”状态。

```c
/**Listing 33. The implementation of Task 1 in Example 8**/
void vTask1( void *pvParameters )
{
UBaseType_t uxPriority;
 /* This task will always run before Task 2 as it is created with the higher 
 priority. Neither Task 1 nor Task 2 ever block so both will always be in 
 either the Running or the Ready state.
 Query the priority at which this task is running - passing in NULL means
 "return the calling task’s priority". */
 uxPriority = uxTaskPriorityGet( NULL );
 for( ;; )
 {
 /* Print out the name of this task. */
 vPrintString( "Task 1 is running\r\n" );
 /* Setting the Task 2 priority above the Task 1 priority will cause
 Task 2 to immediately start running (as then Task 2 will have the higher 
 priority of the two created tasks). Note the use of the handle to task
 2 (xTask2Handle) in the call to vTaskPrioritySet(). Listing 35 shows how
 the handle was obtained. */
 vPrintString( "About to raise the Task 2 priority\r\n" );
 vTaskPrioritySet( xTask2Handle, ( uxPriority + 1 ) );
 /* Task 1 will only run when it has a priority higher than Task 2.
 Therefore, for this task to reach this point, Task 2 must already have
 executed and set its priority back down to below the priority of this
 task. */
 }
}
```

```c
/* Listing 34. The implementation of Task 2 in Example 8  */
void vTask2( void *pvParameters )
{
UBaseType_t uxPriority;
 /* Task 1 will always run before this task as Task 1 is created with the
 higher priority. Neither Task 1 nor Task 2 ever block so will always be 
 in either the Running or the Ready state.
 Query the priority at which this task is running - passing in NULL means
 "return the calling task’s priority". */
 uxPriority = uxTaskPriorityGet( NULL );
 
 for( ;; )
 {
 /* For this task to reach this point Task 1 must have already run and
 set the priority of this task higher than its own.
 Print out the name of this task. */
 vPrintString( "Task 2 is running\r\n" );
 /* Set the priority of this task back down to its original value. 
 Passing in NULL as the task handle means "change the priority of the 
 calling task". Setting the priority below that of Task 1 will cause 
 Task 1 to immediately start running again – pre-empting this task. */
 vPrintString( "About to lower the Task 2 priority\r\n" );
 vTaskPrioritySet( NULL, ( uxPriority - 2 ) );
 } 
}
```

每个任务都可以查询和设置自己的优先级，而不使用有效的任务句柄，而简单地使用NULL。只有当任务希望引用除其自身以外的任务时，例如当任务1更改任务2的优先级时，才需要一个任务句柄。为了允许Task1执行这一点，将在创建Task2时获取并保存Task2句柄，如清单35中的注释中突出显示的那样。

```c
/*Listing 35. The implementation of main() for Example 8*/
/* Declare a variable that is used to hold the handle of Task 2. */
TaskHandle_t xTask2Handle = NULL;
int main( void )
{
 /* Create the first task at priority 2. The task parameter is not used 
 and set to NULL. The task handle is also not used so is also set to NULL. */
 xTaskCreate( vTask1, "Task 1", 1000, NULL, 2, NULL );
 /* The task is created at priority 2 ______^. */
 /* Create the second task at priority 1 - which is lower than the priority
 given to Task 1. Again the task parameter is not used so is set to NULL -
 BUT this time the task handle is required so the address of xTask2Handle
 is passed in the last parameter. */
 xTaskCreate( vTask2, "Task 2", 1000, NULL, 1, &xTask2Handle );
 /* The task handle is the last parameter _____^^^^^^^^^^^^^ */
 /* Start the scheduler so the tasks start executing. */
 vTaskStartScheduler(); 
 
 /* If all is well then main() will never reach here as the scheduler will 
 now be running the tasks. If main() does reach here then it is likely there
 was insufficient heap memory available for the idle task to be created. 
 Chapter 2 provides more information on heap memory management. */
 for( ;; );
}
```

图22演示了示例8任务执行的顺序，生成的输出如图23所示

![image-20220419212038992](C:\Users\86178\Desktop\RTOS\${pic}\image-20220419212038992.png)

![image-20220419220508674](C:\Users\86178\Desktop\RTOS\${pic}\image-20220419220508674.png)

#### 3.10 删除任务

**vTaskDelete()API功能**

任务可以使用vTaskDelete()API函数来删除自身或任何其他任务。注意，只有当vTeeRTOSConfig.h中INCLUDE_vTaskDelete设置为1时，vTaskDelete()API函数才可用。

已删除的任务不再存在，并且无法再次进入正在运行的状态。

空闲任务有责任释放分配给已被删除的任务的内存。因此，合理使用vTaskDelete()API函数的应用程序确保不会完全耗尽所有处理时间是很重要的。

注意：当删除任务时，只有内核本身分配给任务的内存将自动释放。必须显式地释放所分配的任务的实现的任何内存或其他资源。

![image-20220419221556305](C:\Users\86178\Desktop\RTOS\${pic}\image-20220419221556305.png)

**Table 13. vTaskDelete() parameters**

| Parameter Name/Return Value | Description                                                  |
| --------------------------- | ------------------------------------------------------------ |
| pxTaskToDelete              | 要删除的任务的句柄（主题任务)——请参见xTask创建()API函数的px创建任务参数，以了解有关获取任务句柄的信息。任务可以通过传递NULL来代替有效的任务句柄来删除自身。 |

