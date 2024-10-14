视频地址：https://www.bilibili.com/video/BV1r7411A7hq?p=2&vd_source=f93c2f87ac116d0ab5a7d34711f601c7

主讲人：张银奎

### X86调试措施

> INT 3指令，8086引入
>
> **软件断点的基础**`INT 3` 常常被用于在代码中插入断点

> 追踪标志（TF），8086引入
>
> **单步跟踪的基础**
>
> `TF` 是 x86 汇编语言中的标志寄存器（Flag Register）标志之一，代表"Trap Flag"。当 `TF` 被设置（设为1）时，它会产生一个单步中断，即每执行一条指令后都会触发中断。

> 调试寄存器（DR0-DR7），80386引入
>
> **硬件断点的基础，监视变量、IO访问，也可以针对代码**
>
> 调试寄存器（Debug Registers），通常表示为 `DR0` 到 `DR7`，是 x86 架构中的一组寄存器，用于支持硬件调试功能。这些寄存器提供了一种方式来设置断点、观察变量、跟踪指令执行等调试操作。
>
> 以下是关于调试寄存器的一些基本信息：
>
> 1. **DR0 到 DR3：** 这些寄存器用于设置断点。通过在这些寄存器中指定内存地址，可以在程序执行到指定地址时触发断点中断。
> 2. **DR6 寄存器：** 用于指示调试事件的状态。当发生调试事件时，`DR6` 的不同位会被置位，指示事件的类型。
> 3. **DR7 寄存器：** 控制调试寄存器的工作方式。通过设置 `DR7` 的不同位，可以启用或禁用断点，指定断点类型（执行、读取、写入）以及设置触发断点的条件。
>
> ==其实只有6个，没有DR4和DR5==

> 分支监视和记录，Pentium Pro引入
>
> **按分支单步的基础，记录软件的执行流程**





### 软件断点

> 背景知识：
>
> 在计算机体系结构中，代码段、数据段和 I/O（Input/Output）空间是内存的不同部分，用于存储程序的代码、数据以及与外部设备进行输入和输出的信息。以下是对这些概念的简要解释：
>
> 1. **代码段（Code Segment）：**
>    - **作用：** 用于存储程序的指令（代码）。
>    - **特点：** 代码段是只读的，通常包含程序的执行指令。程序在执行时，从代码段中取出指令并执行。修改代码段通常是不允许的，以确保程序的完整性和安全性。
>    - **寄存器：** 在 x86 架构中，CS 寄存器存储代码段的选择子。
> 2. **数据段（Data Segment）：**
>    - **作用：** 用于存储程序的数据。
>    - **特点：** 数据段包含程序运行时使用的变量和数据。与代码段不同，数据段是可读写的，允许程序在运行时修改其中的数据。数据段通常包括全局变量、静态变量和堆内存。
>    - **寄存器：** 在 x86 架构中，DS 寄存器存储数据段的选择子。
> 3. **I/O 空间（I/O Space）：**
>    - **作用：** 用于与外部设备进行输入和输出。
>    - **特点：** I/O 空间是为了处理外部设备的输入和输出而保留的内存区域。它提供了一种通过读写特定地址来与外部设备进行通信的机制。在计算机体系结构中，通常有专门的指令和指令集用于处理 I/O 操作。
>    - **寄存器：** 在一些体系结构中，可能有专门的 I/O 空间地址空间和相关的寄存器用于控制和通信。







> 核心：
>
> - INT 3指令
>
> - 机器码为1字节，即0xCC （中文——烫）CPU一执行到就会断
>
> - 没有数量限制
>
> - 局限性：
>
>   > 属于代码类断点，即可以让CPU执行到代码段内某个地址时停下来，不适用于数据段和I/O空间
>   >
>   > 对于在ROM（只读存储器）中执行的程序（比如BIOS或其他固件程序），无法动态增加软件断点。因为目标内存是只读的无法动态写入断点指令。这时就要用到==硬件断点==。
>
> 





> 知识点：
>
> 1、VS打断点就相当于在那条指令位置插入INT 3（替换一个字节） 
>
> 2、`x ntdll!*readfile*` 是 WinDbg 中的一个命令，用于搜索并显示符合指定模式的符号名称（Symbol Name）及其对应的地址。这个命令的具体含义如下：
>
> - `x`: 表示 "examine" 或 "查看" 的意思。
> - `ntdll!*readfile*`: 是一个通配符模式，用于匹配 `ntdll.dll` 中包含 "readfile" 字符串的所有符号名称。
>
> 解释一下每个部分：
>
> - `ntdll`: 表示目标 DLL 的名称，这里是 Windows 系统动态链接库 `ntdll.dll`。
> - `*readfile*`: 是通配符模式，其中 `*` 表示零个或多个字符的通配符，而 "readfile" 表示包含 "readfile" 字符串的符号名称。
>
> 因此，`x ntdll!*readfile*` 的作用是查找并显示在 `ntdll.dll` 中符号名称中包含 "readfile" 字符串的所有符号及其对应的地址。这对于定位与文件读取相关的函数或符号很有用。
>
> ntdll.dll是**重要的Windows NT内核级文件**。 描述了windows本地NTAPI的接口。 当Windows启动时，ntdll.dll就驻留在内存中特定的 写保护 区域，使别的程序无法占用这个内存区域。
>
> 请注意，使用通配符时要谨慎，确保匹配的模式足够明确，以避免匹配到不想查看的符号。
>
> 返回如下：
>
> ```cmd
> 这些数据显示的是 `ntdll.dll` 中与文件读取相关的一些函数的符号名称及其对应的地址。以下是对每个条目的简要解释：
> 
> 1. **`ntdll!LdrpResReadFile (_LdrpResReadFile@16)`**
>    - 符号名称：`LdrpResReadFile`
>    - 地址：`779a030a`
>    - 解释：这可能是 `ntdll.dll` 中的一个内部函数，与加载资源时的文件读取有关。
> 
> 2. **`ntdll!ZwReadFileScatter (_ZwReadFileScatter@36)`**
>    - 符号名称：`ZwReadFileScatter`
>    - 地址：`77951970`
>    - 解释：`ZwReadFileScatter` 是一个系统调用，用于支持异步文件读取（scatter/gather I/O）。在用户模式下，通常通过 `NtReadFileScatter` 调用。
> 
> 3. **`ntdll!ZwReadFile (_ZwReadFile@36)`**
>    - 符号名称：`ZwReadFile`
>    - 地址：`779516d0`
>    - 解释：`ZwReadFile` 是另一个与文件读取相关的系统调用，用于进行文件读取操作。
> 
> 4. **`ntdll!NtReadFileScatter (_NtReadFileScatter@36)`**
>    - 符号名称：`NtReadFileScatter`
>    - 地址：`77951970`
>    - 解释：`NtReadFileScatter` 是用户模式中的函数，可能通过 `ZwReadFileScatter` 调用相应的内核模式函数。
> 
> 5. **`ntdll!NtReadFile (_NtReadFile@36)`**
>    - 符号名称：`NtReadFile`
>    - 地址：`779516d0`
>    - 解释：`NtReadFile` 是用户模式中的函数，用于进行文件读取操作。通常通过 `ZwReadFile` 调用相应的内核模式函数。
> 
> 这些函数是 Windows 操作系统核心库 `ntdll.dll` 中的一部分，用于处理文件读取的底层操作。函数名中的 `Zw` 和 `Nt` 前缀表示这些函数分别是内核模式系统调用和用户模式库函数的接口。
> ```
>
> 
>
> 
>
> 3、u ntdll!NtReadFile
>
> `u ntdll!NtReadFile` 是在 WinDbg 中用于反汇编（disassemble）指定函数的命令。具体含义如下：
>
> - `u`: 表示 "unassemble" 或 "反汇编" 的意思。
> - `ntdll!NtReadFile`: 是指定要反汇编的函数，这里是 `ntdll.dll` 中的 `NtReadFile`。
>
> 这个命令将会显示 `NtReadFile` 函数的反汇编代码，包括其指令和地址。这对于查看函数的底层实现以及理解其工作原理非常有用。反汇编的结果会显示在输出窗口中，你可以看到该函数的汇编指令。
>
> 请注意，WinDbg 中的反汇编是对二进制代码的近似表示，因此阅读汇编代码需要一定的汇编语言知识。这样的操作通常用于调试、分析代码执行路径或查找潜在问题。
>
> 
>
> 
>
> 4、==bp命令==
>
> `bp 779516e0` 是在 WinDbg 中设置断点的命令。具体含义如下：
>
> - `bp`: 是 "breakpoint" 的缩写，表示设置断点的命令。
>
> - `779516e0`: 是要设置断点的内存地址，这是一个十六进制的地址。在这里，`779516e0` 是一个表示内存位置的十六进制地址。
>
>   
>
> 5、==bl命令==
>
> 通常用于bp命令之后，用于列出已设置的断点（breakpoints）。具体含义如下：
>
> - `bl`: 是 "breakpoint list" 的缩写，表示列出断点列表的命令。
>
> 该命令在执行后会显示当前会话中所有已设置的断点及其相关信息，例如断点的地址、断点类型等。这有助于调试者了解哪些地方设置了断点，以及这些断点的详细信息。
>
> 6、调试过程中不能看到INT 3指令
>
> `INT 3` 是一条软中断指令，通常用于调试目的。当执行到这个指令时，会触发一个中断，调试器会捕获这个中断并中断程序的执行，以便进行调试操作。由于它的特殊性，它可能不会直接出现在源代码或反汇编中，而是由调试器处理。
>
> 当不调试或者break时，就会恢复成原样（删去INT 3）
>
> 
>
> 7、Non-invasive attach
>
> 不可以两个调试器attach到同一个进程
>
> 但是Non-invasive attach是不入侵形式进行调试，其实就是只读
>
> 这样就可以看到INT 3断点指令了。





### 硬件断点 陷阱和JTAG

> 硬件断点的基础是调试寄存器
>
> 上面软件断点笔记写的有问题，张老师说确实是8个调试寄存器
>
> ![image-20240124190045892](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20240124190045892.png)



> 8个调试寄存器：
>
> DR0-DR3：
>
> 4个，各放一个线性地址
>
> DR7：
>
> 设定标志位，四个断点对应上方四个标志位
>
> DR6：
>
> 报告断点状态，CPU每执行一条，设置标志位
>
> DR4和DR5 没用！！！
>
> 理论上来说，每个CPU最多可监视四个地址（软件层可以理解为每个线程最多可以设4个地址



> 硬件断点：
>
> - 基于CPU的调试寄存器
> - 可以对代码、数据访问和IO访问设置断点
> - 断点被触发时，CPU产生的是1号异常
> - 受调试寄存器的数量限制
> - WinDbg的==ba命令==设置的便是硬件断点
> - 在多处理器系统中，硬件断点是与CPU相关的，也就是说针对一个CPU设置的硬件断点并适用于其它CPU
>
> 
>
> 
>
> 硬件断点好处：
>
> 不用在软件上加指令
>
> 
>
> 陷阱标志：
>
> 单步执行标志（TF）
>
> 位于FLAGS寄存器中（8位中最后一个）
>
> 打断点其实就是TF位从0置1，CPU每执行一条指令就检查这个标志位，看到置1了就触发一个异常，然后程序就自然断下来了，当然会又把TF位置回0。
>
> 如果一直置1的问题就在于，CPU看到置1了就执行异常的代码，处理异常的代码是不希望被断下来的
>
> 这其实我联想到C#里的try catch 实际应该也是这么执行的
>
> 
>
> ==断点其实也是一种异常！！==



> X86中的异常：
>
> ![image-20240124191428391](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20240124191428391.png)

> 更多信息：
>
> ==IA-32手册==
>
> 需要啃！！！
>
> http://www.intel.com/products/processor/manuals/index.htm





### windows用户态调试原理

> ![image-20240124192859853](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20240124192859853.png)

> 用户态一旦调试程序了，那个程序就变成freeze状态不能动了



> 知识点：
>
> 1、~*
>
> 2、~0 k
>
> 3、调试时调试器程序会创建一个调试线程，把被调试程序全都freeze掉
>
> 4、syscall指令——从用户态到内核态





### Linux应用程序调试

> Process Trace ——Ptrace api
>
> ![image-20240124195142331](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20240124195142331.png)

> linux有很强的父进程的概念
>
> 每个进程都必须有一个父进程。这个规则有助于构建进程之间的层次结构，形成进程树。
>
> 父进程可以访问子进程的信息。
>
> 
>
> "养父进程"（Adopted Parent Process）通常指的是一个进程被操作系统的一个特殊进程接管，成为这个特殊进程的子进程。
>
> 即调试进程
>
> ![image-20240124195420875](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20240124195420875.png)

> ![image-20240124195437563](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20240124195437563.png)







### Windows操作系统的异常分发操作

> 异常大致有：
>
> - Win32异常
> - 包括CPU异常及windows操作系统所定义的异常
>
> CLR异常
>
> - CLR及.NET程序定义的异常
> - 异常代码为e0434f4d 即.COM
>
> C++异常
>
> - 代码为0xe06d73即.msc



> 知识：
>
> 1、KiDispatchException：
>
> windows分发异常的一个枢纽（核心方法）
>
> windows默认异常是两轮分发
>
> 第一轮先给调试器分发用户态异常，调试器异常复制到用户态（try...catch)
>
> 若第一轮不处理，第二轮也是先给调试器，告诉是最后的处理机会，会杀死进程
>
> ![image-20240127144718353](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20240127144718353.png)
>
> ![image-20240127144807353](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20240127144807353.png)
>
> 2、结构化异常处理SEH
>
> 1. **用户模式异常处理（第一轮）：** 当应用程序发生异常时，首先在用户模式下进行异常处理。这通常是通过操作系统提供的结构化异常处理（Structured Exception Handling，SEH）来完成的。SEH允许应用程序编写自定义的异常处理程序，以捕获并处理特定类型的异常。
> 2. **内核模式异常处理（第二轮）：** 如果在用户模式下无法处理异常（例如，异常太严重，无法在应用程序级别解决），控制权就会转移到操作系统内核的异常处理程序。在内核模式下，操作系统可以采取必要的措施来处理异常，例如终止进程、记录日志、恢复系统状态等。
>
> ![image-20240127145209078](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20240127145209078.png)
>
> 3、==指令== dd 地址
>
> 跳到指定地址的栈上 
>
> 4、如何快速定位核心异常代码
>
> 异常代码结构体exception record有特定异常码
>
> ![image-20240127150353628](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20240127150353628.png)
>
> "c0000094" 是一个 Windows 操作系统异常代码（Exception Code），它表示异常的具体类型。在 Windows 中，异常代码通常以十六进制数表示，前缀 "0x" 表示这是一个十六进制数。
>
> 异常代码 "c0000094" 对应的含义是 STATUS_INTEGER_DIVIDE_BY_ZERO，表示发生了整数除零异常。
>
> ```C++
> typedef struct _EXCEPTION_RECORD {
>     DWORD    ExceptionCode;
>     DWORD    ExceptionFlags;
>     struct _EXCEPTION_RECORD *ExceptionRecord;
>     PVOID    ExceptionAddress;
>     DWORD    NumberParameters;
>     ULONG_PTR ExceptionInformation[EXCEPTION_MAXIMUM_PARAMETERS];
> } EXCEPTION_RECORD, *PEXCEPTION_RECORD;
> //
> ```
>
> 
>
> 5、线程上下文结构体地址
>
> ![image-20240127150710676](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20240127150710676.png)
>
> 识别寄存器上下文 可以看1007f
>
> ```C++
> typedef struct _THREAD_CONTEXT {
>     DWORD ContextFlags;             // 上下文标志，指示哪些寄存器被保存
>     DWORD Dr0;
>     DWORD Dr1;
>     DWORD Dr2;
>     DWORD Dr3;
>     DWORD Dr6;
>     DWORD Dr7;
>     CONTEXT_FLOATING_POINT FloatSave;
>     DWORD SegGs;
>     DWORD SegFs;
>     DWORD SegEs;
>     DWORD SegDs;
>     DWORD Edi;
>     DWORD Esi;
>     DWORD Ebx;
>     DWORD Edx;
>     DWORD Ecx;
>     DWORD Eax;
>     DWORD Ebp;
>     DWORD Eip;                      // 程序计数器
>     DWORD SegCs;
>     DWORD EFlags;                   // 标志寄存器
>     DWORD Esp;                      // 栈指针
>     DWORD SegSs;
>     BYTE  ExtendedRegisters[MAXIMUM_SUPPORTED_EXTENSION]; // 扩展寄存器信息
> } THREAD_CONTEXT, *PTHREAD_CONTEXT;
> ```
>
> ![image-20240127151325709](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20240127151325709.png)





### 编译器的调试过程——调试符号！

> ![image-20240127151443977](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20240127151443977.png)



#### 调试符号

> - 编译过程的副产品
> - 衔接二进制程序与源程序的桥梁
> - 对调试有着重要意义
> - 源代码级调试必须
> - 二进制跟踪时的灯塔

> ![image-20240127151723250](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20240127151723250.png)
>
> windows下调试全部都是pdb文件
>
> ![image-20240127151758382](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20240127151758382.png)



> PDB文件布局：
>
> ![image-20240127152028397](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20240127152028397.png)



> 知识：
>
> 1、peview程序
>
> 是看windows下可执行文件 格式
>
> 2、pdb的大小往往远远大于可执行程序的大小
>
> 因为pdb里记录了更多信息
>
> 3、符号服务器
>
> 符号服务器（Symbol Server）是一个用于存储和管理程序调试符号（Debug Symbols）的中心化服务。在软件开发和调试过程中，调试符号是与程序二进制文件关联的元数据，包含了源代码的结构信息、变量名、函数名等调试信息。
>
> 符号服务器的主要作用是帮助开发人员在调试时将程序的崩溃堆栈信息映射回源代码的相应行号和函数名，从而更轻松地定位和解决问题。
>
> 以下是符号服务器的一些主要功能和优势：
>
> 1. **符号存储：** 符号服务器存储了大量的调试符号，开发人员可以将编译后的调试符号上传到服务器上进行集中管理。
> 2. **符号检索：** 开发人员可以通过符号服务器轻松地检索和获取特定版本程序的调试符号，以便在调试时使用。
> 3. **符号映射：** 当程序发生崩溃或异常时，调试器会尝试从符号服务器下载相应的调试符号，并将崩溃堆栈信息映射回源代码的行号和函数名，从而帮助开发人员定位问题。
> 4. **版本控制：** 符号服务器通常与版本控制系统集成，开发人员可以根据需要上传、管理和检索特定版本的调试符号。
> 5. **减小程序大小：** 在发布时，可以选择不包含调试符号，从而减小程序的体积，但在调试时仍然可以通过符号服务器获取相应的调试信息。
>
> ==学习windbg的关键==
>
> 
>
> ![image-20240127153004197](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20240127153004197.png)
>
> 
>
> 6、linux下调试是DWARF 内嵌在ELF文件中





### 调试器！（核心）

> ![image-20240127153446541](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20240127153446541.png)



> 知识：
>
> 1、Dbgk*
>
> 位于内核中，但是是支持用户态调试的函数
>
> 2、Kd*
>
> 内核调试的相关函数
>
> 3、EProcess
>
> windows内核态 管理进程 最重要的是EProcess结构体
>
> 观察它可以得到很多有用信息
>
> 在 Windows 内核中，每个进程都有一个对应的 `EPROCESS` 结构。这个结构包含了大量的信息，用于描述和控制进程的状态、资源分配、安全性等方面。一些常见的 `EPROCESS` 结构包含的信息有：
>
> - 进程 ID（PID）
> - 父进程指针
> - 进程状态信息（例如，运行、挂起、终止等）
> - 进程优先级
> - 进程创建时间
> - 进程的安全上下文
> - 进程的地址空间描述符等等
>
> `EPROCESS` 结构是 Windows 内核中的一个复杂的数据结构，它包含了许多字段和指针，用于表示和管理进程的各个方面。通过操作 `EPROCESS` 结构，Windows 内核可以进行进程的创建、调度、销毁、资源分配等操作，从而保证系统的稳定性和安全性。
>
> ![image-20240127154038298](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20240127154038298.png)
>
> 4、调试器是面向事件的
>
> ![image-20240127154357812](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20240127154357812.png)







### Windbg学习和使用

> 优势：GUI界面
>
> 用户态 内核态都能调试
>
> 用户态：
>
> 本机调试（Native Debuging）——C语言这种，源代码级别的
>
> 管理调试（Managed Debuging）——C#\VB这种，托管类代码

> ![image-20240128085506679](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20240128085506679.png)
>
> 三种命令
>
> - 常规命令（最常用）
> - 源命令
> - 扩展命令
>
> ![image-20240128085611877](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20240128085611877.png)

> ![image-20240128085647299](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20240128085647299.png)
>
> ![image-20240128085735051](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20240128085735051.png)