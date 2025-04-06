## Docker

---

### 安装
1. 卸载旧版本
   
   ```bash
   yum remove docker \
       docker-client \
       docker-client-latest \
       docker-common \
       docker-latest \
       docker-latest-logrotate \
       docker-logrotate \
       docker-engine
   ```

2. 安装docker的yum库。
   
   ```bash
   yum install -y yum-utils
   ```

3. 配置docker的yum源
   
   ```bash
   yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
   ```

4. 安装docker
   
   ```bash
   yum install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
   ```
   
   查看docker版本
   
   ```bash
   docker -v
   ```

5. 启动docker  
   设置开机启动
   
   ```bash
   systemctl enable docker
   ```
   
   启动
   
   ```bash
   systemctl start docker
   ```
   
   查看docker镜像
   
   ```bash
   docker images
   ```
   
   其他
   
   ```bash
   systemctl status docker
   systemctl stop docker
   systemctl restart docker
   ```

6. 设置镜像加速  
   阿里云镜像获取地址：<https://cr.console.aliyun.com/cn-hangzhou/instances/mirrors>
   
   ```bash
   sudo mkdir -p /etc/docker
   ```
   
    ```bash
    sudo tee /etc/docker/daemon.json <<-'EOF'
    {
      "registry-mirrors": ["https://525cckjy.mirror.aliyuncs.com"]
    }
    ```
   
    ```bash
    EOF
    ```
   
    ```bash
    sudo systemctl daemon-reload
    ```
   
    ```bash
    sudo systemctl restart docker
    ```

### 常用命令

| 命令                  | 解释                               | 示例                                       |
|---------------------|----------------------------------|------------------------------------------|
| docker run          | 启动一个新的容器并运行命令                    | docker run -d ubuntu                     |
| docker ps           | 列出当前正在运行的容器                      | docker ps                                |
| docker ps -a        | 列出所有容器（包括已停止的容器）                 | docker ps -a                             |
| docker build        | 使用 Dockerfile 构建镜像               | docker build -t my-image .               |
| docker images       | 列出本地存储的所有镜像                      | docker images                            |
| docker pull         | 从 Docker 仓库拉取镜像                  | docker pull ubuntu                       |
| docker push         | 将镜像推送到 Docker 仓库                 | docker push my-image                     |
| docker exec         | 在运行的容器中执行命令                      | docker exec -it container_name bash      |
| docker stop         | 停止一个或多个容器                        | docker stop container_name               |
| docker start        | 启动已停止的容器                         | docker start container_name              |
| docker restart      | 重启一个容器                           | docker restart container_name            |
| docker rm           | 删除一个或多个容器                        | docker rm container_name                 |
| docker rmi          | 删除一个或多个镜像                        | docker rmi my-image                      |
| docker logs         | 查看容器的日志                          | docker logs container_name               |
| docker inspect      | 获取容器或镜像的详细信息                     | docker inspect container_name            |
| docker exec -it     | 进入容器的交互式终端                       | docker exec -it container_name /bin/bash |
| docker network ls   | 列出所有 Docker 网络                   | docker network ls                        |
| docker volume ls    | 列出所有 Docker 卷                    | docker volume ls                         |
| docker-compose up   | 启动多容器应用（从 docker-compose.yml 文件） | docker-compose up                        |
| docker-compose down | 停止并删除由 docker-compose 启动的容器、网络等  | docker-compose down                      |
| docker info         | 显示 Docker 系统的详细信息                | docker info                              |
| docker version      | 显示 Docker 客户端和守护进程的版本信息          | docker version                           |
| docker stats        | 显示容器的实时资源使用情况                    | docker stats                             |
| docker login        | 登录 Docker 仓库                     | docker login                             |
| docker logout       | 登出 Docker 仓库                     | docker logout                            |

---