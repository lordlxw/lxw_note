# 代码分析

## 一、基本概念

> C# Environment类能获取到多种比较特殊的文件夹作为分析的手段和依据
>
> 1、C:\Users\admin\AppData\Local——在这个文件夹中，应用程序可以存储一些非共享的数据，如配置文件、缓存文件、临时文件、日志文件等。这些数据是特定于用户的，只有当前用户能够访问和操作这个文件夹中的内容。
>
> 注意：每个用户都有自己的`AppData\Local`文件夹，其中的内容是相互隔离的，不同用户之间无法访问对方的本地应用程序数据。
>
> 2、C:\Users\admin\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup
>
> 这个文件夹可以找到开机自启的程序
>
> 3、 Application Frame Host，表示它是一个UWP应用程序
>
> `ApplicationFrameHost.exe` 是 Windows 操作系统中用于承载通用 Windows 平台 (UWP) 应用程序的一个壳程序。它充当了 UWP 应用程序的容器，负责提供应用程序窗口的管理和生命周期管理。





## 二、Core类库

> 1、定义枚举类型、事件和浏览器库
>
> 2、Url帮助类，Url编码转换
>
> 3、数据库模型类、数据库自检，用的SqlLite 和EFCORE作为DBContext
>
> 通过继承EF的DbConfiguration类，用于配置和自定义数据库上下文的行为，您可以扩展和自定义数据库配置选项和行为。
>
> 



> ApplicationFrameHost.exe”，是Windows 10操作系统的一部分，它在后台进程中。通过这个来判断应用是否是UWP应用。他在C:\Windows\System32系统文件夹下，不能关闭它，否则所有应用程序都将失效

### 数据库表

> AppModels——应用程序表，列名——名称、描述、文件路径、Icon路径、总时长
>
> CategoryModels——记录的是个人个性化分类的应用列表
>
> DailyLogModels——每日数据信息，列名——日期（按天）、时间、AppModelID
>
> HoursLogModels——时段数据，列名——日期（按小时）、时间、AppModelID





### 实现逻辑

当有UWP程序时，加入icon图标保存在AppIcons文件夹中

每天的日志会记录在Log文件夹中

创建快捷方式，设置开机启动

网站的缓存机制，key、value

秒到小时、分钟的转换

调用win32API——获取当前窗口句柄

使用到的api如下：

在C#中，`IntPtr.ToString()`方法的默认实现会将`IntPtr`对象的值转换为一个有符号的十进制字符串。

进程最关键的信息——进程名、进程文件夹路径、进程ID

```具体获取代码
1、获取进程名
Process process = Process.GetProcessById(processID);
            processName = process.ProcessName;
2、获取进程句柄
 IntPtr processHandle = IntPtr.Zero;
processHandle = Win32API.OpenProcess(0x001F0FFF, false, processID);

3、获取进程所在文件路径，buffer是缓冲区
if (processHandle != IntPtr.Zero)
            {
                int MaxPathLength = 4096;
                var buffer = new StringBuilder(MaxPathLength);
                Win32API.QueryFullProcessImageName(processHandle, 0, buffer, ref MaxPathLength);
                processFileName = buffer.ToString();
            }
            
            
4、获取进程ID
int processID = 0;
 Win32API.GetWindowThreadProcessId(hwnd, out processID);
```



1、public static extern IntPtr GetForegroundWindow();——获取当前窗口句柄

2、public static extern IntPtr SetWinEventHook(uint eventMin, uint eventMax, IntPtr hmodWinEventProc, WinEventDelegate lpfnWinEventProc, uint idProcess, uint idThread, uint dwFlags);

```注释
eventMin 和 eventMax：指定要监听的事件范围。可以使用预定义的常量或自定义的事件标识符。
hmodWinEventProc：指定事件处理函数所在的模块句柄。通常为 null。
lpfnWinEventProc：指定一个回调函数，用于处理事件通知。该函数的定义由 WinEventDelegate 委托指定。
idProcess 和 idThread：指定要监听的进程和线程标识符。通常为 0，表示监视所有进程和线程。
dwFlags：指定事件钩子的附加选项和标志。
```

> 安装一个事件钩子，将指定的回调函数与特定事件相关联。当相应的事件发生时，系统将调用该回调函数，以通知应用程序。应用程序可以在回调函数中执行自定义的操作来响应事件。
>
> 需要注意的是，使用事件钩子时，必须小心处理回调函数的逻辑，以避免阻塞或延迟系统的正常操作。此外，当事件钩子不再需要时，应使用 `UnhookWinEvent` 函数来取消事件钩子的安装，以释放相关资源。
>
> 
>
> `WinEventDelegate` 是一个自定义的委托类型，用于定义一个回调函数的签名，该回调函数将被用作事件钩子函数的处理函数。
>
> 在这个上下文中，`WinEventDelegate` 委托指定了回调函数的参数和返回类型，以确保与底层的事件钩子函数签名相匹配。
>
> 在代码中，`WinEventDelegate` 委托的定义可能类似于以下形式：
>
> ```C#
> public delegate void WinEventDelegate(IntPtr hWinEventHook, uint eventType, IntPtr hwnd, int idObject, int idChild, uint dwEventThread, uint dwmsEventTime);
> //每个参数的含义如下：
> 
> hWinEventHook：事件钩子的句柄，用于标识特定的事件钩子。
> eventType：表示事件类型的整数值。具体的事件类型取决于安装钩子时指定的事件范围。
> hwnd：与事件相关联的窗口的句柄。例如，窗口焦点变化事件将提供前台窗口的句柄。
> idObject 和 idChild：事件中特定对象和子对象的标识符。这些标识符的含义取决于具体的事件类型。
> dwEventThread：事件所在的线程的标识符。
> dwmsEventTime：事件发生的时间戳，以毫秒为单位。
> ```

3、 [DllImport("user32.dll")]
        public static extern bool UnhookWinEvent(IntPtr hWinEventHook);

用于取消已安装的事件钩子。`UnhookWinEvent` 函数用于取消先前使用 `SetWinEventHook` 函数安装的事件钩子。通过传递事件钩子的句柄作为参数，可以指定要取消的事件钩子，以释放相关资源。

4、[DllImport("user32.dll", CharSet = CharSet.Auto)]        public static extern int GetWindowThreadProcessId(IntPtr hwnd, out int ID); 

- `hwnd`：表示窗口句柄的 `IntPtr` 对象。它是要获取进程标识符的窗口句柄。
- `processId`：表示进程标识符的 `out` 参数。当函数成功调用后，会将对应窗口的进程标识符写入这个参数中。

`GetWindowThreadProcessId` 函数用于获取给定窗口的线程标识符和进程标识符。通过传递窗口句柄作为参数，可以获取与该窗口关联的线程和进程的标识符。目的：==输入句柄，获得相关联进程标识符==



5、[DllImport("psapi.dll", SetLastError = true)]
        public static extern int GetModuleFileNameExA(IntPtr hProcess, IntPtr hModule, StringBuilder lpFilename, int nSize);

可以通过调用 `Marshal.GetLastWin32Error` 方法来获取最后发生的错误码(int 类型)

`psapi.dll` 提供了一组函数，可以用于检索进程的详细信息，如进程的标识符、模块句柄、模块名称、模块路径等。通过使用这些函数，开发人员可以编写与进程相关的工具、性能监控程序、进程注入等应用程序。

- `hProcess`：进程的句柄，表示要查询的进程。
- `hModule`：模块的句柄，表示要获取其文件名的模块。可以为 `IntPtr.Zero`，表示获取进程的主模块文件名。
- `lpFilename`：一个 `StringBuilder` 对象，用于接收模块的文件名。
- `nSize`：`lpFilename` 缓冲区的大小（以字符为单位）。

`GetModuleFileNameExA` 函数用于获取指定进程中指定模块的完整文件名。通过传递进程句柄和模块句柄，可以获取该模块的文件名。返回值是一个整数，表示函数的执行结果。如果函数成功执行，则返回值为写入 `lpFilename` 缓冲区的字符数（不包括终止符）；如果函数失败，则返回值为 0，并可以通过调用 `Marshal.GetLastWin32Error()` 获取错误码。



6、     [DllImport("kernel32.dll", SetLastError = true)]
        public static extern IntPtr OpenProcess(uint dwDesiredAccess, bool bInheritHandle, int dwProcessId);

     [DllImport("kernel32.dll", SetLastError = true)]
    public static extern bool CloseHandle(UIntPtr hObject);

1. `OpenProcess` 函数：
   - `OpenProcess` 用于打开一个指定进程的句柄，以便对其进行操作。
   - 参数 `dwDesiredAccess` 指定要获取的访问权限，例如读取、写入或执行权限等。
   - 参数 `bInheritHandle` 指定句柄是否可以被新进程继承。
   - 参数 `dwProcessId` 指定要打开的进程的标识符。
   - 函数返回一个进程句柄，用于后续操作该进程。如果打开失败，则返回 `IntPtr.Zero`。
2. `CloseHandle` 函数：
   - `CloseHandle` 用于关闭一个已打开的句柄。
   - 参数 `hObject` 是要关闭的句柄。
   - 函数返回一个布尔值，指示关闭句柄是否成功。



7、[DllImport("user32", SetLastError = true)]
        [return: MarshalAs(UnmanagedType.Bool)]
        public static extern bool EnumChildWindows(IntPtr hWndParent, EnumWindowProc lpEnumFunc, IntPtr lParam);

    [DllImport("kernel32.dll", SetLastError = true, CharSet = CharSet.Auto)]
    public static extern bool QueryFullProcessImageName([In] IntPtr hProcess, [In] int dwFlags, [Out] StringBuilder lpExeName, ref int lpdwSize);

1. `EnumChildWindows` 函数：
   - `EnumChildWindows` 用于枚举指定父窗口下的所有子窗口。
   - 参数 `hWndParent` 是父窗口的句柄，指定要枚举子窗口的父窗口。
   - 参数 `lpEnumFunc` 是一个回调函数，用于处理每个子窗口的信息。
   - 参数 `lParam` 是用户自定义的参数，将传递给回调函数。
   - 函数返回一个布尔值，表示枚举子窗口的结果。
2. `QueryFullProcessImageName` 函数：
   - `QueryFullProcessImageName` 用于获取指定进程的完整可执行文件路径。
   - 参数 `hProcess` 是要查询的进程句柄。
   - 参数 `dwFlags` 是一些标志位，用于控制查询的方式。
   - 参数 `lpExeName` 是一个 `StringBuilder` 对象，用于接收进程的完整可执行文件路径。
   - 参数 `lpdwSize` 是 `lpExeName` 缓冲区的大小，函数会将实际路径的长度写入此参数。
   - 函数返回一个布尔值，表示查询路径是否成功。





8、public const UInt32 PROCESS_QUERY_INFORMATION = 0x400;
        public const UInt32 PROCESS_VM_READ = 0x010;

官方文档地址：https://learn.microsoft.com/en-us/windows/win32/procthread/process-security-and-access-rights

`PROCESS_QUERY_INFORMATION`：

- 这个常量的值是 `0x400`，表示进程访问权限的标志位。
- 当将该标志传递给打开进程句柄的函数（如 `OpenProcess`）时，表示请求对进程的基本信息进行查询。

`PROCESS_VM_READ`：

- 这个常量的值是 `0x010`，表示进程访问权限的标志位。
- 当将该标志传递给打开进程句柄的函数（如 `OpenProcess`）时，表示请求对进程的虚拟内存进行读取。

==`PROCESS_QUERY_INFORMATION` 和 `PROCESS_VM_READ` 分别用于请求查询进程信息和读取进程内存的权限。==





后面还有声音判断、键盘钩子



### 应用设置


   **ConfigModel**

```C#
 /// 关联进程
     public List<LinkModel> Links { get; set; }
/// 常规
    public GeneralModel General { get; set; }
/// 行为
    public BehaviorModel Behavior { get; set; }
```

> LinkMode——就是一个名字对应多个关联进程
>
> GeneralModel——配置是否开机自启、主题模式、概览设置、浏览统计等
>
> BehaviorModel——行为，用于忽略一些指定的应用或URL





### 服务类（Services）

服务通常被设计为具有特定的功能和责任

> 1、AppConfig：IAppConfig——加载配置、获取配置、更新配置、修改配置。跟之前的ConfigModel类有关，就是对应配置的json文件
>
> 
>
> ***2、AppData：IAppData——更新App数据，获取App。用到了AppModel。List<AppModel> GetAppsByCategoryID(int categoryID);根据ID分类APP列表。
>
> 
>
> 3、AppObserver：IAppObserver——观察者，用于监听应用程序获取焦点事件。方法及事件：启动观察、停止观察、切换活跃App时发生
>
> 关键：进程文件路径中若包含 "ApplicationFrameHost.exe"，则说明是 UWP 应用程序的主机进程。
>
> ①根据窗口句柄拿到进程ID，②根据进程ID拿到进程句柄，③根据进程句柄拿到进程文件夹路径
>
> 4、BrowserObserver：IBrowserObserver——观察者，浏览器网站观察服务。方法及事件：启动观察、停止观察、切换WEB站点时发生
>
> 
>
> 5、BrowserWatcher：IBrowserWatcher——观察者，用于观察用户用的是什么浏览器。方法及事件：启动观察、停止观察、切换浏览器时发生 目前有三种浏览器：chrome、msedge、vivaldi
>
> 
>
> 6、Categorys：ICategorys——方法：增删改查分类，获取CategoryModel
>
> 
>
> ***7、Data：IData——方法：保存APP使用时长、获取今天数据、查询指定范围数据、本周数据、指定进程某天、周、月的数据、获取指定日期所有分类时段统计数据、获取指定日期范围所有分类按天统计数据、获取指定年份所有分类按月份统计数据、清空指定时间范围数据、导出数据到EXCEL
>
> 
>
> 8、Database：IDatabase——拿到读取数据库对象、拿到写入数据库对象、关闭写入数据库的上下文对象。在完成写入操作后，你可以调用该方法释放资源或执行清理操作
>
> 
>
> 9、DateTimeObserver：IDateTimeObserver——日期变更观察（用于监听日期变化）。方法及事件：启动、停止、日期即将发生变化（前1秒）的事件
>
> 
>
> 10、Main：IMain——方法及事件：运行服务、启动服务、退出服务。更新某个进程的事件、初始化完成后触发的事件
>
> 
>
> 11、Sleepdiscover：ISleepdiscover——方法及事件：启动、休眠状态发生更改的事件
>
> 
>
> ***12、WebData：IWebData——方法及事件：添加链接浏览时长、更新链接的图标、获取日期范围的站点浏览数据(浏览时长降序排序)、获取、更新、删除网站所有分类、通过分类ID获取分类数据、获取指定时间段的浏览时长统计数据、统计指定时间段的浏览时长、统计指定时间段的网页浏览数量、获取指定时间段的网页浏览记录、 清空指定日期范围数据
>
> 
>
> 13、WebFilter：IWebFilter——网站过滤器，方法：初始化过滤器、指示输入的URL是否被域名配置忽略



### 几个核心类分析：

> 1、Sleepdiscover
>
> SystemEvents类很关键





## 三、UI应用程序

> 首先先将上述Core声明的接口和实现当作服务一个个注册到容器中（基本是单例）
>
> 核心语句：
>
> ```C#
> var main = serviceProvider.GetService<IMainServicer>();//main里有许多依赖的接口
> main.Start();
> ```
>
> Main.cs构造函数里调用：
>
> ```C#
>   public Main(
>             IAppObserver appObserver,
>             IData data,
>             ISleepdiscover sleepdiscover,
>             IAppConfig appConfig,
>             IDateTimeObserver dateTimeObserver,
>             IAppData appData, ICategorys categories,
>             IBrowserObserver browserObserver,
>             IWebFilter webFilter_)
>         {
>             this.appObserver = appObserver;
>             this.data = data;
>             this.sleepdiscover = sleepdiscover;
>             this.appConfig = appConfig;
>             this.dateTimeObserver = dateTimeObserver;
>             this.appData = appData;
>             this.categories = categories;
>             this.browserObserver = browserObserver;
>             _webFilter = webFilter_;
> 
>             IgnoreProcessCacheList = new List<string>();
>             ConfigIgnoreProcessRegxList = new List<string>();
>             ConfigIgnoreProcessList = new List<string>();
> 
>             appObserver.OnAppActive += AppObserver_OnAppActive;  //App活跃时触发
>             sleepdiscover.SleepStatusChanged += Sleepdiscover_SleepStatusChanged; //休眠状态变化时触发
>             appConfig.ConfigChanged += AppConfig_ConfigChanged; //配置改变时触发
>             dateTimeObserver.OnDateTimeChanging += DateTimeObserver_OnDateTimeChanging; //日期时间变化时触发
>         }
> 
> ```
>
> 



> 除了Main.cs还有一个 MainServicer : IMainServicer，更核心的服务
>
> ```C#
> public MainServicer(
>          IMain main,
>          IThemeServicer themeServicer,
>          IInputServicer inputServicer,
>          IAppContextMenuServicer appContextMenuServicer,
>          IWebSiteContextMenuServicer webSiteContext_,
>          IStatusBarIconServicer statusBarIconServicer_)
>      {
>          this.main = main;
>          this.themeServicer = themeServicer;
>          this.inputServicer = inputServicer;
>          this.appContextMenuServicer = appContextMenuServicer;
>          _webSiteContext = webSiteContext_;
>          _statusBarIconServicer = statusBarIconServicer_;
>      }
> ```
>
> 由于AppContextMenu、WebSiteContextMenu、StatusBarIconServicer 都注入了 MainwindowViewModel，意味着大家都能拿到vm这个句柄了，然后就可以操作了。
>
> 最重要的方法是Main.cs中的Run方法
>
> 观察者模式都是通过Start方法，开始调用的
>
> ```C#
> main.cs中
>      public void Start()
>         {
>             activeStartTime = DateTime.Now;
>             activeProcess = null;
> 
>             dateTimeObserver.Start();
>             appObserver.Start();
> 
>             if (config.General.IsWebEnabled)
>             {
>                 browserObserver.Start();
>             }
>         }
> ```
>
> 

### 1、Controls 自定义控件

> DefaultStyleKey



## 四、代码修改

### 1、Win32API中增加对会话用户的监视

```C# Win32API
        #region lxw添加，对登录的用户进行区分
        public enum WTS_INFO_CLASS
        {
            WTSInitialProgram,
            WTSApplicationName,
            WTSWorkingDirectory,
            WTSOEMId,
            WTSSessionId,
            WTSUserName,
            WTSWinStationName,
            WTSDomainName,
            WTSConnectState,
            WTSClientBuildNumber,
            WTSClientName,
            WTSClientDirectory,
            WTSClientProductId,
            WTSClientHardwareId,
            WTSClientAddress,
            WTSClientDisplay,
            WTSClientProtocolType
        }

        //ppBuffer——返回的就是会话ID， int sessionId = Marshal.ReadInt32(buffer);
        [DllImport("wtsapi32.dll", SetLastError = true)]
        public static extern bool WTSQuerySessionInformation(IntPtr hServer, int sessionId, WTS_INFO_CLASS wtsInfoClass, out IntPtr ppBuffer, out int pBytesReturned);

        [DllImport("wtsapi32.dll", SetLastError = true)]
        public static extern void WTSFreeMemory(IntPtr pMemory);

        //根据会话ID获取用户名
        public static string GetSessionUserName(int sessionId)
        {
            IntPtr pBuffer;
            int bytesReturned;

            if (WTSQuerySessionInformation(IntPtr.Zero, sessionId, WTS_INFO_CLASS.WTSUserName, out pBuffer, out bytesReturned))
            {
                string userName = Marshal.PtrToStringAnsi(pBuffer);
                WTSFreeMemory(pBuffer);
                return userName;
            }

            throw new Exception("Failed to query session information.");
        }
        #endregion
```





## END：可以优化的点

> 1、浏览器的选择上除了三个，可以添加
>
> 2、网站计时这边是不是要全一些
>
> 3、获取日期范围的站点浏览数据(浏览时长降序排序)，可以加一个按钮变成可以按降序排序
>
> 4、在源代码上新增代码的存储工程，供wcf程序使用，外部可以直接调用它的sql语句。
>
> 存储过程（Stroed Proceduer）是在大型数据库系统中，一组为了完成特定功能的SQL语句集，经编译后，存储在数据库中。用户通过指定存储过程的名字并给出参数（如果该存储过程有参数）来执行它。再运行存储过程前，数据库已对其进行了语法和句法分析，并给出了优化执行方案。这种已经编译好的过程可极大地改善SQL语句的性能。由于执行SQL语句的大部分工作已经完成，所以存储过程能以极快的速度执行。
>
> 5、bool IsSystemProcess(string processName, string title)，判断指定进程是否属于系统进程时，可以通过array添加新的需要排除在计时之外的进程。     string[] titleArr = { "UnlockingWindow", "Program Manager" }; 根据array名