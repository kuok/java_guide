## MySQL

---

### 安装

1. 创建文件夹，并进入文件夹
   
   ```bash
   mkdir /app/mysql && cd /app/mysql
   ```
   
   ```bash
   yum -y update
   ```

2. 使用wget下载mysql yum源
   
   ```bash
   wget https://dev.mysql.com/get/mysql80-community-release-el7-3.noarch.rpm
   ```

3. 添加 mysql yum 源
   
   ```bash
   sudo yum localinstall mysql80-community-release-el7-3.noarch.rpm -y
   ```

4. 安装 yum 工具 yum-utils :
   
   ```bash
   sudo yum install -y yum-utils
   ```

5. 查看可用的MySQL
   
   ```bash
   yum repolist enabled | grep "mysql.*-community.*"
   ```

6. 查看所有的 MySQL 版本
   
   ```bash
   yum repolist all | grep mysql
   ```

7. 使用指定版本MySQL(这里跳过，直接安装MySQL8.0)  
   假如我想使用MySQL5.7，那么我就需要先关闭MySQL8.0
   
   ```bash
   sudo yum-config-manager --disable mysql80-community
   ```
   
   开启MySQL5.7
   
   ```bash
   sudo yum-config-manager --enable mysql57-community
   ```

8. 查看当前启用的MySQL版本
   
   ```bash
   yum repolist enabled | grep mysql
   ```

9. 安装MySQL
   
   ```bash
   sudo yum install -y mysql-community-server
   ```
   
   可能出现问题1,改为如下命令
   
    ```bash
    sudo yum install -y mysql-community-server --nogpgcheck
    ```

10. 启动MySQL
    
    ```bash
    sudo service mysqld start
    ```

11. 查看MySQL服务状态
    
    ```bash
    sudo service mysqld status
    ```
    
    显示启动成功
    
    ```bash
    Redirecting to /bin/systemctl status mysqld.service
    ● mysqld.service - MySQL Server
       Loaded: loaded (/usr/lib/systemd/system/mysqld.service; enabled; vendor preset: disabled)
       Active: active (running) since Tue 2025-03-11 18:01:29 CST; 40s ago
         Docs: man:mysqld(8)
               http://dev.mysql.com/doc/refman/en/using-systemd.html
      Process: 21449 ExecStartPre=/usr/bin/mysqld_pre_systemd (code=exited, status=0/SUCCESS)
     Main PID: 21550 (mysqld)
       Status: "Server is operational"
       CGroup: /system.slice/mysqld.service
               └─21550 /usr/sbin/mysqld
    
    Mar 11 18:01:22 VM-8-17-centos systemd[1]: Starting MySQL Server...
    Mar 11 18:01:29 VM-8-17-centos systemd[1]: Started MySQL Server.
    ```

12. 设置 MySQL开机启动
    
    ```bash
    systemctl enable mysqld
    ```

13. 初始化MySQL
    查看初始化密码
    
    ```bash
    sudo grep 'temporary password' /var/log/mysqld.log
    ```
    
    显示
    
    ```bash
    2025-03-11T10:01:24.744631Z 6 [Note] [MY-010454] [Server] A temporary password is generated for root@localhost: .,.LE5psXu6E
    ```
    
    使用初始密码登录`rX/Ro,XIo8dm`
    
    ```bash
    mysql -u root -p
    ```
    
    重新设置初始密码
    
    ```sql
    ALTER USER 'root'@'localhost' IDENTIFIED BY '我的密码';
    ```

14. 设置MySQL远程连接  
    mysql 数据库中user 表中的特定用户(root) 的host 的属性值为localhost.  
    进入mysql库中进行特定用户的host 修改
    
    ```sql
    use mysql;
    update user set host='%' where user='root';
    ```
    
    或者：赋予root用户远程访问权限。若允许任意IP地址访问
    
    ```sql
    GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' WITH GRANT OPTION;
    ```
    
    如果你想只为特定IP地址授权远程访问
    
    ```sql
    GRANT ALL PRIVILEGES ON *.* TO 'root'@'特定IP' WITH GRANT OPTION;
    ```
    
    刷新权限
    
    ```sql
    FLUSH PRIVILEGES;
    ```
    
    退出MySQL
    
    ```sql
    quit
    ```
    
    开放防火墙
    
    ```bash
    firewall-cmd --zone=public --add-port=3306/tcp --permanent
    systemctl restart firewalld
    ```
    
    测试连接成功。

---

#### 问题

1. GPG Keys are configured as: file:///etc/pki/rpm-gpg/RPM-GPG-KEY-mysql  
   遇到GPG错误通常是因为软件包的签名验证失败。这可能是由于多种原因，包括密钥过期、密钥未安装或配置错误。  
   安装时禁用GPG检查
   
   ```bash
   sudo yum install -y mysql-community-server --nogpgcheck
   ```

2. 5.7版本赋予远程连接权限语句不同
   
   ```sql
   GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '你的密码' WITH GRANT OPTION;
   ```

---