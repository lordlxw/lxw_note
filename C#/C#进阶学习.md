# C#进阶内容

.Net历史：

<img src="C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20220528103924276.png" alt="image-20220528103924276" style="zoom:50%;" />

## 一、委托（action、func、predicate、comparison）

>什么是委托？====
>
>委托级别很高，和类是一个级别
>
>区别：类的实例叫做对象，==委托的实例叫做委托实例==
>
>核心：普通类型存储的是数据，而委托变量存储的是==行为==
>
>Action和Func——实质就是泛型委托，区别只是变量的个数
>
>Predicate ,有且只有一个参数，返回值只为 bool 类型
>
>Comparison委托:可以用作排序比较的委托

实例1：

``` C#
delegate void Do(); //定义一个委托

public class Program(){
    static void Main(string[] args)
    {
        Help h; //语法糖，直接生成委托实例，而不用new
        h=SayHello; //函数赋给委托实例
        
        h(); //委托实例的调用
        void SayHello(){
            Console.WriteLine("nihao");
        }
    }
}

```

示例2：

```C#
internal class Program
    {
        static void Main(string[] args)
        {

            GoStation(() =>
            {
                Console.WriteLine("我爱你");
            });
            
            Console.Read();
          
        }


        static void GoStation(Action action)
        {
            Console.WriteLine("起床");
            Console.WriteLine("刷牙");
            action();
            Console.WriteLine("吃饭");
        }

        static void Study()
        {
            Console.WriteLine("我要学习");
        }
    }
```







## 二、接口回调函数



> 如多线程的BeginInvoke()方法。委托实例.BeginInvoke(参量名,IAsyncCallback(IAsyncResult ar)-回调函数,)
>
> BeginInvoke返回的也是IAsyncResult 接口类型
>
> IAsyncCallback——当这个异步操作完成后所做的回调操作。即委托一完成就做callback操作

代码：

```C#
 static void Main(string[] args)
        {
            Del dl = TakeAWhile;

            IAsyncResult ar = dl.BeginInvoke(1, 350, (s) => { Console.WriteLine("Finished"); }, null);
            while(!ar.IsCompleted)
            {
                Console.WriteLine("1_1");
                Thread.Sleep(50);
            }
            Console.Read();

        }

        static int TakeAWhile(int data,int ms)
        {
            Console.WriteLine("TaskWaiting ...");
            Thread.Sleep(ms);
            Console.WriteLine("Task Completed");
            return ++data;
        }
    }

    public delegate int Del(int a, int b);
```



> ==异步编程核心：返回void和task——不要使用返回void,用返回task或是task的泛型即可==。
>
> void和task是有区别的，返回void的话调用这个方法的程序是没法追踪这个方法的。换句话说，返回void相当于告诉c#你不在乎这个异步方法会跑出什么东西来，也不在乎这个方法什么时候跑完。
>
> async, await 底层是状态机, 而如果返回值是void的话，调度方是不会有等待行为的，因为没有awaiter

















## 三、扩展方法

> 需要表示为静态类的静态方法

```C#
   public static class PersonExtension
    {
      public  static void ShowTelphoneNum(this Person person)
        {
            person.GetPhone();

        }
    }
```























## 四、EFCore和Ado.Net



### 1、特点与比较

> * 性能上（运行效率）
>
> Ado.Net的性能更高些，直接使用SQLHelper的Command、Connection等命令通过写[SQL语句](https://so.csdn.net/so/search?q=SQL语句&spm=1001.2101.3001.7020)对数据库进行操作。（EF的实体模型，性能上肯定要损失些！！）
>
> * EF最终都是翻译转换成sql去执行的，开发很快捷。ado相对来说你可以自行处理sql存储过程和脚本，灵活性大，不需要进行翻译，但工作量会相对多一些。
>
> * EF框架和Ado.Net，其实简单来说，就是封装和原生的PK了。
>
> * 适用性上
>
>   EF适合较大型的项目，数据量也较大些；而Ado.Net适用于小型项目（执行效率高些）。
>



### 2、ADO.Net

![image-20220529113459996](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20220529113459996.png)



两种方案：DataReader(一直连)  和 DataAdaptor+DataTable(操作完会断开) （都建立在Command上）

>性能上：SqlDataReader一次只在内存中存储一行，减少了系统开销。优于SqlDataAdapter。
>
>读取时：SqlDataReader需通过调用自身Read()方法循环读取数据到指定对象。而SqlDataAdapter可通过调用Fill()方法一次性填充数据到DataSet。还可将对 DataSet 所做的更改解析回数据源。
>
>操作上：SqlDataReader需通过调用自身的Close()方法断开连接。而SqlDataAdapter可以读取完数据库后自动断开连接.

#### 事务

> Key：Transaction 多条语句执行，一句失败，则都不执行
>
> * 开启事务
> * 提交事务
> * 回滚事务

``` C#
   public static int UpdateByTransaction(List<string> sqlList)
        {
            using(SqlConnection conn =new 						                                 SqlConnection(conStr))
            {
                SqlCommand cmd = new SqlCommand();
                cmd.Connection = conn;  
                conn.Open();
                cmd.Transaction = conn.BeginTransaction();  //开启事务
                int result = 0;

                if(cmd.Transaction!=null)
                {
                    cmd.Transaction.Rollback();        //若有一条语句有问题，回滚事务（撤销所有sql语句执行）
                }
                foreach(var sql in sqlList)  //循环提交sql语句
                {
                    cmd.CommandText = sql;
                    result=  cmd.ExecuteNonQuery();
                }
              return result;
            }
            
        }
```



### 3、EF框架——高级开发者才能熟练运用

> EF有数据迁移的功能：要装包：EntityFrameWorkCore.Tools  和  EntityFrameWorkCore
>
> 写好类库（即数据库表中的列），通过 包-程序控制台：Add-Migration （数据库名）可以利用自己写好的实体框架（类库）迁移数据。
>
> 在Config中配置要建立数据库名，update-database命令可以建立真正的数据库
>
> add-migration 命令是代码优先迁移中的关键命令之一。当您对域模型进行更改并需要将它们添加到数据库中时，您将创建一个新的迁移。这是通过 Add-Migration 命令完成的。在最简单的形式中，您只需要提供一个迁移名称。





> 在代码优先迁移中，有四个命令
>
> * enable-migrations
> * add-migration
> * update-database
> * get-migrations



> 缺点：自动建立数据映射，但string-》nvarchar（max）在进行数据库操作时会占用大量内存，需要自己改变变量类型。可以通过给属性 增加特性 实现
>
> [Required]——表面变量非空
>
> [MaxLength(100)]——变量最长100，所以不会是nvarcharmax
>
> [Column(TypeName="varchar(10)")]



### 4、显示加载和延迟加载（不推荐延迟加载）

Q:比如我一个Author表，关联了多个Books，启动延迟加载Author时，是不是会都访问Book表中的内容?

> 是的，如果您启用了延迟加载并访问了 `Author` 实体的关联属性 `Books`，那么在默认情况下，EF Core 会触发额外的数据库查询来获取该作者关联的书籍信息。这种情况下，每次访问 `Books` 属性时，都会执行一次查询来获取相关书籍数据。
>
> 这种行为通常会导致 N+1 查询问题。例如，如果您有多个作者，每个作者都关联了多本书，在遍历作者列表并访问每个作者的 `Books` 属性时，可能会导致执行大量的数据库查询，从而影响性能。
>
> 为了避免 N+1 查询问题，您可以使用显式加载（Eager Loading）来一次性加载所有关联的书籍信息，而不是每次访问 `Books` 属性时都触发查询。以下是使用显式加载的示例：
>
> ```csharp
> using Microsoft.EntityFrameworkCore;
> 
> // ...
> 
> var author = dbContext.Authors.Include(a => a.Books).FirstOrDefault(a => a.Id == authorId);
> ```
>
> 在上面的示例中，通过使用 `Include` 方法，您可以在加载 `Author` 实体时一并加载关联的 `Books` 数据，从而避免了多次查询。
>
> 需要根据实际情况权衡使用延迟加载和显式加载，以优化查询性能。延迟加载可以在某些情况下方便地加载相关数据，但也可能导致性能问题。显式加载则可以更精确地控制何时加载相关数据，避免不必要的查询。

> `dbContext.Authors.Include(a => a.Books)` 这句代码使用了 Entity Framework Core 的显式加载功能，它的作用是在查询 `Authors` 表的数据时，同时加载每个作者关联的 `Books` 数据，以便在同一查询中获取完整的作者和书籍信息。
>
> 具体来说，这行代码执行了以下操作：
>
> 1. `dbContext.Authors`：表示查询 `Authors` 表的数据集。
> 2. `.Include(a => a.Books)`：使用 `.Include` 方法来指示 EF Core 在加载 `Authors` 数据的同时，也加载每个作者关联的 `Books` 数据。这样，查询结果将包含每个作者的完整信息以及他们的书籍信息。
>
> 这并不会仅仅找到 `Books` 属性不为空的数据，而是在加载 `Authors` 数据时，将每个作者关联的书籍数据也一并加载。
>
> 总之，`dbContext.Authors.Include(a => a.Books)` 是在查询 `Authors` 数据时，同时加载每个作者关联的 `Books` 数据，以获取完整的作者和书籍信息。





> 使用显示加载可以避免可能由延迟加载引发的不必要的递归
>
> 延迟加载可能会触发递归加载，特别是在涉及到循环引用的情况下。这可能导致不断地加载关联的数据，直到达到递归的最大深度或引发异常。
>
> 为了避免递归加载问题，您可以采取以下措施之一：
>
> 1. **显式加载（Eager Loading）：** 使用 `.Include` 方法一次性加载需要的数据，从而避免在访问属性时触发递归加载。
> 2. **关闭延迟加载：** 如果不需要延迟加载，可以在上下文中禁用延迟加载功能。
> 3. **避免循环引用：** 在设计数据库和实体类时，尽量避免创建循环引用的关系，以减少递归加载的可能性。
> 4. **限制导航属性：** 考虑只加载需要的数据，而不是加载整个对象图。







## 五、Orm

























## 六、异步编程模式

### 1、线程概念

> * 线程代码 用try catch是捕捉不到异常的，解决方法是在线程执行的方法里加上try catch，任何线程有未处理的异常都会触发AppDomain.CurrentDomain.UnhandledException
> * 可通过IsBackground属性判断线程是否是后台线程（true，设为后台线程）
>
> IsBackground=false（默认） 说明***\**\*不是后台线程，所以主线程即使完成了，子线程也会继续完成\*\**\***
>
> ***\**\*这是一个 后台线程，\*\*IsBackground=true\*\*, 主线程完成后，后台子线程也停止了，\*\*即使 \*\*子线程\*\* 还有没运行完，也要停止\*\*\*\*\****
>
> 线程的创建一般有两种方式：1、委托创建线程 2、Thread类创建线程

### 2、Task和Thread

>#### Task并不是新启一个新线程，而是==运行时==调度可用的资源。
>
>* Thread是申请真实的资源。Thread存在的问题，无法告诉线程在结束时做别的工作，必须进行Join操作，很难使用较小并发组建大型并发。
>
>* Task能解决上述问题，底层使用线程池减少启动延迟，Tasks利用回调方式避免线程等待。
>
>* Task.Run(()=>{});         和 Task.Factory.StartNew(()=>{});  等价
>
>* Task默认使用线程池。即后台线程，当主线程结束时，创建的所有 Tasks都会结束。
>
>* Task的Wait方法会进行阻塞直到操作完成（类比thread的Join）
>
>* Task.Run适合用于短任务，长任务要使用LongRunning
>
> ```C#
> Task.Factory.StartNew(RefreshAll,TaskCreationOptions.LongRunning); //refreshAll方法是长任务，不断刷新界面
> ///
> 如果当前Task上的TaskCreationOptions设置为LongRunning的话，这个task就会委托到Thread中去执行，这个task就会委托到Thread中去执行，这样的好处显而易见，如果长时间运行的task占用着ThreadPool的线程，这时候ThreadPool为了保证线程充足，会再次开辟一些Thread，如果耗时任务此时释放了，会导致ThreadPool线程过多，上下文切换频繁，所以这种情况下让Task在Thread中执行还是非常不错的选择，当然如果你不指定这个LongRunning的话，那就是在ThreadPool上执行///
> ```
>
>
>
>

### 3、Async和Await

Thread中的Join	 和 	Task中的Wait基本一样

Q：I/O密集型和CPU计算区分

A：您的代码会“等待”某些东西，例如来自数据库的数据吗？是即I/O密集型

您的代码会执行昂贵的计算吗？“是”，那么您的工作**受 CPU 限制**。

> * 成对出现，否则就跟同步一样
> * 程序执行到await处主线程休息，await定义的方法交给其他模块执行，await下面代码也不执行，但主线程==并不卡死==，待await模块执行完毕，继续执行后续代码
> * 同步和异步并不会带来速度的区别，但能带来UI线程不阻塞
> *  异步有三种形式
>   * public async void Test01(){}  //void
>   * public async Task Test02(){}  //不带返回值的
>   * public async Task<T> Test03(){}  //带返回值的	

> 使用场景：
>
>   
>
> 如果你的工作是I/ o限制的，使用async和await而不是Task.Run。 您不应该使用Task库。 
>
>  
>
> 如果你的工作是cpu限制的，你关心的是响应性，使用async和await，但通过Task.Run在另一个线程上生成工作。 如果工作适合并发和并行，也可以考虑使用Task库。

### 4、信号（Signaling）

> 某些情况，需要让线程一直处于等待状态，直到接收到其他线程发来的通知，这叫做Signaling（发送信号）

> 最简单的信号结构是 <u>ManualResetEvent</u> （暂停、继续）

> 三个方法：
>
> * WaitOne方法会阻塞当前线程，直到另一个线程通过调用Set方法开启信号
> * 调用Reset方法会将其再次关闭

```C#
	static void Main(string[] args)
        {
            var signal = new ManualResetEvent(false); 
            Task.Run(() =>
            {
                for(int i=0;i<100;i++)
                {
                    if(i==20)
                    {
                        signal.WaitOne();
                    }
                    Console.WriteLine(i);
                }
            }); //暂停在20不动了
            Thread.Sleep(3000);
            Console.WriteLine("三秒后继续");
            signal.Set();  //继续，走完输出100个数字。
            Console.ReadLine();
        }
```



### 5、线程池（灵活也有限制）

> 线程池：由于线程的创建需要时间，可以事先创建许多线程。在需要更多线程时增加线程，在需要释放资源时减少线程。
>
> 限制：线程池中所有线程都是后台线程，不能设置优先级、入池的线程只能用于时间较短的任务。









### 6、Parallel关键词（顺序无关）

> :one: Parallel.For(int start,int end,Action<int> body)   //与for循环类似，执行并行的循环，相当于每次循环一个线程
>
> :two: Parallel.Invoke(ParallelOptions, Action[])
>
> 对给定任务实现并行执行
>
> 此方法可用于执行一组可能并行的操作。
>
> 无法保证操作的执行顺序或操作是否并行执行。 此方法不会返回，直到每个提供的操作都已完成，无论是否由于正常终止或异常终止而完成。
>
> :three:Parallel.ForEachAsync
>
> 异步并发，可以设置并行度



---



##  七、接口

### 1、用处——解耦代码，后期更新

比如一个工厂里，有许多产品，共有属性有颜色，重量。共有方法有称重和售卖。

此众多产品都有这些特性，所以可以将这些共性抽象为一个接口

如果以后有新产品，都可以继承这个接口，而不用修改之前的代码

调用是用接口声明即可，也是实现重载。

List<IProduct> list = 创建各种产品，只要继承该接口都可以





## 八、Linq

### 1、概念

> 根据数据源类型，分为三种
>
> * Linq to Object
> * Linq to ADO.NET
> * Linq to XML

### 2、Linq关键字

> from:可以进行复合查询
>
> select:查询正文必须以 select 或 group 子句结尾
>
> where:条件筛选
>
> orderby:  默认升序，降序关键字(descending)，orderby后第一个排序元素为主要排序，第二个为次要排序
>
> group:对查询结果的分组
>
> into:将分组的结果进行排序、再次查询等操作
>
> join:联结操作，join子句可以来自不同源序列（内连接、分组连接、外连接）

### 3、示例代码

``` C#
int []arr = {1,3,5,6,8,10};
var query = from val in arr select val; //查到所有arr的值
 var sublist3 = from student in list orderby student.Id（descending) select student;  
//默认小到大排序 descending修正为降序排列
```

### 4、方法 及 实现（Lambda）

>IEnumerable<T>接口，提供了大量与查询相关的方法（Linq）
>
>有：
>
>==数值运算==：Aggregate（累加器）、Sum（求和）、Average（求平均值）、Max、Min、
>
>==取值==：All（检查是否序列中所有元素都满足条件）、Any（检查序列中是否有任何一个元素满足条件，真返回true）、Contains（检查数据序列中是否包含特点元素）、Count（返回满足条件元素的数量）、Cast（将IEnumerable中元素转换为指定的数据类型）、FirstOrDefault（返回序列中满足条件的第一个元素，若不存在返回默认值）、
>
>LastOrDefault（返回序列中满足条件的最后一个元素，若不存在返回默认值）、
>
>SingleOrDefault（返回序列中满足条件的唯一元素，若不止一个会引发异常）
>
>==集合操作==：
>
>Reverse（反转元素顺序）
>
>Distinct（返回不重复元素集合）
>
>Concat（连接两个序列，首尾相连）
>
>Except（差集）、Intersect（交集）、Union（并集）、SequenceEqual（比较两序列是否相等）
>
>==提取子集==
>
>Skip（跳过指定数量的元素）、Take（从序列开头返回指定数量的连续元素）
>
>ToArray、ToList

Linq常与Lambda表达式一起使用

### 5、实战用法

> var numbers = new List<int> { 1, 2, 3, 4, 5 };
>
> where：用于过滤数据
>
> ```C#
> var evenNumbers = numbers.Where(x => x % 2 == 0);
> ```
>
> 
>
> select：用于变换数据
>
> ```C#
> var squaredNumbers = numbers.Select(x => x * x);
> ```
>
> 

## 九、反射

> using System.Reflection;
>
> 反射中最重要的类：Type、Assembly
>
> 优势：反射可以获取到类库得所有私有方法
>
> 缺点：反射执行稍慢，安全性不足，易出错，处理反射很复杂
>
> 层次（由大到小）：Assembly---》Module---》
>
> 实质：就是知道一个你能获得所有信息（==树状结构==）
>
> ![image-20240420143021345](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20240420143021345.png)
>
> ```C#
> namespace ConsoleApp1
> {
> internal class Program
> {
>   static void Main(string[] args)
>   {
>       //或者Type t= typeof(Student);
>       Type t = Type.GetType("ConsoleApp1.Student");
>       PropertyInfo[] properties = t.GetProperties();
>       MethodInfo[] methods = t.GetMethods();
> 
>       //通过反射获取类的属性和方法
>       foreach(PropertyInfo property in properties)
>       {
>          Console.WriteLine(property.PropertyType.Name+","  +property.Name);
>       }
>       Console.WriteLine("=====================");
>       foreach(MethodInfo method in methods)
>       {
>           Console.WriteLine(method.Name);
>       }
>       Console.ReadLine();
>   }
> }
> 
> public class Student
> {
>   public int Id { get; set; }
>   public string Name { get; set; }    
>   public void GetName1()
>   {
> 
>   }
>   public void GetId1()
>   {
> 
>   }
> }
> }
> 
> ```
>
> Q：如何用反射创建对象
>
> ```C#
> Assembly assembly = Assembly.LoadFrom("11212");
> Type type = assembly.GetType("11212.object");
> object o = Activitor.CreateInstance(type);//创建对象
> 
> foreach()
> ```



> Q：利用反射获取所有构造函数
>
> ```C#
>         static void Main(string[] args)
>         {
>             Assembly assembly = Assembly.LoadFrom("123.dll");
>             Type type = assembly.GetType("123class");
>             foreach(ConstructorInfo constructor in type.GetConstructors()) //获取类中所有构造方法
>             {
>                 Console.WriteLine(constructor.Name);
>                 foreach(var parameter in constructor.GetParameters()) //获取到构造方法里所有参数类型
>                 {
>                     Console.WriteLine(parameter.ParameterType); //显示类型方法
>                 }
>                 object o = Activator.CreateInstance(type, new object[] { 123 }); //创建构造函数的对象
>             }
> 
> 
> 
>         }
> ```



> Q：通过反射获取方法
>
> ```C#
> 
>             #region 通过反射调用方法
>             Assembly assembly2 = Assembly.LoadFrom("12112111");
> 
>             Type type2 = assembly2.GetType("ClassName");//获取类型名、
>             object o2 = Activator.CreateInstance(type2);
>             foreach(var method in type2.GetMethods())
>             {
>                 Console.WriteLine(method.Name);
>                 foreach(var parm in method.GetParameters())
>                 {
>                     Console.WriteLine(parm.Name+","+parm.ParameterType);
>                 }
>             }
> 
>            MethodInfo methodInfo =  type2.GetMethod("方法名1");//就能获取到该方法，存储到MethodInfo里面
>             methodInfo.Invoke(o2,null); // 第一个参数就是对象，第二个就是传递的参数，如果方法是无参数的，就是null
> 
>             MethodInfo methodInfo2 = type2.GetMethod("方法名2",new Type[] {typeof(int),typeof(string)});//获取指定类型的方法
>             methodInfo.Invoke(o2, new object[] {123,"字符串"}); 
> 
>             #endregion
> ```
>





> 反射实战：sqlHelper改进
>
> ```C#
> public T Find<T>(int id)
> {
> 	Type type = typeof(T);
>     string sql = $"SELECT fstring.Join("", 	type.GetProperties0.Select(p => p.Name))) FROM (dbo][Students] where 			 Studentld={id};
> 
> using(SqlConnection conn = new SalConnection(connStsring)
>    }
> ```
>



### BindingFlags

> `BindingFlags` 是一个枚举类型，用于指定在反射中搜索成员时要使用的绑定标志。这些标志允许你控制反射操作的行为，如获取类型的成员、获取方法、属性、字段等。
>
> 下面是 `BindingFlags` 枚举的一些常用成员：
>
> 1. **Public**：指示反射应搜索公共成员。
> 2. **NonPublic**：指示反射应搜索非公共成员。
> 3. **Instance**：指示反射应搜索实例成员。
> 4. **Static**：指示反射应搜索静态成员。
> 5. **DeclaredOnly**：指示反射应仅搜索当前类型声明的成员，而不搜索基类中的成员。
> 6. **FlattenHierarchy**：指示反射应搜索当前类型及其基类中的所有公共成员。
> 7. **IgnoreCase**：指示反射应忽略成员名称的大小写。
>
> ```C#
> using System;
> using System.Reflection;
> 
> class MyClass {
>     public void Method1() {}
>     public void Method2() {}
> }
> 
> class Program {
>     static void Main() {
>         Type type = typeof(MyClass);
>         MethodInfo[] methods = type.GetMethods(BindingFlags.Public | BindingFlags.Instance);
>         foreach (MethodInfo method in methods) {
>             Console.WriteLine(method.Name);
>         }
>     }
> }
> //搜索一个类型的所有公共实例方法
> //使用 BindingFlags.Public | BindingFlags.Instance 组合了两个标志，以便获取 MyClass 类型的所有公共实例方法。
> ```
>
> 

## 十、常见特性

> 1、你的托管代码是否允许非托管代码调用。这个涉及到.NET的平台互操作问题。比如，C#可以调用C\C++写的DLL,也可以操纵MS Office的组件这就称之为互操作(Invoke)。那么如果你也想让C\C++也来调用你的C#写的程序，此时就需要开放你的类为对COM可见性
>
> //默认即为true,默认情况下，它们对 COM 是可见的。只能使 public 类型可见。而不能使用该属性使原本为 internal 或 protected 的类型对 COM 可见，也不能使不可见类型的成员可见。

```C#
[ComVisible(true)]
public sealed class MyClass{ // 一般来说对COM可见的类都是密封的。
 
     [ComVisible(true)]
     public .... Foo(...){
       // to do sth...
     }
}
```

## 十一、重写方法

> 常见重写方法：
>
> equals 和 getHashcode
>
> 这两个要同时重写

### 1、重写Dispose方法

> 99% C# 会帮我处理好托管资源
>
> 而非托管资源：比如Windows窗口句柄、数据库链接、GDI对象、独占文件锁对象等如果想释放，可以用using语法糖
>
> * Dispose()需要继承IDisposable接口
> * Close和Dispose的区别，Close方法关闭对象，没有完全释放，Dispose()完全释放
> * 99%的情况不用重写Dispose，当有些情况某内存不断增长时需要特定情况特定解决
>
> 代码示例
>
> ```C#
>     public class DisposableTest : IDisposable
>     {
> 
> 
>         private readonly IntPtr unmanagedResource;//非托管内存
>         private readonly SafeHandle managedResource;//托管内存
>         public void Dispose()
>         {
>             Dispose(true); //调用处理方法
>             GC.SuppressFinalize(this); // 让GC别管了，我自己处理
>         }
> 
>         protected virtual void Dispose(bool isManualDisposing)
>         {
>             ReleaseUnmanagedResource(unmanagedResource);
>             if(isManualDisposing)
>             {
>                 ReleaseManagedResource(managedResource);
>             }
>         }
> 
> 
>         public DisposableTest()
>         {
>             unmanagedResource =Marshal.AllocHGlobal(sizeof(int));
>             managedResource = new SafeFileHandle(new IntPtr(), true);
>         }
> 
>         public void ReleaseUnmanagedResource(IntPtr intptr)
>         {
>             Marshal.FreeHGlobal(intptr);
>         }
> 
>         public void ReleaseManagedResource(SafeHandle safeHandle)
>         {
>             if(safeHandle != null)
>             {
>                 safeHandle.Dispose();
>             }
>         }
> 
>         ~DisposableTest()
>         {
>             Dispose(false);
>         }
>     }
> ```
>
> 



## 十二、CLI命令方式

> 与git配合使用

> 1、dotnet new gitignore
>
> 创建一个忽略git更新的文件 ——Ignore Visual Studio temporary files*

> 2、dotnet new sln
>
> 创建一个解决方案文件，默认是当前目录的名字

> 3、dotnet new console -o DemoConsole
>
> 创建一个控制台项目，重命名为demoConsole 

> 4、 dotnet sln add DemoConsoleApp/
>
> 已将项目“DemoConsoleApp\DemoConsoleApp.csproj”添加到解决方案中。

> 5、 dotnet new classlib -o DemoClassLib
>
> 创建一个类库
>
>  dotnet sln add DemoClassLib/
>
> 将类库加到解决方案里

> 6、dotnet add package Dapper
>
> 必须要在类库的文件夹下添加该nuget包，就是必须加到项目下，而不是最大的解决方案下

> 7、ConsoleApp引用DemoClassLib这个类库
>
>  dotnet add reference ../DemoClassLib/DemoClassLib.csproj

> 8、`dotnet restore` 命令使用 NuGet 还原依赖项以及在 project 文件中指定的特定于项目的工具。 
>
> dotnet build 创建bin目录
>
> ------------------
>
> dotnet clean ——清理后
>
> dotnet rebuild——重新创建





## 十三、构造函数

> 静态构造优先级比普通构造高
>
> 1.静态构造函数先于构造函数执行。
>
> 2.静态构造函数只执行一次。





## 十四、抽象方法（用的少）

> 抽象方法一定要写在抽象类中，规范好方法让子类去实现
>
> 抽象方法不能new，只能子类来继承
>
> 抽象方法不常用，常用的是虚方法（virtual）

> 与接口的区别：
>
> * 抽象类——单继承，接口——多继承
> * 抽象类里可以写普通方法，虚方法等，接口只能写规范，不写实现
> *  使用场合：抽象类一般用于不太改动，范围大一点的事件 

抽象类:子类只能接受并在接受后必须实现一个父类安排的任务，若为抽象类任务，必须由子类实现；接口:子类可以接受多个并在接受后必须实现多个父类安排的任务，此任务必须由子类实现。





## 十五、泛型

C# 自带的泛型有 list 和 dictionary

泛型最大优点就是做到了通用

### 1、泛型类

> ```C#
>     public class MyGeneric<T>
>     {
>         private T Value;
>         private int X;
> 
>         public MyGeneric(int x)
>         {
>             X = x;
>         }
> 
>         public MyGeneric()
>         {
> 
>         }
>         public MyGeneric(T value, int x)
>         {
>             Value = value;
>             X = x;
>         }
>     }
> ```
>
> 



### 2、泛型方法

会自动判断数据类型

```C#
        public void Show<T1,T2>(T1 t1,T2 t2)
        {
            Console.WriteLine(t1.ToString());
            Console.WriteLine(t2.ToString());

        }
    static void Main(string[] args)
        {

            Program p = new Program();
            p.Show(123, DateTime.Now);
            p.Show<int, string>(111, "3334");
            Console.Read();

            
        }
```





### 3、泛型接口（略）









### 4、泛型约束：

> 对传入参数类型进行约束，where 关键字 
>
> 大致有四种约束：
>
> * new()约束
> * struct值类型约束
> * class引用类型约束
> * 自定义类型约束
>
> Tips:多个约束情况下，无参构造函数约束一定要写在最后面

```C#

        public static void Show1<T>(T t) where T:new ()  //表示T类型只接受带一个无参数的构造函数
        {
            
        }
```

![image-20221228102722668](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20221228102722668.png)







```c#
    public static void Show1<T>(T t) where T:Person   //表示T类型只接受Person类型
        {
            
        }


        public static void Show2<T>(T t) where T : struct   //表示T类型只接受值类型
        {

        }
```



```C#
      public static void Show3<T,S,K>(T t,S s ,K k)where S:struct where K:Person where T : class   //表示S只接受值类型，K只接受Person类型，T类型只接受引用类型
        {

        }
```



### 5、协变和逆变

> People p = new Teacher();//里氏转换
>
> 但List<People> peoples = new List<Teacher> ();//这样不行 引入协变和逆变
>
> 协变和逆变是针对泛型接口和泛型委托来说的。
>
> out关键字代表协变，in代表逆变
>
> 协变只能返回结果，不能作参数
>
> defalut关键字让函数不会抛出异常
>
> ![image-20221228111654274](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20221228111654274.png)







## 十六、Json操作

> 用newtonSoft库
>
> 最简单的方法：就是json数据和你的类相对应,直接拿到整个数据
>
> ```C#
>        private List<SteelInfo> Do()
>         {
>             string fileName = @"C:\Users\admin\Desktop\规程表工具-2023.01.10\pdisfc.json";
> 
>             if(File.Exists(fileName))
>             {
>                 var customers = JsonConvert.DeserializeObject<List<SteelInfo>>(File.ReadAllText(fileName));
>  
>                 List<SteelInfo> list = new List<SteelInfo>();
>                 
>                 return customers;
>             }
>             else
>                 return null;
>         
>         }
> ```
>
> 

> dynamic动态类型操作json
>
> dynamic其实就是你初始化任意指定的类型，写入json文件

==在传递对象，但是又不想创建一个``class``或者``struct``的时候，ExpandoObject就是一个非常好的选择。==

> 
>
> ```C#
> //创建动态对象       
> List<dynamic> BuildDynamicObject()
>         {
>             var addresses = new List<dynamic> { new { Name = "Jack", ID = "1", Sex = "男" }, new { Name = "Rose", ID = "2", Sex = "女" }, new { Name = "Habiton", ID = "100", Sex = "中" } };
>             return addresses;
>         }
> 
> //将动态对象序列化为json字符串写入json文件并返回
>         static string ConvertDynamicObjectsToJson(List<dynamic> data)
>         {
>             string json = JsonConvert.SerializeObject(data);
>             string fileName = @"C:\Users\admin\Desktop\Test.json";
>             File.Delete(fileName);
>             File.WriteAllText(json, fileName);
>             return json;
>         }
> 
> 
>         //ExpandoObjectConverter——对象转换器
>         static List<ExpandoObject> ReadJsonFileConvertToObject()
>         {
>             string fileName = @"C:\Users\admin\Desktop\Test.json";
>             if (File.Exists(fileName))
>             {
>                 string temp = File.ReadAllText(fileName);
>                 var data = JsonConvert.DeserializeObject<List<ExpandoObject>>(temp);
>                 return data;
>             }
>             return null;
>         }
> ```
>
> 
>
> **“vs可以编辑里面找到 将json/xml转换为类





## 十七、运算符和新特性

```
空合并运算符(??)：
用于定义可空类型和引用类型的默认值。如果此运算符的左操作数不为null，则此运算符将返回左操作数，否则返回右操作数。
例如：a??b 当a为null时则返回b，a不为null时则返回a本身。
空合并运算符为右结合运算符，即操作时从右向左进行组合的。如，“a??b??c”的形式按“a??(b??c)”计算。
```

> public string Title{get;set;}=null!;//表示title字段不可以为空
>
> public string？Content{get;set;}//表示content字段可以为空







## 十八、依赖注入、容器概念及其实现

> 大白话就是一个容器里放多个类，然后可以拿到这个类，调用这个类的方法

### 1、代码实现

> ```C#
> namespace ConsoleApp1
> {
>     /// <summary>
>     /// 依赖注入其实就是把各个类放在一个容器中，实质就是反射（Activator.CreatInstance)
>     /// </summary>
>     internal class Program
>     {
>         static void Main(string[] args)
>         {
>             //HelloService helloService = new HelloService();
>             //ConsumerHelloService consumerHelloService = new ConsumerHelloService(helloService);
> 
>             //HelloService newHelloService = (HelloService)
>             //    Activator.CreateInstance(typeof(HelloService));
>             //ConsumerHelloService newconsumerHelloService = (ConsumerHelloService)
>             //    Activator.CreateInstance(typeof(ConsumerHelloService), newHelloService);
>             //newHelloService.Output();
>             //newconsumerHelloService.Output();
> 
>             //容器 与 依赖注入
>             var container = new DependencyContainer();
>             container.AddDependency(typeof(HelloService));
>             container.AddDependency(typeof(ConsumerHelloService));
> 
>             //从容器中拿到类型并创建实例
>             var resolver = new DependencyResolver(container);
>             //Success
>             resolver.GetService<HelloService>().Output();
> 
>             //Failed,因为是有参构造
>             //resolver.GetService<ConsumerHelloService>().Output();
>             resolver.GetService<ConsumerHelloService>().Output();
>             Console.Read();
>         }
>     }
> 
>     public class HelloService
>     {
>         public void Output()
>         {
>             Console.WriteLine("这是HelloService测试");
>         }
> 
>         public HelloService() { }
>     }
> 
>     public class ConsumerHelloService
>     {
>         HelloService _helloService;
> 
>         public ConsumerHelloService(HelloService helloservice)
>         {
>             this._helloService = helloservice;
>         }
> 
>         public void Output()
>         {
>             _helloService.Output();
>         }
>     }
> 
>     /// <summary>
>     /// 依赖注入所用到的容器
>     /// </summary>
>     public class DependencyContainer
>     {
>         List<Type> _dependencies = new List<Type>(); // 放类型的容器
> 
>         public void AddDependency(Type type)
>         {
>             _dependencies.Add(type);
>         }
> 
>         public void AddDependency<T>()
>         {
>             _dependencies.Add(typeof(T));
>         }
> 
>         public Type GetDependency(Type type)
>         {
>             return _dependencies.FirstOrDefault(x => x.Name == type.Name);
>         }
>     }
> 
>     /// <summary>
>     /// 依赖注入解析器
>     /// </summary>
>     public class DependencyResolver
>     {
>         DependencyContainer _container;
> 
>         public DependencyResolver(DependencyContainer container)
>         {
>             this._container = container;
>         }
> 
>         public T GetService<T>()
>         {
>             var dependency = _container.GetDependency(typeof(T)); //返回类型
> 
>             var constructor= dependency.GetConstructors().Single(); //找到所有构造函数中第一个构造函数
>             var parameters = constructor.GetParameters().ToArray(); //生成默认构造函数所需的参数
> 
>             if(parameters.Length > 0)
>             {
>                 var parameterImplementations = new object[parameters.Length];
>                 for (int i = 0; i < parameters.Length; i++)
>                 {
>                     parameterImplementations[i] = Activator.CreateInstance(parameters[i].ParameterType); //为每个类型创造实例放到object数组中
>                 }
>                 return (T)Activator.CreateInstance(dependency,parameterImplementations); //创建有参实例对象
>             }
>            
>             return (T)Activator.CreateInstance(dependency); //创建类型的无参实例对象
>         }
> 
>         public T GetService<T>( Type paramType)
>         {
>             var type = _container.GetDependency(typeof(T));
>             return (T)Activator.CreateInstance(type,paramType);
>         }
>     }
> }
> ```
>
> 

> ### 2、三种生命周期
>
> > AddTransient(瞬时) 作用: 每次请求，都获取一个新的实例。即使同一个请求获取多次也会是不同的实例(只要请求new 新的)
> >
> > ​       *AddScoped（作用域）作用： 每次请求，都获取一个新的实例。同一个请求获取多次会得到相同的实例
> > ​      AddSingleton（单例）作用：每次都获取同一个实例 （比如：帮助类的内容）*







## 十九、GAC 强名

> 安装.net 框架时，它会安装系统自带的程序集，在C:\windows\assembly——称之为GAC（共享程序集），我们一般类库生成的是私有程序集，每次引用需单独添加。
>
> 而GAC是都能用，强名由：1、主版本号  2、次版本号 3、公钥 组成。
>
> 
>
> 注意：
>
> 点开引用的库，可以看到哪些是系统自带的类库
>
> 系统自带的强名为true 复制本地为false
>
> 
>
> 用xcopy将一台电脑的程序复制到另一台上面， 会发生的问题有：你只是复制了文件夹，而没有复制共享程序集（GAC），所以有时进行xcopy部署的时候，应用程序将无法正确执行。
>
> 所以！解决方法是将依赖项都放在exe所在的目录下。
>
> ![image-20230303100736456](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20230303100736456.png)



> 自己的类库要变成共享程序集，必须要用公钥（用一个工具gacutil来生成公钥），生成一个snk，在你的类库文件中添加assemblykeyfile，成功后会加到cache里，可以在C:\windows\microsoft.net\assembly\GAC_MSIL中看到该类库。 不同的版本将安装到不同的地方。





## 二十、多线程同步技术

> * lock语句
> * interlocked类        
> * monitor类
> * spinlock结构
> * waithandle类
> * mutex类
> * semaphore类
> * event类
> * ReaderWriterLockSlim类
>
> 
>
>  ---前三为进程内部的同步，后四为多个进程间的线程同步



同步技术详解：

### 1、lock

> lock锁住的对象必须是引用类型，==一次只有一个线程能访问相同实例的方法==。一般用法如下：
>
> ```C#
> static object _lock = new object();
> public static void AddOneMillion()
> {
>     for(int i=0;i<100000;i++)
>     {
>         lock(_lock)
>         {
>             Total++;
>         }
>     }
> }
> ```
>
> 注意：操作哪个对象就锁住哪个对象！且对象不要用get，set，因为存取期间，对象没有锁定。
>
> ```C#
>    internal class Program
>     {
>         static void Main(string[] args)
>         {
>          
>             int numTask = 20;
>             var state = new SharedState();
>             var tasks = new Task[numTask];
> 
>             for(int i=0;i<numTask;i++)
>             {
>                 tasks[i] = new Task(new Job(state).DoTheJob);
>                 tasks[i].Start();
>             }
>             for(int i=0;i<numTask;i++)
>             {
>                 tasks[i].Wait();
>             }
>             Console.WriteLine("State总数为:{0}",state.State);
>             Console.Read();
>         }
>     }
> 
>     public class SharedState
>     {
>         public int State { get; set; }
>     }
> 
>     public class Job
>     {
>         SharedState sharedState { get; set; }
>         public Job(SharedState sharedState)
>         {
>             this.sharedState = sharedState;
>         }
>         private object o = new object();
>         public void DoTheJob()
>         {       
>                 for (int i = 0; i < 50000; i++)
>                 {
>                 lock (sharedState)
>                 {
>                     sharedState.State+=1;
>                 }
>                 }
>   
>         }
>     }
> ```
>
> ***：i++不是线程安全的，它的操作包括从内存重取一个值，再加1，再存回内存。如果想做多线程的递增操作，使用interlocked类会快很多，但仅限于简单的同步问题。Interlocked.Increment.





### 2、Monitor类

> C#的lock类其实是monitor的封装。
>
> ```C#
> Monitor.Enter(obj);
>     try
>     {
>         //执行异步方法
>     }
> finally
> {
>     Monitor.Exit(obj);
> }
> ```
>
> 与lock类相比，monitor类主要优点在于：可以添加一个等待被锁定的超时值，这样就可以不用无限期地等待被锁定，而可以使用TryEnter方法（传入超时值）。





### 3、WaitHandle类

> 抽象基类，用于等待一个信号的设置，可以等待不同的信号。



### 4、Mutex类

> 派生自WaitHandle类，互斥类，非常类似于Monitor类，因为它们都只有一个线程能拥有锁定。只有一个线程能获得互斥锁定，访问受互斥保护的同步代码区域。



### 5、Semaphore类

> 信号量非常类似于互斥，其区别是，信号量可以同时由多个线程使用。信号量是一种计数的互斥锁定。
>
> ```C#
>    internal class Program
>     {
>         static void Main(string[] args)
>         {
>             int threadCount = 6;
>             int semaphoreCount = 4;
>             var semaphore = new SemaphoreSlim(semaphoreCount, threadCount);
>             var threads = new Thread[threadCount];
>             for (int i = 0; i < threadCount; i++)
>             {
>                 threads[i] = new Thread(ThreadMain);
>                 threads[i].Start(semaphore);
>             }
>             for(int i=0;i<threadCount; i++)
>             {
>                 threads[i].Join();
>             }
>             Console.WriteLine("所有线程执行完毕");
>             Console.Read();
>         }
> 
>         static void ThreadMain(object o)
>         {
>             SemaphoreSlim semaphore = o as SemaphoreSlim;
>             Trace.Assert(semaphore != null, "o 一定是semaphore类");
>             bool isCompleted = false;
>             while(!isCompleted)
>             {
>                 if(semaphore.Wait(600))
>                 {
>                     try
>                     {
>                         Console.WriteLine("线程{0}锁住了信号量",Thread.CurrentThread.ManagedThreadId);
>                         Thread.Sleep(2000);
>                     }
>                     finally
>                     {
>                         semaphore.Release();
>                         Console.WriteLine("线程{0}释放了信号量", Thread.CurrentThread.ManagedThreadId);
>                         isCompleted = true;
>                     }
>                 }
>                 else
>                 {
>                     Console.WriteLine("线程{0}超时，请等待",Thread.CurrentThread.ManagedThreadId);
>                 }
>             }
>         }
>     }
> ```
>
> ![image-20230306134423103](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20230306134423103.png)



### 6、Event类

> AutoResetEvent、ManualResetEventSlim



### 7、BackgroundWorker类

> BackgroundWorker是·net里用来执行多线程任务的控件，它允许编程者在一个单独的[线程](https://baike.baidu.com/item/线程/103101?fromModule=lemma_inlink)上执行一些操作。耗时的操作（如下载和[数据库事务](https://baike.baidu.com/item/数据库事务/9744607?fromModule=lemma_inlink)）在长时间运行时可能会导致用户界面 (UI) 始终处于停止响应状态。如果您需要能进行响应的用户界面，而且面临与这类操作相关的长时间延迟，则可以使用BackgroundWorker类方便地解决问题。
>
> 就是说 加载某个进度条可以用这个类！
>
> 三个事件——DoWork、ProgressChanged、RunWorkerCompleted
>
> ReportProgress（）方法触发BackgroundWorker控件的ProgressChanged事件，这个事件会将控制权交给UI线程。







## 二十一、安全性

> System.Security
>
> WindowsIndentity类用来验证你windows用户的身份，从而对权限做出区分。
>
> ——Principal类
>
> 在类或方法级别上，可以使用    [PrincipalPermission]特性声明性地说明权限的需求。



### 1、客户端应用程序服务

> 身份验证功能：基于System.Web.Security 的 Membership和Roles类。使用Membership类可以验证、创建、删除和查找用户，改变密码以及与用户相关的许多其他操作。使用Roles类可以添加和删除角色，给用户获取角色，以及改变用户的角色。
>
> ==私钥加密，公钥解密==



### 2、桌面应用程序权限

> FileIOPemission类——对文件io的权限控制
>
> 



## 二十二、本地化

> 本地化就是指你一个软件要在不同地区销售的情况下，你的语言、数字规则都要和当地相符。
>
> 用的是Culture类。System.Globalization 类

### 1、Resource文件和Settings文件的区别

> 这两个仅使用于Winform和WPF
>
> Resources是只读的，是不变的。可以添加字符串、图片、音视频
>
> Settings：看范围，当范围是用户时可以读写，当范围是应用程序是只读。读写修改的内容下次启动不会丢失，相当于一个小型数据仓库。





## 二十三、网络

> WebClient类使用简单，但功能也十分有限，不能使用它提供身份验证证书。
>
> 建议使用WebRequest类
>
> 代码部分
>
> ```C##3
>   //WebClient类
>   public  void DownloadFileFromWeb()
>         {
>             WebClient client = new WebClient();
>             Stream sm = client.OpenRead("http://www.reuters.com/");
>             StreamReader sr = new StreamReader(sm);
>             string line;
>             while ((line = sr.ReadLine()) != null)
>             {
>                 richTextBox1.Text += line;
>             }
>         }
>         
>           //WebRequest 和 WebResponse 类
>     public void DownloadFileFromWeb2()
>         {
>             WebRequest client = WebRequest.Create(("http://www.reuters.com/");
>            WebResponse response = client.GetResponse();
>             StreamReader sr = new StreamReader(sm);
>             string line;
>             while ((line = sr.ReadLine()) != null)
>             {
>                 richTextBox1.Text += line;
>             }
>         }
> ```
>
> 
>
> 
>
> SmtpClient类可以发送邮件。
>
> Socket类提供了网络编程中最高级的控制。



## 二十四、文件和注册表

> 注意：文件夹的概念是Windows独有的，别的系统都称之为目录

> 表示文件和文件夹的.net 类
>
> Directory类和File类只包含静态方法，不能被实例化。读取和写入文件用的是File的静态方法。
>
> DirectoryInfo类和FileInfo类是有状态的，不是静态的，需要实例化后与某个特定的文件夹或文件关联起来。

> ​	Path类的Combine方法可以联合路径名
>
> MessageBox.Show(Path.Combine(@"C:\My Documents", "Readme.txt"));
>
> 返回
> 如果指定的路径之一是零长度字符串，则该方法返回其他路径。如果 path2 包含绝对路径，则该方法返回 path2

读取文件流代码实例(StreamReader)

```C#
FileStream fs = new FileStream(@"c:\My Doc\test.txt",FileMode.Open,FileAccess.Read,FileShare.None);
StreamReader sr = new StreamReader(fs);

```

写入文件流代码类似(StreamWriter)





## 二十五、ADO.NET

>其实就是对象的概念，数据库操作有关。
>
>ADO和ADO.NET不一样，ADO是一个COM组件库。总之ADO.NET更高级。
>
>ADO.NET附带了3个数据库客户端名称空间
>
>:one:SQL Server
>
>:two:ODBC
>
>:three:OLE DB
>
>ADO.NET类最重要的功能是：以断开连接的方式工作。
>
>





## 二十六、重新学习多线程

> Thread——最基本的类
>
> Join()方法——等当前线程全部工作完，再执行下一条语句
>
> abort()方法——终止当前线程，此方法很危险，有可能会导致程序异常终止，建议使用CancellationToken方法来取消线程的执行。

> 默认情况下显示创建的是前台线程
>
> - ### 前台线程：应用程序必须运行完所有的前台线程才能退出，默认创建的线程都是前台线程。
>
> - ### 后台线程：应用程序可以不必考虑后台线程是否已经运行完毕（包括正常退出和异常退出），只要所有的前台线程结束，后台线程自动结束。



### 1、匿名函数中的闭包问题

```C#
  			 int i = 10;
            var thread4 = new Thread(() =>
            {
                PrintNumber(i);
            });
            i = 20;
            var thread5 = new Thread(() =>
            {
                PrintNumber(i);
            });
            thread4.Start();
            thread5.Start();
线程4和线程5都会打印20.
    因为如果在多个lambda表达式中使用相同的变量，它们会共享该变量值。
```

### 2、lock关键字

> 确保当一个线程使用某些资源时，同时其他线程无法使用该资源，有一个线程安全的概念。





### 3、线程同步

> 线程同步的概念是基于共享对象的，如果无须共享对象，那么就无须进行进程同步。

```C#
  class CounterNoLock : CounterBase
    {
        private int count;

        public int Count
        {
            get { return count; }
            set { count = value; }
        }

        internal override void Decrement()
        {
            Interlocked.Decrement(ref count);   
        }

        internal override void Increment()
        {
            Interlocked.Increment(ref count);
        }
    }
```

>Interlocked类本身就是原子操作，我们无需锁定任何对象即可获取到正确的结果。Interlocked类提供了Increment、Decrement和Add等基本数学操作的原子方法，从而我们无需使用锁。



### 4、SemaphoreSlim类和Semaphore类

> semaphoreslim是轻量化的限制同一个资源被访问的类。
>
> 下例：设置数据库访问最大数量为4，大于4就阻塞等待前面的释放再连接

```C#
    internal class Program
    {
        static void Main(string[] args)
        {
            for(int i=0;i<8;i++)
            {
                string threadName = "Thread" + i;
                int secondsToWait = 2 + 2 * i;
                Thread th = new Thread(() => AccessDatabase(threadName, secondsToWait));
                th.Start();
            }
            Console.Read();
        }

        static SemaphoreSlim _semaphore = new SemaphoreSlim(4);//指定访问同一个线程的最大数量为4

        static void AccessDatabase(string name,int seconds)
        {
            Console.WriteLine("{0} 等待着去接触数据库",name);
            _semaphore.Wait();//阻止当前线程，直到可进入为止
            Console.WriteLine("{0} 成功进入数据库", name);
            Thread.Sleep(seconds);
            Console.WriteLine(("{0} 已经完成", name));
            _semaphore.Release();//释放对象一次
        }
    }
```





### 5、AutoResetEvent类

> AutoResetEvent类可以从一个线程向另一个线程发送通知。AutoResetEvent类可以通知等待的线程有某事件发生。

> 线程阻塞成立的一对条件:
> 1.线程中包含waitone();
> 2.AutoResetEvent(bool)的bool为false,即事件状态为非终止状态；
>
> 
>
> WaitOne方法，是用来等待的，如果AutoResetEvent处于非终止态，则一直等待，直到调用set
>
> Set方法，告知我的工作完成了，现在可以走了。set()方法是将事件状态设置为终止状态

````C#
    internal class Program
    {
        static void Main(string[] args)
        {
            Thread th = new Thread(() =>
            {
                Process(10);
            });
            th.Start();
            Console.WriteLine("等待完成工作");
            _workerEvent.WaitOne();
            Console.WriteLine("第一项操作已经完成");
            Console.WriteLine("正在主线程上执行操作");
            Thread.Sleep(5000);
            _mainEvent.Set();
            Console.WriteLine("现在跑在第二线程上");
            _workerEvent.WaitOne();
            Console.WriteLine("第二项操作已经完成");
            Console.Read();
        }

        private static AutoResetEvent _workerEvent = new AutoResetEvent(false);
        private static AutoResetEvent _mainEvent = new AutoResetEvent(false);

        static void Process(int seconds)
        {
            Console.WriteLine("开启一个长作业");
            Thread.Sleep(seconds);
            Console.WriteLine("工作完成了");
            _workerEvent.Set();
            Console.WriteLine("等待主线程完成它的工作");
            _mainEvent.WaitOne();//等待接收信号
            Console.WriteLine("开始第二部操作");
            Thread.Sleep(seconds * 1000);
            Console.WriteLine("工作已经完成");
            _workerEvent.Set();
        }
    }
````

> AutoResetEvent类像一个旋转门，一次只允许一个人通过

### 6、ManualResetEventSlim类

> 相比AutoResetEvent类，ManualResetEventSlim更加灵活
>
> ManualResetEventSlim相当于大门你可以控制通过的人数
>
> 这两个类其实都是基于信号量机制的。





### 7、线程池学习

》









## 二十七、C#中处理XML

> XmlDocument类
>
> System.Xml 名称空间
>
> XmlReader是一种拉模型，它把应用程序请求的数据拉入该应用程序。
>
> 拉模型的优点：可以选择把什么数据发送给应用程序。
>
> 你要什么你就去拉，推模型是所有XMLO数据都必须由应用程序处理，无论是否需要这些数据。
>
> 



> XmlDocumnet类和XmlReader类的区别：
>
> XmlReader——常用于你只需要加载一次文档
>
> XmlDocument——你要重复查看某个节点
>
> 所以用XmlDocument准没错。



### 1、ADO.NET和xml相互转化

```C#
  static void XmlToADONETT()
        {
            DataSet ds = new DataSet("XMLProducts");

            ds.ReadXml("Products.xml");
            DataGrid dg1 = new DataGrid();
            dg1.DataSource=ds.Tables[0];
            foreach(DataTable dt in ds.Tables)
            {
                foreach(DataColumn col in dt.Columns)
                {
                    textbox1.Text += "\t" + col.ColumnName + "-" + col.DataType.FullName + "\r\n";
                }
            }
        }
```



### 2、XML序列化与反序列化

> 用到C#的特性
>
> [XmlElementAttribute]——标记一个成员为XML元素
>
> [XmlRootAttribute]——标记一个类为根元素
>
> [XmlArrayItem("Prod",typeof(Product))]——标记一个数组元素，类型是Product类



### 3、Linq操作xml

> Xdocument 代替了 XmlDocument对象



## 二十八、SQL存储过程

> 关键词：Procedure

```Sql
CREATE PROC sp_test@param1INT,@param2 VARCHAR(16)AS SELECT * FROM test WHERE id-@param1AND t_no=@param2;G0
```

> 存储过程的优点：
> 提高性能
> SQL语句在创建过程时进行分析和编译
> 存储过程是预编译的，在首次运行一个存储过程时，查询优化器对其进行分析、优化，并给出最终被存在系统表中的存储计划，这样，在执行过程时便可节省此开销。
> 降低网络开销存储过程调用时只需用提供存储过程名和必要的参数信息，从而可降低网络的流量
> 便于进行代码移植数据库专业人员可以随时对存储过程进行修改，但对应用程序源代码却毫无影响，从而极大的提高了程序的可移植性，
> 更强的安全性系统管理员可以对执行的某一个存储过程进行权限限制，避免非授权用户对数据的访问在通过网络调用过程时，只有对执行过程的调用是可见的。 因此，恶意用户无法看到表和数据库对象名称、嵌入自己的 Transact-SOL 语句使用过程参数有助于避免 SOL 注入攻击。 因为参数输入被视作文字值而非可执行代码，所以，攻击者将命今插入过程内的 Transact-SQL可以对过程进行加密，这有助于对源代码进行模糊处理。

> **优势在于：存储过程速度快，已经预编译好了**



## 二十九、MSBUILD

> Microsoft 生成引擎是一个用于生成应用程序的平台。 此引擎（也称为 MSBuild）为项目文件提供了一个 XML 架构，用于控制生成平台处理和生成软件的方式。 Visual Studio 会使用 MSBuild，但 MSBuild 不依赖于 Visual Studio。 通过在项目或解决方案文件中调用 msbuild.exe 或 dotnet build，可以在未安装 Visual Studio 的环境中安排和生成产品。

> 有时会说你的sdk版本 和 MSBUILD版本不一致，感觉解决方法就是改变xml文件里的sdk版本。



## 三十、Task和Thread的区别

> `Task` 和 `Thread` 都是在多线程和并发编程中使用的概念，但它们有一些重要的区别。以下是 `Task` 和 `Thread` 的主要区别：
>
> 1. **抽象层级**：
>    - `Thread` 是操作系统级别的线程概念。每个 `Thread` 对象都代表一个操作系统线程，它负责执行代码。
>    - `Task` 是在 .NET 中对异步操作的抽象表示。一个 `Task` 表示一个异步操作单元，可以是单个线程，也可以是线程池中的线程，或者是其他执行上下文。
> 2. **创建和管理**：
>    - 使用 `Thread` 创建和管理线程时，需要直接与操作系统的底层 API 交互，包括线程的创建、启动、停止和同步等。
>    - 使用 `Task` 可以更方便地创建和管理异步操作。.NET 框架提供了高级的任务调度和管理机制，使得异步编程更加容易和可控。
> 3. **线程池**：
>    - `Thread` 不会自动加入线程池，每个创建的 `Thread` 对象都会占用操作系统的资源。手动管理大量线程可能会导致资源浪费和性能问题。
>    - `Task` 默认情况下会使用线程池中的线程执行异步操作，这样可以有效地复用线程并减少线程创建和销毁的开销。
> 4. **并发性能**：
>    - `Thread` 的创建和销毁需要较大的开销，管理大量线程可能会导致性能问题。
>    - `Task` 使用线程池中的线程来执行任务，可以更有效地利用资源并提高并发性能。
> 5. **异常处理**：
>    - 在使用 `Thread` 时，需要自行处理线程抛出的异常，否则异常会导致程序崩溃。
>    - `Task` 具有内置的异常处理机制，可以更方便地捕获和处理异步操作中的异常。
>
> 反正就是Task什么都好
>
> ```C#
> await Task.Delay(10000);
> ==
> Thread.Sleep(10000);
> ```
>
> 





## 三十一、模板（项目模板和项模板）

> 都在C:\Users\admin\Documents\Visual Studio 2022\Templates\文件夹下
>
> 可以自定义模板
>
> 项目——>导出模板——>导出后的文件放在上述文件夹下即可（zip文件）
>
> 加完必须重启

> 项——》对应的与类相似的文件
>
> 项目——》对应project项目
>
> 也可以添加所需的依赖项
>
> 可以新建Code、Custom、WPF文件夹把压缩文件放进去，就相当于模板添加到特定的分类里边了

> 除了创建类模板，你也可以选择wpf模板（包括.cs代码和xaml代码）



## 三十二、弃元

> 在 C# 中，"弃元"（Discard）是一个占位符，通常用于表示一个不需要的值或不需要的变量。弃元用下划线 `_` 表示，它的主要作用是告诉编译器不关心这个值，可以忽略它。弃元通常在以下情况下使用：
>
> 1. **未使用的参数**：当您定义一个方法，但不打算使用其中的某些参数时，可以使用弃元来占位。这有助于明确表示这些参数是故意不使用的，同时避免编译器产生未使用参数的警告。
>
>    ```csharp
>    public void MyMethod(int usedValue, int _)
>    {
>        // 在这里，不使用弃元参数 _
>        Console.WriteLine(usedValue);
>    }
>    ```
>
> 2. **忽略方法返回值**：有时，您可能调用一个方法，但对其返回值不感兴趣，可以使用弃元来忽略它。
>
>    ```csharp
>    _ = SomeMethodWithReturnValue();
>    ```
>
> 3. **模式匹配时的占位**：在模式匹配中，可以使用弃元来忽略某些模式的值。
>
>    ```csharp
>    if (myObject is MyClass { Property1: 42, Property2: _ })
>    {
>        // 在这里，忽略 Property2 的值
>    }
>    ```
>
> 弃元的目的是增加代码的清晰度，明确表示某些值或变量是有意忽略的，从而使代码更易于阅读和理解。





## 三十三、switch case 进阶

> C#中case 值只要是编译时常量都行
>
> C#中每个case后加break不是必要的
>
> 那如果要用到两个执行语句怎么办，比如执行完case A的语句后，下一行可以用goto case B：或者goto case default;
>
> 注意：每个case语句结尾最常见的是break;但也可以是return、goto case、yield break、yield return



> 当case限定条件不够时，可以加when来增加判断条件
>
> ```C#
> switch(item)
> 
> {
> 
> case "apple" when cost<10:
> 
> SellApple();
> 
> break;
> 
> case "banana" when cost<5:
> 
> SellBanana();
> 
> break;
> 
> }
> ```
>
> 



### Switch 表达式

> C# 8.0引入了新的`switch`表达式，它可以更简洁地表示`switch`语句。使用`=>`符号而不是`case`和`break`。
>
> ```C#
> var result = option switch
> {
>     1 => "Option 1",
>     2 => "Option 2",
>     _ => "Default Option"
> };
> 
> ```
>
> 



### Switch 模式匹配

> ```C#
> object obj = "Hello";
> switch (obj)
> {
>     case int i:
>         Console.WriteLine($"It's an int: {i}");
>         break;
>     case string s:
>         Console.WriteLine($"It's a string: {s}");
>         break;
>     default:
>         Console.WriteLine("It's something else.");
>         break;
> }
> 
> ```
>
> 





## 三十四、元组[不用创建新类了]

> C# 的元组（Tuple）是一种有序的、不可变的数据结构，它允许你将多个不同类型的值组合成一个单一的对象。元组的每个元素可以具有不同的数据类型，而且一旦创建，元组的内容不能被修改。
>
> - **返回多个值**：当一个函数需要返回多个相关的值时，可以使用元组。这可以减少创建新的类或结构的需求，从而使代码更简洁。
> - **临时组合数据**：当你需要将多个相关的值组合在一起，以便在代码中传递或存储它们时，元组是一个方便的方式。
> - **模式匹配**：元组可用于模式匹配，帮助你处理复杂的数据结构，特别是在 `switch` 语句中或其他需要条件匹配的地方。
>
> 以下是一些常见的元组用法和示例：
>
> **1、返回多个值**
>
> ```C#
> public (string, int) GetPersonInfo()
> {
>     return ("John", 30);
> }
> 
> // 调用函数并解构返回的元组
> var (name, age) = GetPersonInfo();
> Console.WriteLine($"Name: {name}, Age: {age}");
> 
> ```
>
> **2、组合多个值**
>
> ```C#
> var point = (x: 5, y: 10);
> Console.WriteLine($"X: {point.x}, Y: {point.y}");
> 
> ```
>
> **3、模式匹配**
>
> ```C#
> switch (shape)
> {
>     case (Circle { Radius: > 0 } c):
>         // 处理有效的圆
>         break;
>     case Circle c:
>         // 处理无效的圆
>         break;
>     case (Rectangle { Width: var w, Height: var h } r) when w == h:
>         // 处理正方形
>         break;
>     case Rectangle r:
>         // 处理矩形
>         break;
>     default:
>         // 处理其他情况
>         break;
> }
> 
> ```
>
> 



## 三十五、Expression（表达式）

> ![image-20231125090622832](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20231125090622832.png)
>
> 相当于
>
> ![image-20231125090642540](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20231125090642540.png)



> 上面的是委托
>
> 但==表达式==和委托不同
>
> 委托可以直接引用方法，但表达式不行，表达式是纯粹为了lambda表达式而创建的
>
> ```C#
> Expression<Func<int,bool>> test = i=>i>5;
> //正确，但它不会转换为方法
> //其实是抽象成了一个对象
> 
> ```
>
> ![image-20231125091513672](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20231125091513672.png)
>
> 实质是表达式树
>
> ![image-20231125093035111](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20231125093035111.png)
>
> ![image-20231125095537408](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20231125095537408.png)
>
> ![image-20231125095911240](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20231125095911240.png)



> 当涉及到 LINQ 查询时，有两个主要的类用于执行不同类型的查询操作：`Queryable` 和 `Enumerable`。
>
> ### Enumerable:
>
> 1. **适用范围：** 用于在内存中对 IEnumerable 集合（如数组、列表）进行查询。
>
> 2. **数据源：** 操作在内存中执行，适用于已加载到内存的数据集。
>
> 3. **命名空间：** `System.Linq` 中的 `Enumerable` 类。
>
> 4. 示例用法：
>
>    ```
>    csharpCopy codeint[] numbers = { 1, 2, 3, 4, 5 };
>    var result = Enumerable.Where(numbers, n => n > 3);
>    ```
>
> ### Queryable:
>
> 1. **适用范围：** 用于与实现 `IQueryable` 接口的数据源进行交互，通常用于数据库查询。
>
> 2. **数据源：** 可以将查询转换为底层数据源（如数据库服务器）上执行。
>
> 3. **命名空间：** `System.Linq` 中的 `Queryable` 类。
>
> 4. 示例用法：
>
>    ```
>    csharpCopy codevar dbContext = new YourDatabaseContext(); // 假设有一个数据库上下文
>    IQueryable<int> query = dbContext.YourTable.Where(n => n > 3);
>    ```
>
> ### 总结:
>
> - 如果数据已在内存中，使用 `Enumerable` 进行查询更为方便。
> - 如果数据存储在外部数据源（如数据库），并且数据源实现了 `IQueryable` 接口，那么使用 `Queryable` 可以在数据源上执行查询，而不是在内存中进行。
>
> 在实际使用中，你通常会根据数据的来源和性质选择使用 `Enumerable` 或 `Queryable`。对于内存中的集合，`Enumerable` 更直观且方便。对于数据库查询等情况，`Queryable` 提供了更强大的查询优化和延迟加载的能力。
>
> EntityFramework就是能理解这些表达式树的框架



## 三十六、垃圾回收（GC）

> 垃圾回收主要发生在回收短期对象时。为了优化垃圾回收器的性能，托管堆分为 0、1 和 2 三代，因此它可以分别处理长期和短期对象。垃圾回收器在第 0 代中存储新对象。在应用程序生存期早期创建的对象在集合中幸存下来，这些对象将在第 1 代和第 2 代中提升和存储。由于压缩托管堆的一部分比压缩整个堆更快，因此此方案允许垃圾回收器释放特定一代的内存，而不是在每次执行回收时释放整个托管堆的内存。



## 三十七、超快并行序列操作 - PLINQ(Parallel LINQ)

> ![image-20240308154706792](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20240308154706792.png)





> `AsParallel()` 是 LINQ 中的一个扩展方法，用于将普通的 LINQ 查询转换为并行 LINQ 查询（PLINQ）
>
> ```C#
> var results = Enumerable.Range(0, 100).AsParallel().Select(DoHeavyWork).ToList();
>  static int DoHeavyWork(int i)
>  {
>      // 这里进行耗时操作...
>      // 假设进行一些复杂的计算，然后返回结果
>     for(int x = 0;x<5000000;x++)
>      {
>          i += x;
>      }
>      return i;
>  }
> //输出乱序，但快
> 
> var results = Enumerable.Range(0, 100).Select(DoHeavyWork).ToList();
>  static int DoHeavyWork(int i)
>  {
>      // 这里进行耗时操作...
>      // 假设进行一些复杂的计算，然后返回结果
>     for(int x = 0;x<5000000;x++)
>      {
>          i += x;
>      }
>      return i;
>  }
> //输出有序，但慢
> ```
>
> 通过利用多个线程，PLINQ 能够更充分地利用多核处理器的优势，从而加快数据处理的速度。PLINQ 会动态地分配任务，并根据系统资源和工作负载来调整线程的数量，以达到最佳的执行效率。



### ConcurrentDictionary

> `ConcurrentDictionary` 是 .NET Framework 和 .NET Core 中的一个线程安全的字典实现。它允许多个线程同时读取和写入字典，而无需额外的同步手段（如锁）来保护数据结构的完整性。
>
> ```C#
>  static ConcurrentDictionary<int ,List<int>> TestConcurrentDic=new ConcurrentDictionary<int ,List<int>>();
> TestConcurrentDic.AddOrUpdate(1, new List<int>() { 1,3,5},null);
> //无需委托即这么做
> ```
>
> 当需要对 `ConcurrentDictionary` 进行读取或写入操作时，它会根据键的哈希值将操作映射到对应的分段上，然后获取分段的锁来执行操作。这样，即使多个线程同时对字典进行操作，它们也只会竞争相应分段的锁，不会阻塞其他分段的操作，从而减少了锁的冲突，提高了并发效率。
>
> 每个分段存储的是键值对的集合，而不是键值对的部分。


### WithDegreeOfParallelism

> 最大并行度控制
>
> ```C#
> var results = Enumerable.Range(0, 100).AsParallel().WithDegreeOfParallelism(2).Select(DoHeavyWork).ToList();
> ///设置并行度为 2 可能会导致任务被分成两个较小的部分，在两个线程上并行执行。这样做可能会比并行度为默认值（通常情况下为处理器核心数）更有效，因为它可以更好地平衡线程的资源消耗和任务的执行效率。
> ```
>
> 通过设置并行度，你可以控制并行 LINQ 查询使用的线程数，从而影响到程序的性能和资源消耗。通常来说，如果任务是 CPU 密集型的（比如你的 `DoHeavyWork` 方法），设置并行度可以帮助利用多核处理器的优势，提高程序的执行效率。但是如果任务是 IO 密集型的，设置过大的并行度可能会导致线程间的上下文切换增加，反而降低程序的性能。
>
> 在 WinForms 中创建自定义控件通常更多地涉及到 CPU 密集型操作而不是 IO 密集型操作。





### AsOrdered

> 确保有序
>
> ```C#
> 错误例子：
> var results = Enumerable.Range(0,100).AsParallel().WithDegreeOfParallelism(2).AsOrdered().Select(DoHeavyWork).ToList();
> 
> 正确例子：
> var results = Enumerable.Range(0, 100)
>                         .AsParallel()
>                         .AsOrdered() 
>     					// 在这里调用 AsOrdered()
>                         .WithDegreeOfParallelism(2) 						// 在这里指定并行度
>                         .Select(DoHeavyWork)
>                         .ToList();
> //首先调用了 AsParallel() 方法来创建一个并行查询，然后调用了 AsOrdered() 方法来确保结果保持原始顺序，最后调用了 WithDegreeOfParallelism() 方法来指定并行度。
> ```
>
> 



### AsSequential

> `.AsSequential()` 方法是 LINQ 中的一个方法，它用于将并行查询转换为串行查询。
>
> 在并行 LINQ 中，如果你调用了 `AsParallel()` 方法，LINQ 查询将以并行方式执行，这意味着查询会在多个线程上并行执行以提高性能。但是，在某些情况下，你可能希望将并行查询转换为串行查询，以便更好地控制查询的执行顺序或者避免并行执行带来的性能开销
>
> ```C#
>   var results = ParallelEnumerable.Empty<int>().AsSequential();
> ```
>
> 



### 灵活运用 并行 和 串行

> ```C#
> var collection = Enumerable.Range(0,10).AsParallel( )
> Select(HeavyComputation)//in parallel
> .AsSequential( )
> Select(HeavyComputation);//sequentially
> ```
>
> 在并行模式下执行一组重型计算，然后在串行模式下按顺序再次执行相同的一组重型计算。







### WithCancellation

> 在 C# 中，`Task` 类型代表一个异步操作的执行，它可以通过调用 `CancellationToken` 对象的 `Cancel()` 方法来取消。`WithCancellation` 方法允许你将取消操作与 `Task` 或 `Task<T>` 关联起来，使得在取消操作触发时，异步操作可以安全地终止。
>
> ```C#
> var cts = new CancellationTokenSource();
> 
> var collection = Enumerable.Range(0,10)
> .AsParallel( ).WithCancellation(cts.Token)Select(HeavyComputation)// in parallel
> .AsSequential( )
> Select(HeavyComputation);//sequentially
> 
> cts.Cancel;
> 
> 在你的代码中，WithCancellation(cts.Token) 方法用于为并行查询提供取消支持，这意味着在取消请求发出后，AsParallel() 方法会尽快中止并行计算，并且 Select(HeavyComputation) 方法可能会抛出 OperationCanceledException 异常，以通知调用者该操作已被取消。
> ```
>
> 





### WithExecutionMode

>```C#
>Enumerable.Range(0, 100)
>    .AsParallel()
>    .WithExecutionMode(ParallelExecutionMode.ForceParallelism)
>    .Select(x => HeavyComputation(x))
>    .ToList();
>
>
>//在这个例子中，我们使用 WithExecutionMode 指定了并行查询的执行模式为 ParallelExecutionMode.ForceParallelism，它表示强制使用并行执行模式，即使并行执行可能不够高效。当然，你也可以选择其他的执行模式，比如 ParallelExecutionMode.Default，它会让运行时自行决定是否采用并行执行。
>```
>
>ParallelExecutionMode.Default
>
>它很聪明，能自动知晓该用哪种方式执行



### Linq重要概念

> 
>
>      using System;
>     using System.Linq;
>     
>     class Program
>     {
>         static void Main(string[] args)
>         {
>             var collection = Enumerable.Range(0, 3).Select(Print);
>             foreach (var item in collection)
>         {
>             // 不要在这里调用 Environment.Exit(0);
>             Console.WriteLine(item);
>         }
>     
>         // 在这里调用 Environment.Exit(0);
>         Environment.Exit(0);
>     }
>     
>     static int Print(int n)
>     {
>         Console.WriteLine(n);
>         return n;
>     }
>     }
> 在 C# 中，LINQ 查询并不是在调用的时候立即执行的。相反，它们是延迟执行的，这意味着查询不会立即执行，直到需要其结果的时候才会执行。
>
> 当你使用 `foreach` 循环迭代 `collection` 时，查询才会被执行。在这个过程中，`Print` 方法会被调用，并且集合中的每个元素都会传递给 `Print` 方法进行处理。
>
> 
>
> ```C#
> 若变成
>     var collection = Enumerable.Range(0, 3).AsParallel().Select(Print);
> ```
>
> 



## 三十八、74个linq扩展方法（立即执行与延迟执行）

> 在 LINQ 中，有一些方法是立即执行的，而另一些方法则会延迟执行，这取决于 LINQ 查询的类型以及如何使用它们。
>
> 1. **立即执行方法**：立即执行方法会立即执行查询并返回结果。这些方法包括 `ToList()`、`ToArray()`、`ToDictionary()`、`FirstOrDefault()` 等。当调用这些方法时，LINQ 查询会立即执行并生成结果集。
> 2. **延迟执行方法**：延迟执行方法不会立即执行查询，而是在遍历结果时才会执行。这些方法包括 `Where()`、`Select()`、`OrderBy()` 等。这些方法创建了一个查询表达式，但实际的查询操作会在遍历结果集时才会发生。
>
> 关于缓存，LINQ 查询通常不会自动缓存结果，但你可以通过调用 `ToList()`、`ToArray()` 等方法来在内存中缓存结果，以便多次访问。



### 过滤方法（Filtering）

> 1、Where——过滤出符合谓词的数据
>
> 2、OfType——筛选出指定类型的数据
>
> ```C#
> 	IEnumerable<int> collection = Enumerable.Range(1, 100);
>     List<object> objectCollection = new List<object> { 1, 2, "hello", "4" };
>     #region Filtering 过滤方法
>     var filter1 = collection.Where(x => x > 5);
>     var filter2 = objectCollection.OfType<int>();
>     #endregion
> ```
>
> 



### 分区方法（Partitioning）

> 1、Skip
>
> 2、Take
>
> 3、SkipLast
>
> 4、TakeLast
>
> 5、SkipWhile
>
> 6、TakeWhile 
>
> ```C#
> #region Partition 分区方法
> var part1 = collection.Take(3);//拿前三个
> var part2 = collection.TakeLast(3);//拿最后三个
> var part3 = collection.Skip(1);//跳过第一个
> var part4= collection.SkipLast(1);//跳过最后一个
> var part5 = collection.TakeWhile(x => x > 5); //需要指定谓词
> var part6 = collection.SkipWhile(x => x > 5);
> #endregion
> 
> ```
>
> *** 注意SkipWhile和TakeWhile 
>
> `SkipWhile` 方法是 LINQ 提供的一个用于集合操作的方法，它会跳过集合中满足指定条件的元素，直到遇到第一个不满足条件的元素，并返回该元素及其之后的所有元素。
>
> ```C#
> List<int> numbers = new List<int> { 1, 2, 3,74, 5, 6, 7, 8, 9, 10 ,5,5,5,4};
> var numbersAfterFive = numbers.SkipWhile(x => x <= 5);
> //在这种情况下，数字 74 大于 5，所以跳过操作停止，然后返回从数字 74 开始的所有元素。
> 因此，numbersAfterFive 将包含以下元素：74, 5, 6, 7, 8, 9, 10, 5, 5, 5, 4。
> ```
>
> 
>
> ```C#
>             List<int> numbers = new List<int> { 1, 2, 3,74, 5, 6, 7, 8, 9, 10 ,5,5,5,4};
>             var numbersAfterFive = numbers.TakeWhile(x => x <= 5);
> //正好相反
> 直到遇到第一个大于 5 的元素为止。所以，numbersTakeWhile 将包含元素 1, 2, 3
> ```
>
> 





### 投影方法（Projection）

> Projection（投影）通常指的是将一个数据结构转换成另一个数据结构的过程。在LINQ中，Projection指的是通过选择性地提取或转换数据来创建新的数据结构。



> 1、Select //基本的投影操作 
>
> ```C#
> var pro1 = collection.Select(o=>o.ToString()+"HelloNum");
> var pro2= collection.Select((i,x)=>$"i"+x);
> var pro3 = collection.Select(x => $"Value: {x}");
> ```
>
> 
>
> 2、SelectMany //将数据扁平化处理，两个list<int>可以合并成一个
>
> ```C#
>  List<int> firstList = new List<int> { 1, 2, 3 };
>  List<int> secondList = new List<int> { 4, 5, 6, 7 };
> 
>  IEnumerable<List<int>> intCollections = new List<List<int>> { firstList, secondList };
> 
>  var pro4 = intCollections.SelectMany((val,o)=>o.ToString()+"HelloNum"); // 将数据扁平化处理
> 
> var pro5 = intCollections.SelectMany((val,index)=>val.Select(x=>$"{index}Test{val}".ToString())); // 将数据扁平化处理
> 会输出 
>  0，1
>  0，2
>  0，3
>  1，4
>  1，5
>  1，6
>  1，7
> ```
>
> 
>
> 
>
> 3、Cast（强制转换）
>
> ```C#
> IEnumerable<string> castCollections = new List<string> { "123" };
>  castCollections.Cast<int>();
> ```
>
> 
>
> 4、Chunk（分割）
>
> 用于将一个序列分割成指定大小的块：
>
> ```C#
> var numbers = Enumerable.Range(1, 10);
> 
>         var chunks = numbers.Chunk(3);
> ```
>
> 





### 存在或数量检查（Existence & Quantity Checks）

> 1、Any——有一个符合就跳出迭代
>
> ​            var check1 = collection.Any(o=>o / 10 == 1);
>
> 2、All——检查是否所有的都符合谓词，只有所有的都符合返回true
>
> ​            var check2 = collection.All(o=>o / 10 == 1);
>
> 3、Contains——检查泛型数据是否存在
>
> ​        var check3 = collection.Contains(2);
>
> 这三个都是立即检查的
>
> 注意！！！ 大多数linq扩展方法都是==延迟执行==的



### 序列操作（Sequence manipulation）

> ```C#
>  var seq1 = collection.Append(1);
> //序列末尾添加值
>  
> var seq2 = collection.Prepend(11);
> //序列开头添加值
> ```
>
> 



### 聚合方法（Aggregation Methods）

> 1、Count——立即执行的
>
> 2、TryGetNonEnumeratedCount——立即执行
>
> 3、Max——立即执行
>
> 4、MaxBy——立即执行
>
> 5、Sum——立即执行
>
> 6、Average——立即执行
>
> 7、Aggregate——立即执行（难点）
>
> 8、LongCount——立即执行





### 核心难点Aggregate

> 在 LINQ 中，`Aggregate` 是一个非常强大的方法，用于对序列中的元素执行累积运算，并生成一个结果。`Aggregate` 方法允许你指定一个累积函数，该函数定义了如何对序列中的元素进行累积运算。
>
> 1、重载方法一
>
> ```C#
>  IEnumerable<int> collection = Enumerable.Range(1, 10); 
> var agg1 = collection.Aggregate((x,y)=>x + y);
> 
> //输出55，1-10求和
> ```
>
> 这个 lambda 表达式表示对两个整数进行加法操作，其中 `x` 是累积的结果，`y` 是序列中的每个元素。



> ```C#
>             var agg2 = collection.Select(o=>o.ToString()).Aggregate((x,y)=>$"{x}, {y}");
> //先转成string再以逗号分割 （类似csv）
> ```
>
> 



> 2、重载方法2
>
> ```	C#
> var agg3 = collection.Aggregate(10,(x, y) => x + y); 
> 
> 初始时，累加器的值为 10。
> 累加函数首先将累加器的初始值 10（x）和序列中的第一个元素 1（y）相加，得到 11。
> 然后，累加器的值更新为 11，作为下一个计算的 x。
> 累加函数再将累加器的值 11 和序列中的下一个元素 2 相加，得到 13。
> 依此类推，直到处理完整个序列，得到最终的累积结果。
>     
>     //输出55+10=65
> ```
>
> 





> 3、重载方法3
>
> ```C#
> var agg4 = collection.Aggregate(10, (x, y) => x + y,x=>x=x/2);
> 
> //将执行完最后的结果 再最后做一个选择器函数
> 输出65/2
> ```
>
> 







### 元素操作（Element Operate）

> 这下面的方法都是立即执行的
>
> 1、First —— 返回序列第一个元素
>
>   var oper1 = collection.First(); 
>
> 问题在如果没有就会报错
>
> 2、FirstOrDefault —— 不会报错
>
> var oper2= collection.FirstOrDefault();
>
> 3、Single —— 返回唯一元素
>
> var oper3 = collection.Single();
>
> 如果列表中有且仅有一个元素，它将返回该元素。否则，如果列表为空或者包含多个元素，`Single()` 方法将抛出异常。
>
> 4、SingleOrDefault
>
> 如果列表中有且仅有一个元素，它将返回该元素。如果列表为空，它将返回整数的默认值 0。如果列表包含多个元素，`SingleOrDefault()` 方法将抛出异常。
>
> 5、Last
>
> 6、LastOrDefault
>
> 和最上面的First相似
>
> 7、ElementAt()
>
> 返回指定序列是否存在值，如果无则报异常
>
> 8、ElementAtOrDefault()
>
> 默认返回0





### 转换方法（Conversion Methods）

> 都是立即执行的
>
> 1、ToArray()
>
> 2、ToList()
>
> 3、ToDictionary()
>
> 4、ToHashSet
>
> 5、ToLookUp
>
> ```C#
> class Student
> {
>     public string Name { get; set; }
>     public int Grade { get; set; }
> }
> 
> List<Student> students = new List<Student>
> {
>     new Student { Name = "Alice", Grade = 10 },
>     new Student { Name = "Bob", Grade = 11 },
>     new Student { Name = "Charlie", Grade = 10 },
>     new Student { Name = "David", Grade = 11 },
>     new Student { Name = "Eva", Grade = 10 }
> };
> 
> var studentsByGrade = students.ToLookup(s => s.Grade);
> 
> ```
>
> 



### 生成方法（Generation Methods）

> 1、AsEnumerable 不是立即执行
>
> 2、AsQueryable 不是立即执行
>
> 3、Range 立即
>
> 4、Repeat 立即
>
> 5、Empty 立即



### 集合方法（Set Operation）

> 1. **Distinct**：
>    - `Distinct` 方法用于从集合中移除重复的元素，返回一个包含不重复元素的新集合。
> 2. **DistinctBy**：
>    - `DistinctBy` 是一个自定义版本的 `Distinct` 方法，它允许你根据指定的条件对集合中的元素进行去重。
> 3. **Union**：
>    - `Union` 方法用于合并两个集合，返回一个包含两个集合中所有不重复元素的新集合。
> 4. **Intersect**：
>    - `Intersect` 方法用于获取两个集合的交集，返回一个包含两个集合中共有元素的新集合。
> 5. **Except**：
>    - `Except` 方法用于从一个集合中移除另一个集合中包含的元素，返回一个包含不在第二个集合中的元素的新集合。
> 6. **ExceptBy**：
>    - `ExceptBy` 是一个自定义版本的 `Except` 方法，它允许你根据指定的条件从一个集合中移除另一个集合中包含的元素。
> 7. **IntersectBy**：
>    - `IntersectBy` 是一个自定义版本的 `Intersect` 方法，它允许你根据指定的条件获取两个集合的交集。
> 8. *UnionBy*：
>
> - `UnionBy` 是一个自定义版本的 `Union` 方法，它允许你根据指定的条件合并两个集合，并移除重复元素。
> - 这个方法与 `Union` 的区别在于，`UnionBy` 可以根据自定义的条件来确定两个元素是否相等，而 `Union` 只能使用默认的相等性比较。

> 
>
> 
>
> 9.SequenceEqual**： 立即执行的
>
> - `SequenceEqual` 方法用于确定两个序列是否相等，即它们是否包含相同的元素并以相同的顺序排列。
> - 这个方法对于比较两个序列非常有用，因为它可以在比较序列时检查元素的相等性和顺序性。





### 连接和分组（Join & Group）

> 1、Zip
>
> 2、Join
>
> 3、GroupJoin
>
> 4、Concat
>
> 5、GroupBy

> 当然，请看下面的解释：
>
> 1. **Zip**：
>    - `Zip` 方法将两个集合中相同索引位置的元素一一对应地合并成一个新的元素。
>    - 如果两个集合的长度不同，`Zip` 方法将会返回较短集合的长度。
>    - 示例：
>      ```csharp
>      var numbers = new List<int> { 1, 2, 3 };
>      var words = new List<string> { "one", "two", "three" };
>      var result = numbers.Zip(words, (num, word) => num + " " + word);
>      // 结果为 ["1 one", "2 two", "3 three"]
>      ```
>
> 2. **Join**：
>    - `Join` 方法执行的是内连接，它根据两个集合中的指定键相等的元素进行连接。
>    - `Join` 方法的返回结果是两个集合中的元素根据指定的键进行合并后的结果。
>    - 示例：
>      ```csharp
>      var result = from student in students
>                   join grade in grades on student.ID equals grade.StudentID
>                   select new { student.Name, grade.Grade };
>      ```
>
> 3. **GroupJoin**：
>    - `GroupJoin` 方法执行的是左连接，它将一个集合与另一个集合进行连接，并将结果分组。
>    - `GroupJoin` 方法返回的是一个集合，其中每个元素包含源集合中的一个元素以及与之相关联的子集合。
>    - 示例：
>      ```csharp
>      var result = from student in students
>                   join grade in grades on student.ID equals grade.StudentID into studentGrades
>                   select new { student.Name, Grades = studentGrades };
>      ```
>
> 4. **Concat**：
>    - `Concat` 方法用于将两个集合连接起来，形成一个新的集合。
>    - `Concat` 方法不会去除重复的元素，如果需要去重可以结合 `Distinct` 方法使用。
>    - 示例：
>      ```csharp
>      var result = numbers1.Concat(numbers2);
>      ```
>
> 5. **GroupBy**：
>    - `GroupBy` 方法用于对集合中的元素进行分组，按照指定的键进行分组。
>    - `GroupBy` 方法返回一个集合，其中每个元素表示一个分组，包含分组的键以及该分组中的元素。
>    - 示例：
>      ```csharp
>      var result = from student in students
>                   group student by student.Department into studentGroup
>                   select new { Department = studentGroup.Key, Students = studentGroup };
>      ```
>
> 这些方法可以帮助您在 LINQ 查询中执行各种集合操作，如连接、分组、合并等。



### 排序（Sort）

> 这些方法都是 LINQ 中用于排序集合的方法，它们可以按照指定的顺序对集合进行排序。
>
> 1. **OrderBy**：
>    - `OrderBy` 方法用于按照指定的顺序对集合进行升序排序。
>    - 例如：`collection.OrderBy(item => item.Property)`
>
> 2. **OrderByDescending**：
>    - `OrderByDescending` 方法用于按照指定的顺序对集合进行降序排序。
>    - 例如：`collection.OrderByDescending(item => item.Property)`
>
> 3. **ThenBy**：
>    - `ThenBy` 方法用于在之前的排序基础上，对集合进行额外的升序排序。
>    - 例如：`collection.OrderBy(item => item.Property1).ThenBy(item => item.Property2)`
>
> 4. **ThenByDescending**：
>    - `ThenByDescending` 方法用于在之前的排序基础上，对集合进行额外的降序排序。
>    - 例如：`collection.OrderBy(item => item.Property1).ThenByDescending(item => item.Property2)`
>
> 5. **Reverse**：
>    - `Reverse` 方法用于反转集合中元素的顺序。
>    - 例如：`collection.Reverse()`
>
> 6. **Order**：
>    - `Order` 方法是 `OrderBy` 方法的别名，用于对集合进行升序排序。
>
> 7. **OrderDescending**：
>    - `OrderDescending` 方法是 `OrderByDescending` 方法的别名，用于对集合进行降序排序。
>
> 这些方法允许您在 LINQ 查询中根据指定的条件对集合进行排序，以便以合适的顺序处理数据。通过使用这些方法，您可以轻松地按照自己的需求对集合中的元素进行排序。
