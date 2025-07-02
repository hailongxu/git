
# docker 原理

隔离，配额
隔离：文件系统，用户，init进程，网络，linux采用namespace实现，用到的程序交互界面进行
镜像文件系统：保存每一步的结果，分层文件系统overlay，采用COW技术


## 注意点
docker 容器中的进程都要运行在前台

docker中init晋城市不需要的，直接就是你的主程序

# docker 指令
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
## 镜像容器
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

## docker 存储
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



# 案例

## Nginx

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

## Mysql服务
```yaml
version: '3.8'

services:
  mysql:
    image: mysql:8.0
    
    container_name: mysql_db
    
    environment:
      # MySQL镜像专用变量（必须大写）
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD:-secret}  # 默认密码'secret'
    
    volumes:
      - mysql-data:/var/lib/mysql
      # 初始化SQL脚本挂载（Linux下路径区分大小写）
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql  # 注意文件路径大小写
    
    ports:
      - "3306:3306"  # 主机端口:容器端口
    
    restart: unless-stopped  # 必须小写

volumes:
  mysql-data:
    # 显式命名卷（规范关键字）
    name: mysql-data  # 保持与挂载时名称一致
```

测试Mysql服务
```bash
# 启动服务
docker compose up -d

# 验证数据库初始化
docker exec mysql_db mysql -uroot -psecret -e "SHOW DATABASES;"

# 测试持久化（重启后数据仍在）
docker compose down && docker compose up -d
docker exec mysql_db mysql -uroot -psecret -e "USE testdb; CREATE TABLE test(id INT);"
docker compose restart
docker exec mysql_db mysql -uroot -psecret -e "SHOW TABLES IN testdb;"
```


## WEB + Redis 系统

docker-compose.yaml
```yaml
# 案例需求：
# 1. 使用自定义 bridge 网络，实现 Web 应用与 Redis 的容器间通信。
# 2. Web 应用需通过 /ping 路径访问，并使用 Redis 进行计数。
# 3. 通过 docker-compose 管理多容器服务。
# 4. 提供测试脚本验证 /ping 计数功能。
#
# 说明：
# - mynet 是自定义 bridge 网络，保障容器间隔离与通信。
# - web 服务依赖 redis 服务，需通过网络名访问 redis。
# - 5000:5000 将 Web 服务暴露到主机。
# - 可通过 Dockerfile 构建 web 服务镜像。
#

version: '3.8'

# 自定义 bridge 网络
networks:
  mynet:
    driver: bridge
    name: mynet  # 显式命名网络

services:
  # Redis 服务
  redis:
    image: redis:alpine
    container_name: redis
    networks:
      - mynet
    restart: unless-stopped
    # # 注意：生产环境应设置密码
    # healthcheck:
    #   test: ["CMD", "redis-cli", "ping"]
    #   interval: 5s
    #   timeout: 2s
    #   retries: 3

  # Web 应用服务
  web:
    build: .
    container_name: web
    networks:
      - mynet
    ports:
      - "5000:5000"
    depends_on:
      - redis
      # redis:
      #   condition: service_healthy
    environment:
      - FLASK_ENV=development
    restart: unless-stopped

# 测试:
# docker compose up -d --build 启动服务
# curl http://localhost:5000/ping 验证计数功能
```

Dockerfile
```dockerfile
# 案例需求：
# 构建一个 Python Web 应用镜像，支持通过 Redis 计数。
#
# 关键步骤说明：
# - 选择合适的 Python 基础镜像
# - 安装依赖（如 flask、redis）
# - 拷贝应用代码到镜像中
# - 设置容器启动命令

FROM python:3.10-slim
# 在这里编写你的 Dockerfile

WORKDIR /app

COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt && \
    rm -rf /tmp/* /var/tmp/*

COPY app.py .

EXPOSE 5000

CMD ["python", "app.py"]


# # 构建阶段
# FROM python:3.10 as builder
# WORKDIR /app
# COPY requirements.txt .
# RUN pip install --user -r requirements.txt

# # 运行阶段
# FROM python:3.10-slim
# WORKDIR /app
# COPY --from=builder /root/.local /root/.local
# COPY app.py .
# ENV PATH=/root/.local/bin:$PATH
# CMD ["python", "app.py"]
```

app.py
```python
from flask import Flask
import redis

app = Flask(__name__)
r = redis.Redis(host='redis', port=6379, decode_responses=True)

@app.route('/ping')
def ping():
    count = r.incr('count')
    return f'PONG {count}', 200

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

requirements.txt
```text
flask
redis
```


测试和验证
```shell
# 构建并启动服务
docker compose up -d --build

# 测试接口（应返回递增计数）
curl http://localhost:5000/ping  # PONG 1
curl http://localhost:5000/ping  # PONG 2

# 验证容器间通信
docker exec -it web ping redis

# 查看服务日志
docker compose logs -f

# 清理环境
docker compose down
```