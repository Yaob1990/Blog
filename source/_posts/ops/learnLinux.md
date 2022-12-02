---
title: 基础Linux学习
categories:
  - ops
img: ../../coverImages/linux.jpg
tags:
  - Linux
  - 运维
date: 2020-03-29 20:03:48
---



现在的服务都是部署在宝塔的基础上，基本是0技术要求，学习Linux，准备在下半年的服务换成原生的linux，不再依赖第三方控制面板。以下学习都是以`Centos 7`为基础。

### 常用命令

1. `init 0`: 关机
2. `init 6`: 重启
3. `ip addr`: 获取 ip 地址
4. `pwd`: 显示当前路径
5. `clear`或者`ctrl + l`:  清屏

6. `history` ：查看历史命令
7. `! + num`：使用历史命令

### 用户操作

1. 添加用户：`useradd zhangsan`
2. 设置密码：`passwd zhangsan`
3. 删除用户：`userdel -rf zhangsan`

### Linux 常见目录结构

**root 目录：**linxu 超级权限root 的主目录。*

**home 目录：**系统默认的用户主目录，如果添加用户是不指定用户的主目录，默认在/home
下创建与用户同名的文件夹。*

**bin 目录：**存放系统所需要的重要命令，比如文件或目录操作的命令ls、cp、mkdir 等，另外
/usr/bin 也放了一些系统命令。这些命令对应着文件都是可以执行的。*

**sbin 目录：**存放只有root 超级管理员才能执行的程序*

**boot 目录：**存放着linux 启动时内核及引导系统程序所需要的核心文件，内核文件和grub
系统引导管理器都位于此目录。

**dev 目录：**存放这linux 系统下的设备文件，如光驱等。

**etc 目录：**存放系统的配置文件，作为一些软件启动时默认配置文件读取的目录，如/etc/fstal
存放系统分析信息。*

**mnt 目录：** 临时文件挂载目录、也可以说是测试目录

**opt 目录：** 第三方软件存放目录*

**media 目录：**即插即用型设备挂载点，光盘默认挂载点，通常光盘挂载于/mnt/cdrom 下。

**tmp 目录：**临时文件夹。*

**usr 目录：**应用程序存放目录，安装linux 软件包是默认安装到/usr/local 目录下。*

**var 目录：**目录经常变动，/var/log 存放系统日志，/var/log 存放系统库文件。*

### Linux 文件管理

1. 创建文件： 

   `touch file`

2. 删除文件：

   `rm -rf file`

   **-r:** 递归的删除目录下面文件以及子目录下文

   **-f:** 强制删除，忽略不存在的文件，不给出提示

3. 移动文件/修改文件名

   `mv file1 file2`

4. 查看文件内容

   `cat file`

5. 复制文件

   `cp file1 file2`

6. 批量创建文件

   ```shell
   touch file{1..10}
   touch -rd file{1..10}
   ```

7. 编辑文件

   `vi file`

8. 管道方式查看文件

   ```shell
   cat file | head -3 // 查看前3行
   cat file | tail -3 // 查看后3行
   ```

9. 文件查找

   方法1：

   `find 目录 -name 文件名`

   方法2（更快）：

   ```bash
   // 建立数据库
   updatedb
   // 数据库中查找
   locate aa.txt 
   ```

### Linux 目录管理

1. 创建目录

   `mkdir dir1 dir2 dir3`

2. 重命名/移动目录

   `mv dir 1 dir2`

3. 删除目录

   `rm -rf dir1`

4. 复制目录

   `cp -rf dir1 dir2`

5. 显示目录结构

   `tree(需要安装)`

### Linux 文件类型

1. ​	查看文件类型`ll`

2. ​    常见文件类型

   ```shell
   -rw-r—r— "-“开头的都是普通文件;
   
   drw-r—r— "d"开头的是目录文件;
   
   lrw-r—r— "l"开头的文件都是软链接文件;
   ```

### Linux 打包压缩

1. Zip

   压缩：

   `zip -r public.zip public`

   -r	表示将指定的目录下的所有子目录以及文件一起处理

   解压：

   `unzip public.zip -d dir`

   查看压缩包内容

   `unzip -l public.zip`

2. gz 压缩包

   ```shell
   tar czvf public.tar.gz public // 压缩
   tar xzvf public.atr.gz // 解压
   tar tf public.atr.gz // 查看
   tar cvf public.tar public // 仅打包，不压缩
   tar xvf public.tar // 解压无gz的包
   ```



### Linux 别名管理

```
// 添加别名
alias aa='cat /etc/httpd/conf/httpd.conf'
// 删除别名
unalias aa
// 查看别名
alias
```



### 内存、cup 管理

`top`

1. 第一行

   ```shell
   top - 15:31:47 up 9:30, 3 users, load average: 0.00, 0.02, 0.05
   ```

   依次对应：系统当前时间 up 系统到目前为止i 运行的时间， 当前登陆系统的用户数量， load average 后
   面的三个数字分别表示距离现在一分钟，五分钟，十五分钟的负载情况。



2. 第二行

   ```
   Tasks: 133 total, 1 running, 132 sleeping, 0 stopped, 0 zombie
   ```

   依次对应：tasks 表示任务（进程），133 total 则表示现在有133 个进程，其中处于运行中
   的有1 个，132 个在休眠（挂起），stopped 状态即停止的进程数为0，zombie 状态即僵尸
   的进程数为0 个。

3. top 命令的第三行，cpu 状态：

   ```shell
   %Cpu(s): 0.2 us, 0.4 sy, 0.0 ni, 99.3 id, 0.0 wa, 0.0 hi, 0.1 si, 0.0 st
   ```

   **只看空闲就可以了**：cpu 空闲率为99.3%

   依次对应：
   **us**:user 用户空间占用cpu 的百分比
   **sy**:system 内核空间占用cpu 的百分比
   **ni**:niced 改变过优先级的进程占用cpu 的百分比
   **id**:空闲cpu 百分比
   **wa**:IO wait IO 等待占用cpu 的百分比
   **hi**:Hardware IRQ 硬中断占用cpu 的百分比
   **si**:software 软中断占用cpu 的百分比
   **st**:被hypervisor 偷去的时间

4. top 命令的第四行，内存状态：

   ```
   KiB Mem : 2897496 total, 1995628 free, 191852 used, 710016 buff/cache
   ```

   总内存:2.76g 空闲：1995628/1024/1024=1.9g 已经使用0.18g 缓存区内存0.67g
   缓冲区是从主内存中特地预留出的内存，用来存放特定的一些信息，例如从磁盘中取得的文件表，程序正
   在读取的内容等等

5. top 命令第七行，各进程的监控：

   ```
   PID USER PR NI VIRT RES SHR S %CPU %MEM TIME+ COMMAND
   ```

   依次对应：
   **PID** — 进程id
   **USER** — 进程所有者
   **PR** — 进程优先级
   **NI** — nice 值。负值表示高优先级，正值表示低优先级
   **VIRT** — 进程使用的虚拟内存总量，单位kb。VIRT=SWAP+RES

   **RES** — 进程使用的、未被换出的物理内存大小，单位kb。RES=CODE+DATA
   **SHR** — 共享内存大小，单位kb
   **S** — 进程状态。D=不可中断的睡眠状态R=运行S=睡眠T=跟踪/停止Z=僵尸进程
   **%CPU** — 上次更新到现在的CPU 时间占用百分比
   **%MEM** — 进程使用的物理内存百分比
   **TIME+** — 进程使用的CPU 时间总计，单位1/100 秒
   **COMMAND** — 进程名称（命令名/命令行）

`uptime`

1. uptime

   ```
   top - 15:31:47 up 9:30, 3 users, load average: 0.00, 0.02, 0.05
   ```

   1.服务器工作时间
   2.在线用户
   3.平均负载一分钟，五分钟，十五分钟的负载情况

### 查看登录用户

1. `who`:显示当前正在系统中的所有用户名字，使用终端设备号，注册时间

2. `who am i`: 显示出当前终端上使用的用户

3. `last`:显示近期用户或终端的登录情况

   

### 进程管理

1. 查看进程

   ```shell
   pstree # 查看进程树
   pstree -ap #显示所有详细(进程、子进程、进程号)
   pstree | grep httpd
   pstree -ap | grep httpd
   ```

2. 关闭进程

   ```shell
   pkill httpd # pkill 进程的名字
   kill 2245 # 进程号
   kill -9 2245   # 强制杀死进程
   ```

### 查看端口

```shell
netstat -tunpl |grep httpd 
```

 -t 或--tcp 显示TCP 传输协议的连线状况。

 -u 或--udp 显示UDP 传输协议的连线状况。

 -n 或--numeric 直接使用IP 地址，而不通过域名服务器。

 -p 或--programs 显示正在使用Socket 的程序识别码和程序名称。

 -l 或--listening 显示监控中的服务器的Socket。

### 查看硬盘信息

```
df
df -h # 以人们易读的方式显示，总共多少g 用了多少g
df /home # 查看该文件夹所在磁盘的使用情况
```



### 使用 systemctl 管理服务

systemctl 就是service 和chkconfig 这两个命令的整合，在CentOS 7 就开始被使用了,systemctl
是系统服务管理器命令，它实际上将service 和chkconfig 这两个命令组合到一起。

```shell
systemctl start nginx # 启动服务
systemctl stop nginx # 关闭服务
systemctl restart ngonx # 重启服务
systemctl enable httpd # 设置开机自启动
systemctl disable nginx # 停止开机自启动
systemctl list-unit-files|grep enabled # 列出所有自启动服务
systemctl reload nginx # 重新加载配置
```

### Firewakkd 防火墙设置

#### 基本使用

```shell
systemctl start firewalld # 启动
systemctl stop firewalld # 关闭
systemctl status firewalled # 查看状态
systemctl disable firewalld # 开机禁用
systemctl enable firewalld # 开机启用
```

#### 配置

```shell
firewall-cmd --state # 显示状态
firewall-cmd --zone=public --list-ports # 查看所有打开的端口
firewall-cmd --reload # 更新规则
firewall-cmd --zone=public --add-port=80/tcp --permanent  # 开启80端口，–permanent 永久生效，没有此参数重启后失效
firewall-cmd --reload # 重新载入
firewall-cmd --zone=public --query-port=80/tcp # 查看端口
firewall-cmd --zone=public --remove-port=80/tcp --permanent # 删除
```

