# 零碎点

# 一、

> 1、System.Guid  NewGuid创建一个不重复的32位随机数

> 2、Console.CancelKeyPress += (sender, eventArgs) => 方法。
>
> ctrl+c =》退出

> 3、config配置文件 可在app.config文件中以xml形式配置，注意要在程序集中引用==System.Configuration==这个基本库
>
> 示例：
>
> ``` c#
> app.config中添加：
>     <appSettings>
> 		<add key="key1" value="val1"/>
> 		<add key="key2" value="val2"/>
> 		<add key="key3" value="val3"/>
> 	</appSettings>
>     
> 主函数中：
>     string s1 = 				      		System.Configuration.ConfigurationManager.AppSettings["key1"];
>    Console.WriteLine(s1);
> //可以键值对形式打出val1
> ```
>
> App.config文件名是固定的
>
> 但是，显示所有文件夹后，在bin\Debug下可以看到完全编译后的文件名 例：consoleApp.exe.config——这才是真正的config文件
>
> 所有的config文件的基类都在machine.config中（在.net4.0文件夹中）。必要情况下，可以重写machine.config

> 4、IEnumerable<T>接口，提供了大量与查询相关的方法（Linq）
>
> 有：
>
> ==数值运算==：Aggregate（累加器）、Sum（求和）、Average（求平均值）、Max、Min、
>
> ==取值==：All（检查是否序列中所有元素都满足条件）、Any（检查序列中是否有任何一个元素满足条件，真返回true）、Contains（检查数据序列中是否包含特点元素）、Count（返回满足条件元素的数量）、Cast（将IEnumerable中元素转换为指定的数据类型）、FirstOrDefault（返回序列中满足条件的第一个元素，若不存在返回默认值）、
>
> LastOrDefault（返回序列中满足条件的最后一个元素，若不存在返回默认值）、
>
> SingleOrDefault（返回序列中满足条件的唯一元素，若不止一个会引发异常）
>
> ==集合操作==：
>
> Reverse（反转元素顺序）
>
> Distinct（返回不重复元素集合）
>
> Concat（连接两个序列，首尾相连）
>
> Except（差集）、Intersect（交集）、Union（并集）、SequenceEqual（比较两序列是否相等）
>
> ==提取子集==
>
> Skip（跳过指定数量的元素）、Take（从序列开头返回指定数量的连续元素）
>
> ToArray、ToList
>
> 5、exe是运行在自己的内存地址空间，dll没有自己的地址空间而是运行在你开启的exe环境中的
>
> dll的意义在于其复用性
>
> 6、每个进程都分配4GB虚拟内存，这4gb虚拟内存相当于空头支票。当你真正用的时候操作系统才会给你物理内存（物理页）。
>
> 孤立代码的唯一方式是通过进程来实现的，进程对确保安全有很大帮助，而它们的一大缺点是性能。
>
> 7、私有程序集和共有程序集：
>
> 私有程序集就是不同项目中生成的程序集，仅供本项目使用，或者可以经过配置被某一个其它项目的程序集引用。
>
> 共享程序集是机器级别的共享程序集，它放置一个叫GAC （Global Assembly Cache）的地方，可以被其它所有的私有程序集所引用。注意，GAC中只能放置.dll文件，而不能有.exe 文件。关于如何部署到GAC，具体细节查书。关于程序集的部署配置有很多需要注意的地方。
>
> 
>
> ```c#
> 你编写的程序如果没有进行 签名 不具备 公匙 文化 版本属性的程序集为私有程序集。该程序集只能在程序所在的目录使用。
> 像.net框架的程序集就是公有程序集。
> ```
>
> 8、堆栈和托管堆
>
> ![](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20230201111826311.png)

> 9、String是引用类型，但是与一般的引用类型有区别，字符串是不可改变的。
>
> 10、ref和out 的异同：
>
> 首先：两者都是按地址传递的，使用后都将改变原来参数的数值。
>
> 其次：ref可以把参数的数值传递进函数，但是out是要把参数清空，就是说你无法把一个数值从out传递进去的，out进去后，参数的数值为空，所以你必须初始化一次。这个就是两个的区别，或者说就像有的网友说的，ref是有进有出，out是只出不进。
>
> 11、内敛函数，相当于C++中的inline
>
> C#中使用如下特性
>
> ==C++ 中有个内联函数，使用 inline 来修饰函数，编译器就会对其进行优化，将此函数作为代码判断插入到调用处。==
>
> 函数调用在执行时，首先要在栈中为形参和局部变量分配存储空间，然后还要将实参的值复制给形参，接下来还要将函数的返回地址（该地址指明了函数执行结束后，程序应该回到哪里继续执行）放入栈中，最后才跳转到函数内部执行。这个过程是要耗费时间的。
>
> 另外，函数执行 return 语句返回时，需要从栈中回收形参和局部变量占用的存储空间，然后从栈中取出返回地址，再跳转到该地址继续执行，这个过程也要耗费时间。
>
> 而 C# 中可以通过在函数上使用特性，告诉编译器要对其进行优化，达到相同目的
>
> ​        [MethodImpl(MethodImplOptions.AggressiveInlining)]

> 12、结构体是否有构造函数
>
> 注意！！！结构体可以有构造函数，但必须是有参的，无参构造函数会报错。
>
> 结构体只能继承接口，不能继承父类。
>
> 13、泛型的几个特点
>
> :one:性能好，拆装箱
>
> :two:类型安全，只能添加一个类型的数据
>
> :three:二进制代码重用
>
> 14、Default关键字
>
> 在泛型中，default(T),null赋予引用类型，0赋予值类型。
>
> 15、泛型类的静态成员只能在类的一个实例中共享
>
> 16、yield和foreach的使用
>
> yield return语句返回集合的一个元素，并移动到下一个元素上。可以迭代大量数据，无需一次将所有数据读入内存。
>
> **`yield: 带有yield的函数是一个迭代器，函数返回某个值时，会停留在某个位置，返回函数值后，会在前面停留的位置继续执行，直到程序结束`**
>
> Yield Return关键字的作用就是退出当前函数，并且会保存当前函数执行到什么地方，也就上下文。你发现没下次执行这个函数上次跑来的代码是不会重复执行的。
>
> 17、sizeof运算符只能确定栈中值类型的长度（单位：字节）
>
> 如果对于复杂类型要求size，需要把代码放在==unsafe==块中。
>
> 18、比较引用类型的相等性
>
> ReferenceEquals(x,y);//比较两个引用是否引用类的同一个实例
>
> Equals可以重写，Object.GetHashCode()的方式
>
> 19、运算符重载
>
> ```C#
> 
>  internal class Vector
>  {
>      public int x;
>      public int y;
>      public int z;
> 
>      public Vector(int x, int y, int z)
>      {
>          this.x = x;
>          this.y = y;
>          this.z = z;
>      }
>      public Vector(Vector s)
>      {
>          this.x = s.x;
>          this.y = s.y;
>          this.z = s.z;
>      }
>      public static Vector operator +(Vector lhs, Vector rhs)
>      {
>          Vector result = new Vector(lhs);
>          result.x += rhs.x;
>          result.y += rhs.y;
>          result.z += rhs.z;
>          return result;
>      }
>  }
> ```
>
> C# 要求所有运算符重载都声明为public和static，这表示它们与它们的类或结构相关联。
>
> 20、比较运算符重载要成对！！！
>
> * ==和！=
> * 大于和小于
> * 大于等于和小于等于
>
> 在重载==和！=时，还必须重载Equals和GetHashCode方法
>
> 21、委托和事件
>
> **委托是类型安全的类。**
>
> **事件是类型安全的委托。**
>
> 22、对String的增删其实每次都要创建一个新的副本
>
> 好的解决方式是用StringBuilder类
>
> 23、C#中的正则表达式
>
> ```C#
>  static void Main(string[] args)
>         {
>             StringBuilder stringBuilder = new StringBuilder();
>             string text = @"abcion,sdaion,ionasd,ppaion";
>             const string pattern1 = @"ion\b"; //查找以ion结尾的字符串
>             const string pattern2 = @"\bn"; //查找以n为开头的字符串
>             const string pattern2 = @"\ba\S*ion\b"; //查找以a为开头,以ion为结尾的字符串 \S表示任何不是空白字符的字符
>             MatchCollection myMatch = Regex.Matches(text, pattern1);
>             foreach(Match m in myMatch)
>             {
>                 Console.WriteLine(m.Index); //返回下标index
>             }
>             Console.Read();
>         }
> ```
>
> 
>
> 24、equals方法默认比较的是两个值是否相等。如果是两个引用类型的是比较地址是否相等。
>
> 所以要重写equals方法，C#中当equals方法被重写时，GetHashCode方法也要重写，否则不能保证hash码是唯一的。（通常用亦或的形式）
>
> 
>
> 
>
> 25、共享资源实现异步多线程方式
>
> :one:	lock锁
>
> :two:	Interlocked类的静态方法 //效率更高，但只能和加减法一起用





# 二、Tips for C# programming

>1、多用$符号连接字符串，性能比较好。还有stringbuilder

# 三、常用接口

> 1、ICloneable 克隆 ~引用
>
> ```C#
>   public class Car:ICloneable
>     {
>         public Car(string length, string name, string width)
>         {
>             this.length = length;
>             this.name = name;
>             this.width = width;
>         }
> 
>         public string length { get; set; }  
>         public string name { get; set; }
>         public string width { get; set; }
>         public object Clone()
>         {
>             return new Car(this.length, this.name, this.width);
>             或者return MemberwiseClone();
>             
>         }
> 
>         public override string ToString()
>         {
>             return string.Format("车宽为{0},车长为{1},车名为{2}", this.width,this.length,this.name);
>         }
>     }
> ```
>
> 



# 四、特殊函数

> 1、string.IsNullOrEmpty(string s)
>
> 2、     object.ReferenceEquals(a, b);//判断对象a和对象b是否完全相等



# 五、版本更新新特性

> 1、C# 11——原始字符串
>
> 原始字符串至少以三个双引号`"""`字符开头，以相同数量的双引号字符结束。通常，原始字符串文字在一行使用三个双引号开始字符串，在另一行使用三个双引号结束字符串。
>
> 优势：处理json字符串时头尾加上`"""`就可以了，多行的时候，开头的`"""`和结尾的`"""`必须是单独占用一行，否则就会报错：



# 六、自己的领悟

> 2023/12/5
>
> 终于明白Task.Delay和Thread.Sleep的区别了
>
> 其实就是异步和同步
>
>  `Thread.Sleep` 是一个同步方法，它会导致当前执行线程（通常是主线程）在指定的时间内休眠，而不执行任何其他操作。
>
>  `Task.Delay` 是异步非阻塞的方法。它会创建一个延迟任务，而不会阻塞当前线程。用于Task，Async和Await
>
> 比如你点击一个按钮 切换视图
>
> Thread.Sleep会阻塞你，3S之后再切换到新视图（3S后再进入页面）
>
> 而Task.Delay会先切换到视图，3S之后再更新页面（先进入页面）



> 2023/12/5
>
> 字典的效率问题
>
> ![image-20231205155452550](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20231205155452550.png)
>
> 效率问题在于重复的哈希查找，先判断有没有键值，再做一次赋值操作，很浪费
>
> 如果用TryGetValue的时候要注意是struct值类型还是引用类型。
>
> ==解决方式一==
>
> ```C#
>    			Dictionary<int, string> AA = new();
>             int a = 10;
>             ref var s = ref CollectionsMarshal.GetValueRefOrAddDefault(AA,  a,out var existed);
> //out带出来的bool值existed代表了是否有该键
>             if(!existed)
>             {
>                 //如果不存在，则可以做赋值操作等等
>                 //优势在于只进行了一次哈希查找
>             }
> ```
>
> ==解决方式二==
>
> ```C#
>   internal class Program
>     {
>         static void Main(string[] args)
>         {
>             MyStruct my = new();
>             Dictionary<Guid, MyStruct> AA = new();
>             ref var ValOrNull = ref CollectionsMarshal.GetValueRefOrNullRef(AA, my.Id);
>             if(!Unsafe.IsNullRef(ref ValOrNull)) //如果值不是空引用
>             {
>                 ValOrNull.Id = my.Id;
>             }
>             //m会返回一个值引用或者一个空引用
>             Console.WriteLine("Hello, World!");
>         }
>     }
>   public  struct MyStruct
>     {
>         public Guid Id;
>     }
> ```
>
> 基准测试的效率对比：
>
> ![image-20231205161834703](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20231205161834703.png)
