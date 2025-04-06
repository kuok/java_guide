## Nacos

---

### Docker部署

参考<https://nacos.io/docs/v2.4/manual/admin/deployment/deployment-standalone/?spm=5238cd80.2ef5001f.0.0.3f613b7curtyBZ>

1. 创建文件夹
   
   ```bash
   mkdir /app/nacos && cd /app/nacos
   ```

2. 创建application.properties文件使用mysql数据库  
   上方为mysql信息。下方为鉴权信息。
   
   ```properties
   spring.sql.init.platform=mysql
   
   db.num=1
   db.url.0=jdbc:mysql://localhost:3306/nacos?characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true
   db.user=user
   db.password=pwd
   
   nacos.core.auth.plugin.nacos.token.secret.key=token的base64编码
   nacos.core.auth.server.identity.key=key
   nacos.core.auth.server.identity.value=value
   ```
   
   > 在 Nacos 中，**NACOS_AUTH_TOKEN**、**NACOS_AUTH_IDENTITY_KEY**和**NACOS_AUTH_IDENTITY_VALUE** 是三个与安全认证相关的配置项。它们的作用如下：
   > - **NACOS_AUTH_TOKEN**：表示 Nacos 的认证令牌。当启用认证功能时，需要使用认证令牌进行身份验证。如果未提供正确的认证令牌，将无法访问 Nacos 的管理接口和配置信息。首先，您需要选定一个至少32个字符的文本字符串作为原始密钥。接下来，您需要将这个原始密钥字符串通过Base64编码。参考：<https://nacos.io/blog/faq/nacos-user-question-history15141/> ，这里自己生成，注意是长度大于32的字符的base64编码。
   > - **NACOS_AUTH_IDENTITY_KEY**：表示 Nacos 认证身份的键。当启用认证功能时，需要使用身份键和身份值进行身份验证。身份键可以是用户名、应用程序名称或任何其他标识符。这里使用nacos。
   > - **NACOS_AUTH_IDENTITY_VALUE**：表示 Nacos 认证身份的值。当启用认证功能时，需要使用身份键和身份值进行身份验证。身份值可以是用户名、应用程序名称或任何其他与身份键相对应的值。这里使用nacos密码，后续登录控制台修改为该密码。
3. 数据库初始化  
   新建nacos数据库
   
   ```sql
   create database nacos DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;
   ```
   
   初始化数据库。使用官方初始化sql文件。<https://github.com/alibaba/nacos/blob/master/distribution/conf/mysql-schema.sql?spm=5238cd80.2ef5001f.0.0.3f613b7curtyBZ&file=mysql-schema.sql>
4. docker部署
   > 三个值要与application.properties文件内一致，不然控制台出现Invalid server identity key or value, Please make sure set `nacos.core.auth.server.identity.key` and `nacos.core.auth.server.identity.value`, or open `nacos.core.auth.enable.userAgentAuthWhite`错误。
   
   开启鉴权
   **NACOS_AUTH_TOKEN**:**nacos.core.auth.plugin.nacos.token.secret.key**
   **NACOS_AUTH_IDENTITY_KEY**:**nacos.core.auth.server.identity.key**
   **NACOS_AUTH_IDENTITY_VALUE**:**nacos.core.auth.server.identity.value**
   
   ```bash
   docker run --name nacos-standalone -e MODE=standalone -e NACOS_AUTH_ENABLE=true -e NACOS_AUTH_TOKEN='QWxleG5hY29zdG9rZW4xMjM0NTY3ODkxMDExMTIxMzE=' -e NACOS_AUTH_IDENTITY_KEY='nacos' -e NACOS_AUTH_IDENTITY_VALUE='kuok1995GDD' -v /path/application.properties:/app/nacos/application.properties -p 8848:8848 -d -p 9848:9848  nacos/nacos-server:latest
   ```

5. 防火墙
   
   ```bash
   firewall-cmd --zone=public --add-port=8848/tcp --permanent
   systemctl restart firewalld
   ```

6. 控制台  
   访问<http://host:8848/nacos> 进入控制台。  
   默认不需要密码。默认用户名nacos，登录后修改密码为上方的NACOS_AUTH_IDENTITY_VALUE：nacos.core.auth.server.identity.value

---

### 本地部署

Docker部署后一直退出，估计是太占用内存了，改为本地部署。  
参考：<https://nacos.io/docs/latest/quickstart/quick-start/?spm=5238cd80.2ef5001f.0.0.3f613b7cGhSlhZ>

1. 创建文件夹
   
   ```bash
   mkdir /app/nacos && cd /app/nacos
   ```

2. 上传文件  
   上传文件后，解压
   ```bash
   unzip nacos-server-$version.zip 
   ```
   
   再上传配置文件，开启鉴权
   
    ```propertie
    spring.sql.init.platform=mysql
    db.num=1
    db.url.0=jdbc:mysql://localhost:3306/nacos?characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true
    db.user=root
    db.password=mysql密码
    
    nacos.core.auth.plugin.nacos.token.secret.key=base64编码后的token
    nacos.core.auth.server.identity.key=nacos
    nacos.core.auth.server.identity.value=密码（尽量与控制台登陆密码一致）
    
    ### If turn on auth system:
    nacos.core.auth.system.type=nacos
    nacos.core.auth.enabled=true
    ```

3. 启动  
   进入bin文件夹
   
    ```bash
    sh startup.sh -m standalone
    ```
   
   登陆控制台页面，应该可以登陆了
   
    ```text
    http://tencent.server:8848/nacos/
    ```
   
   查看日志
   
    ```bash
    tail -fn200 /app/nacos/nacos/logs/start.out
    ```
   
   停止
   
    ```bash
    sh shutdown.sh
    ```
   
   其他
   
   > 服务注册
   >
   > ```bash
   > curl -X POST 'http://127.0.0.1:8848/nacos/v1/ns/instance?serviceName=nacos.naming.serviceName&ip=20.18.7.10&port=8080'
   > ```
   >
   > 服务发现
   >
   > ```bash
   > curl -X GET 'http://127.0.0.1:8848/nacos/v1/ns/instance/list?serviceName=nacos.naming.serviceName'
   > ```
   >
   > 发布配置
   >
   > ```bash
   > curl -X POST "http://127.0.0.1:8848/nacos/v1/cs/configs?dataId=nacos.cfg.dataId&group=test&content=HelloWorld"
   > ```
   >
   > 获取配置
   >
   > ```bash
   > curl -X GET "http://127.0.0.1:8848/nacos/v1/cs/configs?dataId=nacos.cfg.dataId&group=test"
   > ```

4. 开机启动  
   新建文件
   
    ```bash
    vim /etc/systemd/system/nacos.service
    ```
   
   输入以下信息，注意是单机模式，且需要指定JAVA_HOME
   
    ```properties
    [Unit]
    Description=Nacos service
    After=network.target
    
    [Service]
    Type=forking
    Environment="JAVA_HOME=/app/java/jdk-21.0.6"
    ExecStart=/bin/bash /app/nacos/nacos/bin/startup.sh  -m standalone  #启动命令 启动脚本换成自己对应的目录即可
    ExecStop=/bin/bash /app/nacos/nacos/bin/shutdown.sh #停止命令 停止脚本换成自己对应的目录即可
    
    [Install]
    WantedBy=multi-user.target
    ```
   
   重新加载文件
   
   ```bash
   systemctl daemon-reload
   ```
   
   启动redis
   
   ```bash
   systemctl start nacos
   ```
   
   开机自启
   
   ```bash
   systemctl enable nacos
   ```
   
   top看了下，2G的内存能占我35%，不知道为啥这么占内存。过几天估计还会挂，起码这次能轻易找到日志了。
   
    ```bash
     4814 root      20   0 3660420 720012  10092 S   0.3 35.2   0:39.06 java
    20701 mysql     20   0 1859816 470172   4616 S   0.3 23.0  16:50.29 mysqld
    20338 rabbitmq  20   0 2281148  84576   2860 S   0.0  4.1  20:18.31 beam.smp
     3625 root      20   0 1056588  83748   6648 S   1.0  4.1 116:37.73 YDService
    ```

---