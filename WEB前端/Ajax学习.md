# Ajax

Ajax 就是 异步的js+xml

最大的优势：无刷新获取数据



## 一、Ajax简介

> 优点：
>
> 允许你根据用户事件来更新部分页面内容
>
> 缺点：
>
> * 没有浏览历史，不能回退
> * 存在跨域问题
> * 爬虫爬不到信息的
>
> 



> 1、Http请求格式和报文
>
> 行： get\post  /?=asda
>
> 头：	Host:
>
> ​		 Cookie: 
>
> ​		Content-Type:text/html;charset=utf-8
>
> ​		 Usr-Agent：Chrome -83
>
> 空行：
>
> 体：get必须为空，post可以不为kong



> 2、Http响应报文
>
> 行：协议版本 Http/1.1 200 OK
>
> 头：Content-Type:text/html;charset=utf-8
>
> ​		Content-length:2048
>
> ​		Content-encoding:gzip
>
> 空行：
>
> 体：
>
> 		<html>		
> 				<head>
> 				    			
> 				</head>
> 				<body>
> 				<h1>hello</h1>
> 				</body>
> 		</html>
>
> 
>



## 二、Ajax实现

###  1、实现代码

> ```js
> const btn = document.getElementById('button');
> btn.onClick=function(){
>     //1、创建对象
>     const xhr=XMLHttpRequest();
>     2、初始化，设置请求方法和url
>     xhr.open('Get','http://127.0.0.1:80/server');
>     //设置请求头信息
>     xhr.setRequestHeader('Content-Type','application/x-www-form-urlencoded')
>     3、发送
>     xhr.send();
>     4、事件绑定，处理服务器返回的结果 readystate有五个值 0~4	
>     0 表示未初始化  1 open方法调用完毕 2 send方法已经调用完毕 3 服务端返回部分结果 4 服务端返回了所有结果
>     xhr.onreadystatechange=function(){
>         if(xhr.readyState==4)
>             {
>                 //判断响应成功码
>                 if(xhr.status==200)
>                     {
>                         //处理结果
> 						console.log(xhr.getAllResponseHeaders());//拿到所有响应头
>                         console.log(xhr.response);//拿到所有响应体
> }
>             }
>     }
> }
> ```
>
> 注意：XHR 是XMLHttpRequest的缩写，F12查看网页也有XHR这个选项，即过滤Ajax请求



### 2、Ajax如何设置请求

> 在Url后+？参数  ?a=100&b=200&c=300





### 3、Ajax中的缓存问题

> IE浏览器的缓存机制 对 已改变的数据不会刷新
>
> 而Google浏览器没这个问题
>
> ### 如何解决IE的缓存机制？
>
> 加时间戳，表示这是另一个请求
>
>  xhr.open('Get','http://127.0.0.1:80/ie?t='+Date.now());





### 4、增加超时机制

> xhr.timeout=2000;
>
> //两秒内没有结果则取消
>
> 超时回调：
>
> xhr.ontimeout=funtion(){
> 	alert("网络异常，请重试")
>
> }



> 手动Ajax取消请求：
>
> x.abort(); （X是XMLHttpRequest对象）





## 5、Ajax处理反复请求（一直向服务器发送请求）

> 实现方式：点第二下就给它取消掉（abort），加一个标志位







## 三、JQuery中发送Ajax请求

》