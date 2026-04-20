# Chapter 08：Docker 部署游戏服务器

> 🎯 目标：掌握用 Docker 和 Kubernetes 部署高可用游戏服务器集群，实现自动化扩缩容。

---

## 8.1 Docker 基础

### 镜像/容器/仓库

```
镜像（Image）= 菜谱（只读的配方）
容器（Container）= 按菜谱做出来的菜（运行中的实例）
仓库（Registry）= 食谱书柜（存放镜像的地方）
```

### 核心命令

```bash
# 构建镜像
docker build -t game-server:v1.0 .

# 运行容器
docker run -d --name game-server -p 8888:8888 game-server:v1.0

# 查看运行中的容器
docker ps

# 查看日志
docker logs -f game-server

# 进入容器
docker exec -it game-server /bin/bash

# 停止/启动
docker stop game-server
docker start game-server

# 构建多架构镜像
docker buildx build --platform linux/amd64,linux/arm64 -t game-server:v1.0 .
```

---

## 8.2 Dockerfile 写游戏服务器

### Go 游戏服务器 Dockerfile

```dockerfile
# -------- 多阶段构建 --------
# 阶段1：编译
FROM golang:1.21-alpine AS builder

WORKDIR /app

# 安装编译依赖
RUN apk add --no-cache git

# 复制源码
COPY . .

# 编译（禁用 CGO，适应更多环境）
RUN CGO_ENABLED=0 GOOS=linux GOARCH=amd64 \
    go build -ldflags="-s -w" -o game-server .

# -------- 阶段2：运行 --------
FROM alpine:3.19

WORKDIR /app

# 安装运行时依赖
RUN apk add --no-cache ca-certificates tzdata

# 从 builder 复制二进制
COPY --from=builder /app/game-server .
COPY --from=builder /app/config ./config/

# 创建非 root 用户
RUN adduser -D -u 1000 gameuser
USER gameuser

# 暴露端口
EXPOSE 8888 8889

# 健康检查
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD wget --quiet --tries=1 --spider http://localhost:8888/health || exit 1

# 启动命令
ENTRYPOINT ["./game-server"]
CMD ["--config", "config/server.yaml"]
```

### 配置文件

```yaml
# config/server.yaml
server:
  host: "0.0.0.0"
  port: 8888

database:
  host: "db"
  port: 3306
  user: "game"
  password: "${DB_PASSWORD}"
  name: "gamedb"

redis:
  host: "redis"
  port: 6379

game:
  max_players: 1000
  tick_rate: 30
```

---

## 8.3 docker-compose 多容器编排

### docker-compose.yaml

```yaml
version: '3.8'

services:
  # -------- 游戏逻辑服务器 --------
  game-server:
    build: ./game-server
    container_name: game-server
    ports:
      - "8888:8888"
    environment:
      - DB_PASSWORD=${DB_PASSWORD}
      - REDIS_HOST=redis
    depends_on:
      db:
        condition: service_healthy
      redis:
        condition: service_started
    restart: unless-stopped
    networks:
      - game-network
    deploy:
      replicas: 2
      resources:
        limits:
          cpus: '2'
          memory: 2G

  # -------- 数据库 --------
  db:
    image: mysql:8.0
    container_name: game-db
    environment:
      MYSQL_ROOT_PASSWORD: ${DB_PASSWORD}
      MYSQL_DATABASE: gamedb
      MYSQL_USER: game
      MYSQL_PASSWORD: ${DB_PASSWORD}
    volumes:
      - db-data:/var/lib/mysql
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      interval: 10s
      timeout: 5s
      retries: 5
    networks:
      - game-network

  # -------- Redis --------
  redis:
    image: redis:7-alpine
    container_name: game-redis
    command: redis-server --appendonly yes
    volumes:
      - redis-data:/data
    networks:
      - game-network

  # -------- Consul（服务发现）--------
  consul:
    image: consul:1.15
    container_name: game-consul
    command: agent -server -ui -bootstrap-expect=1 -client=0.0.0.0
    networks:
      - game-network

networks:
  game-network:
    driver: bridge

volumes:
  db-data:
  redis-data:
```

### 启动命令

```bash
# 构建并启动所有服务
docker-compose up -d --build

# 查看所有服务状态
docker-compose ps

# 查看日志
docker-compose logs -f game-server

# 扩缩容（增加到 4 个游戏服务器）
docker-compose up -d --scale game-server=4

# 停止所有服务
docker-compose down
```

---

## 8.4 Kubernetes 基础

### 核心概念

| 概念 | 说明 |
|-----|------|
| **Pod** | 最小调度单元，一个或多个容器 |
| **Deployment** | 管理 Pod 副本数，滚动更新 |
| **Service** | 给一组 Pod 提供稳定的访问入口 |
| **ConfigMap** | 配置管理 |
| **Secret** | 敏感信息（密码/密钥）|
| **HorizontalPodAutoscaler** | 根据负载自动扩缩容 |

### 游戏服务器 K8s 部署

```yaml
# game-server-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: game-server
  labels:
    app: game-server
spec:
  replicas: 3
  selector:
    matchLabels:
      app: game-server
  template:
    metadata:
      labels:
        app: game-server
    spec:
      containers:
      - name: game-server
        image: registry.example.com/game-server:v1.0
        ports:
        - containerPort: 8888
          name: game
        env:
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: game-secrets
              key: db-password
        resources:
          requests:
            memory: "1Gi"
            cpu: "500m"
          limits:
            memory: "2Gi"
            cpu: "2000m"
        livenessProbe:
          httpGet:
            path: /health
            port: 8888
          initialDelaySeconds: 10
          periodSeconds: 30
        readinessProbe:
          httpGet:
            path: /ready
            port: 8888
          initialDelaySeconds: 5
          periodSeconds: 10
---
# -------- Service：提供负载均衡 --------
apiVersion: v1
kind: Service
metadata:
  name: game-server-service
spec:
  selector:
    app: game-server
  ports:
  - protocol: TCP
    port: 8888
    targetPort: 8888
  type: LoadBalancer
---
# -------- Horizontal Pod Autoscaler：自动扩缩容 --------
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: game-server-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: game-server
  minReplicas: 3
  maxReplicas: 20
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
```

---

## 8.5 环境变量与配置

```bash
# .env 文件
DB_PASSWORD=super_secret_password
REDIS_PASSWORD=redis_pass
SERVER_PORT=8888
LOG_LEVEL=info

# 启动时注入
docker run --env-file .env game-server
```

### Kubernetes Secret

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: game-secrets
type: Opaque
stringData:
  db-password: "super_secret_password"
  redis-password: "redis_pass"
```

---

## 8.6 CI/CD 自动化

### GitHub Actions

```yaml
# .github/workflows/deploy.yml
name: Build and Deploy

on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3

    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v2

    - name: Login to Registry
      uses: docker/login-action@v2
      with:
        registry: registry.example.com
        username: ${{ secrets.REGISTRY_USER }}
        password: ${{ secrets.REGISTRY_TOKEN }}

    - name: Build and Push
      uses: docker/build-push-action@v4
      with:
        context: ./game-server
        push: true
        tags: registry.example.com/game-server:${{ github.sha }}

    - name: Deploy to K8s
      run: |
        kubectl set image deployment/game-server \
          game-server=registry.example.com/game-server:${{ github.sha }}
```

---

## 8.7 本章小结

```
✅ 已掌握：
├── Docker 镜像/容器/仓库概念
├── Dockerfile 多阶段构建（Go 游戏服务器）
├── docker-compose 编排多容器
├── Kubernetes 核心概念（Pod/Deployment/Service）
├── 游戏服务器 K8s 部署 YAML
├── HorizontalPodAutoscaler 自动扩缩容
├── Secret/ConfigMap 配置管理
├── 环境变量与敏感信息
└章 CI/CD 自动化部署

🎉 Stage 5 MMO 专题 —— 完成！

🎉 全部 Stage 5 MMO 网络专题完成！
覆盖：帧同步/状态同步 → KCP/ENet → 预测回滚 → 服务器架构 → Redis缓存 → 跨服战 → 反外挂 → Docker部署
```

---

_📚 参考资料：《Docker 官方文档》《Kubernetes 官方文档》《Go in Action》_
