# Docker 容器化学习笔记：从镜像分层到 compose 编排

## 一、概念引入：容器是什么

Docker 是一个**容器化**（Containerization）平台，把"应用 + 它运行所需的一切（代码、系统库、运行时、环境变量）"打包成一个可移植的**镜像**，并在宿主机内核上以**隔离的轻量进程**（容器）运行。

一句话理解：**Docker 不是虚拟机，它不虚拟硬件，而是隔离进程**——同一套 Linux 内核，多个容器共享它，但各自有独立的文件系统、进程树、网络、PID 空间，看起来像一台台独立的小机器。

容器与裸机、虚拟机的对比：

| 维度 | 裸机（直接跑在物理机上） | 虚拟机（VM） | 容器（Docker） |
| ---- | ------------------------ | ------------ | -------------- |
| 资源隔离 | 物理级 | 需要 Hypervisor 虚拟硬件 | 内核级（Namespaces + Cgroups） |
| 是否需要客户机 OS | 否 | 每个 VM 一个完整 Guest OS | 否，共享宿主内核 |
| 启动速度 | 分钟级 | 分钟级 | 毫秒~秒级 |
| 镜像尺寸 | — | 数 GB | 数十~数百 MB |
| 资源消耗 | 最低 | 高（虚耗不少内存） | 低，仅进程级开销 |
| 隔离强度 | — | 强（内核隔离） | 中（共享宿主内核） |
| 典型场景 | 物理裸金属跑高性能推理 | 多云租户、强隔离 | 微服务、CI/CD、训练/推理部署 |

结论：**在 AI 领域，Docker 几乎是标准交付物**。你把部署好 CUDA/PyTorch/Milvus 的镜像一推，任何机器 `docker run` 即用，彻底消灭"在我机器上是好的"。

## 二、为什么 AI 工程师必须学 Docker

1. **环境一致性**：模型训练对版本极度敏感（CUDA、cuDNN、驱动、PyTorch），容器把这一切锁进镜像，队友/服务器/云端跑出来的结果才可能复现。
2. **复杂依赖一键部署**：一套 RAG 服务通常需要向量库（Milvus）+ 对象存储（MinIO）+ 元数据（etcd）+ 应用本体，docker-compose 一个命令全部拉起（本知识库 `03-向量数据库与Embedding` 下的部署笔记就大量用到它）。
3. **NVIDIA 容器 GPU 直通**：`nvidia-smi` 在容器里也能看到卡，训练脚本在容器内跑与裸机性能损失极小。
4. **版本回滚与灰度**：镜像即版本，`docker pull <tag>` 即可回滚，配合镜像仓库天然是"一键回退"。

## 三、核心机制：三要素 + 三大技术支柱

### 3.1 镜像 / 容器 / 仓库三要素

- **镜像（Image）**：只读模板，由多层文件系统拼成。类比"安装盘"或"类"。
- **容器（Container）**：镜像运行起来的实例，在最上层加了一个可写层。类比"装好并运行中的系统"或"对象实例"。同一个镜像能启动 N 个互不影响的容器。
- **仓库（Registry）**：集中存储与分发镜像的服务，如 Docker Hub、阿里云镜像、公司私有 Harbor。类比"应用商店"。

### 3.2 Namespaces：进程的"假想独立世界"

内核的 Namespaces 机制让每个容器看到**自己独占的资源视图**：自己的 PID 列表（看不到宿主机进程）、自己的网络栈（独立 IP 端口空间）、自己的文件系统挂载、自己的 hostname。本质是给一组进程做了视图隔离——它们"以为"自己单独拥有一台机器。

### 3.3 Cgroups：资源的"限额手"

Cgroups（Control Groups）负责**限制与统计资源**：给容器限定最多用几个大核、多少内存、多少 IO。防止某个失控的容器把整台服务器拖死——这正是容器比普通多进程更强的资源治理点。

### 3.4 Union FS：镜像分层的原理

Docker 的文件系统用 Union FS（联合文件系统，如 OverlayFS）实现**分层**。Dockerfile 的每条指令生成一个只读层，运行容器时在最上面叠一个可写层。好处：

- 多个容器共享相同底层镜像层，**磁盘只存一份**，秒级启动；
- 构建有缓存：改一行 Dockerfile，只有它及之后的层重新构建。

这也解释了为什么一个 RUN 里连用 `&&`：**每层都会永久保存**，用 `apt-get update && apt-get install` 一条 RUN 完成，干净层少一层缓存旧数据。

## 四、关键知识点速查表

| 维度 | 命令/指令 | 一句话记忆 |
| ---- | --------- | ---------- |
| 拉取/查看 | `docker pull`、`docker images`、`docker rmi` | 下镜像、列镜像、删镜像 |
| 跑容器 | `docker run -d --name app -p 8000:8000 -v ./data:/data img` | 后台 + 命名 + 端口 + 卷 |
| 进容器 | `docker exec -it app bash` | 进容器开 shell，排查必用 |
| 看状态 | `docker ps -a`、`docker logs -f app` | 运行中（-a 含已退出）/ 看日志 |
| 删除 | `docker stop/rm`、`docker system prune` | 停机、删容器、一键清理垃圾 |
| 构建 | `docker build -t myapp:v1 .`、`docker push` | 构建并打标签、推仓库 |
| compose | `docker-compose up -d`、`docker-compose down` | 一键起一组、一键停删一组 |
| GPU | `--gpus all` | 容器内暴露全部 GPU |

## 五、实践示例：日常开发闭环

### 5.1 Dockerfile 常用指令

```dockerfile
# FROM：基础镜像，官方 Python 3.11 运行时
FROM python:3.11-slim

# ENV：环境变量（构建期与运行期都生效）
ENV PYTHONUNBUFFERED=1 \
    TZ=Asia/Shanghai

# WORKDIR：设置工作目录并自动创建
WORKDIR /app

# COPY：把本地文件拷进镜像（不要 COPY 已忽略的权重/密钥，配合 .dockerignore）
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt -i https://pypi.tuna.tsinghua.edu.cn/simple

# 再把源码拷进去：先装依赖再拷源码，能最大化利用构建缓存
COPY . .

# CMD / ENTRYPOINT：容器启动时执行的命令
# CMD 可被 docker run 后的参数覆盖；ENTRYPOINT 作为"入口"固定
ENTRYPOINT ["python"]
CMD ["app.py"]
```

ENTRYPOINT 与 CMD 的关系：ENTRYPOINT 定"怎么跑"（固定命令），CMD 定"跑什么"（可覆盖参数）。例如 ENTRYPOINT `["python"]` + `docker run img train.py`，实际执行 `python train.py`——这样想换脚本只需改 CMD 位置的外部参数。

### 5.2 一条最常用的调试链

```bash
docker build -t rag-app:v0.1 .          # 构建镜像
docker run -d --name rag \
  -p 8000:8000 \                        # -p 宿主机端口:容器端口
  -v /root/kate/data:/app/data \        # -v 挂载数据卷，容器内改宿主文件实时可见
  --restart=unless-stopped \            # 意外挂掉自动拉起
  --gpus all \                          # 用上 GPU
  rag-app:v0.1
docker ps                              # 确认容器起来了
docker logs -f rag                     # 看运行日志
docker exec -it rag bash               # 进容器排障：pip list / nvidia-smi / cat /etc/*-release

# 改完代码不想重新构建？挂载源码目录 + 重启即可：
docker run -d --name rag-dev -v /root/kate/rag:/app rag-app:v0.1
docker restart rag-dev
```

## 六、数据卷与网络简述

- **数据卷（Volume）**：容器是"一次性"的，rm 掉数据全没。卷把宿主机目录（`-v 主机:容器`）或命名卷挂进容器，实现持久化 + 宿主/容器互访。数据库类容器（Milvus、MinIO、etcd）**必须挂卷**，否则重启数据就蒸发。
- **容器网络**：默认 bridge 网络下，同一 compose 项目里的容器用**服务名互相访问**（如 `ETCD_ENDPOINTS=etcd:2379`），不需要 IP；对外服务用 `-p` 暴露端口。不同容器项目默认隔离，可自定义 `networks`。

## 七、实践示例：docker-compose 编排 Milvus + MinIO + etcd

单条 `docker run` 只适合单个容器。真实 RAG 部署往往是"一组服务要联动"，这正是 docker-compose（声明式编排）的用武之地。以下示例脱胎于本知识库 `03-向量数据库与Embedding/04-Milvus向量库/milvus用docker-compose部署.md` 与 `05-Minio对象存储/minio用docker-compose部署.md` 的实战写法：

```yaml
# docker-compose.yml
version: "3.5"

services:
  # 元数据协调：Milvus 依赖 etcd
  etcd:
    container_name: milvus-etcd-local
    image: quay.io/coreos/etcd:v3.5.5
    environment:
      - ETCD_AUTO_COMPACTION_MODE=revision
      - ETCD_AUTO_COMPACTION_RETENTION=1000
    volumes:
      - ./volumes/etcd:/etcd
    command: etcd -advertise-client-urls=http://127.0.0.1:2379 -listen-client-urls http://0.0.0.0:2379 --data-dir /etcd
    healthcheck:
      test: ["CMD", "etcdctl", "endpoint", "health"]
      interval: 30s
      timeout: 20s
      retries: 3

  # 对象存储：Milvus 的数据落在这（MINIO 也是独立对象存储服务）
  minio:
    container_name: milvus-minio-local
    image: minio/minio:RELEASE.2023-03-20T20-16-18Z
    environment:
      MINIO_ACCESS_KEY: minioadmin      # 注意：新版用 MINIO_ROOT_USER 更规范
      MINIO_SECRET_KEY: minioadmin
    ports:
      - "9000:9000"                     # API 端口
      - "9001:9001"                     # web 管理台端口
    volumes:
      - ./volumes/minio:/minio_data
    command: minio server /minio_data --console-address ":9001"
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:9000/minio/health/live"]
      interval: 30s
      timeout: 20s
      retries: 3

  # Milvus 单机 standalone
  standalone:
    container_name: milvus-standalone-local
    image: milvusdb/milvus:v2.4.15
    command: ["milvus", "run", "standalone"]
    security_opt:
      - seccomp:unconfined
    environment:
      ETCD_ENDPOINTS: etcd:2379       # 服务名直连，无需写 IP
      MINIO_ADDRESS: minio:9000
    volumes:
      - ./volumes/milvus:/var/lib/milvus
    ports:
      - "19530:19530"                 # gRPC 对外
      - "9091:9091"                   # 健康检查
    depends_on:                       # 控制启动顺序
      - etcd
      - minio
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:9091/healthz"]
      interval: 30s
      start_period: 90s
      timeout: 20s
      retries: 3

networks:
  default:
    name: milvus
```

```bash
docker-compose up -d        # 一键构建/拉取并后台启动全部服务
docker-compose ps           # 看服务健康状态
docker-compose logs -f standalone   # 看 Milvus 日志
curl -X GET "http://localhost:9091/healthz"   # 访问性验证
docker-compose down         # 停止并移除容器（卷里的数据保留在 ./volumes/）
```

关键点：`depends_on` 控顺序、`healthcheck` 定就绪、服务名当域名、`${DOCKER_VOLUME_DIRECTORY:-.}` 可做目录可配置化。唯一的坑要在实战里注意——**版本敏感**：Milvus 版本间镜像与 etcd/MinIO 组合有兼容性要求，换版本前先查官方 `release` 说明。

## 八、常见坑与小结

1. **时区错 8 小时**：官方镜像默认 UTC。两条路：ENV 里 `TZ=Asia/Shanghai`（有些基础镜像是 tzdata 存好才能生效），或挂载 `/etc/localtime:/etc/localtime:ro`，跑起来先 `date` 验证。
2. **容器内以 root 跑有风险**：默认 root 权限过宽。安全做法是在 Dockerfile 里 `RUN useradd -m app && USER app`，或 run 时 `--user $(id -u):$(id -g)`。数据卷目录还要注意属主一致，否则"Permission denied"。
3. **内存/CPU 飚高拖垮宿主**：容器默认不限资源，运行加 `-m 4g --cpus 2`（compose 里 `mem_limit`、`cpus`），否则训练失控会打爆驱逐 k8s 节点或宿主 OOM。
4. **镜像越堆越胖**：基础镜像优先 `slim`/`alpine`；一条 RUN 完成装包并 `rm -rf /var/lib/apt/lists/*`；多阶段构建（builder 层只留运行产物）。`docker images` 你会发现瘦身后动辄减小 60%+。
5. **容器内 PID 1 的问题**：如果 CMD 直接起 bash 而没前置信号处理，`docker stop` 可能关不掉。尽量让入口进程是程序本身（如 `python app.py`），或用 `init` 类镜像。
6. **`.dockerignore` 必不可少**：把 `__pycache__`、`.git`、权重、密钥加进去，否则既慢又可能泄露机密（和 Git 的 `.gitignore` 一个道理）。
7. **don't `docker-compose down -v` 手滑**：-v 会连数据卷一起删，数据库清空，这操作在别人部署库里是要背锅的。

**小结**：Docker 的三大支柱——**Namespaces 隔离视图、Cgroups 限制资源、Union FS 分层存镜像**，背下这两句架构就清爽了。实操则记住一条链路：**Dockerfile 把环境固化成镜像 → docker run/ps 管理运行实例 → docker-compose 把多服务串成一组项目 → 卷管持久化、端口管对外、网络靠服务名**。本知识库的 Milvus、MinIO、以及后续所有"用 docker-compose 部署"的笔记，都是这章内容的直接应用。

## 九、参考资料

- [Docker 官方文档（Get Started）](https://docs.docker.com/get-started/)
- [docker-compose 官方参考](https://docs.docker.com/compose/compose-file/compose-file-v3/)
- [Docker 入门系列：镜像分层原理](https://docs.docker.com/get-started/09_image_basics/)
- [Milvus 官方安装文档（Standalone / docker-compose）](https://milvus.io/docs/install_standalone-docker.md)
- 本知识库配套：`03-向量数据库与Embedding/04-Milvus向量库/milvus用docker-compose部署.md`、`03-向量数据库与Embedding/05-Minio对象存储/minio用docker-compose部署.md`