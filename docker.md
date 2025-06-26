
## docker 指令
```bash
docker info
docker version
docker run hello-world
```
镜像的格式：
```text
<repository>/<image>:<tag>
```
私有镜像，需要login
```shell
docker login <repository>
```

```shell
docker pull alpine # 拉取镜像
docker image ls # 查看镜像
docker run alpine ls -a # 运行容器
docker ps -a # 查看容器
docker inspect alpine --format='{{.Config.Cmd}}' # 查看cmd是啥，默认就是启动这个cmd
docker run -it -d alpin # 后台运行容器， i=interactive t=tty
docker attach <container_id> # 会替换掉init进程，所以退出后，容器也推出了
docker exec # 这个是执行，容器不会退出
```

docker 存储
```shell
docker volume create nginx_data

# 运行 Nginx 容器并挂载卷
docker run -d --name web-volume -p 8081:80 -v nginx_data:/usr/share/nginx/html nginx

# 在容器中创建一个测试页面
docker exec -it web-volume sh -c 'echo "<h1>Hello from Volume Storage</h1>" > /usr/share/nginx/html/index.html'

# 访问页面验证内容
curl http://localhost:8081

# 删除容器
docker rm -f web-volume

# 用同样的配置重新运行容器
docker run -d --name web-volume-2 -p 8081:80 \
   -v nginx_data:/usr/share/nginx/html nginx

# 再次访问页面，内容仍然存在
curl http://localhost:8081

# 查看卷的详细信息
docker volume inspect nginx_data
```
## 容器编排
- `docker compose up`: 创建和启动所有服务
- `docker compose down`: 停止和删除所有服务
- `docker compose ps`: 查看服务状态
- `docker compose logs`: 查看服务日志


## 监控
```bash
# 实时查看容器资源使用状态
docker stats

# 查看容器详细信息
docker inspect <container_id>

# 查看容器内进程
docker top <container_id>

# 查看容器端口映射
docker port <container_id>
```

## 注意点
docker 要求运行的进程，都要运行在前台


## 示例

### Nginx

```dockerfile
FROM ubuntu:22.04
# 安装 nginx 和 curl
RUN apt update && \
    apt install -y nginx curl && \
    rm -rf /var/lib/apt/lists/*
# 创建简单的 HTML 文件
RUN echo "Hello Docker!" > /var/www/html/index.html
# 暴露 80 端口
EXPOSE 80
# 启动 nginx 服务（前台运行）
CMD ["nginx", "-g", "daemon off;"]
```
