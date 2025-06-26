# Halo

---

## 安装
官方文档很详细：<https://docs.halo.run/getting-started/install/jar-file>

1. 创建文件夹，然后按照官方文档继续。
   
    ```bash
    mkdir /app/halo && cd /app/halo
    ```
   下载包
   ```bash
   wget https://dl.halo.run/release/halo-2.21.0.jar -O halo.jar
   ```
   创建 Halo 配置文件`application.yaml`
   ```yaml
   server:
     # 运行端口
     port: 8090
   spring:
     # 数据库配置，支持 MySQL、MariaDB、PostgreSQL、H2 Database，具体配置方式可以参考下面的数据库配置
     r2dbc:
       url: r2dbc:pool:mysql://localhost:3306/halo
       username: user
       password: pwd
     sql:
       init:
         mode: always
         # 需要配合 r2dbc 的配置进行改动
         platform: mysql
   halo:
     # 工作目录位置
     work-dir: /app/halo/halo2
     # 外部访问地址
     external-url: http://localhost:8090
     # 附件映射配置，通常用于迁移场景
     attachment:
       resource-mappings:
         - pathPattern: /upload/**
           locations:
             - migrate-from-1.x
   ```
   注意使用MySQL数据库，需创建数据库并修改配置。
   
   ```sql
   create database halo DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;
   ```
   
   测试启动使用全路径。
   
    ```bash
    /app/java/jdk-21.0.6/bin/java -Dfile.encoding=UTF-8 -server -Xms256m -Xmx256m -jar /app/halo/halo.jar --spring.config.additional-location=optional:file:/app/halo/
    ```
   创建 halo.service 文件
   ```bash
   vim /etc/systemd/system/halo.service
   ```
   ```properties
   [Unit]
   Description=halo.Service
   Documentation=https://docs.halo.run
   After=network-online.target
   Wants=network-online.target
   
   [Service]
   Type=simple
   User=root
   ExecStart=/app/java/jdk-21.0.6/bin/java -Dfile.encoding=UTF-8 -server -Xms256m -Xmx256m -jar /app/halo/halo.jar --spring.config.additional-location=optional:file:/app/halo/
   ExecStop=/bin/kill -s QUIT $MAINPID
   Restart=always
   
   [Install]
   WantedBy=multi-user.target
   ```
   ```bash
   systemctl daemon-reload
   ```
   ```bash
   systemctl start halo
   ```
   ```bash
   systemctl enable halo
   ```

2. 其他
   
    ```bash
    systemctl start halo
    systemctl stop halo
    systemctl status halo
    systemctl enable halo
    systemctl is-enabled halo
    journalctl -fn 200 -u halo
    ```

---