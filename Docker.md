
# 解决什么问题？

- **一句话解释：** Docker 解决了“**在我电脑上明明是好的，怎么到服务器上就挂了？**”这个古老而棘手的问题。
- 为了保持 **环境一致性**
- **隔离性：** Docker 利用了 Linux 内核的 **Namespaces**（实现资源隔离，比如进程、网络）和 **Cgroups**（实现资源限制，比如 CPU、内存）技术，使得每个容器都像一个轻量级的、独立的虚拟机，但它又比虚拟机启动更快、资源占用更少，因为它共享宿主机的操作系统内核。
- **标准化：** Docker 提供了一套标准的打包、运输和运行应用的流程。这使得开发、测试、运维团队可以用同一套标准进行协作，极大地提升了DevOps的效率。

# 镜像和容器有什么区别

- **一句话解释：** 镜像是静态的“**模板**”，容器是动态的“**实例**”。
- **一个比喻：** 我们用面向对象编程来比喻。**镜像 (Image)** 就像是一个**类 (Class)**，它是一个只读的、定义好的模板。而**容器 (Container)** 则是这个类的一个**实例 (Instance)**，即 `new` 出来的一个对象。你可以用同一个镜像（类）创建出很多个一模一样但相互隔离的容器（实例）。

- **镜像的分层结构 (Layers)：** 这是一个非常重要的技术细节。Docker 镜像是**分层**的。`Dockerfile`（我们下一步会讲）中的每一条指令都会创建一个新的层。这些层是只读的，并且会被缓存。
    - **优点1 (高效构建)：** 如果你修改了 `Dockerfile` 的某一行，Docker 在重新构建镜像时，只会重新构建被修改行及其之后的分层，前面的缓存层会被直接复用，大大加快了构建速度。
    - **优点2 (高效存储与分发)：** 如果多个镜像共享相同的基础层（例如，都基于 `ubuntu:20.04`），那么在磁盘上这部分基础层只需要存储一份。

- **容器的读写层：** 当一个容器从镜像启动时，Docker 会在镜像的最上层添加一个**可读写的“容器层”**。所有对容器的修改（如创建文件、修改配置）都发生在此层。原始的镜像是不会被改变的。这被称为“Copy-on-Write”机制。

# Dockerfile
`Dockerfile` 是一个文本文件，它包含了一系列指令，用于告诉 Docker 引擎如何一步步地构建出一个镜像。
示例：
```Dockerfile
# 1. 选择一个包含 Java 运行环境的基础镜像
FROM openjdk:11-jre-slim

# 2. 设置容器内的工作目录
WORKDIR /app

# 3. 将本地打包好的 JAR 文件复制到容器的工作目录中
# 第一个 . 是指本地 target 目录下的 .jar 文件
# 第二个 . 是指容器内的 /app 目录
COPY target/my-app.jar .

# 4. 声明容器对外暴露的端口（纯文档性质）
EXPOSE 8080

# 5. 定义容器启动时要执行的命令
CMD ["java", "-jar", "my-app.jar"]
```

- `FROM`：指定基础镜像，通常是构建镜像的第一条指令。
- `RUN`：执行命令，比如安装软件包。
- `COPY` 或 `ADD`：将文件复制到镜像中。
	- **区别**：功能类似，都是将文件从宿主机复制到镜像中。但 `ADD` 有两个额外的功能：它可以自动解压 `tar` 文件，并且源文件可以是一个 URL。COPY的源文件只能是本地文件。因为 `ADD` 的行为不如 `COPY` 透明和可预测，官方最佳实践推荐**优先使用 `COPY`**。
- `CMD` 或 `ENTRYPOINT`：指定容器启动后执行的命令。
	- **区别**：它们都用于定义容器启动时执行的命令。主要区别在于：`CMD` 提供的命令**可以被 `docker run` 后面的参数覆盖**，它更像是一个‘默认’命令。而 `ENTRYPOINT` 定义的命令**不会被轻易覆盖**，`docker run` 后面的参数会被当作 `ENTRYPOINT` 命令的参数来处理。`ENTRYPOINT` 更适合用于制作‘可执行’的容器。
- `EXPOSE`：声明镜像中使用的端口，主要作为文档用途，不会实际发布端口。

# 多阶段构建 (Multi-stage builds)
多阶段构建（multi-stage build）主要有以下几个优势：

1）**减小镜像体积**：通过在构建过程中仅保留最终需要的文件，可以显著减少 Docker 镜像的体积，从而加快部署和启动时间。 
2）**提高安全性**：减少了镜像中的不必要文件，降低了潜在的安全风险。 
3）**优化构建流程**：可以在不同阶段使用不同的基础镜像，从而更好地管理依赖和构建工具，并提高构建效率。 
4）**易于维护**：构建过程更加清晰和可维护，因为不同的阶段可以对不同的任务负责，使得 Dockerfile 更加简洁和模块化。
一个Go示例：
```dockerfile
# 第一个阶段：构建阶段
FROM golang:1.17 AS builder
WORKDIR /app
COPY . .
RUN go build -o myapp

# 第二个阶段：运行时阶段
FROM alpine:latest
WORKDIR /root/
COPY --from=builder /app/myapp .
CMD ["./myapp"]
```



|特性|**容器**|**虚拟机 (VM)**|
|---|---|---|
|**隔离级别**|进程级别隔离|操作系统级别隔离|
|**底层架构**|共享宿主机的操作系统内核|每个 VM 都有自己的操作系统|
|**启动速度**|秒级|分钟级|
|**资源消耗**|非常小|较大|
|**部署效率**|非常高|相对较低|

# Docker网络模式

## 网络模式类型
- **bridge模式**：默认网络模式，容器通过docker0网桥进行通信
- **host模式**：容器与宿主机共享网络命名空间，性能较好但隔离性差
- **none模式**：容器没有网络接口，完全隔离
- **container模式**：新容器与已存在容器共享网络命名空间

## 端口映射
```bash
# 将容器端口映射到宿主机
docker run -p 8080:80 nginx  # 宿主机8080映射到容器80端口
docker run -P nginx           # 随机映射容器暴露的端口
```

## 容器间通信
- **同一网络下的容器**：可以通过容器名直接通信
- **不同网络下的容器**：需要通过端口映射或创建自定义网络

# Docker数据管理

## 数据卷(Volume)
```bash
# 创建数据卷
docker volume create my-volume
# 挂载数据卷
docker run -v my-volume:/data nginx
```

## 绑定挂载(Bind Mount)
```bash
# 将宿主机目录挂载到容器
docker run -v /host/path:/container/path nginx
```

## tmpfs挂载
```bash
# 内存临时存储
docker run --tmpfs /tmp nginx
```

# Docker Compose

## 核心概念
- **服务(Service)**：一个应用的容器，可以包含多个相同镜像的容器实例
- **项目(Project)**：由一组关联的应用容器组成的一个完整业务单元

## docker-compose.yml示例
```yaml
version: '3.8'
services:
  web:
    build: .
    ports:
      - "5000:5000"
    depends_on:
      - redis
  redis:
    image: redis:alpine
```

## 常用命令
```bash
docker-compose up -d     # 后台启动所有服务
docker-compose down      # 停止并删除容器、网络
docker-compose logs web  # 查看服务日志
docker-compose scale web=3 # 扩容web服务到3个实例
```

# Docker在K8S中的角色

## 容器运行时接口(CRI)
- Docker通过dockershim与K8S集成（已逐步淘汰）
- 现代K8S推荐使用containerd、CRI-O等轻量级运行时

## 镜像规范
- K8S使用OCI标准镜像格式
- Docker镜像是K8S Pod运行的基础

## K8S中的Docker使用
```yaml
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: myapp
    image: nginx:1.21  # Docker镜像
    ports:
    - containerPort: 80
```

# Docker安全

## 安全最佳实践
- **最小权限原则**：使用非root用户运行容器
- **镜像安全**：使用官方镜像，定期更新基础镜像
- **资源限制**：设置CPU、内存限制
- **网络安全**：使用自定义网络，限制容器间通信

## 安全扫描
```bash
# 使用docker scan扫描镜像漏洞
docker scan my-image:latest
```

## 安全上下文
```dockerfile
# 创建非root用户
RUN groupadd -r appuser && useradd -r -g appuser appuser
USER appuser
```

# Docker与CI/CD集成

## Jenkins集成
```groovy
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                script {
                    def image = docker.build("myapp:${env.BUILD_ID}")
                }
            }
        }
        stage('Test') {
            steps {
                script {
                    docker.image("myapp:${env.BUILD_ID}").inside {
                        sh 'npm test'
                    }
                }
            }
        }
        stage('Deploy') {
            steps {
                script {
                    docker.image("myapp:${env.BUILD_ID}").push()
                }
            }
        }
    }
}
```

## GitLab CI集成
```yaml
stages:
  - build
  - test
  - deploy

build:
  stage: build
  script:
    - docker build -t $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA .
    - docker push $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
```

## Docker Registry
- **私有仓库搭建**：使用Docker Registry或Harbor
- **镜像标签策略**：使用Git commit SHA、时间戳等作为标签
- **镜像清理策略**：定期清理旧版本镜像

# Docker性能优化

## 镜像优化
- **减少层数**：合并RUN指令
- **选择合适的基础镜像**：使用alpine、distroless等轻量级镜像
- **多阶段构建**：分离构建和运行环境

## 容器优化
- **资源限制**：合理设置CPU、内存限制
- **健康检查**：配置HEALTHCHECK指令
- **日志管理**：配置日志驱动和轮转策略

## 构建缓存优化
```dockerfile
# 将不常变化的指令放在前面
FROM node:14
COPY package*.json ./
RUN npm ci --only=production
COPY . .
```

# Docker常见问题排查

## 容器启动失败
```bash
# 查看容器日志
docker logs container-name
# 进入失败容器
docker run -it --rm image-name /bin/bash
```

## 镜像构建失败
```bash
# 查看构建过程
docker build --no-cache -t myapp .
# 调试构建过程
docker run -it --rm image-name /bin/bash
```

## 网络问题
```bash
# 查看网络配置
docker network ls
docker network inspect bridge
# 测试网络连通性
docker exec container-name ping target-host
```

## 性能问题
```bash
# 查看容器资源使用
docker stats
# 查看容器进程
docker top container-name
```
