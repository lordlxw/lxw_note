# Tomcat学习

> 由Apache开源免费的WEB服务器，适用于中小型JavaEE项目，可以处理静态和动态页，静态页用HTML，动态页用Servlet和JSP
>
> 大型项目比如银行——用的是WebLogic（Oracle）、WebSphere（IBM）、JBOSS（JBOSS），大型服务器，要收费的
>
> 静态资源有：html、css、js、jpg —— 所有用户访问的都一样
>
> 动态资源有：servlet/jsp，php，asp —— 不同用户访问相同资源得到结果可能不一样，动态资源被访问后，先转成静态资源，再返回给浏览器，通过浏览器进行解析。
>
> Tomcat通过HTTP协议来服务文件
>
> Tocat是一个Servlet容器还是个Web容器，它不支持EJBS或者JMS
>
> 除了Tomcat之外，与它类似的还有一个叫——Jetty

> Tomcat的核心分为3个部分:
> （1）Web容器—处理静态页面；
> （2）catalina — 一个[servlet](https://so.csdn.net/so/search?q=servlet&spm=1001.2101.3001.7020)容器-----处理servlet;
> （3）还有就是JSP容器，它就是把jsp页面翻译成一般的servlet。

## Servlet

> Java Servlet 是运行在 Web 服务器或应用服务器上的程序，它是作为来自 Web 浏览器或其他 HTTP 客户端的请求和 HTTP 服务器上的数据库或应用程序之间的中间层。
>
> 使用 Servlet，您可以收集来自网页表单的用户输入，呈现来自数据库或者其他源的记录，还可以动态创建网页。
>
> Java Servlet 通常情况下与使用 CGI（Common Gateway Interface，公共网关接口）实现的程序可以达到异曲同工的效果。但是相比于 CGI，Servlet 有以下几点优势：
>
> - 性能明显更好。
> - Servlet 在 Web 服务器的地址空间内执行。这样它就没有必要再创建一个单独的进程来处理每个客户端请求。
> - Servlet 是独立于平台的，因为它们是用 Java 编写的。
> - 服务器上的 Java 安全管理器执行了一系列限制，以保护服务器计算机上的资源。因此，Servlet 是可信的。
> - Java 类库的全部功能对 Servlet 来说都是可用的。它可以通过 sockets 和 RMI 机制与 applets、数据库或其他软件进行交互





## 安装与调试

> 第一次安装时用了java8（jdk1.8） 和 Tomcat10.0，结果不兼容，无法启动，与Tomcat10.0相适应的要Java11以上。
>
> 为了适配jdk1.8，我安装了Tomcat9，还是不行，最后装了java20（最新版本）成功登录http://localhost:8080