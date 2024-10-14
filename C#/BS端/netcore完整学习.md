# .Net Core（杨中科）

> net core 为云计算而生，但不止于云
>
> 优点：
>
> 1. 支持独立部署、不互相影响
> 2. 彻底模块化
> 3. 无历史包袱，运行效率高
> 4. 不依赖于IIS
> 5. 跨平台
> 6. 有依赖注入、单元测试等优点
>
> 以前程序比如winform用net framework下需要装.net framework 环境
>
> 而现在用.net core 可以直接独立发布运行。



> 1、MVC介绍
>
> 控制器作用：
>
> 用户页面上发送信息给控制器，控制器收到请求（request）经过数据处理后再返回给浏览器
>
> 控制器（controller）里面写了许多方法
>
> 当一个请求来时，控制器会触发一个相应的方法，此方法可能返回view，可能返回数据，或许两者都返回

> Model作用：
>
> 拥有属性，传递给控制器数据，或是从控制器拿数据
>
> 模型也被用于服务端数据验证，或有时候也能客户端验证

> View:
>
> 是html代码和C# 代码的结合 .cshtml







> 2、如何把控制台应用程序转为web应用
>
> host builder
>
> 两大方法:CreateHostBuilder & ConfigureWebHostDefaults
>
> * 改配置文件<Project Sdk="Microsoft.NET.Sdk.Web">





## 1、Nuget

> 软件下载依赖趋势
>
> Linux:apt、yum
>
> Js:npm
>
> Java:Maven、Gradle
>
> Python:pip

> Nuget 两种使用方式：
>
> 1、CLI
>
> 2、图形界面
>
> 推荐CLI方式
>
> install-package
>
> uninstall-package
>
> 项目依赖中<ItemGroup><PackageReference Include=""/></ItemGroup>里有对包的引用





## 2、异步编程

> 异步 = 不等待
>
> 传统多线程开发很麻烦：
>
> 引入 关键字：async、await
>
> async、await 不等于 “多线程”
>
> ![image-20221009120955816](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20221009120955816.png)

> async 和 await 一般成对出现
>
> 但可以用.Result  就可以不用 await 来得到 异步方法返回的结果
>
> 但.Result本身并不是异步方法，此方法会等待结果出来后执行（所以还是同步）
>
> #### 注意用.Result会有死锁的风险
>
> ![image-20221009124313128](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20221009124313128.png)





> 异步的Lambda表达式
>
> ```C#
> Task.Run(async(obj)=>{
>     await File.ReadAllTextAsync("e:\1.txt","hello");
> });
> ```
>
> 



> Async、Await 内部原理
>
> Async、Await 是语法糖，最终编译成 状态机
>
> 总结：
>
> > async方法会被C# 编译器编译成一个类，会根据await调用拆分为多个状态，对async的调用会被拆分为
> >
> > 对MoveNext的调用
> >
> > 用await看似是等待，经过编译后，其实没有wait，干别的事去了



> ### 异步方法不等于新线程
>
> 异步方法的代码并不会自动在新线程中执行，除非把代码放到新线程中。

> ### 为什么有的异步方法没标Async
>
> 此种情况只能应用于异步函数没有其他额外的业务逻辑时，只是简单的讲异步的返回值返回给调用者。
>
> 其实就是复杂度低可以不用async

> ### 不要用Sleep,会阻塞，Thread.Sleep()是在UI线程上执行的
>
> 因为它会阻塞调用线程，要用await Task.Delay() 

---

> 有时需要提前终止任务，比如：请求超时、用户取消请求。
>
> ### 很多异步方法都有CancellationToken参数，用于获得提前终止执行的信号。
>
> > CancellationToken是一个结构体
> >
> > 下面代码实现目标：超过10S下载不完就提前终止
> >
> > ```c#
> > public static async Task DownloadWebPage(string url,string n,CancellationToken cancellationToken)
> > {
> >     using(HttpClient hc = new HttpClient())
> >     {
> >         string html = await hc.GetStringAsync();
> >         for(int i=0;i<n;i++)
> >         {
> >             if(cancellationToken.IsCancellationRequested)
> >             {
> >                 console.writeline("请求被取消");
> >                 break;
> > }
> > }
> > }
> >     
> >     //下面是定义一个CancellationToken
> >     CancellationTokenSource cts=new CancellationTokenSource();
> >     cts.CancelAfter(5000);
> >     CancellationToken cToken=cts.Token;
> >     //定时定为5S还不结束就终止
> > ```
> >
> > 



> ### WhenAll 的用法
>
> ![image-20221009170925799](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20221009170925799.png)

---









## 3、LINQ 

![image-20221010111240744](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20221010111240744.png)

> 委托->Lambda->Linq
>
> ```C#
>     static void Main(string[] args)
>         {
>             List<int> nums = new List<int>() { 1, 10, 100 };
>            var s=  nums.Where(IsOk).ToList();
>             Console.Read();
>         }
> 
>       public static  bool IsOk(int i)
>         {
>             return i > 10 ? true : false;
>         }
> ```
>
> 

> C#中的var和js中的不一样，仍然是强类型，C#中的弱类型是==dynamic==
>
> Linq里都是 扩展方法 （IEnumable的扩展方法）
>
> 排序方法：
>
> Order()——默认正序排序
>
> OrderByDescending()——倒序
>
> 多层级排序：
>
> var s = items.order((a=>a.salary<10000).ThenBy(a=>a.age>40));

> 数据库中的Linq方法
>
> 聚合函数：max、min、average、sum、count

> .select 可以用投影方法过滤数据
>
> 其实 linq 默认转化为IEnumerable，也有转成List的ToList()方法

> 上述的Linq是方法语法，下面是查询语法（类SQL）
>
> ```C#
> var items = from e in list where e.salary>3000 orderby e.Age select new {e.Name,e.Age};
> ```



> Linq面试
>
> Linq大部分时间不影响性能







## 4、依赖注入、控制反转

> 依赖注入 是 控制反转思想的实现方式
>
> 依赖注入 简化模块组装过程，降低模块的耦合度
>
> ### 简单说：要了就给你
>
> 我要，你就给我

> 两种实现方式：
>
> 1、服务定位器（ServiceLocator）
>
> 2、依赖注入（Dependency Injection）

![image-20221011115521551](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20221011115521551.png)





> ```C#
> 需要安装
>     using Microsoft.Extensions.DependencyInjection;
> 
>  static void Main(string[] args)
>         {
>             ServiceCollection services = new ServiceCollection();
>             services.AddTransient<TestServiceImpl>();
>             using(ServiceProvider sp = services.BuildServiceProvider())
>             {
>                 TestServiceImpl testServiceImpl = sp.GetService<TestServiceImpl>();
>                 testServiceImpl.DoSomething();
>                 Console.Read();
>             }
>         }
> ```
>
> 



> 生命周期
>
> ![image-20221011130323130](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20221011130323130.png)





> ![image-20221011151450911](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20221011151450911.png)





> .net DI 默认使用的是==构造函数注入==，Java有属性注入

```C#
  static void Main(string[] args)
        {
            ServiceCollection services = new ServiceCollection();
            services.AddTransient<Controller>();
            services.AddTransient<ILog,Log>();
            services.AddTransient<IStorage,StorageImpl>();
            services.AddTransient<IConfig,ConfigImpl>();
            using (var s = services.BuildServiceProvider())
            {
             var c =   s.GetRequiredService<Controller>();
                c.Test();
            }
            Console.ReadLine();
        }

 internal interface IConfig
    {
        string GetValue(string name);
    }
 internal class ConfigImpl : IConfig
    {
        public string GetValue(string name)
        {
            return ($"我是config，我的name是{name}");
        }
    }

   internal interface IStorage
    {
        void Save(string content,string name);
    }

   internal class StorageImpl : IStorage
    {
        private readonly IConfig _config;

        public StorageImpl(IConfig config)
        {
            this._config = config;
        }

        public void Save(string content, string name)
        {
           string s = _config.GetValue("123");
            Console.WriteLine($"向服务器{s}上传的文件名是{name}，内容是{content}");
        }
    }

    internal interface ILog
    {
         void Log(string message);
    }

    internal class Log : ILog
    {
        void ILog.Log(string message)
        {
            Console.WriteLine($"我是log，Msg是{message}");
        }
    }

   internal class Controller
    {
        private ILog _log;
        private IStorage _storage;
        public Controller(ILog log, IStorage storage)
        {
            _log = log;
            _storage = storage;
        }

        public void Test()
        {
            _log.Log("开始上传");
            _storage.Save("你好吗","第一个文件");
            _log.Log("上传完毕");

        }
    }
```







## 5、配置系统

![image-20221013160332974](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20221013160332974.png)





### 1、如何在vs中使用json配置文件

> 先新建config.json，nuget包安装Microsoft.Extensions.Configuration 和 Microsoft.Extensions.Configuration.Json.dll
>
> Microsoft.Extensions.Configuration——这个包就涵盖所有配置文件的方法

> ```C#
>   ConfigurationBuilder configurationBuilder = new ConfigurationBuilder();
>             configurationBuilder.AddJsonFile(@"config.json",optional:true,reloadOnChange:true);
>             // optional 表示这个配置文件是否可选 reloadOnChange 表示文件修改后是否重新加载配置
>             IConfigurationRoot configuration = configurationBuilder.Build();
>             string name = configuration["name"];
>             Console.WriteLine(name);
>             Console.Read();
> 
> ```
>
> 

> 还可以直接建一个和json文件格式一样的类来解析
>
> ```c#
> class Proxy
> {
>     string address;
>     int port;
> }
> 
> int main(){
>     IConfigurationRoot configuration = configurationBuilder.Build();
>     Proxy proxy = configuration.GetSection("proxy").Get<Proxy>();
> }
> ```
>
> 

### 2、更方便、普遍的方式——选项方式读取

> 和DI结合
>
> Nuget安装Microsoft.Extensions.Options 和 Microsoft.Extensions.Configuration.Binder 
>
> 当然也要安装Microsoft.Extensions.Configuration 和 Microsoft.Extensions.Configuration.Json





### 省略，看到41 后面不会了，直接net core看



## 6、.Net Core

> 1、MVC
>
> MVC其实包含.net core webApi
>
> 过去程序必须跑在iis上，现在默认跑在自己项目的代理服务器上。
>
> MVC模式
>
> 一个控制器 Test文件夹下的—— TestController——写对模型的处理 return View(Model);
>
> 对应 Views文件夹下——新建Test 页面——把模型导入视图
>
> ![image-20221026084746942](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20221026084746942.png)

> 主流还是前后端分离 目前





> 2、.net core web api
>
> 其实就是前后端分离，创建项目时没有views文件夹，web api 只提供数据
>
> Swagger就是 微软提供的一个供前后端分离项目调试接口的工具





> 3、Rest
>
> Web Api有两种风格：面向过程（RPC），面向Rest（REST）
>
> RPC：控制器/操作方法的形式把服务器端代码当成方法去调用。不关心http谓词，比如/Person/GetAll,/Persons/GetById?Id=8，有QueryString（有长度限制）.风格自然
>
> REST:按照http语言来使用http协议，对开发人员要求更高
>
> * URL用于资源定位：/user/888、/user/888/orders
> * HTTP谓词：Get、Post、Put（整体更新）、Delete、Patch（局部更新）、Option
> * 什么是“幂等”？Delete、Put、Get是幂等，Post不幂等 幂等：动作发生一次和发生多次的结果是一样的
> * Get响应可以被缓存
> * 服务端要通过状态码来反映资源获取的结果。201（新增成功）、403、404（没有权限）、

> Rest的优缺点：
>
> 优点：
>
> :one:语义更清晰
>
> :two:通过http谓词表示不同操作，接口自描述
>
> :three:可以用get请求做缓存
>
> :four:通过http状态码反映服务器端的处理结果，统一错误处理机制
>
> :five:网关等可以分析请求处理结果
>
> 缺点：
>
> :one:真实资源复杂，很难划分，对开发人员要求高
>
> :two:不是所有操作都能简单对应到http谓词中
>
> :three:系统进化可能会改变幂等性
>
> :four:http状态码个数有限
>
> :five:有些环节会篡改非200响应码的响应报文
>
> :six:有的客户端不支持Put、Delete请求





> Url：资源定位
>
> QueryString：Url之外的额外数据
>
> 请求报文体：供PUT、POST提供数据



> 400派观点
> 网关等可以监控HTTP状态码，如果接口频繁出现4xx状态码，那么就说明客户端的代码不完善 2、很多的系统都是按照HTTP状态码的不同含义进行设计的。如果失败了服务器返回的状态码还是200的话，这会违背软件设计的初衷
>
> 200派观点
> 网络的问题归网络、业务的问题归业务。业务错误不应该和技术错误混在一起。把系统日志和业务日志区分开





> 实例：
>
> ![image-20221026105837309](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20221026105837309.png)

------------------------------------------------
```c#
 [HttpGet(Name = "GetWeatherForecast")]
        public IEnumerable<WeatherForecast> Get()
        {
            return Enumerable.Range(1, 5).Select(index => new WeatherForecast
            {
                Date = DateTime.Now.AddDays(index),
                TemperatureC = Random.Shared.Next(-20, 55),
                Summary = Summaries[Random.Shared.Next(Summaries.Length)]
            })
            .ToArray();
        }
```

> 控制器里每个方法都要写httpGet、httpPut等，否则会报错，打不开swagger页面
>
> 控制器不一定一定要继承Controller / ControllerBase，但最好这么做





> Action方法
>
> Action方法既可以同步也可以异步，异步action一般不需要Async结尾
>
> Web Api中Action方法的返回值如果是普通数据类型，那么返回值就会默认被序列化为Json问个事
>
> Web Api中的Action方法返回值同样支持IActionResult类型，不包含类型信息，因此Swagger等无法推断出类型，所以推荐用ActionResult<T>，它支持类型转换，用起来简单



```C#
[HttpGet]
public IActionResult GetById(int id)
  {

            if (id == 1)
                return Ok("成功访问");
            else
                return NotFound("没有成功");
        }

//IActionResult 可以返回 服务器状态码

   [HttpGet]
        public ActionResult<int> GetById(int id)

        {

            if (id == 1)
                return 1;
            else
                return 2;
        }
//也可以用ActionResult泛型类型
```





> ![image-20221026131049778](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20221026131049778.png)







> 4、前后端分离
>
> 后端只写Web Api
>
> 页面由前端人员负责

```c#
测试样例
     [Route("[Controller]/[Action]")]
    [ApiController]
    public class LoginController : Controller
    {
        [HttpGet]
        public IActionResult Index()
        {
            return View();
        }

        [HttpPost]
        public LoginResponse Login(LoginRequest req)
        {
            if(req.UserName=="lxw"&&req.PassWord=="123456")
            {
                var items = Process.GetProcesses().Select(p => new ProcessInfo(p.Id, p.MachineName, p.WorkingSet64));
                return new LoginResponse(true,items.ToArray()); 
            }
            else
            {
                return new LoginResponse(false, null);
            }
        }
    }

    public record LoginRequest(string UserName,string PassWord);
    public record ProcessInfo(long  Id,string Name,long WorkingSet);
    public record LoginResponse(bool Ok, ProcessInfo[]  ProcessInfos);
```

