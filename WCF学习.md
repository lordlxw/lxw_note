# WCF

https://www.youtube.com/watch?v=QmfPmqMk9Xs&list=PLxFKkSyGrz0WLv5462woTEIwY0oD95k--&index=7

## 1、什么是wcf？

> 构建分布式和可互操作的microsoft平台
>
> 程序运行在不同节点上，不同机器运行不同程序（多台服务器性能会好）可以提高程序的可伸缩性
>
> Web服务是可互操作的，因为他们使用http开放协议和xml格式
>
>  但是dotnet并不是可互操作的，若要使用，必须使用dotnet远程服务
>
> 关键：Http （Web服务）和 Tcp（远程服务）
>
> 一套页面无需学习新技术，通过切换==终结点==就可以应用于Http 和 Tcp，无需改变程序，只需改变配置
>
> *TCP使用流式传输，拥有更好的性能

## 2、WCF之前什么技术可以实现分布式

> 1. Enterprise Services
> 2. Dot Net Remoting
> 3. Web Services
>
> 不是.net 程序能否实现Dot Net Remoting？答案是 —— 也许，要注意数据类型的匹配



## 3、TCP中如何实现.net remoting

> 如何让一个类让别人知道是可移动的（让别的应用程序调用）
>
> 旧实现：
>
> MarshalByRefObject类 在支持远程处理的应用程序中，允许跨应用程序域边界访问对象。
>
> ```C#
> public class HelloRemoteService:MarshalByRefObject,IHelloRemoteService
> {
>     public string GetMsg(string name)
>     {
>         return "HEllo" + name;
> }
> }
> ```
>
> 使用时需添加 System.Runtime.Remoting引用



> ```C#
> int main()
> {
>     HelloRemotingService remotingService = new HelloRemotingService();
>     TcpChannel channel = new TcpChannel(8080);
>     ChannelServices.RegisterChannel(channel);
>     RemotingConfiguration.RegisterWellKnownServiceType(typeof(HelloRemotingService),"GetMsg",WellKnownObjectMode.Singleton); //只能一个调用
>         //参数：服务类、唯一标识uri、枚举类
> }
> ```
>
> 





## 4、真正WCF实现

> 接口+类实现
>
> ```C#
> [ServiceContract]
> public interface IHeloService
> {
>     [OperationContract]
>     void DoWork();
> }
> 
> ```
>
> 

> [ServiceContract] 这个特性表示将下方的接口转换为WCF服务
>
> [OperationContract]可以使该方法作为服务给客户端



## 5、Config文件编写及主机调用

一个WCF可以供HTTP或TCP使用，定义两个终结点即可

> ```xml
> <system.serviceModel>
> 	<bindings>
>         <basicHttpBinding> //使用http绑定
>         </basicHttpBinding>
>     </bindings>
>     <services>
> 		<service name="HelloService.HelloService" behaviorConfiguration="mexBehavior"> //服务行为绑定到下方的具体行为
>             <endpoint address="HelloService" binding = "basicHttpBinding" contract = "HelloService.IHelloService"/> //定义终结点和合约，让客户端知道所有的操作方式，其实就是接口名
>             <endpoint address="HelloService" binding = "netTcpBinding" contract = "HelloService.IHelloService"/>  //Tcp绑定
>              <endpoint address="mex" binding = "mexHttpBinding" contract = "IMetadataExchange"/> 
>             
>             <host>//定义基地址
>                 <baseAddress>
>                     <add baseAddress="http://localhost:8080/"/> //Http
>                     <add baseAddress="net.tcp://localhost:8090/"/> //Tcp
>                 </baseAddress>
>             </host>
>     </services>
>     <behaviors>//指定服务行为。将行为与服务本身相关联
>         <serviceBehaviors>
>             <behavior name ="mexBehavior">
>                 <serviceMetadata httpGetEnabled="true"/> //Http用get来请求服务
>             </behavior>
>         </serviceBehaviors>	
>     </behaviors>
> </system.serviceModel>
> ```
>
> 







Host主机调用

```C#
int main()
{
    using ( Service Host = new Service (typeof(HelloService.HelloService)))
    {
        host.Open();
        //主机就开启了
            
    }
}
```



如何在Web程序中调用呢？

引用——WCF服务——输入uri“http://localhost:8080/”

```C#
private void Do()
{
    HelloServiceClient client = new HelloServiceClient("NetTcpBinding_IHelloService");//通过名字来绑定对应的服务
}
```



## 6、如何使用不同端点公开每个服务契约

> 把 具体服务类 继承 多接口 
>
> Tcp服务仅使用防火墙后的TCP协议可用，即公司内网
>
> 而Http服务是都可用的



> 还是修改Config文件
>
> ``` xml
> <configuration>
>     <system.serviceModel>
>         <bindings>
>         </bindings>
>         <services>
>             <service name="CompanyService.CompanyService">
>                 <endpoint address = "CompanyService" binding="basicHttpBinding" contract="CompanyService.ICompanyPublicService"> //合约名是其中一个接口的命名，这个用http
>                 </endpoint>
>                  <endpoint address = "CompanyService" binding="netTcpBinding" contract="CompanyService.ICompanyConfidentialService"> //合约名是另一个接口的命名，这个用http
>                 </endpoint>
>                  <host>//定义基地址
>                 <baseAddress>
>                     <add baseAddress="http://localhost:8080/"/> //Http
>                     <add baseAddress="net.tcp://localhost:8090/"/> //Tcp
>                 </baseAddress>
>             </host>
>             </service>
>         </services>
>         <behaviors>//指定服务行为。将行为与服务本身相关联
>         <serviceBehaviors>
>             <behavior name ="mexBehavior">
>                 <serviceMetadata httpGetEnabled="true"/> //Http用get来请求服务
>             </behavior>
>         </serviceBehaviors>	
>    		  </behaviors>
>     </system.serviceModel>
> </configuration>
> ```
>
> 





## 7、WCF服务应用场景

### 1、编写sql存储过程

> 服务程序提供存储过程的实现
>
> 客户端使用服务类的接口来调用函数
>
> WCF其实就是序列化和反序列化，WCF用DataContractSerializer序列化器
>
> 而且.net3.5后属性无需用特性显示声明为数据协定可序列化了，但私有字段不会被序列化。

> 要想都序列化，需要在类上加上特性
>
> [Serializable]（全都序列化） 或 [DataContract] (明确指出哪几个需要序列化)+[DataMember]——》指定序列化的数据
>
> 序列化顺序默认是按字母顺序的，可以自定义用Order=1、2、3 ，定义顺序

> 若一个类是基类，有好多子类
>
> ![image-20230617155322984](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20230617155322984.png)
>
> 要这么写
>
> ```C#
> [KnownType(typeof(PartTimeEmployee))]
> [KnownType(typeof(FullTimeEmployee))]
> [DataContract]
> public class Employee
> {
>     [DataMember]
>     public int Id{get;set;}
>     
>     [DataMember]
>     public string Name{get;set;}
>     
>    [DataMember]
>     public DateTime Birth{get'set'}
>     
>     public EmployeeType Type{get;set;}
> }
> 
> public enmu EmployeeType
> {
>     FullTimeEmployee=1,
>     PartTimeEmployee=2
> }
> ```
>
> ```C#
> if(employee.GetType()== typeof(FullTimeEmployee))
> {
> SqlParameter parameterAnnualSalary = new SqlParameterTParameterName ="@AnnualSalary"，Value((FullTimeEmployee)employee).AnnualSalary
> cmd.Parameters.Add(parameterAnnualSalary);
> }
> else
>     
> {
>     
> }
> ```
>
> 





## 8、Message日志功能_消息合同契约

> 可以通过wcf配置实现 后缀名为.svclog，通过SOAP
>
> 查看请求 和 响应
>
> ![image-20230617161003319](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20230617161003319.png)



等于用一个包装器包装了 请求 和 响应

```C#
[MessageContract(IsWrapped=true,WrapperName="EmployeeRequestObject",WrapperNamespace="http://MyCompany.com/Employee")]
public class EmployeeRequest
{
    [MessageHeader(Namespace="http://MyCompany.com/Employee")]
    public string LicenseKey{get;set;}
    
    [NessageBodyMember(Namespace="http://MyCompany.com/Employee")]
    public inc EmployeeId{get;set;}
}
//这是请求




[MessageContract(IsWrapped=true,WrapperName="EmployeeRequestObject",WrapperNamespace="http://MyCompany.com/Employee")]
public class EmployeeInfo
{
    public EmployeeInfo()
    {
        
    }
    
     public EmployeeInfo(Employee employee)
    {
        this.Id=employee.Id;
        this.Name=employee.Name;
    }
    
    [MessageBodyMember(Order=1,Namespace="http://MyCompany.com/Employee")]
    public int Id{get;set;}
    
    [MessageBodyMember(Order=2,Namespace="http://MyCompany.com/Employee")]
    public string Name{get;set;}
    
    。。。
}
//这是响应体
```





## 9、wcf服务控制并发

> 在 WCF (Windows Communication Foundation) 中，`ServiceBehavior` 特性用于控制服务的行为和配置它的生命周期、线程管理等方面。它是服务端控制的一个重要工具，可以通过它调整服务的多个方面，如实例化模式、并发管理、错误处理等。
>
> 
>
> ### **ServiceBehavior 特性概述**
>
> `ServiceBehavior` 特性是应用在服务类上的，主要用于配置服务的运行时行为。通过该特性，我们可以调整服务的以下几个方面：
>
> - **InstanceContextMode**：服务的实例化模式（例如，单例、每请求等）。
> - **ConcurrencyMode**：服务的并发模式（如何处理多个请求）。
> - **AutomaticSessionShutdown**：在会话结束后，是否自动关闭服务实例。
> - **IncludeExceptionDetailInFaults**：是否在故障中包含详细的异常信息。
>
> 
>
> 服务端代码：
> ```C#
>   [ServiceContract]
>   public interface **ISinglePerCall**
>   {
>     [OperationContract]
>     int **GetValue1**();
>     [OperationContract]
>     int **GetValue2**();
>   }
>   **[ServiceBehavior(InstanceContextMode=InstanceContextMode.PerCall, ConcurrencyMode=ConcurrencyMode.Single)]**
>   public class **SinglePerCall** : **ISinglePerCall**
>   {
>     private int **InstanceVariable** = 0;
>     private **static** int **StaticVariable** = 0;
>     public int GetValue1()
>     {
>       return **++InstanceVariable;**
>     }
> 
> ​    public int GetValue2()
> ​    {
> ​      return **++StaticVariable;**
> ​    }
>   }  这里我们定了服务契约ISinglePerCall，它里面包含了两个方法GetValue1()和GetValue2()。**GetValue1()的主要作用是把实例变量的值加1返回；GetValue2()的主要作用是把静态变量的值加1返回。**
>   服务的行为模式是：Single并发+PerCall实例
>   **由于PerCall实例模式会为每个请求生成一个新的服务实例，所以，我们调用GetValue1()返回的结果应当永远都是1。因为每个服务实例在生成的时候都默认为InstanceVariable设为0；但调用GetValue2()的时候并不会都是1，因为GetValue2()读取的是静态变量自增后的值，静态变量被所有实例所共享，所以它会从1开始递增。**由于在这里多个服务实例同时运行，对共享的静态变量的访问有可以产生冲突，故会出现下面的运行结果。要解决这个问题还需要我们手动编写同步代码。
> ```
>
> 
>
> 客户端代码：
> ```C#
>  class SinglePerCall
>   {
>     **private static SRSinglePerCall.SinglePerCallClient client = new Client.SRSinglePerCall.SinglePerCallClient();**
>     public static void Main(string[] args)
>     {
>         
>       **for (int i = 0; i < 10; i++)**
>       {
>         **Thread thread = new Thread(new ThreadStart(DoWork));**
>         thread.Start();
>         while (!thread.IsAlive) ;
>       }
>     }
>     public static void DoWork()
>     {
>       
>       Console.WriteLine("线程" + Thread.CurrentThread.GetHashCode() + ":/t实例变量的值：" +**client.GetValue1**());
>       Console.WriteLine("线程" + Thread.CurrentThread.GetHashCode() + ":/t静态变量的值：" +**client.GetValue2**());
>     }
>   }  在客户端中我启用了10个线程向[服务端]
>     
> ```
>
> https://www.cnblogs.com/aaa6818162/p/4194297.html#:~:text=%E5%BD%93%E5%A4%9A%E4%B8%AA%E7%BA%BF%E7%A8%8B%E5%90%8C%E6%97%B6%E8%AE%BF%E9%97%AE%E7%9B%B8%E5%90%8C%E7%9A%84%E8%B5%84%E6%BA%90%E7%9A%84%E6%97%B6%E5%80%99%E5%B0%B1%E4%BC%9A%E4%BA%A7%E7%94%9F%E5%B9%B6%E5%8F%91%EF%BC%8CWCF%E7%BC%BA%E7%9C%81%E6%83%85%E5%86%B5%E4%B8%8B%E4%BC%9A%E4%BF%9D%E6%8A%A4%E5%B9%B6%E5%8F%91%E8%AE%BF%E9%97%AE%E3%80%82%20%E5%AF%B9%E5%B9%B6%E5%8F%91%E8%AE%BF%E9%97%AE%E9%9C%80%E8%A6%81%E6%81%B0%E5%BD%93%E5%A4%84%E7%90%86%EF%BC%8C%E6%8E%A7%E5%88%B6%E4%B8%8D%E5%A5%BD%E4%B8%8D%E4%BB%85%E4%BC%9A%E5%A4%A7%E5%A4%A7%E9%99%8D%E4%BD%8EWCF%E6%9C%8D%E5%8A%A1%E7%9A%84%E5%90%9E%E5%90%90%E9%87%8F%E5%92%8C%E6%80%A7%E8%83%BD%EF%BC%8C%E8%80%8C%E4%B8%94%E8%BF%98%E6%9C%89%E5%8F%AF%E8%83%BD%E4%BC%9A%E5%AF%BC%E8%87%B4WCF%E6%9C%8D%E5%8A%A1%E7%9A%84%E6%AD%BB%E9%94%81%E3%80%82,%E5%9C%A8WCF%E4%B8%AD%E4%BD%BF%E7%94%A8%20ServiceBehaviorAttribute%E4%B8%AD%E7%9A%84ConcurrencyMode%E5%B1%9E%E6%80%A7%E6%9D%A5%E6%8E%A7%E5%88%B6%E8%BF%99%E4%B8%AA%E8%AE%BE%E7%BD%AE%E3%80%82



## 10、控制吞吐量

> ```web.config
>   <behaviors>
>     <serviceBehaviors>
>       <behavior>
>         <!-- 吞吐量限制 -->
>         <serviceThrottling maxConcurrentCalls="100" maxConcurrentSessions="100" maxConcurrentInstances="100" />
>       </behavior>
>     </serviceBehaviors>
>   </behaviors>
> ```
>
> `axConcurrentCalls` 和 `maxConcurrentInstances` 在实际运行时会相互作用，并且 WCF 会根据这两个配置的最小值来限制并发量。
>
> 具体来说，WCF 的并发请求数会受到这两个参数的 **共同限制**，但 **最终并发数** 会是 **`maxConcurrentCalls`** 和 **`maxConcurrentInstances`** 的 **最小值**。这意味着：
>
> - **`maxConcurrentInstances`** 控制服务实例的最大数量。
> - **`maxConcurrentCalls`** 控制每个实例同时处理的最大请求数。
>
> ![image-20250209191439935](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20250209191439935.png)



##

## WCF-Security（安全）

> Authentication & Authorization
>
> ==例外== httpbinding在wcf中不提供自带的加密机制
>
> **`HttpBinding`**（即 **`basicHttpBinding`**）本身并不提供高级的安全机制。它主要用于支持简单的 **SOAP** 消息交换，并且不提供像 **消息加密** 或 **身份验证** 等安全特性。其主要特点是简单、高效，但不适合处理对安全有较高要求的场景。
>
> ### `basicHttpBinding` 的安全性限制
>
> - **传输安全**：`basicHttpBinding` 在默认配置下通过 **HTTP** 协议进行通信，因此它无法提供如 **HTTPS** 一样的 **加密** 或 **消息保护**。如果需要传输安全，你必须使用 **`basicHttpsBinding`**，该绑定将 HTTP 协议替换为 HTTPS，启用 **TLS** 加密通道。
> - **消息安全**：`basicHttpBinding` 默认情况下不支持对消息本身进行加密和签名。这意味着即使使用 **HTTPS** 保护了通道，消息内容仍然是明文的，无法保证消息的完整性或加密。对于需要对消息进行加密、签名等保护的应用场景，应选择 **`wsHttpBinding`** 或 **`wsDualHttpBinding`**，这些绑定支持更强的 **WS-Security** 功能，能够对消息进行签名、加密等保护。
>
> 
>
> 
>
> **"Pros and cons"** 是一个常见的表达，意思是 **优点和缺点**，用来比较某事物的好处和坏处。它通常用于讨论某个选择、决定或方案时的 **利与弊**。
>
> - **Pros** 指的是某事物的 **优点、好处** 或 **积极的一面**。
> - **Cons** 指的是某事物的 **缺点、不利之处** 或 **负面影响**。



## 11、binding模式

> 以下是 WCF 中常用 `Binding` 类型的总结表格，列出了它们的协议、特点以及适用场景：
>
> | **Binding 类型**            | **协议** | **特点**                                                     | **适用场景**                             |
> | --------------------------- | -------- | ------------------------------------------------------------ | ---------------------------------------- |
> | **BasicHttpBinding**        | HTTP     | 兼容传统的 ASP.NET Web Services (ASMX)，使用 SOAP 格式       | 简单的 Web 服务通信，跨平台互操作性      |
> | **WSHttpBinding**           | HTTP     | 支持 WS-* 协议（如安全性、可靠性），功能比 BasicHttpBinding 强 | 安全、可靠、事务处理要求较高的 Web 服务  |
> | **NetTcpBinding**           | TCP      | 高效的二进制消息传输，支持双向通信和可靠消息传递             | 高性能的内网通信，低延迟和高吞吐量的应用 |
> | **NetNamedPipeBinding**     | 命名管道 | 本地进程间通信，速度最快，仅限同一机器上的客户端和服务端     | 本地进程间通信（IPC），高效的本地通信    |
> | **NetMsmqBinding**          | MSMQ     | 支持异步消息传递，确保消息的可靠性和顺序性                   | 异步消息处理、消息队列应用               |
> | **WSDualHttpBinding**       | HTTP     | 支持双向通信，客户端和服务端可以相互通信                     | 双向通信场景，如推送通知和实时通信       |
> | **WSFederationHttpBinding** | HTTP     | 支持 WS-Federation 协议，用于身份验证和授权                  | 联合身份验证和授权场景                   |
> | **CustomBinding**           | 可自定义 | 允许开发者根据需求自定义协议、传输和编码方式                 | 完全自定义的通信需求                     |
>
> 这个表格简洁地总结了 WCF 各种 `Binding` 的协议、特点和常见应用场景，帮助选择最适合的绑定类型来满足不同的需求。
>
> ### **WebHttpBinding**
>
> - **协议**：HTTP
> - **特点**：专门用于 **RESTful** 服务，支持 **JSON** 和 **XML** 格式的消息传递，适合 **无状态**、轻量级的 HTTP 通信。它不需要 SOAP 消息格式，通常用于 Web API 风格的服务。





## 12、日志记录

> 







---



# 遇到的问题

## 1、多种binding方式的区别

>  webhttpbinding是REST风格的绑定，您只需点击一个URL，然后从Web服务中获取大量XML或JSON。
>
>  
>
>   basichttpbinding和wshttpbinding是两个基于SOAP的绑定，与REST有很大的不同。SOAP的优势在于拥有WSDL和XSD来详细描述服务、其方法以及传递的数据（REST风格并不具备这种功能）。另一方面，您不能只使用浏览器浏览到wshttpbinding端点并查看XML（例如这种绑定的服务如果通过形如http://localhost:端口/testservice.svc的地址访问，将会报http400错误），您必须使用SOAP客户端，例如wcftestclient或您自己的应用程序。
>
>  
>
> 
>  basichttpbinding和wshttpbinding的区别如下：
>
>  
>
>  basichttpbinding是非常基本的绑定-soap 1.1，在安全性方面不多，在功能方面不多，但与现有的任何SOAP客户机都兼容——>互操作性好，功能和安全性差
>
>  
>
>  wshttpbinding是一个全面的绑定，它支持大量的ws-*功能和标准-它有更多的安全功能，您可以使用会话连接，您可以使用可靠的消息传递，您可以使用事务控制，您可以使用流式处理大数据，但wshttpbinding也有点“笨重”并且当你的消息在网络中传输时，会有很多开销。





## 2、webget导致pinvoke异常

> 当在 WCF 服务中添加 `WebGet` 特性时，确保使用的是适合 RESTful 服务的 `webHttpBinding` 配置，并且客户端和服务端的配置匹配。如果错误仍然存在，查看更详细的错误日志可能会提供更多的线索。
>
> 
>
> 默认情况下，WCF 服务使用的是 ==SOAP 协议==，特别是使用 `basicHttpBinding` 或 `wsHttpBinding` 这类 SOAP 协议的绑定。
>
> 
>
> 配置文件：
>
> ```web.config
> <?xml version="1.0" encoding="utf-8"?>
> <configuration>
> 
> 	<appSettings>
> 		<add key="aspnet:UseTaskFriendlySynchronizationContext" value="true" />
> 	</appSettings>
> 	<system.web>
> 		<compilation debug="true" targetFramework="4.7.2" />
> 		<httpRuntime targetFramework="4.7.2"/>
> 	</system.web>
> 	<system.serviceModel>
> 		<behaviors>
> 			<serviceBehaviors>
> 				<behavior>
> 					<!-- 启用 HTTP GET 和 HTTPS GET 的元数据交换 -->
> 					<serviceMetadata httpGetEnabled="true" httpsGetEnabled="true"/>
> 					<!-- 启用调试信息 -->
> 					<serviceDebug includeExceptionDetailInFaults="true"/>
> 				</behavior>
> 			</serviceBehaviors>
> 			<endpointBehaviors>
> 				<behavior name="restbehavior">
> 					<webHttp defaultBodyStyle="Wrapped" defaultOutgoingResponseFormat="Json" />
> 				</behavior>
> 			</endpointBehaviors>
> 
> 		</behaviors>
> 		<protocolMapping>
> 			<add binding="basicHttpsBinding" scheme="https"/>
> 		</protocolMapping>
> 		<serviceHostingEnvironment aspNetCompatibilityEnabled="true" multipleSiteBindingsEnabled="true"/>
> 		<services>
> 			<service name="MyService.PatientService" >
> 				<endpoint address="" binding="webHttpBinding"  contract="MyService.IPatientService" behaviorConfiguration="restbehavior">
> 
> 				</endpoint>
> 			</service>
> 		</services>
> 
> 
> 
> 	</system.serviceModel>
> 	<system.webServer>
> 		<modules runAllManagedModulesForAllRequests="true"/>
> 		<!--
>         若要在调试过程中浏览 Web 应用程序根目录，请将下面的值设置为 True。
>         在部署之前将该值设置为 False 可避免泄露 Web 应用程序文件夹信息。
>       -->
> 		<directoryBrowse enabled="true"/>
> 	</system.webServer>
> 
> </configuration>
> 
> ```
>
> 核心：
>
> ```xml
> <endpointBehaviors>
> 	<behavior name="restbehavior">
> 		<webHttp defaultBodyStyle="Wrapped" defaultOutgoingResponseFormat="Json" />
> 	</behavior>
> </endpointBehaviors>
> 
> 注意：配置了defaultOutgoingResponseFormat="Json"，默认是返回xml的
> ```
>
> 





## 3、发送post请求

> 注意：传参的时候要传根节点。当设置BodyStyle = WebMessageBodyStyle.Bare 特性，就不用传
>
> 
>
> 示例代码：
>
> ```C#
> //接口
> [OperationContract]
> [WebInvoke(Method = "POST", UriTemplate = "AddPatient", BodyStyle = WebMessageBodyStyle.Bare, RequestFormat = WebMessageFormat.Json, ResponseFormat = WebMessageFormat.Json)]
> string AddPatient(MyResponse patient);
> 
> 
> //实现
>    public string AddPatient(MyResponse patient)
>    {
>        // 处理逻辑，例如将患者信息添加到数据库等
>        return patient.Name+patient.Message;
>    }
> ```
>
> 
>
> 
>
> postman调用
>
> ```postman
> http://localhost:5681/PatientService.svc/AddPatient
> 
> Body：
> {"Name": "John Doe","Message": "Hello from Postman"}
> ```
>
> 
