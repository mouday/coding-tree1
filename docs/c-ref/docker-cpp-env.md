# Docker搭建CPP编译环境

## Dockerfile

CPP编译环境示例

```shell
# 1. 选择一个可靠的基础镜像，这里以 Ubuntu 22.04 LTS 为例
FROM ubuntu:22.04

# 2. 设置环境变量，避免安装过程中的交互式提示
ENV DEBIAN_FRONTEND=noninteractive

# 3. 安装常用的C++开发工具链和库
RUN apt-get update && apt-get install -y \
    build-essential \
    cmake \
    git \
    valgrind \
    clang \
    gdb \
    gcc \
    g++ \
    libboost-all-dev \
    nasm \
    qemu-system-x86 \
    # 安装完后，清理APT缓存以减小镜像体积
    && rm -rf /var/lib/apt/lists/*

# 4. 设置容器内的工作目录
WORKDIR /workspace
```

构建操作

```shell
# 构建镜像
docker build -t cpp-env:latest .

# 查看镜像
docker images

# 移除镜像
docker image rm 4a0db4b8dc33

# 启动镜像
docker run -it --rm cpp-env:latest /bin/bash

docker run -it --rm \
    -v $(pwd):/workspace \
    --ulimit core=-1 \
    --cap-add=SYS_PTRACE \
    --security-opt seccomp=unconfined \
    cpp-env:latest /bin/bash


# 查看正在运行的容器
docker ps

# 连接
docker exec -it <容器名或ID> /bin/bash

# 启动/停止
docker start <容器名>
```

技巧：将启动命令放到脚本中，可以直接运行脚本进入容器

```shell
#!/bin/bash
# bash ./cpp-env.sh
docker run -it --rm \
    -v $(pwd):/workspace \
    --ulimit core=-1 \
    --cap-add=SYS_PTRACE \
    --security-opt seccomp=unconfined \
    cpp-env:latest /bin/bash
```

## 一、镜像操作（补充上次未提及的）

| 操作 | 命令 |
| :--- | :--- |
| 查看镜像 | `docker images` 或 `docker image ls` |
| 拉取镜像 | `docker pull <镜像名>:<标签>` |
| 构建镜像 | `docker build -t <镜像名>:<标签> .` |
| 删除镜像 | `docker rmi <镜像ID或名>` |
| 给镜像打标签 | `docker tag <原镜像> <新仓库/新标签>` |
| 导出镜像为文件 | `docker save -o <文件名.tar> <镜像名>` |
| 从文件导入镜像 | `docker load -i <文件名.tar>` |
| 查看镜像历史层 | `docker history <镜像名>` |
| 清理悬空镜像 | `docker image prune` |

---

## 二、容器操作（最常用）

| 操作 | 命令 |
| :--- | :--- |
| **运行容器**（前台） | `docker run <镜像名>` |
| **运行容器**（后台） | `docker run -d <镜像名>` |
| 运行并进入交互终端 | `docker run -it <镜像名> /bin/bash` |
| 运行并命名容器 | `docker run --name <容器名> <镜像名>` |
| 端口映射 | `docker run -p 宿主机端口:容器端口 <镜像名>` |
| 挂载宿主机目录 | `docker run -v 宿主机路径:容器路径 <镜像名>` |
| **查看运行中的容器** | `docker ps` |
| 查看所有容器（含停止的） | `docker ps -a` |
| **停止容器** | `docker stop <容器ID或名>` |
| **启动已停止的容器** | `docker start <容器ID或名>` |
| 重启容器 | `docker restart <容器ID或名>` |
| **进入运行中的容器** | `docker exec -it <容器名> /bin/bash` |
| 删除容器（需先停止） | `docker rm <容器ID或名>` |
| 强制删除运行中的容器 | `docker rm -f <容器ID或名>` |
| 查看容器日志 | `docker logs <容器名>` |
| 实时跟踪日志 | `docker logs -f <容器名>` |
| 查看容器内进程 | `docker top <容器名>` |
| 查看容器资源占用 | `docker stats <容器名>`（不加名看所有） |
| 拷贝文件到容器 | `docker cp <本地文件> <容器名>:<容器路径>` |
| 拷贝文件从容器 | `docker cp <容器名>:<容器路径> <本地文件>` |
| 暂停/恢复容器进程 | `docker pause <容器名>` / `docker unpause <容器名>` |
| 等待容器退出 | `docker wait <容器名>` |
| 导出容器文件系统 | `docker export <容器名> -o <文件名.tar>` |
| 从导出文件导入为镜像 | `docker import <文件名.tar> <镜像名:标签>` |

---

## 三、数据卷操作（持久化数据）

| 操作 | 命令 |
| :--- | :--- |
| 创建数据卷 | `docker volume create <卷名>` |
| 查看所有卷 | `docker volume ls` |
| 查看卷详情 | `docker volume inspect <卷名>` |
| 删除未使用的卷 | `docker volume prune` |
| 删除指定卷 | `docker volume rm <卷名>` |
| 挂载卷运行容器 | `docker run -v <卷名>:<容器路径> ...`（新版也可以用 `--mount`） |

---

## 四、网络操作

| 操作 | 命令 |
| :--- | :--- |
| 查看所有网络 | `docker network ls` |
| 创建网络 | `docker network create <网络名>` |
| 删除网络 | `docker network rm <网络名>` |
| 将容器连接到网络 | `docker network connect <网络名> <容器名>` |
| 断开容器网络 | `docker network disconnect <网络名> <容器名>` |
| 查看网络详情 | `docker network inspect <网络名>` |

---

## 五、系统与清理

| 操作 | 命令 |
| :--- | :--- |
| 查看 Docker 占用磁盘 | `docker system df` |
| 一键清理（容器、镜像、网络、缓存） | `docker system prune -a`（加 `-a` 删除所有未使用镜像） |
| 查看 Docker 版本信息 | `docker version` |
| 查看系统信息 | `docker info` |
| 登录 Docker 仓库 | `docker login` |
| 登出仓库 | `docker logout` |

---

## 六、常见场景组合命令

### 1. 开发环境：启动一个 C++ 开发容器，挂载代码目录，进入
```bash
docker run -it --rm -v $(pwd):/workspace -w /workspace my-cpp-dev-env:latest /bin/bash
```
- `-it`：交互终端  
- `--rm`：退出后自动删除容器（方便临时测试）  
- `-v`：挂载当前目录到容器内 `/workspace`  
- `-w`：进入容器后默认工作目录为 `/workspace`

### 2. 后台运行一个 Web 服务（如 nginx），并映射端口
```bash
docker run -d --name my-nginx -p 8080:80 nginx:latest
```

### 3. 查看并进入某个运行中容器
```bash
docker ps                # 得到容器名
docker exec -it <容器名> /bin/bash
```

### 4. 清理所有停止的容器和悬空镜像
```bash
docker container prune
docker image prune
# 或者一步到位（更激进）
docker system prune -a --volumes
```

### 5. 调试容器网络（进入容器用 `ping` 等工具）
```bash
docker run -it --rm alpine /bin/sh
# 然后安装工具：apk add iputils-ping curl
```

---

## 七、实用小技巧

- **别名简化**：在 `~/.bashrc` 中加入 `alias dk='docker'`、`alias dkc='docker exec -it'`  
- **自动补全**：安装 `bash-completion` 和 `docker-completion` 插件  
- **查看容器内环境变量**：`docker exec <容器> env`  
- **在宿主机执行容器内的命令**：`docker exec <容器> ls -l /`  
- **限制容器资源**：`docker run -m 512m --cpus=1 ...`（内存 512M，CPU 1核）  

如果你还想了解 **`docker-compose`** 的常用操作（多容器编排），或者需要针对某个具体场景（比如调试、构建镜像优化）给出命令组合，可以告诉我，我继续帮你梳理。

## core dump

修改 core_pattern 只转储进程自身的内存（不转储文件映射）
如果你不需要分析映射文件，可以在容器内设置 core 文件的过滤：

```bash
echo 0x33 > /proc/self/coredump_filter
```

这个掩码可以控制哪些内存段被包含进 core 文件（比如去掉文件映射段）。但通常不建议这样做，因为可能会丢失有用的信息。
