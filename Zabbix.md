# Zabbix

> 监控有：Zabbix、Grafana、Prometheus(对docker的监控)
>
> 为什么要监控？
>
> 答：监视主机，架构状态，网站扩容，随着用户的增多，服务随时可能会被oom（Out of memory）
>
> 过去的监控：通过脚本
>
> 现在的监控：Zabbix（数据收集及展示）+Grafana（数据展示更高端）
>
> ​						OpenFalcon（小米）
>
> ​						Prometheus（普罗米修斯）

Zabbix Agent：

> 将数据提供或发送回Zabbix服务器

Zabbix Proxy:

> 如果agent和server距离太远，则可以在server端附近部署一台proxy，代替server来监控远处的主机

SNMP Agent：

> SNMP：简单网络管理协议——应用层协议
>
> udp:161端口



## 1、Linux系统检查各种命令

| 项目          | 对应检查命令                                                 |
| ------------- | ------------------------------------------------------------ |
| 网站/业务/api | **curl/wgt**                                                 |
| 服务          | **systemctl/**service/chkconfig(c6)                          |
| 进程          | **ps/pstree**/pgrep/pidstat/top/htop                         |
| CPU           | **top/htop/**vmstat/mpstat/lscpu/cpuinfo/w/uptime/sar        |
| 内存          | **top/free/ps/iotop(swap)**/vmstat/mpstat/sar/hcache(buffer+cache) |
| 磁盘          | **iotop**/iostat/sar    #磁盘测试命令 dd,fio                 |
| 网络          | **iftop(整体带宽使用情况)/nethogs(精确到进程)** /nstat/ifstat/mtr/sar/ip/route |
| 硬件          | Megacli(监控raid)/ipmitool(温度，CPU风扇转速)/lm_sensors(温度) |

## 2、Zabbix监控架构

> 一条：选LTS！(Long time support)

> C/S架构：
>
> Agent客户端获取数据——发送给Zabbix Server服务端(C语言)——数据会被存放在数据库——Zabbix Web页面展示数据
>
> 服务端需要啥数据跟客户端说，客户端发给服务端，服务端收到数据放到数据库中，数据被展示到web前端（nginx—php架构），LAMP/LNMP

​		LAMP 是一个缩写，它指一组通常一起使用来运行动态网站或者服务器的自由软件：

- Linux，操作系统；

- [Apache](http://www.oschina.net/p/apache+http+server)，网页服务器；

- [MySQL](http://www.oschina.net/p/mysql)，数据库管理系统（或者数据库服务器）；

- [PHP](http://www.oschina.net/p/php) 和有時 [Perl](http://www.oschina.net/p/perl) 或 [Python](http://www.oschina.net/p/python)，脚本语言。、

  

  LNMP：指Linux，N指Nginx，M一般指MySQL，也可以指MariaDB，P一般指PHP

你想监控谁，就在要监控的那台设备上安装Zabbix客户端



小容量——服务端、数据库、界面展示 放在一起

分布式其实就是将这些在一起的东西拆分开

> 系统：
>
> Centos7.X 仅支持客户端安装，不支持服务端安装
>
> 所以Zabbix客户端yum安装，服务端编译安装



## 3、部署Zabbix服务端流程

> :one: 部署ngnix+php环境并测试
>
> :two:部署数据库maridb 10.5及以上
>
> :three:编译安装zabbix-server服务端及后续配置
>
> :four:部署前端代码进行访问
>
> :five:web访问