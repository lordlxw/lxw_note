* 视频地址：https://www.bilibili.com/video/BV1bS4y1471A/?spm_id_from=333.337.search-card.all.click&vd_source=f93c2f87ac116d0ab5a7d34711f601c7



# Jenkins教程（从配置到实战）

> CI/CD

## 第一部分：GitLab安装部署

> 1、Jenkins在开发过程中所属位置
>
> 2、Jenkins+Maven+Git持续集成
>
> > Jenkins服务器（测试服务器，JDK）
> >
> > Gitlab服务器（托管代码）——Jenkins会从Gitlab服务器拉取代码
> >
> > Jenkins主服务器（需要内置Maven、JDK）
>
> 3、安装硬件环境和知识储备



### Gitlab安装

> Gitlab——代码托管平台
>
> 两种安装方式：
>
> ①SSH下安装——就是靠命令 npm 看文档即可
>
> ②Docker下安装——https://docs.gitlab.cn/jh/install/docker.html
>
> * 镜像，安装docker
> * 安装后配置开机启动项
> * 使用容器安装gitlab
> * * 1、添加容器 docker run ... 可能会发生端口冲突
>   * 2、启动容器  docker start gitlab
>   * 3、查看已存在的容器 docker ps -a
>   * 4、进入容器 docker exec -it gitlab /bin/bash
> * 就可以访问 域名uri来访问jenkins了
> * 管理员账号登录 用户名：root 密码存在文件中 cat /etc/gitlab/initial_root_password

---

### Jenkins服务器和测试服务器

> 测试服务器上只要安装一个jdk
>
> Jenkins服务器部署安装部署
>
> * 1、准备一个干净的环境（Centos8以上）
> * 2、检查是否安装了java -version
> * 3、看哪些java包可以用 java yum search java | grep jdk，然后 yum install -y XXX 安装完jdk
> * 4、下载jenkins的war包，然后上传至服务器。jenkins.war 
> * 5、java -jar jenkins.war 运行这个命令后，Jenkins 会在你的机器上启动一个 Web 服务器，通常是一个 **内嵌的 Jetty 服务器**，并通过默认的端口（8080）对外提供 Jenkins 的 Web 界面。
> * 6、首次登录。默认管理员 用户和密码 已经配好了 在隐藏文件夹 secrets里
> * 7、jenkins有很多插件/样式
>
> 
>
> 在Jenkins服务器上安装maven
>
> * 官网maven.apache.org 下载好 上传至服务端
> * tar xcvf apache-maven***.tar.gz，解压缩maven
> *  执行 到那个目录下执行maven /user/local/maven/bin/mvn
>
> 
>
> 
>
> Jenkins服务器也要装Git，不然无法执行拉取的。
>
> yum install git





## 第二部分：Jenkins配置Maven+Git

>初始化的Jenkins网页是不带maven的
>
>需要manage jenkins里安装插件——Maven Integration
>
>
>
>Jenkins里是按Item（任务）来配置的
>
>创建一个新任务后，在配置里面配置好Repository URL 有权限的需要添加凭证
>
>分支也要选好
>
>
>
>1、Jenkins服务器也要装Git，不然无法执行拉取的。 
>
>yum install git
>
>
>
>2、Pre Steps：
>
>构建前所做的一些操作，可以是脚本
>
>
>
>3、Build：
>
>Jenkins需要知道Maven安装在哪里 Global Tool Configuration 
>
>需要配置Maven的系统环境变量
>
>
>
>**POM 文件**是 **Project Object Model** 的缩写，中文意思是 **项目对象模型**。它是 Maven 构建工具中的一个核心文件，通常命名为 `pom.xml`，用于定义和描述 Java 项目的基本信息、依赖关系、构建配置等。Maven 是一个自动化构建工具，用于管理 Java 项目的构建、依赖和生命周期。
>
>
>
>4、Maven配置
>
>
>
>5、Pom.xml配置



## 第三部分：测试服务器安装部署

> 核心：把Jar包扔到测试服务器上
>
> 用到Jenkins插件：
>
> publish over ssh（安装一下） 
>
> 
>
> Maven里在Post Steps 有选项——Send files or execute commands over SSH（刚刚安装的那个插件）
>
> 
>
> 配置好测试服务器的 地址。
>
> 
>
> 核心的有一个Exec Command——（nohup是后台运行的意思） 可以写shell脚本 （处理复杂的逻辑）
>
> 比如拉取完git远程仓库代码后，可以执行 重启服务的命令
>
> cd /root/docker
> docker-compose restart uat



> SSH Publishers超时机制
>
> 高级里可以配置Exec Timeout
>
> *输出命令时一定注意不要让窗口卡住，等待输入，可以通过写入来控制







> Pre Steps——前置动作
>
> 跟Post Steps类似，可以做杀死进程的操作。







> 配webhooks可以设置回调钩子
>
> 
>
> ==就可以自动重新生成Jenkins项目了== 不用手动触发
>
> 注意：在Gitlab服务器和Jenkins服务器上都需要配置，一个设置token一个发送请求。
>
> 让jenkins对外公布一个api







### 定时任务——Cron表达式

> 除此之外，还可以设置每天构建
>
> 构建触发器——Build periodically
>
> https://crontab.guru



### Poll SCM触发器

> **Poll SCM**" 是 Jenkins 中的一种触发构建的方式，指的是通过 **轮询** 源代码管理（SCM，Source Code Management）系统来检查是否有新的提交（commit），如果检测到有新的提交，就触发 Jenkins 构建任务。
>
> 在 Jenkins 中，**SCM** 通常指的是代码仓库，如 Git、Subversion 等，**Poll SCM** 就是定期检查这些代码仓库是否有新的更改（例如新的 commit 或 push）。如果发现新提交，Jenkins 会自动触发一个新的构建过程。

## Docker部署

》
