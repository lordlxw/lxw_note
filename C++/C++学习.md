链接：https://www.youtube.com/watch?v=_bYFu9mBnr4&list=PLxFKkSyGrz0WgU4KmCMF86tyKZw2VL2By&index=1&t=25s



## 1、编译

> g++ ./youtube.cpp
>
> - "make"是一个构建工具，它负责根据Makefile定义的规则和依赖关系来决定需要重新编译的文件，并自动执行编译和构建操作。
> - "g++"是C++编译器，它是实际执行编译操作的工具，将C++源代码转换为可执行文件。
>
> Windows会生成一个a.exe文件
>
> Mac/Unix会生成一个a.out文件

> 用using namespace std; 看上去很省力，但在真实程序开发中很蠢，别用，会引起命名混乱



> 命令
>
> 1、g++ main.cpp —— 默认生成a.exe（不带调试信息）
>
> 2、g++ -g main.cpp -o my_app ——带-g表示带调试信息，-o表示指定输出名，则会生成my_app.exe
>
> 3、vscode中debug生成launch.json 可以生成和程序有关的调试信息
>
> 4、2、g++ -g main.cpp swap.cpp -o mysecond_app ——编译链接两个文件生成mysecond_app.exe



> 问：
>
> 1、printf 和  cout区别
>
> 2、std::bool
>
> 3、字符串的find方法，源代码如何实现
>
> 还有find_first_of方法，找不到返回NPOS，即-1的补码值
>
> 4、C++的字面量-literals

## CMake

> 可以生成复杂的makefile
>
> CMakeLists.txt
>
> ```CMake
> project(MYSWAP)
> 
> add_executable(my_cmake_swap main.cpp swap.cpp)
> ```
>
> 



## 2、数据类型

### 1、bool类型

特点：除了0是false，其他全是true

> ```C++
>  bool a = 1;
>     cout << a << endl;
>     cout << std::boolalpha << a << endl;
> ```

>std::boolalpha 可以强制转换成true / false



### 2、浮点型

> 三种：float、double、long double
>
> ```C++
> #inclue<float.h>
> float  a = 10.0/3;
> a = a*1000000000;
> cout << std::fixed << a<<endl; //会输出标准化的数字，而不是 e...
> cout << FLT_DIG<<endl; //float能保证的有效位数最多是6位
> ```
>
> float是最不值得信任的类型
>
> float能保证的有效位数最多是6位，即第7位开始就不值得信任了。
>
> 同理，double对应DBL_DIG,有效位数是15位
>
> long double 对应 LDBL_DIG,有效位数是18位
>
> 



### 3、Const、Enum、Define

> 都是不可变的常量



### 4、数字类型

> #include<cmath>
>
> C++的数字类型分为 NAN（负数开根号） 和 +-INF （无穷大）
>
>   cout << remainder(123, 61) << endl;
>
> 答案是1.000000，即余数的标准化个数
>
> 如果用 10 % 3.25 就会报错，要用 remainder（10，3.25）约等于 fmod(10,3.25)
>
> fmax(10,100) //输出最大值100

> 都是针对负数而言
>
> trunc（向上取整） 和 floor （向下取整）
>
> round(数字) //五舍五入





### 5、String类 和 C的Strings 比较 和 String的输入方法

> C++的String 比 C的Strings强得多
>
> String 也就是 一个字符数字
>
> #include<string>
>
> string  == std::string
>
> 未赋值的string类输出就是空（不是null，是""）
>
> .length()返回的是真实字符串的长度

> C的所谓string就是字符数组 char name[]，以\0结尾，不能随意改变数组大小

```C++
int main()
{
    string text;
    cin>>text;
    cout<<text<<endl;
}
//若你输入Hello World 则只会输出Hello，即空格前的字符串
```



> String的几种输入方法
>
> 1、cin
>
> 遇到空格即中断
>
> ```C++
> int main()
> {
>     cin>>a;
>     cin>>b;
>     cout<<a;
>     cout<<b;
> }
> //输入hello world
> 则直接输出两个数字
> ```
>
> 
>
> 2、getline(cin,变量名)



### 6、进制转换

> int number = 30；
>
> int number = 0X30；//16
>
> int number = 030；//8进制

> 输出时——std::cout<<std::hex<<number<<std::endl;



## 3、集合类型

> 1、Array数组——固定大小，若输出一个没有初始化的数组，会出现错误数字，==指针传递==
>
> 2、Vector 相等于C#的Array——动态大小，==值传递==
>
> 如果要编译vector，必须用C++11，即 g++ aaa.cpp -std=c++11
>
> 传递的时候要注意要传引用还是本身
>
> 3、STL::Array相当于 数组和Vector的折中，提供了更多操作的方法——也是静态的和[]一样，==值传递==
>
> - 提供的固定大小的数组容器，类似于普通数组，但提供了更多的功能和安全性。`std::array`的大小在编译时确定，无法动态调整。
> - 优点：固定大小，不需要手动管理内存；提供了多种操作和功能；可以通过迭代器访问元素；可以使用`size()`函数获取元素数量。
> - 缺点：大小固定，无法动态调整；相比数组，可能有轻微的性能开销。****
>
> 例： std::array<int ,10> data={1,2,3}  输出10个数字的话后面全是0（正常输出了）
>
> STL::array 和 vector 都是引用赋值，而原生[]不支持赋值

> 迭代的简便写法：
>
> ```C++
> int data[] = {1,2,3,4,5,6};
> //data也可以是std:array或是vector
> for(int n:data)
> {
>  	std::cout<<n<<endl;   
> }
> ```
>
> 





## 4、流式数据传输

> #include<iostream>
>
> std::ofstream file("hello.txt",std::ios:app);
>
> //使用`std::ios::app`打开模式：`std::ios::app`是`std::ios`命名空间中定义的打开模式之一，表示以"追加"的方式打开文件。使用该模式打开文件后，写入的数据将会追加到文件的末尾，而不是覆盖已有内容。



## 5、方法和传参

> 1. 值传递（Pass by Value）：将参数的值复制给函数的形式参数。在函数内部对形参的修改不会影响原始变量的值。值传递是一种简单、安全的方式，适用于传递基本类型或小型对象，但对于大型对象来说，复制的开销可能较大。
> 2. 引用传递（Pass by Reference）：通过引用将参数绑定到函数的形式参数。对形参的修改会影响原始变量的值。引用传递可以用于函数内部修改函数外部的变量，也可以减少复制的开销。引用传递对于大型对象或需要修改参数值的情况非常有用。
> 3. 指针传递（Pass by Pointer）：通过指针将参数传递给函数。指针传递与引用传递类似，允许在函数内部修改参数的值，并且可以传递空指针。使用指针传递需要注意空指针的处理和解引用操作。
> 4. const引用传递（Pass by Const Reference）：将参数通过const引用传递给函数。这样可以避免参数的复制开销，同时限制函数对参数的修改，确保参数的不可变性。const引用传递适用于大型对象或不需要修改参数值的情况。
> 5. 右值引用传递（Pass by Rvalue Reference）：C++11引入的特性，通过右值引用将参数传递给函数。右值引用传递适用于可移动的对象或需要使用移动语义的情况，可以提高性能。





## 6、C++文件组织

> 一般就三种
>
> 1、头文件(.h)——声明
>
> 2、实现文件(.cpp)——实现
>
> 3、主文件（主函数）——组织和调用



### 1、makefile

> 用途：每次编译不编译整个文件，而是变化的部分文件
>
> makefile通常用于linux或mac
>
> file1.cpp --> file1.o
>
> file2.cpp --> file2.o
>
> 如果file1.cpp 变化了
>
> 通常解决方法：
>
> 编译file1，编译file2，链接在一起 拿到a.out
>
> 
>
> makefile能很好地解决这个问题
>
> ```makefile
> math:math_stuff.o math_utils.o
> 	g++ math_stuff.o math_utils.o -o math
> 
> math_stuff.o: math_stuff.cpp
> 	g++ -c math_stuff.cpp
> 
> math_utils.o: math_utils.cpp
> 	g++ -c math_utils.cpp
> 
> clean:
> 	rm *.o math
> ```
>
> make——会执行编译你变化的obj



## 7、函数模板

```C++
template <typename T>
void Swap(T &a, T &b)
{
    T temp = a;
    a = b;
    b = temp;
}
//T可以换成别的符号

如果要做到数组类型的交换也一样
    void Swap(T a[],T b[])
{
    
}
```

