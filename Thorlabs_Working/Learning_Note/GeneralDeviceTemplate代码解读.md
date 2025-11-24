# WPF部分

## 前人经验

> Clone最新代码，根据GUI设计确定项目类型，Material-UnComplexProjectCode；Fluent-ComplexProjectCode，执行相应文件夹中的bat文件拷贝文件。
> 设备名称替换，以设备名称为NavitarZoomLenses为例：
> 工具：Notepad++（替换文件内容），批量重命名（替换文件名）
> Notepad++：Filters为*.*  Thorlabs.Device替换为Thorlabs.NavitarZoomLenses
> Notepad++：Filters为*.*  TempDevice替换为NavitarZoomLenses
> Notepad++：Filters为*.sln*;*.csproj*  Device替换为NavitarZoomLenses
> 批量重命名：TempDevice替换为NavitarZoomLenses
> 工程文件及文件夹名替换：手动将3个文件夹和sln及 csproj文件名中的Device替换为NavitarZoomLenses
> Plugin.xml、App.config内容替换： DeviceName、Device、 TEMPDEVICE、Demo Device替换为NavitarZoomLenses
> 项目配置：纯C#项目，保留AnyCPU即可。包含C++的项目，删除AnyCPU，确认项目是32位还是64位，或者两者皆要。若只要其中一种的，删除所有项目中的另一种平台。若在vs中无法添加某种平台，显示相同平台名称已存在等错误，可直接用txt打开csproj文件，在<Platforms>x86</Platforms>中直接写所需平台。注：需确保每个solution的platform和每个项目的platform匹配



## 核心页面

> 1、SelectDeviceWindow——选择设备窗口
>
> 2、



## 核心服务



### ConnectService

> 主窗口订阅连接事件：
>
> ea.GetEvent<ConnectFinishEvent>().Subscribe(ConnectFinishEventExecute, true);
>
> **连接思路：**
>
> 用户点击连接按钮
>     ↓
> MainWindowViewModel.ConnectionCommandExecute()
>     ↓
> ConnectService.ConnectDevice(deviceDetail)
>     ↓
> deviceDetail.GetEntry().GenerateDevicePlugin()
>     ↓
> plugin.Connect()  // 调用CSL1CommandProvider的方法
>     ↓
> ConnectService发布ConnectFinishEvent
>     ↓
> MainWindowViewModel和MainContentViewModel收到事件
>     ↓
> 执行各自的ConnectFinishEventExecute方法



#### 

#### 底层通信：CSL1CommandProvider



### 设备发现

#### 1、Ethernet设备

> 通过 CSL1EthernetList() 方法扫描网络
>
> 支持==动态添加==新设备



#### 2、Port设备

> 通过 CSL1PortList() 方法扫描串口
>
> USB/Serial
>
> 序列号：M009
>
> 波特率：115200



### 多语言切换

> LocalizationManager (单例管理器)（管理多个 本地化语言，比如exe/module）
>     ↓ 管理
> ILocalizationService (接口)
>     ↓ 实现
> LocalizationServiceTemplate<T> (模板基类)
>     ↓ 继承
> LocalizationService (Shell项目实现)
>
> ShellLocalizationKey





### 状态码

> - 正数 = 成功（通常表示设备数量、字节数、句柄等）
>
> - 0 = 成功但无结果（如没有找到设备）
>
> - 负数 = 错误代码
>
>   



### 页面的注入

> 服务继承接口并实现（以太网/串口）
>
> 服务注册页面和服务的关系。RegisterViewWithRegion
>
> （regionName，view）的映射



## 控件库

> TelerikUI
>
> * RadChartView
>
>
> SciChart是什么
>
> 由SciChart移植到了TelerikUI（便宜一些）







## 疑问

> 1、为什么调试Shell 可以进到 Plugin里
>
> 明明都依赖的Shared
>
> 2、连接方法是哪里Publish的，pub/sub只看到了订阅。是C++pub的吗？
>
> 3、图像用的是什么控件，第三方还是？
>
> 4、状态码-3代表什么
>
> 5、C++的代码有吗
>
> 6、保存的语言/样式是怎么持久化的
>
> 7、单设备 / 多设备  区别在哪





## 需要做的事

>1、看post
>
>2、写post
>
>3、看代码
>
>4、优化代码
>
>5、看官方设备文档
>
>6、Telerik控件库学习
>
>*7、额外看别的代码



## 德政指点

> 1、多语言是怎么实现的
>
> 2、maincontent 有线程池，为什么这么做，Invoke加lambda表达式 （队列）好处是什么
>
> 3、exe是引用的shared，shared里定义接口会找到plugin？createplugin？getmainview
>
> 4、主题切换看一下
>
> 5、合并squash merge



## 学到的部分

> 1、针对SciChart（需要许可证），不能直接调试，先运行，然后在attach上去。



# C++部分

> uart_library.h







# CSL1

## 设备本身

> ###  LED状态总结
>
> | LED颜色 | 状态含义     | 光源状态 | 操作建议     |
> | :------ | :----------- | :------- | :----------- |
> | 🔵 蓝色  | 设备已通电   | 光源关闭 | 可以开启光源 |
> | 绿色    | 设备正常工作 | 光源开启 | 光源正在运行 |
> | ❌ 无光  | 设备未通电   | 无状态   | 检查电源连接 |
>
> ### 💡 使用提示
>
> 1. 首次使用：连接USB后看到蓝色LED，说明设备正常通电
>
> 1. 开启光源：按下Power按钮或通过软件控制，LED会从蓝色变为绿色
>
> 1. 安全操作：绿色LED表示光源正在工作，注意不要直视光源
>
> 1. 故障判断：如果连接后没有LED亮起，检查USB连接和电源供应



## 服务

> 更新服务 
>
> 定时轮询服务
>
> 多语言服务 —— 了解程度：4
>
> LocalizationService 和 PluginLocalizationService 分别 注入泛型的key，
>
> Template优势：
>
> - 扩展性：加枚举类 + 加语言文件 = 新功能的多语言支持
> - 复用性：所有服务共享相同的底层逻辑
> - 统一性：一个管理器控制所有本地化服务
> - 维护性：修改文本不需要重新编译
> -  类型安全：编译时检查本地化键
>
> 主题切换服务 —— 了解程度：4.5
>
> 为什么要继承自DependencyObject？
>
> 因为控件也要本地化。
>
> SetKind - 设置本地化键 SetPath - 设置属性路径



> 持久化保存在 userConfig文件夹。
>
> MaterialPalette.Palette 通过这个来修改Telerik控件的主题（亮还是暗）

### ==DataService==

> 抽象基类DataService
>
> 一个串行队列ConcurrentQueue，保证线程安全。（不需要自己加 lock）`ConcurrentQueue` 保证的是 **入队顺序**（FIFO）
>
> 自己实现的Invoke和InvokeAsync（同步和异步）方法，入队，执行委托
>
> QueueThread 的作用和 在连接/断开时启动/停止。	
>
> QueueExecution 就是一个死循环线程，它：
>
> - 持续运行：只要设备连接着，线程就一直运行
> - 智能休眠：没有命令时自动休眠，节省资源
> - 即时响应：有命令时立即唤醒处理
> - 串行执行：按顺序执行所有委托，确保设备安全
>
> Get 方法：需要返回值，使用**同步**调用
>
> - 立即获得结果
>
> - 支持流程控制
>
> - 便于错误处理
>
> Set 方法：不需要返回值，使用**异步**调用
>
> - 不阻塞UI
>
> - 快速响应
>
> - 后台执行

## 窗口

> 1、选项窗口
>
> 功能：改 主题 和 语言
>
> 2、帮助窗口
>
> 功能：
>
> 3、选择设备窗口
>
> 4、选择单一设备窗口
>
> 5、支持窗口
>
> 6、更新窗口
>
> 7、==主窗口==




## PluginInterfaces

>DevicePlugin&DeviceSDK&DeviceAction
>
>DevicePlugin (设备插件)
>    ↓ 包含
>DeviceSDK (设备SDK)
>    ↓ 管理
>DeviceAction (设备操作)
>
>Plugin的View是按需加载的，只有设备连接成功了，才会显示页面。
>
>```C#
>if (state == ConnectState.connect)
>{
>    this.ToolContent.Content = new RibbonContent();
>    this.MainContent.Content = device.Plugin.GetPluginUI().GetMainView();
>    connectButton.LargeImage = (ImageSource)Resources["Link_Broken_OutlineDrawingImage"];
>    RibbonButtonHelper.SetAttachedSmallImage(connectButton, (ImageSource)Resources["Link_Broken_OutlineDrawingImage"]);
>    connectButton.Text = localizationService.GetLocalizationString(Localization.ShellLocalizationKey.DisconnectRibbonButton);
>    ThorlabsProduct.SerialNo = device.SerialNumber;
>    Header= ThorlabsProduct.ProductShortDisplayName + " - " + ThorlabsProduct.SerialNo;
>}
>```
>
>



# 准备个PPT

> 1、复杂设备 和  简单设备
>
> 2、多设备
>
> 3、以太网
>
> 4、Plugin如何和关联
>
> 5、多语言，主题切换，保存在哪（user.config (clickOnce)?)
>
> 6、DataService池的作用 Func委托
>
> ---
>
> CSL1
>
> 6、CSL1设备是干什么的
>
> 7、优化点在哪
>
>
> PPT大纲
>
> 1、GeneralDeviceTemplate 模板介绍
>
> 如何构建（简单设备 和 复杂设备）
>
> Plugin和Exe分离
>
> 导航逻辑
>
> 2、CSL1介绍
>
> 设备功能
>
> 图像图表（Telerik 和 SciChart）



# 问题

> 1、Disconnect(); 为什么要新起一个线程？
>
> 直接调用Disconnect()会发生什么？可能发生==死锁==
>
> ```场景
> // ❌ 直接调用（死锁）：
> 轮询线程 → 获取锁 → 异常 → Disconnect() → Stop() → Wait() → 死锁
> 
> // ✅ Task.Run（正常）：
> 轮询线程 → 获取锁 → 异常 → 新起Task → 继续执行 → 释放锁 → 新线程执行Disconnect
> ```
>
> Task.Run *避免了在轮询线程中停止轮询服务*
>
> *轮询线程在调用 Stop()*
>
> pollingTask.Wait() 会阻塞在那里是因为死锁
>
> PollingService 的 Pause、Resume 和 Stop（完全停止）
>
> 设备断开---》Stop
>
> 设备连接---》PollingService轮询开始
>
> 轮询线程不能调用 Stop() 方法来停止自己，所以需要在新线程中执行。
>
> 2、人工操作 和 设备上操作 Polling Pulling（1S）？冲突 这时候是怎么解决的，如何保证数据一致性
>
> 锁机制保护,所有状态修改都通过同一个锁_update_lock保护



# AI问题

> ### 第一轮：整体架构理解
>
> 问题1： 请描述一下这个Thorlabs模版设备项目的整体架构，包括主要的项目结构和它们之间的关系。
>
> - TempDevice.Shared：包含所有共享的接口、抽象类、通用服务和工具类
>
> - TempDevice.Plugin：包含具体的设备插件实现（串口和以太网），通过Prism模块化机制注册
>
> - TempDevice.Shell：主应用程序，通过依赖注入容器管理所有服务，通过区域管理器显示不同的视图
>
> 
>
> 问题2： 这个项目使用了哪些设计模式？请举例说明。
>
> - 工厂模式：IEntry.GenerateDevicePlugin()方法
>
> - 策略模式：不同的连接方式（串口vs以太网）
>
> - 观察者模式：Prism的事件聚合器系统
>
> - 依赖注入模式：Prism的IoC容器
>
> 
>
> 问题3： 项目中的TempDevice.Shared、TempDevice.Plugin和TempDevice.Shell分别承担什么职责？
>
> - Shared：提供设备管理、连接服务、事件系统、本地化等基础设施
>
> - Plugin：实现具体的设备通信协议和UI界面
>
> - Shell：提供主窗口、菜单、工具栏等应用程序框架



> ### 第二轮：核心接口和抽象类
>
> 问题4： IDevicePlugin接口定义了哪些核心方法？为什么需要这个接口？
>
> - 设备标识属性：SerialNumber、ProductName、Manufacture、DeviceHandle
>
> - 核心方法：Connect()、Disconnect()、Reconnect()、IsConnected()、Dispose()
>
> 为什么需要这个接口：
>
> - 提供统一的设备操作契约
>
> - 支持多态性，允许不同类型的设备（串口、以太网）使用相同的接口
>
> - 便于依赖注入和单元测试
>
> 
>
> 问题5： IEntry抽象类的作用是什么？PortEntry和EthernetEntry是如何实现它的？
>
> IEntry抽象类的作用：
>
> - 提供设备发现和管理的入口点
>
> - 负责注册视图到区域管理器
>
> - 实现设备列表获取和插件生成
>
> - 每个连接类型（串口、以太网）都有自己的Entry实现
>
> 
>
> 问题6： DeviceDetail抽象类的作用是什么？它和IDevicePlugin有什么关系？
>
> DeviceDetail抽象类：
>
> - 继承自BindableBase，支持数据绑定
>
> - 包含基本设备信息：产品名、序列号、制造商、友好名称、连接状态
>
> - 提供Clone()和GetEntry()抽象方法
>
> - 子类如PortDeviceDetail和EthernetDeviceDetail可以添加特定属性（如端口号、IP地址）
>
> 
>
> - DeviceDetail：是数据模型，存储设备的静态信息和状态
>
> - IDevicePlugin：是功能实现，提供设备的操作方法
>
> DeviceDetail是设备的数据表示，IDevicePlugin是设备的功能实现，它们通过CommonServicesObject组合在一起，形成完整的设备管理单元。



> ### 第三轮：连接管理
>
> 问题7： ConnectService类的主要职责是什么？它是如何管理设备连接的？
>
> 设备连接管理服务，主要负责：
>
> 1. 设备连接管理：建立、维护和断开设备连接
> 2. 连接状态跟踪：管理已连接设备和正在连接中的设备
> 3. 设备插件生命周期管理：创建和管理设备插件实例
> 4. 事件发布：在连接状态变化时发布相应事件
>
> 问题8： 在ConnectService中，connectedDevices和connectingDevices这两个字典的作用是什么？为什么要区分它们？
>
> - connectedDevices：已连接且可用的设备
>
> - connectingDevices：正在连接或重连中的设备（防重复机制）
>
> - 目的：而是防止重复连接同一设备，点的太快连接两次
>
> 
>
> 问题9： ReconnectManager是如何工作的？它和ReconnectWindowViewModel之间是什么关系？
>
> 多设备重连？
>
> -  注册机制：ReconnectManager管理ReconnectWindowViewModel的注册
>
> - ✅ Telerik提醒窗口：确实使用了Telerik内置的提醒窗口，包括生命周期管理、UI控制、异步操作



> ### 第四轮：事件系统
>
> 问题10： 项目使用了什么事件系统？请解释ConnectStartEvent、ConnectFinishEvent等事件的作用。
>
> - 基于IEventAggregator接口
>
> - 使用PubSubEvent<T>作为事件基类
>
> 松耦合，
>
> ConnectStartEvent作用：
>
> - 通知UI组件显示连接进度指示器
>
> - 更新设备状态显示（如"正在连接..."）
>
> - 禁用相关操作按钮（防止重复操作）
>
> - 记录连接日志
>
> ConnectFinishEvent
>
> - 通知UI组件更新设备状态显示
>
> - 显示连接成功/失败的消息
>
> - 启用/禁用相关操作按钮
>
> 
>
> 问题11： 为什么需要ReconnectSingleStartEvent和ReconnectSingleFinishEvent？它们和普通的重连事件有什么区别？
>
> ### 第五轮：具体实现细节
>
> 问题12： 在PortEntry.TryListDevice()方法中，代码是如何解析设备信息的？请解释第47-65行的逻辑。
>
> 问题13： TempDevicePortPlugin和TempDeviceEthernetPlugin在连接方式上有什么不同？
>
> 问题14： PollingService是如何实现轮询的？cancelNext标志的作用是什么？
>
> ### 第六轮：工具类和辅助功能
>
> 问题15： LogHelper类是如何配置日志的？它使用了什么日志框架？
>
> 问题16： CustomSettingsServices类如何管理友好名称？为什么要序列化这些数据？
>
> 问题17： GenerateStaticMethod类中的FindVisualChild和FindVisualParent方法的作用是什么？
>
> ### 第七轮：配置和本地化
>
> 问题18： ThorlabsProduct类如何获取配置信息？这些配置信息存储在哪里？
>
> 问题19： 本地化系统是如何工作的？LocalizationService如何加载和管理多语言支持？
>
> 问题20： 项目支持哪些主题？主题切换是如何实现的？



> PollingService
>
> 1、为什么要waitForStop？pollingEvent字段有什么用，如果没有它会怎么样？
>
> 事件Invoke是同步的：会等待订阅者完成，把事情做完再结束。（阻塞在那）
>
> waitOne就是在等Set信号。
>
> 2、pollingControlSignal的作用
>
> 核心功能：暂停、恢复轮询。
>
> *// ManualResetEventSlim：轻量级，性能更好*
>
> private ManualResetEventSlim? pollingControlSignal;
>
> *// ManualResetEvent：重量级，功能更完整*
>
> private ManualResetEvent pollingEvent;
>
> 暂停和继续的逻辑也要等一组任务执行完。
