# PostgreSQL

---

## 安装

官网有详细脚本<https://www.postgresql.org/download/linux/redhat/>

1. 创建文件夹

   ```bash
   mkdir /app/postgresql && cd /app/postgresql
   ```

   ```bash
   yum -y update
   ```

2. Install the repository RPM:

   ```bash
   sudo yum install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm
   ```

3. Install PostgreSQL:

   ```bash
   sudo yum install -y postgresql15-server
   ```

4. Optionally initialize the database and enable automatic start:

   ```bash
   sudo /usr/pgsql-15/bin/postgresql-15-setup initdb
   sudo systemctl enable postgresql-15
   sudo systemctl start postgresql-15
   ```

5. 默认密码  
   默认情况下，PostgreSQL数据库的超级管理员用户名为“postgres”。超级管理员是具有最高权限的用户，可以执行所有数据库操作。  
   在安装PostgreSQL数据库时，您需要为超级管理员设置一个初始密码。如果您没有更改默认设置，那么默认密码是空的，即没有密码。  
   以postgres用户登录

   ```bash
   su - postgres
   ```

   登录PostgreSQL

   ```bash
   psql
   ```

   修改密码

   ```sql
   ALTER USER postgres WITH PASSWORD '新密码';
   ```

   退出PostgreSQL

   ```sql
   \q
   ```

   退出postgres用户

   ```bash
   exit
   ```

6. 授权远程登陆  
   查找主配置文件：postgresql.conf

   ```bash
   find / -name postgresql.conf
   ```

   ```bash
   [root@VM-8-17-centos postgresql]# find / -name postgresql.conf
   /var/lib/pgsql/15/data/postgresql.conf
   ```

    ```bash
    vim /var/lib/pgsql/15/data/postgresql.conf
    ```

   设置关键配置项：

   ```properties
   listen_addresses = '*'  # 允许所有IP连接
   port = 5432            # 默认端口
   ```

   查找客户端认证文件：pg_hba.conf

   ```bash
   find / -name pg_hba.conf
   ```

   ```bash
   vim /var/lib/pgsql/15/data/pg_hba.conf
   ```

   ```bash
   [root@VM-8-17-centos postgresql]# find / -name pg_hba.conf
   /var/lib/pgsql/15/data/pg_hba.conf
   ```

   修改认证方式，允许所有IP通过密码访问

   ```conf
   # IPv4 local connections:
   host    all             all             0.0.0.0/0               scram-sha-256
   ```

   开放防火墙

   ```bash
   firewall-cmd --zone=public --add-port=5432/tcp --permanent
   systemctl restart firewalld
   ```

   重启服务生效

   ```bash
   sudo systemctl restart postgresql-15
   ```

7. 其他

   ```bash
   systemctl status postgresql-15
   systemctl stop postgresql-15
   systemctl restart postgresql-15
   ```

   客户端测试连接成功。

---