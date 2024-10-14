# CMD命令基础

## 一、基础命令

> 1、查看文件命令
>
> dir /a ——显示当前文件夹所有文件
>
> dir /ar——显示只读文件
>
> dir /as——显示系统文件
>
> cd 切换命令





> 2、拷贝复制重命名命令
>
> copy 源文件夹/源文件 目标地址——将源文件/源文件夹复制到目标地址
>
> xcopy 源文件夹 目标地址 参数 ——xcopy是copy的增强命令，可以实现对多个子目录进行拷贝
>
> /s 复制目录和子目录（不包括空目录） /e 复制目录和子目录（包括空目录）
>
> rename 源文件名 目标文件名
>
> move 源文件 目标地址 ——可以移动时直接重命名



> 3、磁盘修复命令
>
> chkdsk
>
> convert——将fat卷转为ntfs，convert f: /fs:ntfs //将驱动器F已经变成NTFS
>
> diskpart——进入分区模式 list disk 查看磁盘
>
> ```bash
> diskpart //进去磁盘分区模式
> select disk 1 // 选择磁盘1
> clean //格式化磁盘
> creat partition primary //创建主分区
> format fs=ntfs quick label = “E:” //定义磁盘
> ```
>
> 
>
> 



> 4、快速找到特定文件
>
> 借助for循环
>
> ```bash
> for /r c:\ %i in (*.txt) do echo %i   //找出c盘里所有txt后缀名文件并打印
> ```



> 5、批处理入门命令
>
> start——在命令行下允许一个程序（盘符、文件、文件夹、网址、程序）
>
> call——程序互相调用 call demo.bat //在同一路径可以直接调用，不同路径要加路径名



> 6、临时提升管理员权限 
>
> runas 
>
> ```bash
> runas /noprofile /user :mymachine administrator cmd  //提升到管理员权限
> hostname//查看主机名
> ```



> 7、net use命令
>
> 和对方主机进行连接，映射数据到本机
>
> 前提条件：对方开放IPC$共享，在同一网络下
>
> 早期IPC入侵 很常见。 IPC入侵，即通过使用Windows 系统中默认启动的[IPC$](https://baike.baidu.com/item/IPC%24/9482475?fromModule=lemma_inlink)共享，来达到侵略主机，获得计算机控制权的[入侵](https://baike.baidu.com/item/入侵/74119?fromModule=lemma_inlink)。此类入侵主要利用的是计算机使用者对[计算机安全](https://baike.baidu.com/item/计算机安全/2950375?fromModule=lemma_inlink)的知识缺乏，通常不会给计算机设置密码或者密码过于简单，因此才导致被黑客的有机可乘。
>
> 1）、建立空连接
>
> net use \\\IP\ipc$ "" /user:""           //这句话有三个空格
>
> 2）、建立非空连接
>
> net use \\\IP\ipc$ "用户名" /user:"密码"   
>
> 3）、映射默认共享
>
> net use z:\\\IP\c$ "密码" /user:"用户名" //将对方的c盘映射为自己的z盘，其他类推
>
> 如果已经和目标建立了ipc$,则可以直接访问
>
> ```basg
> 则可以直接用IP+盘符+$访问 net use z:\\IP\c$
> ```
>
> 4）、删除一个ipc$连接
>
> net use \\ip\ipc$ /del
>
> 5）、删除共享映射
>
> net use c: /del           //删除映射的C盘



> 8、net 命令
>
> net user——查看本机所有用户
>
> net user userNew /add ——创建一个用户名为userNew的用户
>
> net user userNew /del ——删除
>
> net user userNew /active ——激活用户
>
> net share ——查看共享
>
> net share f=F:/ ——共享F盘
>
> net share f /delete—— 取消共享
>
> net view \\\主机名 ——查看特定主机共享
>
> net start + 服务——开启服务
>
> net stop+ 服务——关闭服务
>
> netsh interface tcp show global ——查看tcp全局参数
>
> ![image-20230212091646011](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20230212091646011.png)
>
> 



> 9、shutdown关机命令
>
> shutdown /r -t  120——120s后关机
>
> shutdown  /a ——取消关机设定

> 10、netstat网络相关命令
>
> netstat -ano  // 查看所有网络连接
>



> 11、wget下载网络资源
>
> cmd不带，必须要下载，放在windows/system32文件夹下
>
> 例子
>
> ```bash
> wget www.baidu.com  //爬取远程网址的html界面
> wget -r 网站 //爬取整个网站
> ```



> 12、系统文件修复
>
> sfc命令——必须管理员权限
>
> sfc /scannow ——开启系统扫描
>
> 注：C:\Windows\Logs\CBS 里面的日志文件能否删除此日志文件是提供有关脱机处理故障的详细信息的处理日志文件，可以删除
>
> sfc /verifile=文件名 ——验证文件的完整性

> 13、nslookup命令
>
> nslookup 主要用来诊断域名系统 (DNS) 基础结构的信息。查询DNS的记录，查询域名解析是否正常，在网络故障时用来诊断网络问题。
>
> nslookup 域名(www.baidu.com)

## 二、批处理.bat文件编写

> 注意事项：
>
> 1. ELSE 子句必须出现在同一行上的 IF 
> 2. /i 表示忽略大小写
> 3. set var1=asd //设置局部变量
> 4. setx PATH “%path;文件夹路径” //设置永久变量  setx PATH “%path;d:” //设置永久变量 
> 5. set 命令可以查看当前机器的环境变量
> 6. 特殊字符  
> 7. * 命令管道符：| —— 将第一条命令结果作为第二条命令参数
>    * 组合命令：&——当第一个命令执行失败，后面命令继续执行
>    * 组合命令：&&——当第一个命令执行失败，后面命令也不会执行。第一条成功，第二条也成功
>    * 组合命令：||——当第一个命令失败后才执行第二条命令
> 8. rem 或 :: 是bat文件里的注释部分
> 9. exit命令==return
> 10. goto命令——跳转 goto part1          :part1 echo iam ready
> 10. 死循环打开某个进程 start 进程名 %0
> 10. 循环次数打开某进程 for /l %%i in (1,1,3) do start 进程名
> 10. sort命令 -排序 sort demo.txt > 1.txt 将排序好的demo.txt重定向到1.txt
> 10. cmd命令清理系统垃圾
> 10. for循环语法：for /l %参数名 in (start,step,end) do command [参数]
>
> ```bash
> for /d %i in(*) do echo %i             //查询所有文件夹并打印
> for /r f:\%i in(*.txt) do echo %i                    //递归遍历所有文件夹
> ```
>
>  16.

> ```ba
> @echo off
> if exist F:\  (echo 存在哦)  else  (echo  不存在哦)
> pause  >nul
> ```
>
> ```bash
> @echo off
> set var1=hello
> set var2=HELLO
> if /i %var1%==%var2%  (echo 相等)  else  (echo  不相等)
> pause  >nul
> ```
>
> ```bash
> @echo off
> if exist c:\it  (echo 存在it文件夹)  else  (md  c:\it & echo 创建it文件夹成功)
> pause  >nul
> ```
>
> ping 2个数据包检测是否联通
>
> ```bash
> @echo off
> set /p var=请输入IP地址:
> ping  -n 2 %var%
> pause >nul
> ```
>
> 批量生成文件夹
>
> ```bash
> @echo off
> for /l %i in (1,1,10) do md c:\%i.txt
> pause >nul
> 
> 
> 
> 
> 或者 读取文本文件内容创建文件夹  /f 读取文件内容
> @echo off
> for /f %%i in (c:\demo.txt) do md c:\%%i.txt
> pause >nul
> ```
>
> 清理系统垃圾
>
> ```bash
> @echo off
> echo 请勿关闭本窗口!
> echo 正在清除系统垃圾文件，请稍等
> del /f /s /q %systemdrive% *.tmp
> del /f /s /q %windir% prefetch *.* rd /s /q %windir%\temp & md
> %windir%\temp 
> del /f /s /q "userprofile%\ Local Settings\Temp\*.*"
> del /f /s /q %systemdrive% *._mp
> del /f /s /q %windir% *.bak
> del /f /s /q %systemdrive%\ *.log
> del /f /s /q %systemdrive%\ *.gid
> del /f /s /q %systemdrive%\ *.chk
> del /f /s /q %systemdrive%\ *.old
> del /f /s /q %systemdrive%\recycled\*.*
> del /f /s /q %userprofile%\cookies\ *.*
> del /f /s /q %userprofile%\ recent\ *.*
> del /f /s /q "%userprofile%\ Local Settings \Temporary Internet
> Files \*.*"
> del /f /s /q "%userprofile%\recent\*.*del"
> echo 清除系统垃圾完成!
> ```
>
> 





## 三、Kail渗透

kali首先是一个操作系统,更确切的说它是一个基于Linux kernel的操作系统,该系统从BackTrack发展而来。建虚拟机导入就行了

> 1、namp的使用
>
> > nmap 网站域名 ——探测目的主机防火墙状态
> >
> > nmap -F 网站域名——加速探测的速度
> >
> > nmap -O 网站域名——查看主机的操作系统
> >
> > nmap -sT 网站域名——查看tcp端口开放情况
> >
> > nmap -sU 网站域名——查看udp端口开放情况
> >
> > nmap 192.168.1.100/24——扫c段0-255区间
> >
> > nmap -iL 1.txt——从文本中读取IP地址
