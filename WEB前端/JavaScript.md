## **Javascript**

## 一、基础

> 一种解释性语言。面向对象的语言，JS解释器有自己的内存管理机制，可以自动垃圾回收
>
> JS是动态类型语言，运行时才编译，var的存在
>
> 目前JavaScript*脚本的运行环境有浏览器和node.js环境两种*。

> JS中严格区分大小写
>
> 六种数据类型：
>
> String——字符串
>
> Number——数值
>
>  js中所有数值都是Number类型，最大值是1.79*10的308次，超出这个就是infinity，NAN——Not a number
>
> Boolean——布尔值 除了0，-0，NAN，null,undefined都是true
>
> Undefined——未定义，声明了但没赋值
>
> Null——空值
>
> Object——对象
>
> 其中Object属于引用数据类型

> 类型转换：将其他类型转换为String、Number、Boolean
>
> Tostring()不会改变原变量类型
>
> 



## 1、JS中的模式匹配

> RegExp方法——正则匹配



## 2、let、var、const的区别

> var 允许多次定义变量
>
> let 和 const 都不允许重复定义，但let可以修改值，const定义完之后不能修改
>
> 注意：const是数组的情况下，可以利用下标来控制元素 const content=[] content[0]="111" content[1]="222"

>1. const let 是块级[作用域](https://search.bilibili.com/all?from_source=webcommentline_search&keyword=作用域) var没有块级作用域,var只有函数和全局作用域
>2. const let 不存在变量声明的提前，var有声明的提前，所以const和let在声明变量/常量 之前，是没办法获取到的，称为暂时性死区temporal dead zone
>3. const let是ES6提出的，var是ES5
>4. const 声明的是常量，常量不能被修改，let和var声明的是变量，可以被修改
>5. const在声明时必须赋值，而let和var不需要
>6. let 和const不能重复声明同一个值:如 let a=1 ; let a =2 这样是不被允许的，但var可以，最后一个var声明的值会覆盖之前的 如：var b =1 ;var b =2 console.log(b)  结果为2

## 3、InnerText 和 InnerHtml区别

> InnerText会去除空格和标签
>
> InnerHtml可以识别html标签，保留空格和换行



## 4、InnerText 和 textContent的区别

> innerText 和 textContent 的作用都是获取元素之间的文本内容，但是它们的在获取到文本内容的结果也有很大的不同



> - innerText 输出结果是在和页面中显示是一样的在同一行显示，而 textContent 输出结果和代码中的显示是一致的。
> - 如果一个元素之间包含了script标签或者style标签，innerText是获取不到这两个元素之间的文本的，而textContent可以。
> - textContent会把空标签解析成换行（几个空标签就是几行），innerText只会把block元素类型的空标签解析换行，并且如果是多个的话仍看成是一个



## 5、JS的三等号

> ***\*==先转换类型再比较，===先判断类型，如果不是同一类型直接为false。\****
>
> ***\*alert(1 ==\** \**“\**\**1\**\**”\**\**); // true\****
>
> ***\*alert(1 === “1\**\**”\**\**); // false\****



## 6、获取html元素的方式

> 通过html标签的id来获得元素——document.getElementById("")方法
>
> 通过查询选择器——document.querySelector("")方法
>
> 此方法可以查询到类，比如body
>
> 



## 7、JS数组

> 可以放不同类型的数据，包括函数
>
> let lxw=["lxw",35,100];
>
> lxw.push(12);
>
> lxw.pop();
>
> shift 和 unshift  和上面类似，pop和push是在数组最后操作，而shift和unshift是对首个元素操作。
>
> unshift() 方法可向[数组](https://so.csdn.net/so/search?q=数组&spm=1001.2101.3001.7020)的开头添加一个或更多元素，并返回新的长度。
>
> let castle={
> 
>     title:"live like a emporer",
> 
>     price:"$500,000,000",
> 
>     isSuperHost:true,
> 
>     images:["img/castle.jpg"],
> 
>     dosomething : function(){
> 
>         console.log("i m a func");
> 
>     }
> 
> }
> 
> castle.dosomething();
> 
> 







## 8、循环

> for(let count=1;count<11;count++){
> }



##  9、随机数

> let randomNum=Math.random()——生成一个0-1的随机数
>
> Math.random()*6——0-5.99999
>
> Math.floor(3.5)——得到3，向下取整
>
> let ra = Math.floor(-7.1);——得到-8
>
> Math.floor(Math.random()*6);//得到0-5整数的随机数



## 10、输入输出

> console.log
>
> alert("提示");
>
> prompt("请输入");





## 11、button_click

> let inputBtn = document.getElementById("input-btn");
>
> inputBtn.addEventListener("click",function(){
>
>   console.log("11");
>
> });
>
> 添加事件监听

> 语法：
>
> *element*.addEventListener(*event, function, useCapture*);
>
> 第一个参数是事件的类型 (如 "click" 或 "mousedown").
>
> 第二个参数是事件触发后调用的函数。
>
> 第三个参数是个布尔值用于描述事件是冒泡还是捕获。该参数是可选的。

> ## 事件冒泡或事件捕获？
>
> 事件传递有两种方式：冒泡与捕获。
>
> 事件传递定义了元素事件触发的顺序。 如果你将 <p> 元素插入到 <div> 元素中，用户点击 <p> 元素, 哪个元素的 "click" 事件先被触发呢？
>
> 在 *冒泡* 中，内部元素的事件会先被触发，然后再触发外部元素，即： <p> 元素的点击事件先触发，然后会触发 <div> 元素的点击事件。
>
> 在 *捕获* 中，外部元素的事件会先被触发，然后才会触发内部元素的事件，即： <div> 元素的点击事件先触发 ，然后再触发 <p> 元素的点击事件。
>
> 默认值为 false, 即冒泡传递，当值为 true 时, 事件使用捕获传递。
>
> 说人话就是：
>
> 冒泡——由内及外 捕获——由外到内

## 二、进阶

## 1、Filter

> 类似于WPF的converter，就是对对象数据作第二次映射
>
> filters are deprecated”表示使用了一个已经被弃用的过滤器
>
> 在Vue 3中，过滤器（Filters）已经被废弃，不再是官方推荐的用法。取而代之的是使用计算属性（Computed Properties）或者方法（Methods）来实现类似的功能。
>
> 比如：1——正确，2——错误
>
> ```vue
> <template>
>   <img alt="Vue logo" src="./assets/logo.png">
>   <div>
>     <table border="1">
>       <tr v-for="(item,index) of OrderData" :key="index">
>         <td>{{ item.price}}</td>
>         <td>{{ item.orderId}}</td>
>         <td>{{ item.owner}}</td>
>         <td>{{ item.status | getOrderStatus }}</td>
>       </tr>
>     </table>
>   </div>
>   <HelloWorld msg="Welcome to Your Vue.js App"/>
> </template>
> 
> <script>
> import HelloWorld from './components/HelloWorld.vue'
> import OrderData from '@/data/order'
> import OrderFilter from '@/filters/order'
> export default {
>   name: 'App',
>   filters:{OrderFilter},
>   components: {
>     HelloWorld
>   },
>   data() {
>     return {
>       OrderData: OrderData,
>     }
>   }
> }
> </script>
> ```
>
> 
>
> ```js
> export default{
>     getOrderStatus(status){
>         switch(status)
>         {
>             case 1:
>                 return "待付款";
>             case 2:
>                 return "待发货";
>             case 3:
>                 return "已付款未发货";
>             case 4:
>                 return "已付款已发货";
>             case 5:
>                 return "签收完成";
>         }
>     }
> }
> ```
>
> 



## 2、XHR-XML HTTP Request

> 发送异步请求
>
> ```javascript
> function B()
> {
>     var xhr = new XMLHttpRequest();
>      
>      xhr.onload = function () {
>          // 输出接收到的文字数据
>          document.getElementById("demo").innerHTML=xhr.responseText;
>      }
>       
>      xhr.onerror = function () {
>          document.getElementById("demo").innerHTML="请求出错";
>      }
>      // 发送异步 GET 请求
>      xhr.open("GET", "https://www.runoob.com//try/ajax/ajax_info.txt", true);
>      xhr.send();
> }
> ```
>
> 





## 3、Promise异步请求

> Promise 对象代表一个异步操作，有三种状态：Pending（进行中）、Resolved（已完成，又称 Fulfilled）和 Rejected（已失败）

> 与XHR的区别：
>
> `XMLHttpRequest`（XHR）和 `Promise` 是两个不同概念的东西，虽然它们都涉及异步操作，但用途和机制不同。
>
> 1. **XMLHttpRequest (XHR)**:
>    - `XMLHttpRequest` 是一个用于发起 HTTP 请求的浏览器内置对象，它最初用于在不刷新整个页面的情况下更新页面内容（Ajax）。
>    - XHR 允许你发送请求到服务器，等待服务器响应，并在响应到达时执行回调来处理数据。
>    - XHR 是传统的异步操作方式，通过监听事件（如 `onload`、`onerror`）来处理异步操作的结果。
> 2. **Promise**:
>    - `Promise` 是一种更现代的异步操作机制，用于更优雅地处理异步任务。
>    - Promise 是一个包装器，它可以将异步操作包装成一个对象，这个对象有三种状态：pending（进行中）、fulfilled（已完成，又称 resolved）和rejected（已拒绝）。
>    - 通过 `then` 方法，你可以将回调函数附加到 Promise，以在异步操作完成时执行。
>
> 主要区别：
>
> - XHR 主要用于发起网络请求，获取数据。它是一种底层的手段。
> - Promise 是一种处理异步操作结果的更高级别的机制。它将异步操作封装成一个对象，提供了更好的语法和结构，使得处理异步代码更加清晰和可维护。
>
> 在现代 JavaScript 中，`Promise` 已经被 `async/await` 这种更为便捷的异步编程方式取代，`async/await` 基于 Promise，使得异步代码看起来更像同步代码。

> ```js
> function C()
> {
>     // 创建一个Promise对象
> const myPromise = new Promise((resolve, reject) => {
>   // 模拟一个异步操作
>   setTimeout(() => {
>     const randomNum = Math.random();
>     if (randomNum > 0.5) {
>       resolve(randomNum); // 成功时调用resolve，并传递结果
>     } else {
>       reject("Operation failed"); // 失败时调用reject，并传递错误信息
>     }
>   }, 1000);
> });
> 
> // 使用Promise
> myPromise
>   .then(result => {
>     console.log("Operation succeeded. Result:", result);
>   })
>   .catch(error => {
>     console.error("Operation failed. Error:", error);
>   });
> }
> ```
>
> 

> ### Promise 的构造函数
>
> Promise 构造函数是 JavaScript 中用于创建 Promise 对象的内置构造函数。
>
> Promise 构造函数接受一个函数作为参数，该函数是同步的并且会被立即执行，所以我们称之为起始函数。起始函数包含两个参数 resolve 和 reject，分别表示 Promise 成功和失败的状态。
>
> 起始函数执行成功时，它应该调用 resolve 函数并传递成功的结果。当起始函数执行失败时，它应该调用 reject 函数并传递失败的原因。
>
> Promise 构造函数返回一个 Promise 对象，该对象具有以下几个方法：
>
> - then：用于处理 Promise 成功状态的回调函数。
> - catch：用于处理 Promise 失败状态的回调函数。
> - finally：无论 Promise 是成功还是失败，都会执行的回调函数。



> 声明一个Promise并调用，有点像C#的Task
>
> ```js
> function print(delay, message) {
>     return new Promise(function (resolve, reject) {
>         setTimeout(function () {
>             console.log(message);
>             resolve();
>         }, delay);
>     });
> }
> 
> print(1000, "First").then(function () {
>     return print(4000, "Second");
> }).then(function () {
>     print(3000, "Third");
> });
> ```
>
> 





## 4、异步函数

> 其实就是Promise的封装
>
> 上面的代码可以替换如下：
>
> ```js
> async function asyncFunc() {
>     await print(1000, "First");
>     await print(4000, "Second");
>     await print(3000, "Third");
> }
> asyncFunc();
> ```
>
> 





## 5、闭包

> 不是很理解
>
> 核心：==闭包是一种保护私有变量的机制，在函数执行时形成私有的作用域，保护里面的私有变量不受外界干扰。==
>
> 直观的说就是形成一个不销毁的栈环境。
>
> ```js
> <!DOCTYPE html>
> <html>
> <head>
> <meta charset="utf-8">
> <title>菜鸟教程(runoob.com)</title>
> </head>
> <body>
> 
> <p>局部变量计数。</p>
> <button type="button" onclick="myFunction()">计数!</button>
> <p id="demo">0</p>
> <script>
> var add = (function () {
>     var counter = 0;
>     return function () {return counter += 1;}
> })();
> function myFunction(){
>     document.getElementById("demo").innerHTML = add();
> }
> </script>
> 
> </body>
> </html>
> ```
>
> 每次都能+1





## 6、原型链（protoType）

> ## prototype 继承 原型继承
>
> 所有的 JavaScript 对象都会从一个 prototype（原型对象）中继承属性和方法：
>
> - `Date` 对象从 `Date.prototype` 继承。
> - `Array` 对象从 `Array.prototype` 继承。
> - `Person` 对象从 `Person.prototype` 继承。
>
> 所有 JavaScript 中的对象都是位于原型链顶端的 Object 的实例。
>
> JavaScript 对象有一个指向一个原型对象的链。当试图访问一个对象的属性时，它不仅仅在该对象上搜寻，还会搜寻该对象的原型，以及该对象的原型的原型，依次层层向上搜索，直到找到一个名字匹配的属性或到达原型链的末尾。
>
> `Date` 对象, `Array` 对象, 以及 `Person` 对象从 `Object.prototype` 继承。
>
> ### 添加属性和方法
>
> 有的时候我们想要在所有已经存在的对象添加新的属性或方法。
>
> 另外，有时候我们想要在对象的构造函数中添加属性或方法。
>
> 使用 prototype 属性就可以给对象的构造函数添加新的属性：
>
> ```js
> <!DOCTYPE html>
> <html>
> <head>
> <meta charset="utf-8">
> <title>菜鸟教程(runoob.com)</title>
> </head>
> <body>
> 
> <h2>JavaScript 对象</h2>
> 
> <p id="demo"></p>
> 
> <script>
> function Person(first, last, age, eye) {
>   this.firstName = first;
>   this.lastName = last;
>   this.age = age;
>   this.eyeColor = eye;
> }
> 
> Person.prototype.nationality = "English";
> Person.prototype.name = function() {
>   return this.firstName + " " + this.lastName;
> };
> var myFather = new Person("John", "Doe", 50, "blue");
> document.getElementById("demo").innerHTML =
> "我父亲的国籍是 " + myFather.nationality; 
> </script>
> 
> </body>
> </html>
> ```
>
> 



## 7、页面跳转和刷新

> ## history.back()——加载历史列表中的前一个 URL
>
> history.forward() 方法加载历史列表中的下一个 URL
>
> 除此之外可以用 **history.go()** 这个方法来实现向前，后退的功能。
>
> ```
> function a(){
>     history.go(1);  // go() 里面的参数表示跳转页面的个数 例如 history.go(1) 表示前进一个页面
> }
> function b(){
>     history.go(-1);  // go() 里面的参数表示跳转页面的个数 例如 history.go(-1) 表示后退一个页面
> }
> history.go() 这个方法来实现刷新的功能。
> 
> function a(){
>     history.go(0);  // go() 里面的参数为0,表示刷新页面
> }
> ```





## 8、浏览器信息

> ```js
> <!DOCTYPE html>
> <html>
> <head>
> <meta charset="utf-8">
> <title>菜鸟教程(runoob.com)</title>
> </head>
> <body>
> 	
> <div id="example"></div>
> <script>
> txt = "<p>浏览器代号: " + navigator.appCodeName + "</p>";
> txt+= "<p>浏览器名称: " + navigator.appName + "</p>";
> txt+= "<p>浏览器版本: " + navigator.appVersion + "</p>";
> txt+= "<p>启用Cookies: " + navigator.cookieEnabled + "</p>";
> txt+= "<p>硬件平台: " + navigator.platform + "</p>";
> txt+= "<p>用户代理: " + navigator.userAgent + "</p>";
> txt+= "<p>用户代理语言: " + navigator.language + "</p>";
> document.getElementById("example").innerHTML=txt;
> </script>
> 
> </body>
> </html>
> ```
>
> ## 警告!!!
>
> 来自 navigator 对象的信息具有误导性，不应该被用于检测浏览器版本，这是因为：
>
> - navigator 数据可被浏览器使用者更改
> - 一些浏览器对测试站点会识别错误
> - 浏览器无法报告晚于浏览器发布的新操作系统



## 9、window自带的窗体

> 警告框、确认框、提示框
>
> 1、alert
>
> 2、confirm
>
> ```js
> var r=confirm("按下按钮");
> if (r==true)
> {
>     x="你按下了\"确定\"按钮!";
> }
> else
> {
>     x="你按下了\"取消\"按钮!";
> }
> ```
>
> 3、prompt
>
> ```js
> var person=prompt("请输入你的名字","Harry Potter");
> if (person!=null && person!="")
> {
>     x="你好 " + person + "! 今天感觉如何?";
>     document.getElementById("demo").innerHTML=x;
> }
> ```
>
> 



## 10、cookie

> ## 什么是 Cookie？
>
> Cookie 是一些数据, 存储于你电脑上的文本文件中。
>
> 当 web 服务器向浏览器发送 web 页面时，在连接关闭后，服务端不会记录用户的信息。
>
> Cookie 的作用就是用于解决 "如何记录客户端的用户信息":
>
> - 当用户访问 web 页面时，他的名字可以记录在 cookie 中。
> - 在用户下一次访问该页面时，可以在 cookie 中读取用户访问记录。
>
> Cookie 以名/值对形式存储，如下所示:
>
> username=John Doe
>
> 当浏览器从服务器上请求 web 页面时， 属于该页面的 cookie 会被添加到该请求中。服务端通过这种方式来获取用户的信息。

> ## 使用 JavaScript 创建Cookie
>
> JavaScript 可以使用 **document.cookie** 属性来创建 、读取、及删除 cookie。
>
> JavaScript 中，创建 cookie 如下所示：
>
> document.cookie="username=John Doe";
>
> 您还可以为 cookie 添加一个过期时间（以 UTC 或 GMT 时间）。默认情况下，cookie 在浏览器关闭时删除：
>
> document.cookie="username=John Doe; expires=Thu, 18 Dec 2043 12:00:00 GMT";
>
> 您可以使用 path 参数告诉浏览器 cookie 的路径。默认情况下，cookie 属于当前页面。
>
> document.cookie="username=John Doe; expires=Thu, 18 Dec 2043 12:00:00 GMT; path=/";



> Cookie的作用：
>
> 
> Cookie通常用于保存一些小而简单的数据，如：
>
> 1. **用户身份认证信息：** 在用户登录后，服务器可以发送一个包含用户标识信息的Cookie，以便在用户的每个请求中识别他们。
> 2. **会话状态：** 用于跟踪用户的会话，例如购物车中的商品、表单数据的保存等。
> 3. **个性化设置：** 存储用户的偏好设置，如网站主题、语言、字体大小等。
> 4. **广告和跟踪：** 用于在不同页面之间跟踪用户的活动，以提供个性化广告或分析用户行为。
> 5. **安全性标志：** 用于实现安全机制，例如防止跨站请求伪造（CSRF）攻击，可以将一个随机生成的令牌存储在Cookie中。
> 6. **记住我功能：** 在用户选择“记住我”选项时，可以将持久性Cookie用于长时间内免登录。
>
> 需要注意的是，由于Cookie是在客户端存储的，因此不应将敏感信息（如密码）存储在Cookie中，以避免安全风险。此外，根据GDPR等隐私法规，网站需要明示地告知用户他们的数据将被存储在Cookie中，并获得用户的同意。





> 检查cookie是否存在
>
> ```js
> <!DOCTYPE html>
> <html>
> <head>
> <meta charset="utf-8">
> <title>菜鸟教程(runoob.com)</title>
> </head>
> <head>
> <script>
> function setCookie(cname,cvalue,exdays){
> 	var d = new Date();
> 	d.setTime(d.getTime()+(exdays*24*60*60*1000));
> 	var expires = "expires="+d.toGMTString();
> 	document.cookie = cname+"="+cvalue+"; "+expires;
> }
> function getCookie(cname){
> 	var name = cname + "=";
> 	var ca = document.cookie.split(';');
> 	for(var i=0; i<ca.length; i++) {
> 		var c = ca[i].trim();
> 		if (c.indexOf(name)==0) { return c.substring(name.length,c.length); }
> 	}
> 	return "";
> }
> function checkCookie(){
> 	var user=getCookie("username");
> 	if (user!=""){
> 		alert("欢迎 " + user + " 再次访问");
> 	}
> 	else {
> 		user = prompt("请输入你的名字:","");
>   		if (user!="" && user!=null){
>     		setCookie("username",user,30);
>     	}
> 	}
> }
> </script>
> </head>
> 	
> <body onload="checkCookie()"></body>
> 	
> </html>
> ```
>
> 
