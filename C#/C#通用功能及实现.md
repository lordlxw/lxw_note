## 1、判断进程是否多开

> 通过mutex互斥变量实现
>
> ```C#
>     public partial class App : Application
>     {
>         private System.Threading.Mutex mutex;
> 
>         private void OnStartup(object sender, StartupEventArgs e)
>         {
>             //  阻止多开和用户主动启动
>             if (e.Args.Length == 0 || IsRuned())
>             {
>                 Shutdown();
>             }
>         }
>         #region 获取当前程序是否已运行
>         /// <summary>
>         /// 获取当前程序是否已运行
>         /// </summary>
>         private bool IsRuned()
>         {
>             bool ret;
>             mutex = new System.Threading.Mutex(true, System.Reflection.Assembly.GetEntryAssembly().ManifestModule.Name, out ret);
>             if (!ret)
>             {
>                 return true;
>             }
>             return false;
>         }
>         #endregion
>     }
> //ret=true,表示当前是唯一实例
> ```
>
> 



## 2、DBContext——读者写者问题

> 有写锁和读锁，用EFCore，接口及实现
>
>     public class Database : IDatabase
>     {
>         private TaiDbContext _writerContext;
>         private readonly object _writeLocker = new object();
>         private readonly object _readLocker = new object();
>         private readonly object _closeLocker = new object();
>         private int _outTime = 60;
>         private int _readerNum = 0; //0表示没读，1代表在读
>         private bool _isWriting = false;
>         private bool _isReading = false;
>         public TaiDbContext GetReaderContext()
>         {
>             lock (_readLocker)
>             {
>                 _isReading = true;
>                 int duration = 0;
>                 while (_isWriting)  //检查是否在写（60S）
>                 {
>                     Thread.Sleep(1000);
>                     duration++;
>                     if (duration >= _outTime)
>                     {
>                         break;
>                     }
>                 }
>                 _writerContext?.Dispose(); //超过60s还在写就强制dispose掉
>                 _readerNum++;
>                 var db = new TaiDbContext();
>                 db.Database.Connection.StateChange += Connection_StateChange;
>                 return db;
>             }
>         }
>     
>         private void Connection_StateChange(object sender, System.Data.StateChangeEventArgs e)
>         {
>             if (e.CurrentState == System.Data.ConnectionState.Closed)
>             {
>                 _readerNum--;
>                 _isReading = _readerNum > 0;
>             }
>         }
>                                                        
>         public void CloseReader()
>         {
>         }
>                                                     
>         public TaiDbContext GetWriterContext()
>         {
>             lock (_writeLocker)
>             {
>                 int duration = 0;
>                 while (_isWriting || _isReading)
>                 {
>                     Thread.Sleep(1000);
>                     duration++;
>                     if (duration >= _outTime)
>                     {
>                         break;
>                     }
>                 }
>                 _isWriting = true;
>                 _writerContext = new TaiDbContext();
>                 return _writerContext;
>             }
>         }
>                                                     
>         public void CloseWriter()
>         {
>             lock (_closeLocker)
>             {
>                 _writerContext?.Dispose();
>                 _isWriting = false;
>             }
>         }
>     }
>     
>     
>      public interface IDatabase
>         {
>             TaiDbContext GetReaderContext();
>             //void CloseReader();
>             TaiDbContext GetWriterContext();
>             void CloseWriter();
>         }





## 3、EFCore声明（自检数据库） 

> ```C# EFCore声明
>     public class TaiDbContext : DbContext
>     {
>         /// <summary>
>         /// 每日数据
>         /// </summary>
>         public DbSet<DailyLogModel> DailyLog { get; set; }
>         /// <summary>
>         /// 时段数据
>         /// </summary>
>         public DbSet<HoursLogModel> HoursLog { get; set; }
>         public DbSet<AppModel> App { get; set; }
>         /// <summary>
>         /// 分类
>         /// </summary>
>         public DbSet<CategoryModel> Categorys { get; set; }
>         /// <summary>
>         /// 网站
>         /// </summary>
>         public DbSet<WebSiteModel> WebSites { get; set; }
>         /// <summary>
>         /// 网站分类
>         /// </summary>
>         public DbSet<WebSiteCategoryModel> WebSiteCategories { get; set; }
>         /// <summary>
>         /// 网页浏览记录（每小时）
>         /// </summary>
>         public DbSet<WebBrowseLogModel> WebBrowserLogs { get; set; }
>         /// <summary>
>         /// 网页链接
>         /// </summary>
>         public DbSet<WebUrlModel> WebUrls { get; set; }
> 
>         private static string _dbFilePath = Path.Combine(FileHelper.GetRootDirectory(), "Data", "data.db");
>         public TaiDbContext()
>        : base(new SQLiteConnection()
>        {
>            ConnectionString = $"Data Source={_dbFilePath}",
>            BusyTimeout = 60
>        }, true)
>         {
>             DbConfiguration.SetConfiguration(new SQLiteConfiguration());
>         }
> 
>         protected override void OnModelCreating(DbModelBuilder modelBuilder)
>         {
>             var model = modelBuilder.Build(Database.Connection);
>             new SQLiteBuilder(model).SelfCheck();
>         }
> 
>         public void SelfCheck()
>         {
>             Database.ExecuteSqlCommand("select count(*) from sqlite_master where type='table' and name='tai'");
>         }
>     }
> ```
>
> 



## 4、单表接口和实现增删改查

> ```C# 接口
>     public interface IAppData
>     {
>         /// <summary>
>         /// 更新app数据，要先调用GetApp获得后更改并传回才有效
>         /// </summary>
>         void UpdateApp(AppModel app);
>         /// <summary>
>         /// 保存app数据更改
>         /// </summary>
>         //void SaveAppChanges();
>         /// <summary>
>         /// 通过进程名称获取app
>         /// </summary>
>         /// <param name="name"></param>
>         /// <returns></returns>
>         AppModel GetApp(string name);
>         AppModel GetApp(int id);
>         /// <summary>
>         /// 获取所有app
>         /// </summary>
>         /// <returns></returns>
>         List<AppModel> GetAllApps();
>         void AddApp(AppModel app);
>         /// <summary>
>         /// 加载已存储的app列表，仅建议在启动时调用一次，无必要请勿再次调用
>         /// </summary>
>         void Load();
>         /// <summary>
>         /// 获取app列表通过分类ID
>         /// </summary>
>         /// <param name="categoryID">分类ID</param>
>         /// <returns></returns>
>         List<AppModel> GetAppsByCategoryID(int categoryID);
>     }
> ```
>
> ```实现
>     public class AppData : IAppData
>     {
>         private List<AppModel> _apps;
> 
>         private readonly IDatabase _databse;
>         private readonly object _locker = new object();
>         public AppData(IDatabase databse)
>         {
>             _databse = databse;
>         }
>         public List<AppModel> GetAllApps()
>         {
>             return _apps;
>         }
> 
>         public void Load()
>         {
>             Debug.WriteLine("加载app开始");
>             using (var db = _databse.GetReaderContext())
>             {
>                 _apps = (
>                     from app in db.App
>                     join c in db.Categorys
>                     on app.CategoryID equals c.ID into itdata
>                     from n in itdata.DefaultIfEmpty()
>                     select app
>                     ).ToList()
>                     .Select(m => new AppModel
>                     {
>                         ID = m.ID,
>                         Category = m.Category != null ? m.Category : null,
>                         CategoryID = m.CategoryID,
>                         Description = m.Description,
>                         File = m.File,
>                         IconFile = m.IconFile,
>                         Name = m.Name,
>                         TotalTime = m.TotalTime
>                     })
>                     .ToList();
>             }
>         }
> 
>         /// <summary>
>         /// 更新app数据，要先调用GetApp获得后更改并传回才有效
>         /// </summary>
>         /// <param name="app"></param>
>         public void UpdateApp(AppModel app_)
>         {
>             try
>             {
>                 using (var db = _databse.GetWriterContext())
>                 {
>                     var app = db.App.FirstOrDefault(c => c.ID.Equals(app_.ID));
>                     if (app != null)
>                     {
>                         app.TotalTime = app_.TotalTime;
>                         app.IconFile = app_.IconFile;
>                         app.Name = app_.Name;
>                         app.Description = app_.Description;
>                         app.File = app_.File;
>                         app.CategoryID = app_.CategoryID;
>                         db.SaveChanges();
>                     }
>                     _databse.CloseWriter();
>                 }
>             }
>             catch(Exception e)
>             {
>                 Logger.Error(e.ToString());
>             }
>         }
>         public AppModel GetApp(string name)
>         {
>             lock (_locker)
>             {
>                 return _apps.Where(m => m.Name == name).FirstOrDefault();
>             }
>         }
>         public AppModel GetApp(int id)
>         {
>             lock (_locker)
>             {
>                 return _apps.Where(m => m.ID == id).FirstOrDefault();
>             }
>         }
>         public void AddApp(AppModel app)
>         {
>             lock (_locker)
>             {
>                 if (_apps.Where(m => m.Name == app.Name).Any())
>                 {
>                     return;
>                 }
>                 try
>                 {
>                     using (var db = _databse.GetWriterContext())
>                     {
>                         var r = db.App.Add(app);
>                         int res = db.SaveChanges();
>                         if (res > 0)
>                         {
>                             Debug.WriteLine("add done!" + r.ID);
> 
>                             _apps.Add(app);
>                         }
>                         _databse.CloseWriter();
>                     }
>                 }catch(Exception e)
>                 {
>                     Logger.Error(e.ToString());
>                 }
>             }
>         }
> 
> 
>         public List<AppModel> GetAppsByCategoryID(int categoryID)
>         {
>             lock (_locker)
>             {
>                 return _apps.Where(m => m.CategoryID == categoryID).ToList();
>             }
>         }
>     }
> ```
>
> 





## 5、WIN32API的声明及调用

> 声明
>
> ```C#
>     public static class Win32API
>     {
>         //返回当前活动窗口的句柄
>         [DllImport("user32.dll")]
>         public static extern IntPtr GetForegroundWindow();
> 
>         //用于将指定的窗口设置为当前前台窗口，并将其激活到前台。
>         [DllImport("user32.dll")]
>         public static extern IntPtr SetForegroundWindow(IntPtr hwnd);
> 
> 
>         public delegate void WinEventDelegate(IntPtr hWinEventHook, uint eventType,
>           IntPtr hwnd, int idObject, int idChild, uint dwEventThread, uint dwmsEventTime);
> 
> 
>         //安装一个事件钩子，以便在系统中指定的事件发生时接收通知。
>         [DllImport("user32.dll")]
>         public static extern IntPtr SetWinEventHook(uint eventMin, uint eventMax, IntPtr
>           hmodWinEventProc, WinEventDelegate lpfnWinEventProc, uint idProcess,
>           uint idThread, uint dwFlags);
> 
>         [DllImport("user32.dll")]
>         public static extern bool UnhookWinEvent(IntPtr hWinEventHook);
> 
>         //根据句柄获得线程id
>         [DllImport("user32.dll", CharSet = CharSet.Auto)]
>         public static extern int GetWindowThreadProcessId(IntPtr hwnd, out int ID);
> 
>         [DllImport("psapi.dll", SetLastError = true)]
>         public static extern int GetModuleFileNameExA(IntPtr hProcess, IntPtr hModule, StringBuilder lpFilename, int nSize);
>         //[DllImport("kernel32.dll", SetLastError = true, CharSet = CharSet.Auto)]
>         //static extern bool QueryFullProcessImageName(IntPtr hProcess, uint dwFlags, [Out, MarshalAs(UnmanagedType.LPTStr)] StringBuilder lpExeName, ref uint lpdwSize);
> 
>         //打开一个已存在的进程，并返回该进程的句柄（HANDLE）。
>         [DllImport("kernel32.dll", SetLastError = true)]
>         public static extern IntPtr OpenProcess(uint dwDesiredAccess, bool bInheritHandle, int dwProcessId);
> 
>         [DllImport("kernel32.dll", SetLastError = true)]
>         public static extern bool CloseHandle(UIntPtr hObject);
> 
> 
>         /// <summary>
>         /// 定义父子进程的结构体
>         /// </summary>
>         internal struct WINDOWINFO
>         {
>             public uint ownerpid;
>             public uint childpid;
>         }
> 
>         [DllImport("user32.dll", SetLastError = true)]
>         public static extern uint GetWindowThreadProcessId(IntPtr hWnd, out uint lpdwProcessId);
> 
>         //使用 EnumChildWindows 函数可以遍历指定父窗口下的所有子窗口，并通过回调函数对每个子窗口进行处理，
>         //例如获取子窗口的句柄、属性、样式等信息，或者执行其他操作。
>         [DllImport("user32", SetLastError = true)]
>         [return: MarshalAs(UnmanagedType.Bool)]
>         public static extern bool EnumChildWindows(IntPtr hWndParent, EnumWindowProc lpEnumFunc, IntPtr lParam);
> 
> 
> 
>         //获取进程所在文件夹路径
>         [DllImport("kernel32.dll", SetLastError = true, CharSet = CharSet.Auto)]
>         public static extern bool QueryFullProcessImageName([In] IntPtr hProcess, [In] int dwFlags, [Out] StringBuilder lpExeName, ref int lpdwSize);
> 
> 
>         public delegate bool EnumWindowProc(IntPtr hWnd, IntPtr parameter);
> 
>         public const UInt32 PROCESS_QUERY_INFORMATION = 0x400; //检索有关进程的某些信息
>         public const UInt32 PROCESS_VM_READ = 0x010;  //读取进程中的内存
> 
>         //private const uint WINEVENT_OUTOFCONTEXT = 0;
>         //private const uint EVENT_SYSTEM_FOREGROUND = 3;
>         //用于获取指定窗口的子窗口的应用程序名称。
>         public static string UWP_AppName(IntPtr hWnd, uint pID)
>         {
>             WINDOWINFO windowinfo = new WINDOWINFO();
>             windowinfo.ownerpid = pID;
>             windowinfo.childpid = windowinfo.ownerpid;
> 
>             IntPtr pWindowinfo = Marshal.AllocHGlobal(Marshal.SizeOf(windowinfo));
> 
>             Marshal.StructureToPtr(windowinfo, pWindowinfo, false);
> 
>             EnumWindowProc lpEnumFunc = new EnumWindowProc(EnumChildWindowsCallback);
>             EnumChildWindows(hWnd, lpEnumFunc, pWindowinfo);
> 
>             windowinfo = (WINDOWINFO)Marshal.PtrToStructure(pWindowinfo, typeof(WINDOWINFO));
>             if (windowinfo.childpid == windowinfo.ownerpid)
>             {
>                 return null;
>             }
>             IntPtr proc;
>             if ((proc = OpenProcess(PROCESS_QUERY_INFORMATION | PROCESS_VM_READ, false, (int)windowinfo.childpid)) == IntPtr.Zero) return null;
> 
>             int capacity = 2000;
>             StringBuilder sb = new StringBuilder(capacity);
>             QueryFullProcessImageName(proc, 0, sb, ref capacity);
> 
>             Marshal.FreeHGlobal(pWindowinfo);
> 
>             return sb.ToString(0, capacity);
>         }
> 
>         /// <summary>
>         /// 用于枚举指定窗口的子窗口。
>         /// </summary>
>         /// <param name="hWnd"></param>
>         /// <param name="lParam"></param>
>         /// <returns></returns>
>         private static bool EnumChildWindowsCallback(IntPtr hWnd, IntPtr lParam)
>         {
>             WINDOWINFO info = (WINDOWINFO)Marshal.PtrToStructure(lParam, typeof(WINDOWINFO));
> 
>             uint pID;
>             GetWindowThreadProcessId(hWnd, out pID);
> 
>             if (pID != info.ownerpid) info.childpid = pID;
> 
>             Marshal.StructureToPtr(info, lParam, true);
> 
>             return true;
>         }
> 
> 
>         /// <summary>
>         /// 获取鼠标坐标
>         /// </summary>
>         /// <param name="lpPoint"></param>
>         /// <returns></returns>
>         [DllImport("user32.dll", SetLastError = true)]
>         [return: MarshalAs(UnmanagedType.Bool)]
>         private static extern bool GetCursorPos(out POINT lpPoint);
> 
>         [StructLayout(LayoutKind.Sequential)]
>         public struct POINT
>         {
>             public int X;
>             public int Y;
> 
>             public static implicit operator Point(POINT point)
>             {
>                 return new Point(point.X, point.Y);
>             }
>         }
> 
>         public static Point GetCursorPosition()
>         {
>             POINT lpPoint;
>             GetCursorPos(out lpPoint);
>             return lpPoint;
>         }
> 
>         #region 声音判断
>         /// <summary>
>         /// 指示系统当前是否在播放声音
>         /// </summary>
>         /// <returns></returns>
>         public static bool IsWindowsPlayingSound()
>         {
>             try
>             {
>                 IMMDeviceEnumerator enumerator = (IMMDeviceEnumerator)(new MMDeviceEnumerator());
>                 IMMDevice speakers = enumerator.GetDefaultAudioEndpoint(EDataFlow.eRender, ERole.eMultimedia);
>                 IAudioMeterInformation meter = (IAudioMeterInformation)speakers.Activate(typeof(IAudioMeterInformation).GUID, 0, IntPtr.Zero);
>                 if (meter != null)
>                 {
> 
>                     float value = meter.GetPeakValue();
> 
>                     // this is a bit tricky. 0 is the official "no sound" value
>                     // but for example, if you open a video and plays/stops with it (w/o killing the app/window/stream),
>                     // the value will not be zero, but something really small (around 1E-09)
>                     // so, depending on your context, it is up to you to decide
>                     // if you want to test for 0 or for a small value
>                     return value > 1E-08;
>                 }
>                 else
>                 {
>                     return false;
>                 }
>             }
>             catch (Exception ec)
>             {
>                 Logger.Error(ec.Message);
>                 return false;
>             }
>         }
> 
>         [ComImport, Guid("BCDE0395-E52F-467C-8E3D-C4579291692E")]
>         private class MMDeviceEnumerator
>         {
>            
>         }
> 
>         private enum EDataFlow
>         {
>             eRender,
>             eCapture,
>             eAll,
>         }
> 
>         private enum ERole
>         {
>             eConsole,
>             eMultimedia,
>             eCommunications,
>         }
> 
>         [InterfaceType(ComInterfaceType.InterfaceIsIUnknown), Guid("A95664D2-9614-4F35-A746-DE8DB63617E6")]
>         private interface IMMDeviceEnumerator
>         {
>             void NotNeeded();
>             IMMDevice GetDefaultAudioEndpoint(EDataFlow dataFlow, ERole role);
>             // the rest is not defined/needed
>         }
> 
>         [InterfaceType(ComInterfaceType.InterfaceIsIUnknown), Guid("D666063F-1587-4E43-81F1-B948E807363F")]
>         private interface IMMDevice
>         {
>             [return: MarshalAs(UnmanagedType.IUnknown)]
>             object Activate([MarshalAs(UnmanagedType.LPStruct)] Guid iid, int dwClsCtx, IntPtr pActivationParams);
>             // the rest is not defined/needed
>         }
> 
>         [InterfaceType(ComInterfaceType.InterfaceIsIUnknown), Guid("C02216F6-8C67-4B5B-9D00-D008E73E0064")]
>         private interface IAudioMeterInformation
>         {
>             float GetPeakValue();
>             // the rest is not defined/needed
>         }
>         #endregion
> 
>         #region 键盘钩子
>         [StructLayout(LayoutKind.Sequential)]
>         public class KeyboardHookStruct
>         {
>             public int vkCode;  //定一个虚拟键码。该代码必须有一个价值的范围1至254
>             public int scanCode; // 指定的硬件扫描码的关键
>             public int flags;  // 键标志
>             public int time; // 指定的时间戳记的这个讯息
>             public int dwExtraInfo; // 指定额外信息相关的信息
>         }
> 
>         private const int WH_KEYBOARD_LL = 13;
>         public const int WM_KEYDOWN = 0x0100;
>         private const int WH_MOUSE_LL = 14;
>         public const int WM_LBUTTONDBLCLK = 0x202;
>         public const int WM_WHEEL = 0x20a;
>         public const int WM_KEYUP = 0x101;
>         public const int WM_SYSKEYDOWN = 0x104;
>         public const int WM_SYSKEYUP = 0x105;
>         /// <summary>
>         /// 设置键盘钩子
>         /// </summary>
>         /// <param name="proc"></param>
>         /// <returns></returns>
>         public static IntPtr SetKeyboardHook(LowLevelKeyboardProc proc)
>         {
>             using (Process curProcess = Process.GetCurrentProcess())
>                 using (ProcessModule curModule = curProcess.MainModule)
>                 {
>                     return SetWindowsHookEx(WH_KEYBOARD_LL, proc,
>                         GetModuleHandle(curModule.ModuleName), 0);
>                 }
>             
>         }
> 
> 
>         /// <summary>
>         /// 设置鼠标钩子
>         /// </summary>
>         /// <param name="proc"></param>
>         /// <returns></returns>
>         public static IntPtr SetMouseHook(LowLevelKeyboardProc proc)
>         {
>             using (Process curProcess = Process.GetCurrentProcess())
>             using (ProcessModule curModule = curProcess.MainModule)
>             {
>                 return SetWindowsHookEx(WH_MOUSE_LL, proc,
>                     GetModuleHandle(curModule.ModuleName), 0);
>             }
>         }
>         public delegate IntPtr LowLevelKeyboardProc(
>             int nCode, IntPtr wParam, IntPtr lParam);
>         //wParam 通常是一个与消息有关的常量值，也可能是窗口或控件的句柄。lParam 通常是一个指向内存中数据的指针。
> 
> 
>         /*安装一个钩子（Hook），以便用于拦截并处理键盘输入事件。
>         钩子回调函数 LowLevelKeyboardProc 将在键盘事件发生时被调用，可以在回调函数中对键盘事件进行处理，例如记录按键、屏蔽某些按键等。
>         */
>         [DllImport("user32.dll", CharSet = CharSet.Auto, SetLastError = true)]
>         public static extern IntPtr SetWindowsHookEx(int idHook,
>             LowLevelKeyboardProc lpfn, IntPtr hMod, uint dwThreadId);
> 
>         //用于卸载之前安装的钩子，hhk: 要卸载的钩子句柄（Hook Handle）。该参数是之前通过 SetWindowsHookEx 函数安装钩子时返回的句柄。
>         [DllImport("user32.dll", CharSet = CharSet.Auto, SetLastError = true)]
>         [return: MarshalAs(UnmanagedType.Bool)]
>         public static extern bool UnhookWindowsHookEx(IntPtr hhk);
> 
>         /*在全局钩子过程中传递钩子信息给下一个钩子或目标窗口的过程。为了实现多个钩子函数按顺序处理钩子事件的机制
>          * 钩子就是键盘、鼠标的输入
>         */
>         [DllImport("user32.dll", CharSet = CharSet.Auto, SetLastError = true)]
>         public static extern IntPtr CallNextHookEx(IntPtr hhk, int nCode,
>             IntPtr wParam, IntPtr lParam);
> 
> 
>         //用于获取指定模块的句柄。它接受一个字符串参数 lpModuleName，用于指定要获取句柄的模块名称。
>         [DllImport("kernel32.dll", CharSet = CharSet.Auto, SetLastError = true)]
>         public static extern IntPtr GetModuleHandle(string lpModuleName);
>         #endregion
> 
>         #region 获取窗口信息，获取的坐标将存储在提供的 RECT 结构体对象 lpRect 中，并通过函数的返回值指示是否成功获取矩形区域。
>         [DllImport("user32.dll")]
>         [return: MarshalAs(UnmanagedType.Bool)]
>         public static extern bool GetWindowRect(IntPtr hWnd, out RECT lpRect);
> 
> 
>         //指定结构体在内存中的布局方式。结构体的成员按照定义的顺序一个接一个地连续存储在内存中。
>         [StructLayout(LayoutKind.Sequential)]
>         public struct RECT
>         {
>             public int Left;        // x position of upper-left corner
>             public int Top;         // y position of upper-left corner
>             public int Right;       // x position of lower-right corner
>             public int Bottom;      // y position of lower-right corner
> 
>             public int Width
>             {
>                 get
>                 {
>                     return Right - Left;
>                 }
>             }
>             public int Height
>             {
>                 get
>                 {
>                     return Bottom - Top;
>                 }
>             }
> 
>         }
> 
>         #endregion
> 
>         //函数用于获取指定窗口的标题栏文本长度。它接受一个窗口句柄（hWnd）作为参数，并返回标题栏文本的字符数（长度）
>         [DllImport("user32.dll", CharSet = CharSet.Auto)]
>         public static extern int GetWindowTextLength(IntPtr hWnd);
>         //函数用于获取指定窗口的标题栏文本。返回值是复制到缓冲区中的字符数（不包括空终止符 \0）。
>         [DllImport("user32.dll", CharSet = CharSet.Auto)]
>         public static extern int GetWindowText(IntPtr hWnd, StringBuilder lpString, int nMaxCount);
>     }
> ```
>
> 

> 具体实现方法
>
> ```C#
>         //用于获取指定窗口的子窗口的应用程序名称。
> 该方法通过枚举指定窗口的子窗口，查找与给定进程 ID 匹配的 UWP 应用程序窗口，然后获取该窗口所属进程的完整路径。
>         public static string UWP_AppName(IntPtr hWnd, uint pID)
>         {
>             WINDOWINFO windowinfo = new WINDOWINFO();
>             windowinfo.ownerpid = pID;
>             windowinfo.childpid = windowinfo.ownerpid;
> 
>             IntPtr pWindowinfo = Marshal.AllocHGlobal(Marshal.SizeOf(windowinfo));
> 
>             Marshal.StructureToPtr(windowinfo, pWindowinfo, false);
> 
>             EnumWindowProc lpEnumFunc = new EnumWindowProc(EnumChildWindowsCallback);
>             EnumChildWindows(hWnd, lpEnumFunc, pWindowinfo);
> 
>             windowinfo = (WINDOWINFO)Marshal.PtrToStructure(pWindowinfo, typeof(WINDOWINFO));
>             if (windowinfo.childpid == windowinfo.ownerpid)
>             {
>                 return null;
>             }
>             IntPtr proc;
>             if ((proc = OpenProcess(PROCESS_QUERY_INFORMATION | PROCESS_VM_READ, false, (int)windowinfo.childpid)) == IntPtr.Zero) return null;
> 
>             int capacity = 2000;
>             StringBuilder sb = new StringBuilder(capacity);
>             QueryFullProcessImageName(proc, 0, sb, ref capacity);
> 
>             Marshal.FreeHGlobal(pWindowinfo);
> 
>             return sb.ToString(0, capacity);
>         }
> 
>   private static bool EnumChildWindowsCallback(IntPtr hWnd, IntPtr lParam)
>         {
>             WINDOWINFO info = (WINDOWINFO)Marshal.PtrToStructure(lParam, typeof(WINDOWINFO));
> 
>             uint pID;
>             GetWindowThreadProcessId(hWnd, out pID);
> 
>             if (pID != info.ownerpid) info.childpid = pID;
> 
>             Marshal.StructureToPtr(info, lParam, true);
> 
>             return true;
>         }
> ```
>
> 





## 6、测量字符串控件长度

> ```C#
>  /// <summary>
>         /// 测量字符串控件尺寸
>         /// </summary>
>         /// <param name="textBlock"></param>
>         /// <returns></returns>
>         public static Size MeasureString(TextBlock textBlock)
>         {
>             var formattedText = new FormattedText(
>                 textBlock.Text,
>                 CultureInfo.CurrentCulture,
>                 FlowDirection.LeftToRight,
>                 new Typeface(textBlock.FontFamily, textBlock.FontStyle, textBlock.FontWeight, textBlock.FontStretch),
>                 textBlock.FontSize,
>                 Brushes.Black,
>                 new NumberSubstitution());
> 
>             return new Size(formattedText.Width, formattedText.Height);
>         }
> ```
>
> 





## 7、INotifyPropertyChanged继承作为更新控件的基类

> ```C#
>  public class UINotifyPropertyChanged : INotifyPropertyChanged
>     {
>         public event PropertyChangedEventHandler PropertyChanged;
> 
>         public void OnPropertyChanged([CallerMemberName] string propertyName = "")
>         {
> 
>             PropertyChangedEventHandler handler = PropertyChanged;
>             if (handler != null)
>             {
>                 handler.Invoke(this, new PropertyChangedEventArgs(propertyName));
>             }
> 
>         }
>     }
> //使用 CallerMemberName 特性，可以通过省略属性名称的显式传递，而是在调用 OnPropertyChanged 方法时自动获取调用者的成员名称。这样可以简化代码并减少错误的可能性，因为属性名称会在编译时确定。
> ```
>
> 





## 8、图像和Bitmap的相互转换

> ```C#
>         public static ImageSource ToImageSource(Icon icon)
>         {
>             Bitmap bitmap = icon.ToBitmap();
>             IntPtr hBitmap = bitmap.GetHbitmap();
> 
>             ImageSource wpfBitmap = Imaging.CreateBitmapSourceFromHBitmap(
>                 hBitmap,
>                 IntPtr.Zero,
>                 Int32Rect.Empty,
>                 BitmapSizeOptions.FromEmptyOptions());
> 
>             if (!DeleteObject(hBitmap))
>             {
>                 throw new Win32Exception();
>             }
> 
>             return wpfBitmap;
>         }
> 
>    [DllImport("gdi32.dll", SetLastError = true)]
>         private static extern bool DeleteObject(IntPtr hObject);
> ```
>
> 



## 9、C#获取鼠标位置(X、Y)

> ```C#
> public static Point GetCursorPosition()
>         {
>             POINT lpPoint;
>             GetCursorPos(out lpPoint);
>             return lpPoint;
>         }
> 
>  [DllImport("user32.dll", SetLastError = true)]
>         [return: MarshalAs(UnmanagedType.Bool)]
>         private static extern bool GetCursorPos(out POINT lpPoint);
> 
>  [StructLayout(LayoutKind.Sequential)]
>         public struct POINT
>         {
>             public int X;
>             public int Y;
> 
>             public static implicit operator Point(POINT point)
>             {
>                 return new Point(point.X, point.Y);
>             }
>         }
> ```
>
> 





## 10、MD5加密（Hash）

>  using (var md5 = MD5.Create())
>                   {
>                       var hash = md5.ComputeHash(icon);
>                       var md5Str = BitConverter.ToString(hash).Replace("-", "").ToLowerInvariant();
>
> }
>
> //icon是字节数组类型，返回的hash也是byte[]类型





## 11、设置应用程序开机自启

> 需要依赖包：
>
> Taskscheduler
>
> ```C#
>   public class SystemCommon
>     {
>         /// <summary>
>         /// 设置开机启动
>         /// </summary>
>         /// <param name="startup"></param>
>         /// <returns></returns>
>         public static bool SetStartup(bool startup = true)
>         {
>             string TaskName = "Tai task";
>             var logonUser = System.Security.Principal.WindowsIdentity.GetCurrent().Name;
>             string tai = Path.Combine(
>                   AppDomain.CurrentDomain.BaseDirectory,
>                    "Tai.exe");
>             string taskDescription = "Tai开机自启服务";
> 
>             using (var taskService = new TaskService())
>             {
>                 var tasks = taskService.RootFolder.GetTasks(new System.Text.RegularExpressions.Regex(TaskName));
>                 foreach (var t in tasks)
>                 {
>                     taskService.RootFolder.DeleteTask(t.Name);
>                 }
> 
>                 if (startup)
>                 {
>                     var task = taskService.NewTask();
>                     task.RegistrationInfo.Description = taskDescription;
>                     task.Triggers.Add(new LogonTrigger { UserId = logonUser });
>                     task.Principal.RunLevel = TaskRunLevel.Highest;
>                     task.Actions.Add(new ExecAction(tai, null, AppDomain.CurrentDomain.BaseDirectory));
>                     task.Settings.StopIfGoingOnBatteries = false;
>                     task.Settings.DisallowStartIfOnBatteries = false;
>                     taskService.RootFolder.RegisterTaskDefinition(TaskName, task);
>                 }
>             }
>             return false;
>         }
> 
> 
>     }
> ```
>
> 



## 12、设置VS调试时必须按管理员权限才能启动

> ```md
> 在 项目 上 添加新项 选择“应用程序清单文件” 然后单击 添加 按钮
> 
> 添加后，默认打开app.manifest文件，将：
> 
> <requestedExecutionLevel  level="asInvoker" uiAccess="false" />
> 
> 修改为：
> 
> <requestedExecutionLevel level="requireAdministrator" uiAccess="false" />
> 然后打开 项目属性 ，将 应用程序 标签页中的 资源 中的 清单 修改为新建的 app.manifest。
> 
> 重新生成项目，再次打开程序时就会提示 需要以管理员权限运行。
> ```
>
> 





## 13、Winform查看某个页面是否曾被打开过

> ```C#
>         //判断是否打开过标签
>         private bool IsFormOpened(string formname)
>         {
>             bool formFound = false;
>             ArrayList formList = new ArrayList();
>             IEnumerator enumer = this.DP_Main.Contents.GetEnumerator();
>             while (enumer.MoveNext())
>             {
>                 formList.Add((Form)(enumer.Current));
>             }
>             foreach (Form myForm in formList)
>             {
>                 //Determines whether the specified form is already open
>                 if (myForm.Name == formname)
>                 {
>                     myForm.Activate();
>                     formFound = true;
>                 }
>                 else
>                 {
>                     //To close some need to be closed forms
>                     if (myForm.GetType().BaseType.Name == "CisdiFrame")
>                     {
>                         if (((CISDI.FRAME.CisdiFrame)myForm).FrameType == 0)
>                         {
>                             myForm.Close();
>                         }
>                     }
>                 }
>             }
>             return formFound;
>         }
> ```
>
> 



## 14、全屏幕截屏

> 也是利用了Win32API
>
> 在 Windows 平台上，全屏截图可以使用 `BitBlt` 函数进行。`BitBlt` 函数是 Win32 API 中的一个位图传输函数，用于从一个设备上下文（DC）复制一个矩形区域的位图到另一个设备上下文中。通过使用 `BitBlt` 函数，我们可以实现全屏幕截图。
>
> ```C#
>         #region DllImport
>         [DllImport("gdi32.dll")]
>         public static extern long BitBlt(
>             IntPtr hdcDest, //The handle of the target device environment
>             int nXDest, // The x coordinate of the upper-left corner of the rectangular region of the target device environment
>             int nYDest, // The y coordinate of the upper-left corner of the rectangular region of the target device environment
>             int nWidth, // The value of the width of the rectangular area of the target device environment
>             int nHeight, // The height of a rectangular area of the target device environment value
>             IntPtr hdcSrc, // The handle of source device environment
>             int nXSrc, //The x coordinate of the upper-left corner of the rectangular area of the source device environment
>             int nYSrc, //The y coordinate of the upper-left corner of the rectangular area of the source device environment
>             int dwRop
>         );// Raster Operators        
> 
>         [DllImportAttribute("gdi32.dll")]
>         public static extern IntPtr CreateDC(
>             string lpszDriver, // Driver Name
>             string lpszDevice, // Device Name
>             string lpszOutput, // null
>             IntPtr lpInitData // Arbitrary printer data
>         );
>         #endregion
>         private void button1_Click(object sender, EventArgs e)
>         {
> 
>                 //Screen screenshot function  
>                 IntPtr dc1 = CreateDC("DISPLAY", null, null, (IntPtr)null);
>                 //Create a monitor's DC
>                 Graphics g1 = Graphics.FromHdc(dc1);
>                 //Create a new Graphics object from a specified device's handle
>                 Image MyImage = new Bitmap(Screen.PrimaryScreen.Bounds.Width, Screen.PrimaryScreen.Bounds.Height, g1);
>                 //According to the size of the screen to create a bitmap object with which the same size
>                 Graphics g2 = Graphics.FromImage(MyImage);
>                 //To get the screen handle
>                 IntPtr dc3 = g1.GetHdc();
>                 //To get the bitmap handle
>                 IntPtr dc2 = g2.GetHdc();
>                 //To capture the current screen into the bitmap object
>                 BitBlt(dc2, 0, 0, Screen.PrimaryScreen.Bounds.Width, Screen.PrimaryScreen.Bounds.Height, dc3, 0, 0, 13369376);
>                 //To copy the current screen to the bitmap
>                 g1.ReleaseHdc(dc3);
>                 //Release off the screen handle
>                 g2.ReleaseHdc(dc2);
>                 //Release off the bitmap handle
>                 saveFileDialog1.DefaultExt = "jpg";
>                 saveFileDialog1.Filter = "jpg files (*.jpg)|*.jpg";
>                 saveFileDialog1.Title = "Save Picture As...";
>                 DialogResult result = saveFileDialog1.ShowDialog();
>                 if (result == DialogResult.OK)
>                 {
>                     MyImage.Save(saveFileDialog1.FileName, ImageFormat.Jpeg);
>                 }
>                 g1.Dispose();
>                 MyImage.Dispose();
>                 g2.Dispose();
>             
>         }
> ```
>
> HDC CreateDC(    LPCTSTR lpszDriver,   // 指定设备驱动程序的名称，通常为 NULL 或 "DISPLAY" 表示创建屏幕设备上下文    LPCTSTR lpszDevice,   // 指定设备的名称，通常为 NULL 表示使用默认设备    LPCTSTR lpszOutput,   // 指定设备输出，通常为 NULL 表示使用默认输出    const DEVMODE* lpInitData // 指向一个 DEVMODE 结构，用于设备初始化，通常为 NULL 表示使用默认初始化 );
>
> 通常可以使用 `NULL` 或 `"DISPLAY"` 表示使用默认的显示设备驱动程序。





## 15、打印

>     [DllImport("gdi32.dll")]
>             public static extern long BitBlt(
>                 IntPtr hdcDest, //The handle of the target device environment
>                 int nXDest, // The x coordinate of the upper-left corner of the rectangular region of the target device environment
>                 int nYDest, // The y coordinate of the upper-left corner of the rectangular region of the target device environment
>                 int nWidth, // The value of the width of the rectangular area of the target device environment
>                 int nHeight, // The height of a rectangular area of the target device environment value
>                 IntPtr hdcSrc, // The handle of source device environment
>                 int nXSrc, //The x coordinate of the upper-left corner of the rectangular area of the source device environment
>                 int nYSrc, //The y coordinate of the upper-left corner of the rectangular area of the source device environment
>                 int dwRop
>             );// Raster Operators 
>        private Bitmap screenImage;
>         private void getScreen()
>         {
>             //Get form
>             Graphics ga = this.CreateGraphics();
>             screenImage = new Bitmap(this.ClientRectangle.Width, this.ClientRectangle.Height, ga);
>             //Create a picture used to save
>             Graphics gs = Graphics.FromImage(screenImage);
>             //To get the context of this form equipment
>             IntPtr dcSrc = ga.GetHdc();
>             //The top and bottom of the bitmap devices
>             IntPtr dcDest = gs.GetHdc();
>             //Copy of the bitmap
>             BitBlt(dcDest, 0, 0, this.ClientRectangle.Width,
>             this.ClientRectangle.Height, dcSrc, 0, 0, 13369376);
>             //Release form equipment
>             ga.ReleaseHdc(dcSrc);
>             //Release the bitmap devices
>             gs.ReleaseHdc(dcDest);
>         }
>     
>         private void tsb_Print_Click(object sender, EventArgs e)
>         {
>             getScreen();
>             this.printDialog1.Document = this.printDocument1;
>             //Show the print window
>             if (this.printDialog1.ShowDialog() == DialogResult.OK)
>             {
>                 try
>                 {
>                     this.printDocument1.Print();
>                 }
>                 catch (Exception excep)
>                 {
>                     MessageBox.Show(excep.Message, "Print Error", MessageBoxButtons.OK, MessageBoxIcon.Error);
>                 }
>             }
>         }





## 16、前台窗口切换触发事件

>     public class AppObserver : IAppObserver
>     {
>         private IntPtr hook;
>         Win32API.WinEventDelegate winEventDelegate;
>         public AppObserver()
>         {
>             winEventDelegate = new Win32API.WinEventDelegate(WinEventProc);
>         }
>         private void WinEventProc(IntPtr hWinEventHook, uint eventType, IntPtr hwnd, int idObject, int idChild, uint dwEventThread, uint dwmsEventTime)
>         {
>             Handle(hwnd);
>         }
>     
>         public event ObserverEventHandler OnAppActive;
>                 
>         public  void Start()
>         {
>             hook = Win32API.SetWinEventHook(0x0003, 0x0003, IntPtr.Zero,
>             winEventDelegate, 0, 0, 0);  //注册一个系统事件监听器
>                 
>             var window = Win32API.GetForegroundWindow();
>              Handle(window); //核心方法
>         }
>         /// <summary>
>         /// ①根据窗口句柄拿到进程ID，②根据进程ID拿到进程句柄，③根据进程句柄拿到进程文件夹路径
>         /// </summary>
>         /// <param name="hwnd"></param>
>         private async void Handle(IntPtr hwnd)
>         {
>             string processName = string.Empty;
>             string processFileName = string.Empty;
>             string processDescription = string.Empty;
>                 
>             await Task.Run(() =>
>             {
>                 string[] processInfo = GetProcessInfoByWindowHandle(hwnd);
>                 int processID = int.Parse(processInfo[2]);
>                 processName = processInfo[0];
>                 processFileName = processInfo[1];
>                 if (processFileName.IndexOf("ApplicationFrameHost.exe") != -1)
>                 {
>                     processFileName = Win32API.UWP_AppName(hwnd, (uint)processID);
>                     if (processFileName == null)
>                     {
>                         Thread.Sleep(1500);
>                 
>                         processFileName = Win32API.UWP_AppName(hwnd, (uint)processID);
>                         processName = GetUWPAPPName(processFileName);
>                 
>                         IntPtr foregeroundHwnd = Win32API.GetForegroundWindow();
>                 
>                         if (foregeroundHwnd != hwnd)
>                         {
>                             //    获取后焦点变了
>                             processInfo = GetProcessInfoByWindowHandle(foregeroundHwnd);
>                             processID = int.Parse(processInfo[2]);
>                             processName = processInfo[0];
>                             processFileName = processInfo[1];
>                         }
>                 
>                     }
>                     else
>                     {
>                         processName = GetUWPAPPName(processFileName);
>                     }
>                 }
>                 FileVersionInfo info = FileVersionInfo.GetVersionInfo(processFileName);
>                 processDescription = info.FileDescription;
>                 int titleCapacity = Win32API.GetWindowTextLength(hwnd) * 2;
>                 StringBuilder stringBuilder = new StringBuilder(titleCapacity);
>                 Win32API.GetWindowText(hwnd, stringBuilder, stringBuilder.Capacity);
>             });
>                        
>             EventInvoke(processName, processDescription, processFileName, hwnd);
>         }
>                 
>         private void EventInvoke(string processName, string description, string filename, IntPtr handle)
>         {
>             var args = new AppObserverEventArgs(processName, description, filename, handle);
>             OnAppActive?.Invoke(args);
>         }
>                 
>         private string GetUWPAPPName(string processFileName)
>         {
>             string processName = string.Empty;
>             if (!string.IsNullOrEmpty(processFileName) && processFileName.IndexOf("\\") != -1)
>             {
>                 processName = processFileName.Split('\\').Last();
>                 processName = processName.Replace(".exe", "");
>             }
>             return processName;
>         }
>                 
>         //根据窗口可以获得进程id，根据进程id可以获取进程
>         private string[] GetProcessInfoByWindowHandle(IntPtr hwnd)
>         {
>             string processName, processFileName = string.Empty;
>                 
>             int processID = 0;
>                 
>             Win32API.GetWindowThreadProcessId(hwnd, out processID);
>                 
>             Process process = Process.GetProcessById(processID);
>             processName = process.ProcessName;
>             IntPtr processHandle = IntPtr.Zero;
>             processHandle = Win32API.OpenProcess(0x001F0FFF, false, processID);
>                 
>             if (processHandle != IntPtr.Zero)
>             {
>                 int maxLength = 4096;
>                 var buffer = new StringBuilder(maxLength);
>                 Win32API.QueryFullProcessImageName(processHandle, 0, buffer, ref maxLength);
>                 processFileName = buffer.ToString(); //进程文件路径
>             }
>             else
>             {
>                 MessageBox.Show("无法获取进程句柄");
>             }
>             return new string[] { processName, processFileName, processID.ToString() };
>         }
>                 
>         public void Stop()
>         {
>             Win32API.UnhookWinEvent(hook);
>         }
>     }



> Win32触发事件回调
>
> ```C#
>     public class Win32API
>     {
>         public delegate void WinEventDelegate(IntPtr hWinEventHook, uint eventType,
>          IntPtr hwnd, int idObject, int idChild, uint dwEventThread, uint dwmsEventTime);
> 
>         //安装一个事件钩子，以便在系统中指定的事件发生时接收通知。
>         [DllImport("user32.dll")]
>         public static extern IntPtr SetWinEventHook(uint eventMin, uint eventMax, IntPtr
>           hmodWinEventProc, WinEventDelegate lpfnWinEventProc, uint idProcess,
>           uint idThread, uint dwFlags);
> 
>         //返回当前活动窗口的句柄
>         [DllImport("user32.dll")]
>         public static extern IntPtr GetForegroundWindow();
> 
>         //根据句柄获得线程id
>         [DllImport("user32.dll", CharSet = CharSet.Auto)]
>         public static extern int GetWindowThreadProcessId(IntPtr hwnd, out int ID);
> 
>         //打开一个已存在的进程，并返回该进程的句柄（HANDLE）。
>         [DllImport("kernel32.dll", SetLastError = true)]
>         public static extern IntPtr OpenProcess(uint dwDesiredAccess, bool bInheritHandle, int dwProcessId);
> 
> 
>         [DllImport("kernel32.dll", SetLastError = true)]
>         public static extern bool CloseHandle(UIntPtr hObject);
> 
>         //获取进程所在文件夹路径
>         [DllImport("kernel32.dll", SetLastError = true, CharSet = CharSet.Auto)]
>         public static extern bool QueryFullProcessImageName([In] IntPtr hProcess, [In] int dwFlags, [Out] StringBuilder lpExeName, ref int lpdwSize);
> 
> 
>         //private const uint WINEVENT_OUTOFCONTEXT = 0;
>         //private const uint EVENT_SYSTEM_FOREGROUND = 3;
>         //用于获取指定窗口的子窗口的应用程序名称。
>         public static string UWP_AppName(IntPtr hWnd, uint pID)
>         {
>             WINDOWINFO windowinfo = new WINDOWINFO();
>             windowinfo.ownerpid = pID;
>             windowinfo.childpid = windowinfo.ownerpid;
> 
>             IntPtr pWindowinfo = Marshal.AllocHGlobal(Marshal.SizeOf(windowinfo));
> 
>             Marshal.StructureToPtr(windowinfo, pWindowinfo, false);
> 
>             EnumWindowProc lpEnumFunc = new EnumWindowProc(EnumChildWindowsCallback);
>             EnumChildWindows(hWnd, lpEnumFunc, pWindowinfo);
> 
>             windowinfo = (WINDOWINFO)Marshal.PtrToStructure(pWindowinfo, typeof(WINDOWINFO));
>             if (windowinfo.childpid == windowinfo.ownerpid)
>             {
>                 return null;
>             }
>             IntPtr proc;
>             if ((proc = OpenProcess(PROCESS_QUERY_INFORMATION | PROCESS_VM_READ, false, (int)windowinfo.childpid)) == IntPtr.Zero) return null;
> 
>             int capacity = 2000;
>             StringBuilder sb = new StringBuilder(capacity);
>             QueryFullProcessImageName(proc, 0, sb, ref capacity);
> 
>             Marshal.FreeHGlobal(pWindowinfo);
> 
>             return sb.ToString(0, capacity);
>         }
> 
>         public const UInt32 PROCESS_QUERY_INFORMATION = 0x400; //检索有关进程的某些信息
>         public const UInt32 PROCESS_VM_READ = 0x010;  //读取进程中的内存
> 
> 
>         //使用 EnumChildWindows 函数可以遍历指定父窗口下的所有子窗口，并通过回调函数对每个子窗口进行处理，
>         //例如获取子窗口的句柄、属性、样式等信息，或者执行其他操作。
>         [DllImport("user32", SetLastError = true)]
>         [return: MarshalAs(UnmanagedType.Bool)]
>         public static extern bool EnumChildWindows(IntPtr hWndParent, EnumWindowProc lpEnumFunc, IntPtr lParam);
> 
>         public delegate bool EnumWindowProc(IntPtr hWnd, IntPtr parameter);
> 
>         /// <summary>
>         /// 定义父子进程的结构体
>         /// </summary>
>         internal struct WINDOWINFO
>         {
>             public uint ownerpid;
>             public uint childpid;
>         }
> 
>         /// <summary>
>         /// 用于枚举指定窗口的子窗口。
>         /// </summary>
>         /// <param name="hWnd"></param>
>         /// <param name="lParam"></param>
>         /// <returns></returns>
>         private static bool EnumChildWindowsCallback(IntPtr hWnd, IntPtr lParam)
>         {
>             WINDOWINFO info = (WINDOWINFO)Marshal.PtrToStructure(lParam, typeof(WINDOWINFO));
> 
>             int pID;
>             GetWindowThreadProcessId(hWnd, out pID);
> 
>             if (pID != info.ownerpid) info.childpid = (uint)pID;
> 
>             Marshal.StructureToPtr(info, lParam, true);
> 
>             return true;
>         }
> 
> 
>         [DllImport("user32.dll")]
>         public static extern bool UnhookWinEvent(IntPtr hWinEventHook);
> 
>         [DllImport("user32.dll", CharSet = CharSet.Auto)]
>         public static extern int GetWindowTextLength(IntPtr hWnd);
> 
>         //函数用于获取指定窗口的标题栏文本。返回值是复制到缓冲区中的字符数（不包括空终止符 \0）。
>         [DllImport("user32.dll", CharSet = CharSet.Auto)]
>         public static extern int GetWindowText(IntPtr hWnd, StringBuilder lpString, int nMaxCount);
>     }
> ```
>
> 0x0003代表前景窗口已更改事件。





## 17、xaml定义快捷键的方法

> 在 XAML 中，`_` 符号通常用于定义快捷键（Access Key）。在上述示例中，`Content="_Save"` 中的 `_` 就是用来定义按钮的快捷键，也称为访问键或助记键。
>
> 快捷键用于在用户界面中提供键盘快捷方式，使用户可以通过按下特定的组合键来触发按钮或其他控件的操作，而无需使用鼠标点击。在 Windows 操作系统中，通常使用 Alt 键来激活快捷键。
>
> 在 `Content` 属性中的 `_Save`，其中的 `_` 表示后面的字符 "S" 将成为按钮的快捷键。在用户界面中，通常这个快捷键将会被标示为下划线 "S"，表示按下 Alt + S 时可以触发按钮的点击事件。
>
> ```xaml
> <Button Content="_Save"/> 
> ```
>
> 





## 18、Helper类连接数据库，增删改查操作

> MysqlHelper类
>
> 利用nuget里的mysql.data连接操作数据库
>
> ```C#
> using System;
> using System.Collections;
> using System.Collections.Specialized;
> using System.Data;
> using MySql.Data.MySqlClient;
> using System.Configuration;
> using System.Data.Common;
> using System.Collections.Generic;
> using System.Text;
> 
> namespace TestStudent
> {
>     public static class MySqlHelper
>     {
>         #region 分页
>         /// <summary>
>         /// row_number()分页(MySqlDataReader)
>         /// </summary>
>         /// <param name="pageSize">取数据条数</param>
>         /// <param name="currentPage">当前页</param>
>         /// <param name="para">示例 "字段列表", "表名", "条件", "排序"</param>
>         /// <returns></returns>
>         public static DataTable GetPageList(string connectionString, int currentPage, int pageSize, string[] para, out int recordCount)
>         {
>             StringBuilder sbSql = new StringBuilder();
> 
>             sbSql.AppendFormat("select {0} from {1} ", para[0], para[1]);
>             if (!string.IsNullOrEmpty(para[2]))
>             {
>                 sbSql.AppendFormat("where {0} ", para[2]);
>             }
> 
>             if (!string.IsNullOrEmpty(para[3]))
>             {
>                 sbSql.AppendFormat("order by {0} ", para[3]);
>             }
> 
>             sbSql.AppendFormat("limit {0},{1}", pageSize * (currentPage - 1), pageSize);
> 
>             //计算记录数
>             StringBuilder sbCount = new StringBuilder();
>             sbCount.AppendFormat("select count(1) from {0} ", para[1]);
>             if (!string.IsNullOrEmpty(para[2]))
>             {
>                 sbCount.AppendFormat("where {0} ", para[2]);
>             }
> 
>             recordCount = Convert.ToInt32(ExecuteScale(connectionString, sbCount.ToString(), null));
> 
>             return ExecuteDataTable(connectionString, sbSql.ToString(), null);
>             //return ExecuteReader(strSql);
>         }
>         #endregion
> 
>         #region  执行简单SQL语句
>         /// <summary>
>         /// 执行SQL语句，返回影响的记录数
>         /// </summary>
>         /// <param name="SQLString">SQL语句</param>
>         /// <returns>影响的记录数</returns>
>         public static int ExecuteNonQuery(string connectionString, string sqlString, MySqlParameter[] parameters)
>         {
>             using (MySqlConnection connection = new MySqlConnection(connectionString))
>             {
>                 using (MySqlCommand cmd = new MySqlCommand(sqlString, connection))
>                 {
>                     connection.Open();
>                     if (parameters != null)
>                     {
>                         cmd.Parameters.AddRange(parameters);
>                     }
> 
>                     int rows = cmd.ExecuteNonQuery();
>                     return rows;
>                 }
>             }
>         }   
> 
>         /// <summary>
>         /// 执行查询语句，返回DataSet
>         /// </summary>
>         /// <param name="SQLString">查询语句</param>
>         /// <returns>DataSet</returns>
>         public static DataSet ExecuteDataSet(string connectionString, string sql, MySqlParameter[] parameters)
>         {
>             using (MySqlConnection connection = new MySqlConnection(connectionString))
>             {
>                 DataSet ds = new DataSet();
> 
>                 connection.Open();
> 
>                 MySqlCommand cmd = new MySqlCommand(sql, connection);
> 
>                 if (parameters != null)
>                 {
>                     cmd.Parameters.AddRange(parameters);
>                 }
> 
>                 MySqlDataAdapter da = new MySqlDataAdapter(cmd);
> 
>                 da.Fill(ds);
> 
>                 return ds;
>             }
>         }
> 
>         /// <summary>
>         /// 执行查询语句，返回DataSet
>         /// </summary>
>         /// <param name="SQLString">查询语句</param>
>         /// <returns>DataTable</returns>
>         public static DataTable ExecuteDataTable(string connectionString, string sql, MySqlParameter[] parameters)
>         {
>             using (MySqlConnection connection = new MySqlConnection(connectionString))
>             {
>                 DataTable dt = new DataTable();
> 
>                 connection.Open();
> 
>                 MySqlCommand cmd = new MySqlCommand(sql, connection);
> 
>                 if (parameters != null)
>                 {
>                     cmd.Parameters.AddRange(parameters);
>                 }
> 
>                 MySqlDataAdapter da = new MySqlDataAdapter(cmd);
> 
>                 da.Fill(dt);
> 
>                 return dt;
>             }
>         }
> 
>         public static object ExecuteScale(string connectionString, string sql, MySqlParameter[] parameters)
>         {
>             using (MySqlConnection conn = new MySqlConnection(connectionString))
>             {
>                 DataTable dt = new DataTable();
>                 conn.Open();
>                 MySqlCommand cmd = new MySqlCommand(sql, conn);
>                 if (parameters != null)
>                 {
>                     cmd.Parameters.AddRange(parameters);
>                 }
> 
>                 return cmd.ExecuteScalar();
>             }
>         }
>         #endregion
>     }
> }
> 
> ```
>
> 
>
> 实现增删改查
>
> ```C#
> using System;
> using System.Collections.Generic;
> using System.Linq;
> using System.Text;
> using System.Data;
> using MySql.Data.MySqlClient;
> 
> namespace TestStudent
> {
>     public class Student
>     {        
>         public int Id { get; set; }        
>         public string Name { get; set; }
>         public string Gender { get; set; }
>     }
> 
>     public class StudentDAO
>     {
> //        create table t_student
> //(
> //  id varchar(50) primary key,
> //  name varchar(50) ,
> //  address varchar(50)
> //)
>         public static  List<Student> QueryAll()
>         {
>             List<Student> list = new List<Student>();
>             string sql = "select * from student_info";
>             foreach (DataRow row in MySqlHelper.ExecuteDataTable("server=127.0.0.1;database=student;uid=root;pwd=lxw19980714;", sql, null).Rows)
>             {
>                 Student student = new Student();
>                 student.Id = Convert.ToInt32(row["id"]);
>                 student.Name = Convert.ToString(row["name"]);
>                 student.Gender = Convert.ToString(row["Gender"]);
> 
>                 list.Add(student);
>             }
> 
>             return list;
>         }
> 
>         public static void Insert(Student student)
>         {
>             string sql =string.Format("insert into student_info(Id,Name,Gender) values('{0}','{1}','{2}')", student.Id,student.Name,student.Gender);
>             MySqlHelper.ExecuteNonQuery("server=127.0.0.1;database=student;uid=root;pwd=lxw19980714;", sql, null);
>         }
> 
>         public static void Update(Student student)
>         {
>             string sql = string.Format("update student_info set Name='{0}',Gender='{1}' where Id='{2}'", student.Name,student.Gender, student.Id);
>             MySqlHelper.ExecuteNonQuery("server=127.0.0.1;database=student;uid=root;pwd=lxw19980714;", sql, null);
>         }
> 
>         public static void Delete(string id)
>         {
>             string sql = "delete from student_info where Id=" + id;
>             MySqlHelper.ExecuteNonQuery("server=127.0.0.1;database=student;uid=root;pwd=lxw19980714;", sql, null);
>         }
> 
> 
> 
>     }
> }
> 
> ```
>
> 





## 19、Orm框架中泛型实体作为基类（都有ID主键）

> ```C#
>   /// <summary>
>     /// 泛型实体基类
>     /// </summary>
>     /// <typeparam name="TPrimaryKey">主键</typeparam>
>     public abstract class EntityBase<TPrimaryKey>
>     {
>         /// <summary>
>         /// 主键
>         /// </summary>
>         public virtual TPrimaryKey Id { get; set; }
> 
>         public override bool Equals(object obj)
>         {
>             return base.Equals(obj);
>         }
> 
>         public override int GetHashCode()
>         {
>             return base.GetHashCode();
>         }
>     }
> 
>     #region Implements
> 
>     /// <summary>
>     /// Guid 类型主键实体基类
>     /// </summary>
>     public abstract class EntityBase : EntityBase<Guid>
>     { }
> 
>     #endregion Implements
> ```
>
> 
