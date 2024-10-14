# Udemy排名第一的JavaScript课程



三天学完（共200集）



> 网页的console控制台其实就是js代码



### 一、基础部分

> 1、Const、Let 和var
>
> let是ES6 新的，相比var有不少优势
>
> let声明变量，const声明常量
>
> let是块作用域，var是函数范围内的





> 2、字符串表示
>
> JS 中单引号和双引号无任何区别，二者均用于表示字符串字面量。
>
> 单引号和双引号混合使用时，内层引号将被视为字符串的一部分。
>
> JS中类型会自动转换，比如数字转字符串
>
> js中``反引号
>
> 从 ECMAScript 6 开始，表示字符串引入了新的方法，即使用反引号（`）来表示模板字符串。
>
> 例：console.log(`I m ${firstName},a ${year-birthYear} years old`)
>
> 反引号不用打换行符\n ，直接在反引号内换行即可。



> 3、类型转换
>
> Number(字符串)——类型转换，原始值不会改变，如果字母想转成数字，则是NAN类型
>
> 类型自动转换时，加号 数字变成字符串拼接 减号 字符串变数字反向
>
> 2+3+4+‘5’ ——输出95
>
> 2+3+4-'5'——输出4
>
> 







> 4、虚假值
>
> JS中：0、''、undefined、null、NAN ——会自动转换成false
>
> 任何非0数字或字符串都是真，注意空对象也是真，但是空字符串是假
>
> 数字0是假，字符串0为真
>
> ```javascript
> let height;
> if(height)
>     {
>         
>     }
> else
>     {
> }
> undefined类型 未定义式，走到else处
> ```
>
> 





> 5、比较运算符 == 和 ===
>
> 18===18，比较两数是否相等，返回true，不但判断值，还判断类型
>
> ===是严格相等运算符，必须完全相等，它不做类型转换
>
> ‘18‘ == 18，返回true
>
> !== (类型相同，其值也相同)   和  != 



> 6、Prompt方法
>
> # prompt()方法用于显示可提示用户进行输入的对话框。方法返回用户输入的字符串。
>
> prompt("请输入电话号码");





> 7、激活严格模式
>
> 在js代码第一行中
>
> ’use strict‘;
>
> 严格模式禁止我们做某些事，会产生可见的错误
>
> 其实就是错误会在控制台显示红色





> 8、函数
>
> function logger(apples,oranges){
>
> ​	console.log(apples,oranges);
>
> }
>
> 也可以有返回值
>
> ### JS的函数类型：命名函数、匿名函数、箭头函数
>
> 箭头函数很重要！
>
> const CalAge = birthYear => 2037-birthYear;

![image-20220824094350879](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20220824094350879.png)







> 9、数组
>
> const friends=['Jack','Mark','Alice']
>
> console.log(friends);
>
> 或
>
> const friends = new Array('Jack','Mark','Alice')
>
> friends.length ——3个
>
> 虽然是const类型，但能改变数组里的值，因为数组是指针
>
> 数组的基本操作：
>
> * push——添加到最右边
> * unshift——添加到最左边
> * pop——移除最右边
> * shift——移除最左边
> * indexof——返回索引，没有就是-1
> * includes——返回bool值，注意是严格测试===
> * array.min()
> * array.max()
> * array1.concat(array2)





> 10、对象
>
> JS中的对象其实是键值对，用{}表示
>
> const LXW={Name:'lxw',job:'teacher',friends:['Michael','Alice','Tom']};
>
> 获取对象属性 用.表示 或者 对象名['key']
>
> 对象里 值 也可以放函数 的返回值
>
> 对象 内部 函数 用this指的就是对象本身



> 11、控制语句
>
> for循环：
>
> for(let rep=1;rep<10;rep++)
>
> while循环



> 12、Node JS 实时服务器
>
> JS中实时编辑，两种方法
>
> :one:可以安装liveServer扩展
>
> :two:安装Node.js环境（服务器）
>
> 平时，js是通过浏览器的V8引擎将js代码编译成机器代码，所以脱离浏览器，js代码将无法运行
>
> Node是V8引擎的容器，Node是用C++写的，可以直接运行在电脑上





> 13、什么是DOM
>
> DOM代表文档对象模型，实际啥html文档的结构化表示（class 和 id 以及各种选择器）
>
> DOM允许我们使用JS访问HTML元素和样式来操作它们
>
> 所以说DOM和html代码和js代码之间的一个联络点
>
> Document作为整个页面对象的入口点，需要它来选择元素
>
> ![image-20220824183138717](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20220824183138717.png)

例

```javascript
document.querySelector('.message').textContent='woaini'
//将一个message类中的文本改成woaini
```

querySelector只会选中符合条件的第一个，想要选中多个类要用querySelectorAll

*const* btnsOpenModal = *document*.querySelectorAll('.show-modal');

querySelector和getElementById 效果大致一样，但后者处理速度要快一些



> 14、处理点击事件（click）
>
> *document*.querySelector('.again').addEventListener('click', *function* () {
>
>  score = 20;
>
>  secretNumber = Math.trunc(Math.random() * 20) + 1;
>
> }
>
> .again是按钮的class名
>
> ### 为一个按钮添加事件监听器，click告诉它触发的条件是点击
>
> addEventListener方法是js中最常用的事件监听方法

```javascript
document.addEventListener('keydown', function (e) {
  // console.log(e.key);

  if (e.key === 'Escape' && !modal.classList.contains('hidden')) {
    closeModal();
  }
});
//按键按下esc时触发
```

要移除一个class 可以用 .classlist.remove(类名)



> 15、JS中的作用域
>
> #### 三类：全局作用域、函数作用域、块作用域（ES6）
>
> 块作用域就是{}的内容，注意块作用域里只适用于let或const声明的变量
>
> #### 若在块作用域中用var声明，则函数内部仍然能访问
>
> 用Var声明的对象其实是window窗口对象的一个属性Window.X





> 16、JS中this的用法
>
> 注意：箭头函数中没有自己的this，所以表示的是父级元素（比如window或其他父级对象），但有时我们需要的就是父级元素！很重要!
>
> 普通函数，this的对象就是本身
>
> 所以说箭头函数内要用this一定要当心，它指向的不是当前对象



> arguments关键字
>
> arguments 只能在常规函数里面使用（箭头函数不行）, 在函数外使用 就会报错
> (1)arguments的作用 : 获取函数的所有实参
> (2)arguments是一个伪[数组](https://so.csdn.net/so/search?q=数组&spm=1001.2101.3001.7020)
> (3)修改了形参, arguments也会随之改变
> (4)修改了arguments的值, 形参也会发生改变











---



### 二、进阶部分

> 1、JS有垃圾回收机制，是动态类型语言（弱类型）
>
> 几乎每个浏览器都有js引擎（用的最多的是V8）
>
> JS过去是是解释型语言，现在混合使用解释和编译（即时编译型）
>
> JS中有头等函数——First-Class Function，即函数可以相当于变量来调用
>
> ```javascript
> const demo = ()=>{
>     model.classList.add('hidden');
> }
> overlay.addEventListener('click',demo);
> ```
>
> 要用Web Api必须基于浏览器





> 2、JS原始类型 和 引用类型
>
> ### 注意：JS中只有按值传递，没有按引用传递
>
> | Primitives（栈） | Objects（内存堆） |
> | ---------------- | ----------------- |
> | Number           | Object Literal    |
> | String           | Arrays            |
> | Boolean          | Functions         |
> | Undefined        |                   |
> | Null             |                   |
> | Symbol           |                   |
> | BigInt           |                   |
>
> 
>
> ![image-20220826101501410](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20220826101501410.png)



> 如何改变对象，一个变都变呢？
>
> ## 通过浅拷贝 ！
>
> const jessicaCopy = Object.assign({},jessica);
>
> 浅拷贝只会拷贝属性在第一级（有层级关系的，如果你对象下面还有一个数组，则无效了），而深度克隆会复制所有内容







> 3、数组解构（Destructuring）
>
> 分解大的数据结构成较小的数据结构
>
> ```Javascript
> *const* arr=[1,2,3,4,5]
> 
> *const* [a,b,c,d,e]=arr;
> 
> *console*.log(a,b,c,d,e);
> ```



> 解构运算符中间留空隙用于跳过
>
> const[first,,second]=restaurant.food;



> 利用解构来完成数组元素的交换，可以省去临时变量temp
>
> [main,secondary] =[secondary,main]



> 嵌套解构
>
> ```javascript
> const nested = [2,5,[7,9]]
> const [i,,[j,k]]=nested //返回 2 ， [7,9]
> ```



> 设置缺省值
>
> const [p=1,q=2,r]=[8,9]





> 4、解构对象
>
> 对解析数据很有用
>
> 可以换名字：
>
> const {name:restaurantname,openingHours:hours} = restaurant;
>
> ```javascript
> const {menu=[],starterMenu:starters=[]}=restaurant;
> //这里定义对象starterMenu，起别名为starter，默认为空，如果有数据则取restaurant对象里的数据
> ```
>
> 先定义函数，后传入对象
>
> ```javascript
> const asd = {
>   name:'lxw',
>   order:function(t){
>     console.log(t);
>   }
> }
> 
> asd.order({
>   time:'today',
>   weight:20000,
>   height:1
> });
> ```
>
> 





> 5、扩展运算符
>
> "==...=="扩展运算符可以将数组中的每个元素或对象中的每个键值对取出 并且不能再模板字符串中使用
>
> ```javascript
> let array=[2,6,11,1];
> let ppye=[1,...array,234];
> 可以很轻松地完成数组任意位置地添加
> ```
>
> 扩展运算在数组操作时很有用
>
> const menuCopy=[...restaurant.mainMenu]
>
> //这一步相当于直接创建了一份restaurant对象的一个子单元的浅拷贝
>
> 扩展运算符适用于可迭代对象，字符串也行
>
> 





> 6、其余模式（rest）...
>
> 跟上面的扩展相反，将多个元素压缩成一个数组
>
> const [a,b,...others]=[1,2,3,4,5]
>
> 可以得到a=1，b=2，others=[3,4,5]
>
> 其余元素必须是最后一个元素，否则不知道其余是多少
>
> rest模式常用于函数参数的可变扩展（感觉类似于C# 的params）
>
> ```javascript
> const add = function(...numbers)
> {
>     let sum=0;
>     for(let i=0;i<numbers.length;i++)
>         {
>             sum+=numbers[i];
>         }
> }
> add(2,3);
> add(3,5,6,1,2);
> ```
>
> 
>
> 





> 7、JS中也有短路现象
>
> or 和 And
>
> or操作链中会找第一个拥有真实value的值，如果没有才返回相应的null 或者 undefined
>
> &&相反，找到第一个为false的值，找到了则返回该值。为真就继续走，输出最后一个真实的值
>
> ```javascript
> rest.number = rest.number||10;
> 等价为
> rest.number||=10;
> ```
>
> #### 逻辑空运算符：？？
>
> rest.number??=10;







> 8、for-of 循环
>
> 相当于foreach，你可以continue 或者 break
>
> for(const item of menu)
>
> {
>
> }





> 9、对象数组的可选链接(optional chaining)
>
> 当某个子对象不存在时，不会继续搜索子子对象
>
> console.log(restaurant.openingHours.mon?.open)
>
> 只有当mon属性在？.之前存在时，才会接下去继续判断
>
> 即如果mon属性不存在，不会接下去判断
>
> 或者这样用和？？一起用
>
> console.log(users[0]?.name??"User is empty");
>
> 判断user对象第一个对象的name属性是否存在，若不存在，输出user is empty





> 10、循环对象（即时是不可迭代的）
>
> ```javascript
> for(const day of Object.keys(openingHours))
>     {
>         console.log(day);
>     }//获取键
> Object.values(对象名) // 获取值
> ```
>
> Object.entries() 方法返回一个给定对象自身可枚举属性的键值对数组。
>
> 其排列与使用 for...in 循环遍历该对象时返回的顺序一致（区别在于 for-in 循环还会枚举原型链中的属性）。
>
> 语法
> Object.entries(obj)
>
> 参数
> obj：可以返回其可枚举属性的键值对的对象。
>
> 返回值
> 给定对象自身可枚举属性的键值对数组。
>
> 描述
> Object.entries()返回一个数组，其元素是与直接在object上找到的可枚举属性键值对相对应的数组。属性的顺序与通过手动循环对象的属性值所给出的顺序相同。



> 11、新的数据结构 sets 和 maps
>
> 一个sets没有重复对象，set里放的是可迭代对象
>
> .size——可返回无重复值的数量，注意不是length
>
> .has('pizza')——判断是否存在
>
> .add('bread')——添加对象
>
> .delete('rice')——根据名字删除对象
>
> ### 注意：set不能通过像数组那样的索引来访问



> map很重要
>
> 对象和maps的区别：
>
> ### 在maps中，key可以有任何类型，而在对象中，键基本上都是字符串
>
> ```javascript
> const res = new Map();
> res.set('name','lxw');
> res.set(1,"zz")
> res.set('categories',['Italian','China','England'])
> res.set([1,2],'Test')
> res.get(1);
> ```
>
> .size
>
> .has
>
> .get
>
> 







> 12、数据来源和数据结构选择
>
> * 源程序数据
> * 用户界面（从网页或者DOM）
> * 从外部源（WebApi）
>
> set要比array速度快
>
> object和map的区别：
>
> map更适合用于简单的键值存储，因为它提供了更好的性能。并且易于迭代
>
> object的优势在于其编写和访问数据比较容易









### 三、高阶部分

> 1、高阶函数+回调函数
>
> ```javascript
> const transformer = function(str,fn){
>     fn(str);
> }
> 
> transformer ('JavaScript',upperFirstword);
> 
> //传一个变量和一个函数进去，这里的fn就是回调函数
> ```
>
> 事件监听的本质其实就是一个高阶函数（addEventListener）+一个回调函数（func）
>
> 这是一种抽象概念

> 函数返回其他函数
>
> ```javascript
> const greet=function(greeting)
> {
>     return function(name){
>         console.log('${greeting}+${name}');
>     }
> }
> 
> const greetHey = greet('Hey');
> greetHay('Lxw');
> //或者直接调用
> greet('Hello')('Lxw');
> -------------------------------------------------------------
>     //利用箭头函数实现
>     
> const greetArr=greeting=>name{
>     console.log('${greeting} &{name}');
> }
> greetArr('Hello')('Lxw');
> ```
>
> 





> 2、函数的调用和应用（call & apply）
>
> book.call(对象名，函数参数1，函数参数2)；
>
> 由于不同对象的this不同，用book.call可以实现不同对象调用同一功能的函数
>
> apply和call的区别在于，apply传的对象是数组
>
> const flightData=[583,'George'];
>
> book.apply(swiss,flightData); 
>
> ### Apply现在已经不用了
>
> ### 用book.call(swiss,...flightData) 可以同样实现







> 3、Bind方法
>
> ### 绑定其实就是为了控制this的指向
>
> 绑定不会立即调用该函数，相反，它返回一个新函数
>
> 绑定到对象身上，相当于指定了this，也可以传入参数





> 4、async和await
>
> 立即调用函数表达式
>
> ```js
> (function(){
>     console.log("12345");
> })();
> 或者
> (()=> console.log("12345"))()
> ```





> 5、闭包(Closure)
>
> 闭包产生了一个函数记住所有存在的变量
>
> ```js
> const secureBooking=function(){
>     let passengerCount=0;
>     return function(){
>         passengerCount++;
>         console.log(`${passengerCount} passengers`);
> }
> }
> const booker = secureBooking();
> booker();
> booker();
> booker();
> //可看见控制台 打印 1 passengers 2 passengers 3 passengers
> ```
>
> 一个函数总算可以访问变量环境创建它的执行上下文，即时在执行环境消失后
>
> 闭包基本上就是这个变量环境附加到函数中，与该函数创建时的环境一模一样
>
> 闭包是封闭变量执行上下文的环境，在其中创建一个函数，即时在那个执行上下文消失之后
>
> console.dir(booker);
>
> 闭包跟事件函数息息相关





> 6、数组进阶学习
>
> > :one:切片（slice）
> >
> > ```js
> > let arr=['a','b','c','d','e']
> > console.log(arr.slice(2))
> > //返回从第二个位置开始的数组  c，d，e  等价于 arr.slice(2,4)
> > arr.slice(-2)
> > //返回d，e
> > ```
> >
> > 切片不改变原始数组
>
> > :two: 拼接Splice
> >
> > 此会改变原数组
> >
> > arr.splice(2) //去除原数组的前2个元素
> >
> > arr.splice(1,2)//去除第一个元素开始后面2个元素 返回 a d
>
> > :three:反转Reverse
> >
> > 此方法会改变原数组
>
> > :four:连接Concat
> >
> > 此方法不会改变原数组
> >
> > 用于连接两个数组
> >
> > arr1.concat(arr2)
>
> > :five:分隔(Join)
> >
> > 此方法不会改变原数组
>
> > :six:foreach
> >
> > 与for of 比较
> >
> > const array=[1,2,3,4,5]
> >
> > for(const item of array)
> >
> > {
> >
> > console.log(item);
> >
> > }
> >
> > ### foreach实质是个高阶函数
> >
> > array.foreach(function(item){
> >
> > console.log(item);
> >
> > })
> >
> > ```js
> > const array=[1,2,3,4,5]
> > array.forEach(function(item,index,arrayNow))
> > ```
> >
> > forEach传参——第一个总是item，第二个总是索引，第三个是我们正在遍历的那个数组即array
> >
> > 注意：forEach不能用continue和break，如果你要跳出循环需要用for of
>
> > forEach和map 、 set 使用
> >
> > ```js
> > const currencies=new Map([
> >     	['USD':'United States dollar'],
> >         ['EUR':'Europe currency'],
> >         ['GBP':'Pound Sterling']
> > ]);
> > currencies.forEach(funtion(value,key,map){
> >            console.log('${key}:${value}')        
> >                    })
> > ```
> >
> > set没有key forEach遍历时 key和value是相同的





> 7、数组最重要的三个数据转换方法（Map、Filter、Reduce）
>
> Map：
>
> MAP会返回一个新数组
>
> ```js
> const movements=[200,450,-200,-300,500]
> 
> cost eurToUsd=1.1;
> 
> const movementsUSD=movements.map(function(mov){
>     return mov*eurToUsd;
> })
> ```
>
> 
