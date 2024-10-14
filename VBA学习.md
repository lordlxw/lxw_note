# VBA学习 

2023/2/19

> 一、简介
>
> VBA就是基于VB开发的人针对excel的操作
>
> 快捷键Alt+F11 进入宏界面
>
> 宏：指录制并且机器自动生成的代码——死板
>
> VBA：人编写的VB程序——灵活



> 2、如何使用
>
> F8——单步调试



> 2.1 定义变量
>
> n = 1 
>
> y = n*100
>
> [C1]=n;
>
> ```vb
> Sub 宏1()
> '
> ' 宏1 宏
> '
> 
>  temp = inputbox("请输入年份")
> n = 2
> [C1] = n
> [c2] = 2 * n
> [c3] = 3 * n
> [d4] = n * n
> p = n * n * n * n
> [d5] = p
>  Msg Box p
> End Sub
> 
> ```
>
> 2.2 变量的数据类型
>
> byte——1B
>
> integer——2B 
>
> Long——4B
>
> String——10B+字符串长度
>
> Date——8B
>
> Varient——16个B或22个B
>
> 声明变量的数据类型 
>
> dim 变量名 as 数据类型
>
> dim n as integer
>
> 简写 dim n%
>
> dim y$，定义y为字符串型
>
> 如果没有声明数据类型，则系统改类型为Varient
>
> 
>
> ```VB
> Sub test03()
>     Dim s&
>     s=Timer  //定义s为当前时间
>     
> End Sub
> ```
>
> &表示连接符，用于字符串拼接





> 2.3对象
>
> 常用对象：
>
> Workbooks（“工作簿名”）
>
> ActiveWorkbook 活动工作簿
>
> ThisWorkBook ‘代码所在工作簿’
>
> Sheets——工作表集合
>
> Sheets（n）第n个工作表——按顺序
>
> Sheetn 第n个工作表——按系统工作表名
>
> ActiveSheet 活动工作表
>
> Range（”单元格地址“）  比如Range（”C11“）=1  
>
> ==Range可以表达一整行，一整列或一整个区域==
>
> Range（”10：10“）.select 选中第十行
>
> Range（”C：C“）.Select 选中第三列
>
> Range（”c11：e15“）=3
>
> Cells（行，列） cells（11，3）=3
>
> [A1]单元格简写
>
> Activecell活动单元格
>
> Selection 选择的区域





> 2.4 属性
>
> Value——值
>
> Path——文件路径
>
> Name——名称
>
> Sheets.Count——工作表的数量
>
> Activecell.address——活动单元格的地址
>
> Interior.ColorIndex——内部属性的颜色属性
>
> ThisWorkbook.Sheets(3).Range("a1:b10").interior.colorindex=56 //将这块区域底色设置为黑色





> 2.5方法（绿标）
>
> 表达方式：对象在前，方法在后
>
> Add——新建方法
>
> Open（工作簿路径）——打开方法
>
> ActiveWorkbook.Clost——关闭活动工作簿 
>
> Range("b1").Copy("a1")——将b1贴到a1
>
> Range("b1").Copy range（”a1“）.PasteSpecial xlPasteValue ——只复制值
>
> Range（”a1“）.Cut[a13] 将a1移动到a13，a1的内容清空
>
> Delete
>
> Clear
>
> ClearContents
>
> Range("b1").Activate——使B1单元格变成活动单元格



> 2.6 IF 方法
>
> Excel 里 if(true 或 false,"成立","不成立")
>
> VBA写法
>
> 块if 不在一行写完时，要用EndIf表示块结束
>
> ```VB
> Sub test()
>     dim n%,x%
>     n=2
>     x=3
> if n>x Then
>         MsgBox "n>x"
>     else
>         msgbox ”x>n“
>     End if
> end sub
> 
> ```
>
> * 如果if 不在 一行写完一定要写end if
>
> if多重判断
>
> if ... else if
>
> ```vb
> Sub 判断()
>     if(cells(2,6))>=15000 Then
>         Cells(2,7)="贵宾"
>     elseif cells(2,6)>=10000
>         cells(2,7)="高级"
>     elseif cells(2,6)>=5000
>         cells(2,7)="中级"
>     else 
>         cells(2,7)="普通"
>     endif
> Endsub
>         
> ```
>
> 





> 2.7 For循环 和 Foreach
>
> for循环格式：默认步长为1
>
> for 变量名=x to x ”循环内容“ 
>
> next
>
> ```VB
> Sub test()
> Dim n%
>     For n = 2 To 19
>     If (Cells(n, 2) < 60) Then
>         Cells(n, 2).Interior.ColorIndex = 3
>     End If
> Next
> End Sub
> 
> ```
>
> 设定步长为4 	for n=2 to 52 step 4
>
> ​							next
>
> for循环嵌套
>
> ```vb
> Sub test()
> Dim n%, y%
> For n = 2 To 19
>     For y = 1 To 3
>     
>     If (Cells(n, 2) < 60) Then
>         Cells(n, 2).Interior.ColorIndex = 3
>     Next y
> Next n
>     End If
> End Sub
> 
> ```
>
> 
>
> For each ... next 
>
> 循环对象集合
>
> workbooks、worksheets、range区域
>
> 格式：
>
> for each 变量名 in 对象集合
> 循环内容
> next
>
> ```vb
> Sub test()
>     Dim s as workbook
>     for each s in workbooks
>     msg s.name
>     nest
> end sub
> ```
>
>  





> 2.8 End的用法
>
> End获取数据边界,便于获取所有行号和列号
>
> * End(xlUp)——上
> * End(xlDown)——下
> * End(xlToLeft)——左
> * End(xlToRight)——右
>
> 例子
>
> ```vb
> Sub test1()
>    x= range("a1").End(xlToRight).column  //获取a1为起始点到最右边的列号
> h= range("a1").End(xlDown).row  //获取a1为起始点到最下边的行号
> End sub
> ```
>
> 
>
> 
>
> 
>
> 
>
> 
>
> row和rows的区别
>
> row返回单元格所在的行号，如果是区域，则返回首行的行号
>
> rows代表行的集合，返回range对象 Rows(1).Select //选中第一整行
>
> Range("a5:e10").Rows("1").Select——选中a5到e10的首行
>
> 
>
> 获取工作表所有行数和列数
>
> rows.count //获取最大行号
>
> columns.count  //获取最大列号
>
> 
>
> 
>
> end定位边界不准，它是将遇到的第一个空格作为终止
>
> 所以我们可以这么做：最右边定位到最左边，最下边定位到最上边来确定内容范围
>
> ```vb
> sub test1()
>     cells(rows.count,"f").end(xlup).row
> 	cells(1,columns.count).end(xltoleft).select
> End sub
> ```





> 2.8 UsedRange 和 CurrentRegion
>
> UsedRange——worksheet的一个属性，代表指定工作表的**所用区域**
>
> 例：Worksheet(1).usedrange.select
>
> UsedRange适用于表比较干净，没有其他杂项的情况
>
> 
>
> CurrentRegion——单元格的一个属性，代表指定单元格所在的区域
>
> Range("a1").CurrentRegion //选中a1单元格所在区域
>
> 与excel 操作，选中一个单元格ctrl+a全选操作类似





> 2.9偏移
>
> 格式：
>
> ```vb
> 单元格.offset(偏移行，偏移列)从0开始
> 单元格.(偏移行，偏移列)从1开始
> 
> 例：
> Range("a1").Offset(8,4).Select
> ```
>
> 



> 3.0Exit函数 //提前结束，相当于break
>
> Exit Do——只能写在Do循环里面
>
> Exit For——只能写在For循环里面
>
> Exit Sub——只能写在Sub子过程里





> 3.1错误语句处理
>
> On Error Resume Next  //当代码出错时忽略，继续运行



> 三、Excel利用ODBC连接数据库
>
> EXCEL 和 Access数据库连接需要经过ADO
>
> EXCEL 和 Mysql数据库连接需要经过ADO 和 ODBC
>
> EXCEL 和 Mysql连接步骤
>
> 第一步 找到ODBC，windows直接搜索即可
>
> 第二步 网上下载相关ODBC的驱动程序，mysql就上mysql官网依次类推
>
> 第三步 odbc中选择excel 文件 点击添加相关驱动，下一步下一步，输入数据库的账号、密码。登陆上去选择相应的数据库。
>
> 第四步 工具-引用相关的连接库文件 （Microsoft ActiveX Data Objects 2.8 Library）写相关的ado代码，引入对象 注：==provider后面填的是对应数据库的引擎==
>
> ```VB
> //1、给连接对象取名字
> Dim conn As ADODB.Connection 声明对象变量
> Set conn = New ADODB.Connectior 创建对象变量
>     
>     //2、建立数据库的连接，例子是DB2数据库
>       
> conn.Open "Provider=IBMDADB2;Data Source=dbshot2;Persist Security Info=True;User ID=scc;Password=scc"
>     
>  //3、执行sql语句
>     conn.Execute(sql)
> ```
>
> 

》