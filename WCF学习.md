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

