## Halo

---

### 安装
官方文档很详细：<https://docs.halo.run/getting-started/install/jar-file>

1. 创建文件夹，然后按照官方文档继续。
   
    ```bash
    mkdir /app/halo && cd /app/halo
    ```
   
   注意使用MySQL数据库，需创建数据库并修改配置。
   
   ```sql
   create database halo DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;
   ```
   
   测试启动使用全路径。
   
    ```bash
    /app/java/jdk-21.0.6/bin/java -Dfile.encoding=UTF-8 -server -Xms256m -Xmx256m -jar /app/halo/halo.jar --spring.config.additional-location=optional:file:/root/.halo2/
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