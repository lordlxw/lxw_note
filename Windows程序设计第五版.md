# Windows程序设计第五版

## 一、简介

> ## Windows自身基本上就是一个动态链接库的集合。
>
> 1、Windows的三个动态连结程式库，代表了Windows三个主要子系统，被称为Kernel、User和GDI
>
> Windows包括一个机制，能够在执行时连结使用动态连结程式库中常式的程式。

Kernel处理所有在传统上由作业系统核心处理的事务——记忆体管理、档案I/O和多任务管理

User指用户界面，实现所有视窗运作机制

GDI是一个图形装置界面，允许程序在荧幕上显示文字和图形



**DLL——Dynamic Linking Library**

```C#
   public void Set_balloon(ToolTip toolTip, int i, string strCoilInfo,bool flag_tooltip)
        {
            if (this.InvokeRequired)
            {
                SetBalloonCallback d = new SetBalloonCallback(Set_balloon);
                this.Invoke(d, new object[] { toolTip, i, strCoilInfo,flag_tooltip });
            }
            else
            {
                if (flag_tooltip)
                {
                    return;
                }
                else
                {

                    toolTip.SetToolTip(positionTextBox[i], strCoilInfo);
                    SetBalloonStyle(toolTip);
                    SetBackColor(toolTip, Color.Cyan);
                }
            }
        }


        delegate void SetBalloonCallback(ToolTip toolTip, int i, string strCoilInfo, bool flag_tooltip);
```

 



## 二、小段

### 1、字符集编码

> 大致分为ASCII码（00-7F，用一个字节表示）和GBK（两个字节拼凑组成汉字）
>
> Unicode编码——是一种编码格式，范围0-0x10FFFF,容纳100多万个编码
>
> UTF-16（两个字节存储），就是两个字节对齐，不够高位补0
>
> UTF-8（一个字节存储），即可变长度字符编码，如果你文档全是英文字母（一个字节表示），网络传速度快。

### 2、C语言中的宽字符

> char ——Ascii编码
>
> wchar_t ——Unicode编码格式
>
> 两者区别：
>
> char str[]="中国";——实际占五个字节两个中文字（四字节）+结尾一字节
>
> wchar_t str[]=L"中国"; ——L代表是以UTF-16形式存储，总共六个字节，结尾两个字节结尾
>
> 这两者由于编码格式不同，存在内存里的值也完全不同，所以对应程序也不同

> 函数对应关系：
>
> char：					printf——strlen——strcpy——strcat
>
> wchar_t_str——wprintf——wcslen——wcscpy——wcscat
>
> 
>
> 计算char的strlen时，遇到00结束，长度不包括00
>
> 计算w_char(宽字符)的strlen时，遇到00 00 结束，长度不包含00 00 

> 所以unicode编码格式就是宽字符

```C
	char str1[]="中国";
	wchar_t str2[]=L"中国";
	printf("%d",strlen(str1));
	printf("%d",wcslen(str2));

//打印输出结果为strlen1=4
//strlen2=2
```

### 3、什么是Win32 Api

> 主要是存放在C:\\WINDOWS\system32下的dll
>
> 注意：system32文件夹下的是64位的
>
> ​			wow64里面的才是32位的
>
> 几个重要的dll：
>
> :one:Kernel32.dll：最核心的功能模块，比如管理内存，进程和线程相关的函数
>
> :two:User32.dll：Windows用户界面相关应用程序接口，如创建窗口和发送信息等
>
> :three:GDI32.dll：图形设备接口，包含画图和显示文本的函数

C中只要包含#include<windows.h>即可使用上述api

两种不同的编码方式，所以win32Api提供的函数都是两份

> 不要被win32里的类型吓到：就是起别名
>
> LPCTSTR->LPCSTR->const char*
>
> Uint:unsigned int
>
> 对应关系如下：
>
> | Win32                       | C         |
> | --------------------------- | --------- |
> | CHAR                        | char      |
> | WCHAR                       | wchar_t   |
> | TCHAR(宏，自动转换编码格式) | char      |
> | PSTR                        | char *    |
> | PWSTR                       | wchar_t * |
> | PTSTR(宏，自动转换编码格式) | char *    |
> |                             |           |
> |                             |           |
>





### 4、 进程

| 分区                   | x86 32位Windows        |
| ---------------------- | ---------------------- |
| 空指针赋值区（64KB）   | 0x00000000——0X0000FFFF |
| 用户模式区             | 0x00010000——0X7FFEFFFF |
| 64KB禁入区             | 0X7FFF0000——0X7FFFFFFF |
| 内核（所有进程都一样） | 0X80000000——0XFFFFFFFF |

 

> 进程创建过程
>
> 1、双击.exe，调用createProcess函数
>
> 2、创建内核对象EPROCESS
>
> 3、映射系统DLL（ntdll.dll）
>
> 4、创建线程内核对象（ETHREAD)
>
> 5、系统启动该线程，映射DLL（ntdll.LdrInitializeThunk)，线程开始执行
>
> 注：EPROCESS、ETHREAD都在内核那2G公共空间里。



> ##### 启动一个进程时，会传入一个StartupINFO的结构体（里面记录了进程的信息），如果没有父进程，则父进程是explorer.exe
>
> 



### 5、句柄

> 内核对象：像进程、线程、文件、互斥体、事件等在内核都有一个相应的结构体，这些结构体由内核负责管理。

![image-20220817113430376](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20220817113430376.png)

> 句柄就是用来找相应的内核对象
>
> ![image-20220817114127453](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20220817114127453.png)
>
> 换句话说，有了句柄，才能够操作内核对象。多进程共享一个内核对象。
>
> CloseHandle——释放句柄，即句柄访问数减一
>
> 句柄是否允许被继承（置1），子进程将父进程标志位为1的句柄继承下来
