# C++游戏服务器编程







## 一、TCP/IP详解

![image-20221008125848106](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20221008125848106.png)

> 数据链路层传输数据最少为46B（不足补0），最大为1500B（以太网）——MTU
>
> 若传输数据大于1500B，则在IP层需要分片

> ### IP连接是不可靠，无连接的
>
> #### 网络数据都是按大端方式传输的

> IP协议首部（最少20B）
>
> ![image-20221008133615391](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20221008133615391.png)



---

> TCP部分
>
> TCP是全双工的
>
> 问：TCP是如何把IP无连接不可靠变成有连接可靠的
>
> TCP将应用程序的传输数据分割成合适的数据块，TCP提供流量控制和拥塞控制
>
> 1、TCP首部：
>
> 无选项是 20 个字节 
>
> ![image-20221009195524280](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20221009195524280.png)
>
> 标志位详解：
>
> URG：紧急指针，会把第17字节的Urgent Pointer置为1，先处理此消息。
>
> Windows Size：占2个字节，表示发送数据的大小
>
> RST：要找的机器的服务没有打开，或者请求超时
>
> Time-Wait——确认对方收到自己的关闭消息（一般是客户端用）
>
> Close-Wait——等待自己应用程序关闭
>
> Time-Wait期间 socket pair 还没完全释放	

![image-20221009203655008](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20221009203655008.png)



![image-20221009205157529](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20221009205157529.png)

> Fin_Wait1 客户端主动关闭的
>
> Fin_Wait2 收到服务端被动关闭的确认
>
> Fin：表示服务端没有别的消息再发送给客户端了



> Flag 对应（三次握手）
>
> Flag——SYN 表示client主动发给server
>
> Flag——SYN、ACK 且ACK=1 表示server收到发给client
>
> Flag——ACK、且ACK=1 表示client也收到server了。连接正式建立



> Flag对应（四次挥手）
>
> Flag——Fin、ACK client关闭（client端）
>
> Flag——Ack Server端
>
> Flag——Fin,ACK Server端
>
> Flag——Ack ， Client端





## 二、C++知识点

> class A{
> }
>
> //编译器会为我们自动生成四个构造函数
>
> A(){} //默认构造
>
> A(const A&){}  //默认拷贝构造
>
> A& operator = (const A&) {return *this}; //重载等于号
>
> inLine ~A(){} //默认析构
>
> 

> Class B：A{
> }
>
> //多了一个基类的构造函数

> Class C{
>
> C（int a）{}
>
> }
>
> //会生成拷贝构造、重载等于、析构，而不会生成默认构造（那个空的）





## 三、Shell学习（服务器开发所需的）

》