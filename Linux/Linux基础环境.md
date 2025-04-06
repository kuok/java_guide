# Linux基础环境

---

## 原则

1. 软件安装包默认放在 `/app/软件名`，后续所有该软件的配置，日志文件均放在该文件夹下。
2. 命名全部小写。

---

## JAVA

1. 创建文件夹，并进入文件夹
   
   ```bash
   mkdir /app/java && cd /app/java
   ```

2. 上传软件包  
   ll命令查看上传成功
   
   ```bash
   [root@VM-8-17-centos java]# ll
   total 681292
   -rw-r--r-- 1 root root 169022667 Mar 11 17:15 jdk-11.0.25_linux-x64_bin.tar.gz
   -rw-r--r-- 1 root root 182840393 Mar 11 17:15 jdk-17.0.13_linux-x64_bin.tar.gz
   -rw-r--r-- 1 root root 197405999 Mar 11 17:14 jdk-21_linux-x64_bin.tar.gz
   -rw-r--r-- 1 root root 148362647 Mar 11 17:13 jdk-8u431-linux-x64.tar.gz
   ```

3. 解压软件包
   
   ```bash
   tar -zxvf jdk-21_linux-x64_bin.tar.gz
   ```

4. 配置环境变量  
   打开环境变量文件
   
   ```bash
   vim /etc/profile
   ```
   
   在末尾添加以下变量
   
   ```vim
   #set jdk envirment 
   export JAVA_HOME=/app/java/jdk-21.0.6
   export JRE_HOME=$JAVA_HOME/jre
   export CLASSPATH=$JAVA_HOME/lib:$JRE_HOME/lib:$CLASSPATH
   export PATH=$JAVA_HOME/bin:$JRE_HOME/bin:$PATH
   ```
   
   重新加载环境变量
   
   ```bash
   source /etc/profile
   ```

5. 查看java版本
   
   ```bash
   java -version
   ```
   
   显示结果如下，安装成功。
   
   ```bash
   [root@VM-8-17-centos mysql]# java -version
   java version "21.0.6" 2025-01-21 LTS
   Java(TM) SE Runtime Environment (build 21.0.6+8-LTS-188)
   Java HotSpot(TM) 64-Bit Server VM (build 21.0.6+8-LTS-188, mixed mode, sharing)
   ```

---

## dnf

yum安装dnf

   ```bash
   yum -y update
   yum install dnf
   ```

失败可能要先添加 EPEL Repo

   ```bash
   yum install epel-release
   ```

---

