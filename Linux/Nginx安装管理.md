# Nginx

---

## Nginx Proxy Manager

官网：<https://nginxproxymanager.com/setup/>

1. 创建文件夹，并进入文件夹
   
   ```bash
   mkdir /app/nginxproxymanager && cd /app/nginxproxymanager
   ```

2. 按照官网写配置文件

   ```bash
   vim docker-compose.yml
   ```
   
   ```yaml
   services:
     app:
       image: 'jc21/nginx-proxy-manager:latest'
       restart: unless-stopped
       ports:
         # These ports are in format <host-port>:<container-port>
         - '80:80' # Public HTTP Port
         - '443:443' # Public HTTPS Port
         - '81:81' # Admin Web Port
         # Add any other Stream port you want to expose
         # - '21:21' # FTP
   
       environment:
         # Uncomment this if you want to change the location of
         # the SQLite DB file within the container
         # DB_SQLITE_FILE: "/app/nginxproxymanager/data/database.sqlite"
   
         # Uncomment this if IPv6 is not enabled on your host
         DISABLE_IPV6: 'false'
   
       volumes:
         - ./data:/data
         - ./letsencrypt:/etc/letsencrypt
   ```

3. 启动
   
   ```bash
   docker compose up -d
   ```

4. 登陆81端口管理页面  
   默认登录的用户名：
   
   ```text
   admin@example.com
   ```
   
   密码：
   
   ```text
   changeme
   ```
   
   腾讯云可能会报错：`502 bad gateway`.  
   查看日志
   
    ```bash
    docker compose logs app
    ```
   
   发现卡在
   
    ```text
    Fetching https://ip-ranges.amazonaws.com/ip-ranges.json
    ```
   
   大概就是npm启动的时候会去调用AWS接口拉一个什么ip-ranges数据库，然后有些云服务商屏蔽了AWS接口（比如腾讯云）。然后启动脚本就卡在这里，启动就失败了，然后相当于npm的管理后台就没启动起来。  
   需要进入容器内部把这个行为屏蔽掉。
   
    ```bash
    docker exec -it nginxproxymanager-app-1 bash
    ```
   
    ```bash
    vim index.js
    ```
   
   发现没vim，先安装
   
    ```bash
    apt update
    ```
   
    ```bash
    apt-get install vim
    ```
   
    ```bash
    vim index.js
    ```
   
   把`/app/index.js`里面的 `.then(internalIpRanges.fetch)` 给注释掉。
   
    ```js
    .then(() => {
                return apiValidator.loadSchemas;
    })
    //.then(internalIpRanges.fetch)
    .then(() => {
                internalCertificate.initTimer();
    ```
   
   退出docker容器
   
    ```bash
    exit
    ```
   
   重启npm
   
    ```bash
    docker restart nginxproxymanager-app-1
    ```
   
   再进入81端口，输入默认用户和密码进入。

---

## Nginx

现在Nginx有可视化工具Nginx Proxy Manager了，自带面板，操作极其简单，非常适合配合 docker 搭建的应用使用。后台还可以一键申请 SSL 证书，并且会自动续期，方便省心。  
所以不再装Nginx了，两者有冲突。

1. 直接yum安装
   
   ```bash
   yum install nginx
   ```

2. 设置开机启动
   
   ```bash
   systemctl enable nginx
   ```

3. 其他
   
   ```bash
   systemctl status nginx
   systemctl start nginx
   systemctl stop nginx
   systemctl restart nginx
   ```

4. 打开防火墙
   
   ```bash
   firewall-cmd --zone=public --add-port=80/tcp --permanent
   firewall-cmd --zone=public --add-port=443/tcp --permanent
   firewall-cmd --reload
   ```

5. 其他  
   查看Nginx配置文件
   
    ```bash
    nginx -t
    ```

---