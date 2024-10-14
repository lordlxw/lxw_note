https://www.youtube.com/watch?v=_8nLSsK5NDo&list=PL82C6-O4XrHdiS10BLh23x71ve9mQCln0

## 基本概念

> 实质就是一个控制台应用
>
> 使用UseStartup<Startup>来自动执行配置
>
> 在 ASP.NET Core Web API 项目中，`Startup` 类的 `ConfigureServices` 和 `Configure` 方法会在应用程序启动时自动执行。这是因为 ASP.NET Core 框架在应用程序启动时会寻找 `Startup` 类，并调用其中的这两个方法来配置服务和应用程序管道。
>
> 具体来说：
>
> 1. **ConfigureServices 方法：** 这个方法用于配置应用程序的服务依赖注入。在这里，你可以注册应用程序需要的各种服务，例如数据库上下文、身份验证服务、仓储、日志记录器等。这些服务的配置将影响整个应用程序的生命周期。这个方法在应用程序启动时由框架调用，确保所有需要的服务在应用程序正式处理请求之前都已经被注册。
>
> 2. **Configure 方法：** 这个方法用于配置应用程序的请求处理管道（middleware pipeline）。在这里，你可以按顺序添加各种中间件，这些中间件会按照添加的顺序依次处理传入的请求。例如，身份验证、路由、异常处理等都可以通过中间件来实现。这个方法也在应用程序启动时由框架调用。
>
>    
>
>    WebApi由于只用控制器，不用像MVC项目需要AddMvc，只需AddControllers即可



> 控制器需要继承自ControllerBase，配上特性[ApiController]
>
> `ApiController` 是 ASP.NET Core 中用于定义 Web API 控制器的一个特性。这个特性提供了一些额外的功能和行为，使得创建和管理 Web API 更加方便。当你在控制器类上应用了 `ApiController` 特性时，框架会根据这个特性自动执行一些默认的行为和配置，以减少样板代码和提高开发效率。
>
> 以下是 `ApiController` 特性的一些用途和功能：
>
> 1. **自动模型验证：** 当应用了 `ApiController` 特性时，框架会自动对请求的模型参数进行验证，如果模型验证失败，框架会返回适当的错误响应。
> 2. **自动 HTTP 状态码：** `ApiController` 特性为常见的 HTTP 方法（如 GET、POST、PUT、DELETE 等）提供了自动的 HTTP 状态码返回，这样你就不需要在每个方法中手动编写状态码返回。
> 3. **自动 HTTP 特性路由：** `ApiController` 特性会自动应用基于 HTTP 方法的默认路由模板，这减少了路由配置的重复性代码。
> 4. **自动内容协商：** `ApiController` 特性支持自动内容协商，根据请求的 Accept 标头选择返回的内容格式（如 JSON、XML 等）。
> 5. **异常处理：** `ApiController` 特性提供了默认的异常处理行为，将异常转化为适当的错误响应，使错误信息更容易理解。
> 6. **数据注解和绑定：** `ApiController` 特性支持数据注解和模型绑定，使你可以在请求参数上使用数据注解来指定验证规则、绑定源等。



> 几个重要的中间件:(在Configure方法中配置)
>
> Routing
>
> Authentication
>
> AddExceptionPage



> 中间件
>
> 在 ASP.NET Core 中，中间件构成了应用程序的请求处理管道，用于处理传入的 HTTP 请求。中间件通过连接在一起，形成一个管道，每个中间件都可以对请求执行操作，并将请求传递给下一个中间件。以下是关于中间件中的 `Run`、`Use`、`Next` 和 `Map` 方法的解释：
>
> 1. **Use 方法：**
>    `Use` 方法是最常见和重要的中间件方法之一。它用于添加中间件到管道中。每当一个请求流经管道时，添加的中间件都会按照添加的顺序依次执行。中间件可以执行各种操作，如身份验证、日志记录、路由处理等。
>
>    ```csharp
>    app.Use(async (context, next) =>
>    {
>        // 执行一些操作
>        await next(); // 调用下一个中间件
>    });
>    ```
>
> 2. **Run 方法：**
>    `Run` 方法是一个特殊的中间件方法，它终结管道并处理请求，后面的中间件将不会被调用。这通常用于定义一个终结点，比如返回一个页面或一个特定的响应。
>
>    ```csharp
>    app.Run(async context =>
>    {
>        await context.Response.WriteAsync("Hello, World!");
>    });
>    ```
>
> 3. **Next 参数：**
>    在 `Use` 方法中，`next` 参数是一个 `Func<Task>` 类型的委托，它表示管道中的下一个中间件。通过调用 `next()` 方法，你可以将请求传递给下一个中间件。如果不调用 `next()`，则后续的中间件将不会被执行。
>
> 4. **Map 方法：**
>    `Map` 方法用于根据请求的路径前缀将请求分配给特定的分支管道。这样可以实现基于路径的不同处理逻辑，从而将不同的部分路由到不同的中间件管道。匹配则执行，不匹配则到下一个中间件去
>
>    ```csharp
>    app.Map("/api", apiApp =>
>    {
>        apiApp.UseAuthentication();
>        apiApp.UseEndpoints(endpoints => { /* 配置 API 端点 */ });
>    });
>    
>    app.Map("/admin", adminApp =>
>    {
>        adminApp.UseAuthorization();
>        adminApp.UseEndpoints(endpoints => { /* 配置管理端点 */ });
>    });
>    ```
>
> 通过组合使用这些方法，你可以构建一个强大而灵活的中间件管道，用于处理不同类型的请求、应用程序逻辑和业务需求。



### 自定义一个中间件

> 需继承自IMiddleware
>
> ```C#
> public class CustomMiddleware:IMiddleware
> {
>     public async Task InvokeAsync(HttpContext context,RequestDelegate next)
>     {
>         await context.response.WriteAsync("Test1");
>         
>         await next(context);
>         
>         await context.response.WriteAsync("Test1");
>     }
> }
> ```
>
> 要使用这个中间件需在ConfigureServices中注入
>
> ```C#
> services.AddTransient<CustomMiddleware>();
> ```
>
> 注入完再在Configure方法中配置中间件
>
> ```C#
> app.UseMiddleware<CustomMiddleware>();
> ```
>
> 



### Routing相关的中间件

> * UseRouting()
> * UseEndpoint(),最好使用  endpoints=>endpoints.MapControllers();
>
> ```C#
> 	[Route("books/(id}/author/(authorId}")]
>     public string GetAuthorAddressById(int id, int authorId)
> {
>     return "hello author address " + id +"" + authorid;
> }
> //传参方式
> http:localhost:5050/books/1/author/2
>     
>     或者
>     [route()"search")]
>     public string search(int id,int authorid,string name)
> {
>     
> }
> //传参方式
> http:localhost:5050/search?id=1&authorid=23&name="lxw"
> ```
>
> 
