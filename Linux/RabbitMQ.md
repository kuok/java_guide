## RabbitMQ

---

### 安装

1. 创建文件夹
   
   ```bash
   mkdir /app/rabbitmq && cd /app/rabbitmq
   ```
   
   ```bash
   yum -y update
   ```

2. 安装erlang
   
   ```bash
   curl -s https://packagecloud.io/install/repositories/rabbitmq/erlang/script.rpm.sh | sudo bash
   ```
   
   ```bash
   sudo yum install erlang-23.3.4.11-1.el7.x86_64
   ```
   
   ```bash
   erl -v
   ```

3. 安装RabbitMQ
   
   ```bash
   curl -s https://packagecloud.io/install/repositories/rabbitmq/rabbitmq-server/script.rpm.sh | sudo bash
   ```
   
   ```bash
   sudo yum install rabbitmq-server-3.8.16-1.el7.noarch
   ```
   
   安装管理界面
   
   ```bash
   rabbitmq-plugins enable rabbitmq_management
   ```
   
   开放防火墙
   
   ```bash
   firewall-cmd --zone=public --add-port=5672/tcp --permanent
   firewall-cmd --zone=public --add-port=15672/tcp --permanent
   systemctl restart firewalld
   ```

4. 启动  
   启动服务
   
   ```bash
   systemctl start rabbitmq-server
   ```
   
   开启开机启动
   
   ```bash
   systemctl enable rabbitmq-server
   ```
   
   其他
   
   ```bash
   systemctl status rabbitmq-server
   systemctl stop rabbitmq-server
   systemctl restart rabbitmq-server
   ```

5. 添加用户  
   安装后会自动生成guest账号，但是guest账号只能在本机登陆。
   
   ```bash
   rabbitmqctl add_user admin 密码
   ```
   
   设置管理员权限
   
   ```bash
   rabbitmqctl set_user_tags admin administrator
   ```
   
   ```bash
   rabbitmqctl set_permissions -p / admin "." "." ".*"
   ```
   
   重启
   
   ```bash
   systemctl restart rabbitmq-server
   ```
   
   访问15672端口，登录成功。
6. 其他操作  
   修改密码
   
   ```bash
   rabbitmqctl  change_password  用户名  新密码
   ```
   
   查看用户列表
   
   ```bash
   rabbitmqctl list_users
   ```
   
   删除用户
   
   ```bash
   rabbitmqctl delete_user Username
   ```

---
