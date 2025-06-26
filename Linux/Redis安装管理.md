# Redis

---

## 安装
1. 创建文件夹
   
   ```bash
   mkdir /app/redis && cd /app/redis
   ```

2. 安装gcc依赖
   
   ```bash
   gcc -v
   ```
   
   ```bash
   yum install -y gcc
   ```

3. 下载redis安装包
   
   ```bash
   wget https://download.redis.io/releases/redis-7.4.2.tar.gz
   ```
   
   ```bash
   tar -zxvf redis-7.4.2.tar.gz
   ```

4. 安装
   
   ```bash
   cd redis-7.4.2
   ```
   
   ```bash
   make && make install
   ```

5. 启动
   开放防火墙
   
   ```bash
   firewall-cmd --zone=public --add-port=6379/tcp --permanent
   systemctl restart firewalld
   ```
   
   ```bash
   /usr/local/bin/redis-server
   ```
   
   此时是以前台方式启动，先退出。
6. 修改配置  
   先备份
   
   ```bash
   cp redis.conf redis.conf.bck
   ```
   
   创建文件夹保存持久化文件以及日志
   ```bash
   mkdir rdb
   ```
   
   修改配置
   ```bash
   vim redis.conf
   ```
   
   搜索功能：/+要搜索的内容，n：下一个搜索结果，N：上一个搜索结果
   
   ```text
   # 将bind 127.0.0.1 -::1注释掉，在下边增加：bind 0.0.0.0
   
   # 将protected-mode yes修改为：protected-mode no
   
   # daemonize 的值从 no 修改成 yes（Redis服务默认是前台运行，需要修改为后台运行）
       daemonize no ---> daemonize yes

   # 设置redis记录日志，默认不记录日志（redis.log为文件名）
       logfile " " ---> logfile "redis.log"
   
   # requirepass foobared注释去掉并在后加上密码（注意中间加个空格）
       requirepass foobared ---> requirepass 密码
   
   # 将日志以及持久化文件保存到rdb文件夹
       dir ./ ---> dir /app/redis/redis-7.4.2/rdb/

   ```
   
   重新以配置文件方式启动，这次是后台运行，redis.conf不必写全路径。
   
   ```bash
   /usr/local/bin/redis-server redis.conf
   ```
   
   查看redis进程
   
   ```bash
   ps -ef | grep redis
   ```
   
   ```bash
   [root@VM-8-17-centos redis-7.2.0]# ps -ef | grep redis
   root      5755     1  0 01:10 ?        00:00:00 /usr/local/bin/redis-server 127.0.0.1:6379
   root      6313   399  0 01:11 pts/0    00:00:00 grep --color=auto redis
   ```
   
   关闭redis
   
   ```bash
   kill -9 5755
   ```

7. 开机自启动  
   新建文件
   
   ```bash
   vim /etc/systemd/system/redis.service
   ```
   
   `/etc/systemd/system` 和 `/lib/systemd/system` 的区别  
   `/usr/lib/systemd/system`  
   这个目录用于存放由系统软件包安装的服务单元文件。这些文件是由软件包维护人员提供的，随着软件包的升级，可能会被自动更新或替换。
   不推荐用户直接编辑这个目录下的服务文件，因为更改可能在软件包升级期间丢失。  
   `/etc/systemd/system/`  
   这个目录用于存放用户自定义的服务单元文件。这些文件是由系统管理员根据自身需求定制的服务配置，不会受到软件包升级的影响。
   建议在此目录下存放自定义的服务单元文件，以便保留定制化配置。  
   输入以下信息
   
   ```properties
   [Unit]
   Description=redis-server
   After=network.target
   
   [Service]
   Type=forking
   # ExecStart需要按照实际情况修改成自己的地址
   ExecStart=/usr/local/bin/redis-server /app/redis/redis-7.4.2/redis.conf
   PrivateTmp=true
   
   [Install]
   WantedBy=multi-user.target
   ```
   
   重新加载文件
   
   ```bash
   systemctl daemon-reload
   ```
   
   启动redis
   
   ```bash
   systemctl start redis
   ```
   
   开机自启
   
   ```bash
   systemctl enable redis
   ```
   
   其他
   
   ```bash
   systemctl status redis
   systemctl stop redis
   systemctl restart redis
   redis-server --version
   redis-cli  --version
   ```
   
   客户端连接成功。

---