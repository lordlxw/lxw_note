# Win32API

## 第一章、窗口管理和图形设备接口

### 1.1、窗口管理

> **==窗口分为：==**
>
> 1、桌面窗口：大白话就是屏保桌面，本质是一张位图
>
> 2、应用程序窗口：由标题栏、菜单栏、System菜单、最小化（最大化）、改变边框、客户区、水平/垂直滚动条等组成。也是多张位图组成
>
> ![image-20230129143840744](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20230129143840744.png)
>
> 3、控制框、对话框以及消息框
>
> * 控制框：类似于打开文件选择
> * 对话框：一个或含有多个控制框的窗口
> * 消息框：给用户一些提示或警告的窗口



> 窗口涉及的属性有：
>
> * 窗口类
> * 窗口名
> * 窗口风格
> * 父窗口或属主窗口
> * 尺寸
> * 位置
> * 定位
> * 子窗口标识或菜单句柄
> * 实例句柄
> * 创建数据
>
> > 每个Windows 应用程序都得用 WinMain作为入口,函数WinMain 完成系列工作包括注册主窗口的窗口类并创建主窗口。WinMain 调用函数 ReisterClass,注册主窗口类，函数 CreateWindowEx 创建主窗口，用ShowWindow来显示窗口
>
> 窗口句柄：
>
> 创建了窗口之后,创建函数返回唯一标识窗口的窗口句柄,应用程序在其它函数中用这个句柄以确保是对该窗口的操作。窗口句柄属于 HWND 数据类型;应用程序必须在说明一个窗口句柄的变量时使用这种类型。
>
> 子窗口至少有一个父窗口，最大的父窗口就是桌面。
>
> 子窗口只能有一个父窗口，但一个父窗口可以有多个子窗口。



