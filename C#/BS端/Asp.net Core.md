ASP.NetCore Learning



## 一、基本知识

> 1、AspNetCore默认采用外部托管<AspNetCoreHostingModel>OutOfProcess</AspNetCoreHostingModel> 
>
> | 进程内托管                         | 进程外托管                            |
> | ---------------------------------- | ------------------------------------- |
> | 进程名：w3wp,exe or iisexpress.exe | dotnet.exe                            |
> | 只有一个服务器                     | 两台服务器                            |
> | 性能更好                           | 在内部和外部Web服务器间代理请求的损耗 |
>
> 整个应用程序运行在Asp.NetCore内置的Kestrel中。

> 2、Main方法作为启动入口

> 3、CreateDefaultBuilder()执行：设置web服务器、加载主机和应用程序配置表信息、配置日志记录

> 4、.netCore加载文件——LaunchSetting.Json
>
> 本地相关的配置信息

> 5、.Net Core中的配置源：优先级从上至下
>
> * appSettings.Json appsettings.{Environment}.json,不同环境下对应不同的托管环境
> * User secrets——用户机密
> * Environment variables——环境变量
> * Command-line arguments——命令行参数

> 6、NetCore中的中间件
>
> Logging、StaticFiles、MVC 回环
>
> * 可同时被访问和请求
> * 可以处理请求后，将请求传给下一个中间件
> * 可以处理请求后，并将管道短路
> * 可以处理传出响应
> * 中间件是按照添加顺序执行的

> 7、app.run是终端中间件不能有多个，否则会引起管道短路，只显示第一个的内容

> 8、ASP默认不支持静态文件。要想运行html、css、png等形式，需要添加静态文件。
>
> 注意定义时的顺序。default要在静态之前
>
> app.UseStaticFiles(); //添加静态文件
>
> 网页上调用：http://localhost:5000/htmlpage.html
>
> app.UseDefaultFiles——调默认文件



## 二、Azure（微软云计算）基于HyperV

> 云计算——租用其他公司的计算能力，azure=租用微软的虚拟机，对标亚马逊的AWS（基于Linux），阿里云抄袭的就是AWS
>
> 实质：就是把虚拟机托管到云端
>
> 优势：经济高效，可缩放，具有弹性，安全可靠（分布式和冗余存储）微软——三个备份，全球性——世界范围的数据中心

> 服务：
>
> IaaS：架构即服务，通过网络来检查和托管，不提供系统
>
> PaaS：平台即服务，包括服务器，存储空间和网络等基础结构
>
> SaaS：软件即服务（主推），利润高，全都给你搭好了，完全的云软件

![image-20220625202824806](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20220625202824806.png)

租服务：比如手机验证码服务





## 三、Razor Pages

> Razor Page 使编码页面集中，场景简单，更高效。 razor页面中有view和viewmodel 
>
> 该页面的所有内容都是孤立的，并在一个位置，可以用Razors来做MVC一切通过使用像

> * Razor页面框架的路由将url映射到磁盘
> * Razor页面需要根文件夹（Pages）
> * Index.cshtml是默认文件夹

## 四、JWT认证

传统：session 依赖于 cookie 即依赖于浏览器，不利于前后端分离的项目

> JWT认证：json web token——一般用于前后端分离/微信小程序/app
> 传统的token：
> 用户登录，服务端给返回token，并将token保存在服务端
> 以后用户再来访问时，需要携带token，服务端获取token后，再去数据库中获取token进行校验

> JWT：
> 用户登录，登录成功，jwt创建一个token，jwt生成的token由三段字符串组成，.分隔
> 第一段：Header，内部包含哈希256，json转为字符串，做base64url加密
> 第二段：Payload，放自定义信息，json转为字符串，做base64url加密
> 第三段：将第一段+第二段拼接，对此拼接物进行hash256加密，对hash256加密后的密文再做一次base64url加密
> 服务端给用户返回一个token（服务端不保存）
> 以后用户来访问，需要携带token，服务端获取token后，再做token的校验
> 优势：无需再服务器端保存token

## 五、常见功能

### 1、自定义验证器

> ```C#
> public class ValidationDomainAttribute:ValidationAttribute{
>     public override bool IsValid(object value)
>     {
>         string[] strings=value.ToString().Split('@');
>         return strings[1].ToUpper() == allowedDomain.ToUpper();
>     }
>     public readonly string allowedDomain;
>     public ValidationDomainAttribute(string allowedDomain){
>         this.allowedDomain = allowedDomain;
>     }
> }
> ```
>
> 





### 2、使用Session

> 装aspnetcore.Session包



## 六、自带的依赖注入

> 在 ASP.NET Core 中，有许多内置的服务已经被注入到依赖注入容器中，以便在应用程序中使用。以下是一些常见的已经注入的服务：
>
> 1. **ILogger<T>**：用于记录日志的服务，其中 `<T>` 是记录日志的类的类型。
> 2. **IConfiguration**：用于访问应用程序配置的服务，可以通过键值对的方式访问配置项。
> 3. **IHostingEnvironment / IWebHostEnvironment**：提供有关应用程序运行环境的信息，例如应用程序的根路径、环境名称等。
> 4. **IHttpClientFactory**：用于创建和管理 HTTP 客户端的工厂，有助于优化 HTTP 请求的性能。
> 5. **IServiceProvider**：用于访问应用程序的依赖注入容器，可以手动解析其他服务。
> 6. **IMemoryCache**：用于缓存数据在内存中，以提高应用程序的性能。
> 7. **DbContext**：如果你使用 Entity Framework Core 进行数据访问，`DbContext` 用于与数据库交互。
> 8. **IActionContextAccessor**：提供有关当前请求的信息，如控制器、动作方法等。
> 9. **IUrlHelperFactory**：用于生成 URL 的帮助器，用于构建 URL。
> 10. **IViewRenderService**：在控制器中呈现视图的服务，用于生成视图的 HTML。
> 11. **ITempDataDictionary**：用于在请求之间传递临时数据的服务。
> 12. **IWebHostEnvironment**：提供有关 Web 主机环境的信息，类似于 IHostingEnvironment。



## 七、中间件

> 中间件模式是23种设计模式的一种，其实是管道模式（也有人说是装饰模式）的一种实现。
>
> 大多数情况下，中间件都是以Use开头的扩展方法。
>
> 通过Startup.cs种的Configure方法来设置请求处理管道
>
> 必须在UseStaticFiles之前通过注册UseDefaultFiles来提供默认文件。
>
> UseDefaultFiles是一个URL重写器，实际没有提供文件，它只是将URL重写，还是由静态文件中间件提供。



### UseFileServer中间件

> UseFileServer结合了UseStaticFiles(),UseDefaultFiles()和UseDirectoryBrowser()中间件的功能。
>
> DirectoryBrowser()中间件支持目录浏览





## 八、控制器

> .net core 中无需显示继承ControllerBase
>
> 只需名字后有Controller即可
>
> 

### MVC中如何从Controller传递数据到View

> * 使用ViewData
> * 使用ViewBag
> * 使用强类型模型对象，也成为**强类型视图**



> ViewData["PageTitle"] = "Hello"; 后台代码写好
>
> `ViewData` 是一个 `ViewDataDictionary` 类型的字典，用于存储临时的键值对数据，以便在控制器中传递到视图。这些数据在控制器和视图之间共享，但是由于是弱类型的字典，所以在使用时需要进行类型转换。因为是动态的，所以可能在编译时不会进行类型检查。
>
> ```index.cshtml
> <h1>@ViewData["PageTitle"]</h1>
> 
> ```
>
> 

> ViewBag.PageTitle = "Hello";
>
> `ViewBag` 是一个动态属性，它是对 `ViewData` 的包装，提供了更方便的语法，但仍然是弱类型的。使用 `ViewBag` 可以通过属性访问数据，不需要像 `ViewData` 那样使用索引器。
>
> ```index.cshtml
> <h1>@ViewBag.PageTitle</h1>
> 
> ```
>
> 

> 最常用和首选项是强类型视图传递
>
> 即
>
> ```C#
>    public ViewResult Details()
>         {
>             Student s = _studentRepository.GetStudent(1);
>             
>             
>             return View(model);
>         }
> ```
>
> View代码接收
>
> ```index.cshtml
> @model Demo.Model.Student //Student类所在的位置
> <html>
> 	<h1> 姓名：@Model.Name</h1>
> 	<h1> 年龄：@Model.Age</h1>
> 	<h1> 性别：@Model.Sex</h1>
> </html>
> 
> ```
>
> 注意model的大小写，在指令(@model)中m为小写，但在调用对象属性时M是大写
>
> 强类型视图优点：
>
> 提供编译类型检查和智能提示
>
> 除此之外，cshtml（Razor视图）也支持迭代类型
>
> 如下：
>
> ```index.cshtml
> @model IEnumerable<Demo.Models.Student>
> <tbody>
> @foreach(var stu in Model)
> <tr>
> 	<td>@student.Id</td>
> 	
> </tr>
> </tbody>
> ```
>
> 在 ASP.NET Core Razor 视图中，每个视图只能绑定一个模型（Model）。
>
> 所以最好绑定ViewModel，里面有很多个Model





## 九、布局视图

> 其实就是各个子界面拼成一个完整的界面
>
> _Layout.cshtml
>
> ```_Layout.cshtml
> <!DOCTYPE html>
> 
> <html>
> <head>
>     <meta name="viewport" content="width=device-width" />
>     <title>@ViewBag.Title</title>
> </head>
> <body>
>     <div>
>         @RenderBody()
>     </div>
> </body>
> </html>
> ```



### 如何使用布局视图呢？

> 比如Details.cshtml中要使用_Layout.cshtml
>
> @RenderBody()是注入视图特定内容的位置，后面就是注入就行
>
> 需声明
>
> ```Details.cshtml
> @model Demo2.Model.Student
> @{
>     Layout = "~/Views/Shared/_Layout.cshtml";
>     ViewBag.Title = "asdasd ";
> }
> <html>
>     <head>
>         
>     </head>
> 
>     <body>
>         <h1>@Model.Name</h1>
>     </body>
> </html>xxxxxxxxxx16 1@{2@model Demo2.Model.Student3@{4    Layout = "~/Views/Shared/_Layout.cshtml";5    ViewBag.Title = "asdasd ";6}7<html>8    <head>9        10    </head>1112    <body>13        <h1>@Model.Name</h1>14    </body>15</html>16}
> ```



> 但是这项工作很繁琐，如果要想使用其他布局文件，则需更新每个视图
>
> MVC为我们提供了Razor视图开始文件，也就是==_ViewStart.cshtml文件==
>
> ```_ViewStart.cshtml
> @{
>     Layout = "_Layout";
> }
> //是创建时自动生成的
> ```
>
> 可以有多个布局视图
>
> ```_ViewStart.cshtml
> @{
>     Layout = "_Layout";
>     if(User.IsInRole("Admin"))
>     {
>         Layout = "_AdminLayout";
>     }
>     else
>     {
>         Layout = "_NonAdminLayout";
>     }
> }
> ```
>
> 





## 十、路由

> MVC中有两种路由技术，常规路由 和 属性路由
>
> ​            app.UseMvcWithDefaultRoute();
>
> //默认路由——{controller=Home}/{action=Index}/{id?} 
>
> ?表示参数可选



### 使用Route()属性来定义路由

> 可以在控制器上方或方法上方加特性
>
> [Route("")]
>
> [Route("Home")]
>
> [Route("Home/Index")]
>
> 表名三种方式都能导航到同一个方法
>
> 如果[Route("[action]")] 则会自动导航到方法名
>
> [Route("[controller]/[action]")]

> 使用属性路由时，属性路由需要在实际使用它们的操作方法上设置。属性路由比传统路由提供了更大的灵活性。通常情况下，常规路由用于服务html页面的控制器，而属性路由则用于服务RESTful API的控制器。



### UseEndpoints()

> 终结点路由，使用它是因为原有的路由中间件功能不够丰富。
>
> UseEndpoints可以处理跨中间件系统(MVC.gRPC、SignalR)的路由系统
>
> ```c#
>      app.UseEndpoints(endpoints =>
>             {
>                 endpoints.MapGet("/", async context =>
>                 {
>                     await context.Response.WriteAsync("11111");
>                 });
>                 endpoints.MapGet("/2", async context =>
>                 {
>                     await context.Response.WriteAsync("22222");
>                 });
>             });
> 
> 
>      app.UseEndpoints(o =>
>             {
>                 o.MapControllers();
>                 o.MapRazorPages();
>             });
> ```
>
> 





## 十一、使用LibMan包管理器安装样式库

> 1、右键项目：添加客户端库
>
> 选择cdnjs——twitter-bootstrap@5.3.1
>
> 


## 十二、TagHelper

> ```index.cshtml
> <a asp-controller="home" asp-action="details" asp-route-id="@student.id">查看</a>
> ```
>
> TagHelper中的Link标签可通过添加新属性来增强标准的HTML标签

> 使用TagHelper的优势：
>
> 无需方法的硬编码

> 如果没有使用TagHelper 则如下要对URL路径进行硬编码
>
> ```index.cshtml
> <a href="/home/details/@student.Id">查看</a>
> ```
>
> 



Image TagHelper

> ```index.cshtml
> <img src="~/images/noimage.png" asp-append=version="true"/>
> ```
>
> //Image TagHelper增强了img标签属性，为静态图像
>
> 文件提供了**缓存清除**行为。



Environment TagHelper

> 可以根据不同环境设置不同样式
>
> ```a.
> <environment include="Development">
> 
> ​	<link href="~/lib/bootstrap/css/bootstrap.css" rel="stylesheet"/>
> 
> </environment>
> //开发环境加载本地bootstrap样式
> 
> <environment include="Staging,Production">
> 
> ​	<link href="远程CDN地址"/>
> 
> </environment>
> //其余环境采用cdn获取
> ```
>
> 



## 十三、模型验证

> 略





## 十四、404错误和异常拦截

> 如何统一处理ASP.NETCORE中的404错误
>
> * UseStatusCodePages()
> * UseStatusCodePagesWithRedirects()
> * UseStatusCodePagesWithReExecute()
>
> ```C#
>  public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
>         {
>             if (env.IsDevelopment())
>             {
>                 app.UseDeveloperExceptionPage();
>             }
>  }
> //路由都不匹配时404自动跳转到默认异常页面
> 
> 
>  if (env.IsDevelopment())
>             {
>                 app.UseDeveloperExceptionPage();
>             }
>             else
>             {
>                 app.UseStatusCodePagesWithRedirects("/Error/{0}");
>             }
> //重定向到自己写的Error视图{0}——表示占位符，会自动接收HTTP中的状态码，地址栏会变！
> 
> 另外一个UseStatusCodePagesWithReExecute其实和上面那个类似，但它只重新执行管道而不发出重定向请求，所以地址栏不会变。
> ```
>
> 





## 十五、使用Logger来记录日志（ILogger注入)

> ```C#
>    public class ErrorController
>     {
>         private ILogger<ErrorController> _logger;
> 
>         public ErrorController(ILogger<ErrorController> logger)
>         {
>                 this._logger = logger;
>         }
> 
>         [Route("Error")]
>         private IActionResult Error()
>         {
>             _logger.LogError("发生错误咯");
>             return View("Error");
>         }
>     }
> ```
>
> 



## 十六、ASP.NET Core Identity框架

> 需要nuget:Microsoft.AspNetCore.Identity.EntityFrameworkCore
>
> ***如果这样，AppDbContext类就必须继承IdentityDbContext了
>
> 同时配置服务
>
> ```C#
> services.AddIdentity<IdentityUser,IdentityRole>().AddEntityFrameworkStores<AppDbContext>;
> ```
>
> 还要添加验证中间件
>
> ```C#
>      app.UseAuthentication();
> ```
>
> 

> Identity自带的两个服务——UserManager 和 SignInManager



```注册一个权限
using Microsoft.AspNetCore.Identity;

public class AccountController
{
    private readonly UserManager<ApplicationUser> _userManager;
    private readonly SignInManager<ApplicationUser> _signInManager;
    private readonly RoleManager<IdentityRole> _roleManager;

    public AccountController(
        UserManager<ApplicationUser> userManager,
        SignInManager<ApplicationUser> signInManager,
        RoleManager<IdentityRole> roleManager)
    {
        _userManager = userManager;
        _signInManager = signInManager;
        _roleManager = roleManager;
    }

    public async Task<IActionResult> Register(RegisterViewModel model)
    {
        if (ModelState.IsValid)
        {
            var user = new ApplicationUser { UserName = model.Email, Email = model.Email };

            var result = await _userManager.CreateAsync(user, model.Password);

            if (result.Succeeded)
            {
                // 创建 "Admin" 角色（如果不存在）
                if (!await _roleManager.RoleExistsAsync("Admin"))
                {
                    await _roleManager.CreateAsync(new IdentityRole("Admin"));
                }

                // 将用户添加到 "Admin" 角色
                await _userManager.AddToRoleAsync(user, "Admin");

                // 注册成功，执行其他操作
            }
            // 处理其他结果...
        }
        // 处理其他情况...
    }
}

```

