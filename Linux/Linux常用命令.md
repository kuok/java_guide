# Linux常用命令

---

## 基础

1. 显示隐藏文件夹

    ```bash
    ll -a
    ```

2. 查看IP

    ```bash
    curl ip.sb
    ```

---

## 检查架构
>
> 部分软件本地安装包分为arm和x64,如java。 需根据不同架构下载不同版本软件。

* ARM64-<https://download.oracle.com/java/21/latest/jdk-21_linux-aarch64_bin.tar.gz>
* x64-<https://download.oracle.com/java/21/latest/jdk-21_linux-x64_bin.tar.gz>

---

### 查询架构

直接查看架构

```bash
[root@VM-8-17-centos ~]# arch
x86_64
```

```bash
arch
```

---

### 系统信息

```bash
[root@VM-8-17-centos ~]# uname -m
x86_64
```

```bash
uname -m
```

完整系统信息

```bash
[root@VM-8-17-centos ~]# uname -a
Linux VM-8-17-centos 3.10.0-1160.119.1.el7.x86_64 #1 SMP Tue Jun 4 14:43:51 UTC 2024 x86_64 x86_64 x86_64 GNU/Linux
```

```bash
uname -a
```

---

### 系统位

查看是32位还是64位

```bash
[root@VM-8-17-centos ~]# getconf LONG_BIT
64
```

```bash
getconf LONG_BIT
```

---

## 防火墙

centOS默认开启了防火墙，安装软件后需要将对应端口开放才能外网访问。

---

### 与docker的冲突

安装docker后，添加端口会报错。因为安装docker后使用的zone是docker，而不是默认的public。之后的命令中需要把zone改为docker。

```bash
You're performing an operation over default zone ('public'),
but your connections/interfaces are in zone 'docker' (see --get-active-zones)
```

```bash
firewall-cmd --get-active-zones
```

```bash
firewall-cmd --list-port --zone=docker
```

安装docker后开放常用端口

```bash
firewall-cmd --zone=docker --add-port=80/tcp --permanent
firewall-cmd --zone=docker --add-port=443/tcp --permanent
firewall-cmd --zone=docker --add-port=3306/tcp --permanent    #mysql
firewall-cmd --zone=docker --add-port=5432/tcp --permanent    #postgresql
firewall-cmd --zone=docker --add-port=5672/tcp --permanent    #rabbitmq
firewall-cmd --zone=docker --add-port=15672/tcp --permanent   #rabbitmq
firewall-cmd --zone=docker --add-port=8090/tcp --permanent    #halo
firewall-cmd --zone=docker --add-port=6379/tcp --permanent    #redis
firewall-cmd --zone=docker --add-port=8848/tcp --permanent    #nacos
firewall-cmd --zone=docker --add-port=9848/tcp --permanent    #nacos
firewall-cmd --zone=docker --add-port=9849/tcp --permanent    #nacos
firewall-cmd --zone=docker --add-port=7848/tcp --permanent    #nacos
firewall-cmd --zone=docker --add-port=3000/tcp --permanent    #3000数据库
firewall-cmd --reload
```

---

### 查看防火墙状态

```bash
systemctl status firewalld
```

```bash
[root@VM-8-17-centos ~]# systemctl status firewalld
● firewalld.service - firewalld - dynamic firewall daemon
   Loaded: loaded (/usr/lib/systemd/system/firewalld.service; disabled; vendor preset: enabled)
   Active: inactive (dead)
     Docs: man:firewalld(1)
```

或

```bash
firewall-cmd --state
```

```bash
[root@VM-8-17-centos ~]# firewall-cmd --state
not running
```

---

### 端口

查看是否开启某端口

```bash
firewall-cmd --query-port=80/tcp
```

查看已经开放的端口列表

```bash
firewall-cmd --list-port
```

开启某端口  
> --permanent永久生效，没有此参数重启后失效  
> 可以是一个端口范围，如1000-2000/tcp

```bash
firewall-cmd --zone=public --add-port=80/tcp --permanent
```

移除某端口

```bash
firewall-cmd --zone=public --remove-port=80/tcp --permanent
# 
firewall-cmd --permanent --remove-port=80/tcp
```

---

### 开启，关闭，重启

```bash
systemctl start firewalld
```

```bash
systemctl stop firewalld
```

```bash
systemctl restart firewalld
```

---

### 重新加载

修改后一般要重新加载防火墙生效。

```bash
firewall-cmd --reload
```

断开所有连接，重启服务加载

```bash
firewall-cmd --complete-reload
```

---

### 开机重启

查看是否开机重启

```bash
systemctl is-enabled firewalld
```

开机重启

```bash
systemctl enable firewalld
```

关闭开机重启

```bash
systemctl disable firewalld
```

---

### 拒绝所有包

查看是否拒绝所有

```bash
firewall-cmd --query-panic
```

拒绝所有包

```bash
firewall-cmd --panic-on
```

取消拒绝状态

```bash
firewall-cmd --panic-off
```

---

### zone概念

“zone”概念，即事先准备好的若干套防火墙策略模板，一般默认使用公共（public）区域。  
所有区域（zone）如下，一般默认使用公共区域（public），由 firewalld 提供的区域按照从不信任到信任的顺序排序。

```bash
firewall-cmd --get-zones
```

```bash
[root@VM-8-17-centos ~]# firewall-cmd --get-zones
block dmz drop external home internal public trusted work
```

* public(公共) —— [默认]公网访问，不受任何限制。
* work(工作) —— 用于工作区。基本信任的网络，仅仅接收经过选择的连接。
* home(家庭) —— 用于家庭网络。基本信任的网络，仅仅接收经过选择的连接。
* trusted(信任) —— 接收的外部网络连接是可信任、可接受的。
* block(限制) —— 任何接收的网络连接都被IPv4的icmp-host-prohibited信息和IPv6的icmp6-adm-prohibited信息所拒绝。
* dmz(隔离区) —— 英文"demilitarized zone"的缩写，此区域内可公开访问，它是非安全系统与安全系统之间的缓冲区。
* drop(丢弃) —— 任何接收的网络数据包都被丢弃，没有任何回复。仅能有发送出去的网络连接。
* external(外部) —— 允许指定的外部网络进入连接，特别是为路由器启用了伪装功能的外部网。
* internal(内部) —— 内部访问。只限于本地访问，其他不能访问。

查看默认zone

```bash
firewall-cmd --get-default-zone
```

```bash
[root@VM-8-17-centos ~]# firewall-cmd --get-default-zone
public
```

修改默认zone

```bash
firewall-cmd --set-default-zone=work
```

---

## 查看登陆信息

---

### 登陆信息

使用`cat`命令结合`/var/log/secure`文件来查看系统的登录信息。该文件记录了通过SSH，Telnet，FTP等方式登录系统的相关信息。

```bash
cat /var/log/secure
```

查看试图暴力破解我的主机的IP

```bash
grep "Failed password for invalid user" /var/log/secure | awk '{print $13}' | sort | uniq -c | sort -nr
```

将暴力破解的IP加入黑名单

```bash
cat /var/log/secure |  grep "Failed password for invalid user" | awk '{print $13}' | sort | uniq -c | sort -n | tail -10 |awk '{print "sshd:"$2":deny"}' >> /etc/hosts.allow
```

```bash
cat /etc/hosts.allow
```

`lastb`命令用于显示最后一次登录失败的记录。它会列出登录失败的用户ID、登录IP地址、登录时间和登录失败的原因。

* “-n “：只显示最近的个登录失败记录。
* “-f “：指定一个包含登录失败信息的文件，而不是默认的/var/log/btmp文件。
* “-w”：以宽格式显示输出，包括终端和源字段。
* “-R”：以反向顺序显示输出，最新的记录在最上面。

```bash
lastb
```

`last`命令显示成功登录的信息。  
last - 用于显示用户最近登录信息。单独执行last命令，它会读取/var/log/wtmp的文件，并把该给文件的内容记录的登入系统的用户名单全部显示出来。

```bash
last
```

---

### 当前登录用户

who - 是显示目前登录系统的用户信息。执行who命令可得知目前有那些用户登入系统，单独执行who命令会列出登入帐号，使用的终端机，登入时间以及从何处登入。

* -b, --boot 最近一次系统启动的时间
* -m：打印当前连接客户端使用的用户以及连接的客户端的IP，此参数的效果和指定"am i"字符串相同；

```bash
who
```

w - 用于显示已经登陆系统的用户列表，并显示用户正在执行的指令。执行这个命令可得知目前登入系统的用户有那些人，以及他们正在执行的程序。单独执行w命令会显示所有的用户，您也可指定用户名称，仅显示某位用户的相关信息。

```bash
w
```

---

### 用新用户代替root用户登录

创建新用户

```bash
useradd Alex
```

创建密码

```bash
passwd Alex
```

修改`/etc/ssh/sshd_config`文件中的`PermitRootLogin`配置项设置为`NO`，表示禁止root登录。  
`MaxAuthTries` 设置为10，表示限制失败次数，失败次数对所有用户起作用。

```bash
cd /etc/ssh/ && cp sshd_config sshd_config.bakup
```

```bash
vi sshd_config
```

重启SSHD服务

```bash
systemctl restart sshd.service
```

此时用root用户登录会报错`Access denied`  
需使用新建用户登录，再切换至root用户。

```bash
su - root
```

不想用时把配置文件改回来，再删除用户。

```bash
userdel -r Alex
```
