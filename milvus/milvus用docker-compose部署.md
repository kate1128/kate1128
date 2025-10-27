### 单机
#### 安装
yaml文件：

[Releases · milvus-io/milvus](https://github.com/milvus-io/milvus/releases)

可以展开看一下它部署的组件

```bash
version: '3.5'

services:
  etcd:
    container_name: milvus-etcd-local
    image: quay.io/coreos/etcd:v3.5.5
    environment:
      - ETCD_AUTO_COMPACTION_MODE=revision
      - ETCD_AUTO_COMPACTION_RETENTION=1000
      - ETCD_QUOTA_BACKEND_BYTES=4294967296
      - ETCD_SNAPSHOT_COUNT=50000
    volumes:
      - ${DOCKER_VOLUME_DIRECTORY:-.}/volumes/etcd:/etcd
    command: etcd -advertise-client-urls=http://127.0.0.1:2379 -listen-client-urls http://0.0.0.0:2379 --data-dir /etcd
    healthcheck:
      test: ["CMD", "etcdctl", "endpoint", "health"]
      interval: 30s
      timeout: 20s
      retries: 3

  minio:
    container_name: milvus-minio-local
    image: minio/minio:RELEASE.2023-03-20T20-16-18Z
    environment:
      MINIO_ACCESS_KEY: minioadmin
      MINIO_SECRET_KEY: minioadmin
    ports:
      - "9001:9001"
      - "9000:9000"
    volumes:
      - ${DOCKER_VOLUME_DIRECTORY:-.}/volumes/minio:/minio_data
    command: minio server /minio_data --console-address ":9001"
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:9000/minio/health/live"]
      interval: 30s
      timeout: 20s
      retries: 3

  standalone:
    container_name: milvus-standalone-local
    image: milvusdb/milvus:v2.4.15
    command: ["milvus", "run", "standalone"]
    security_opt:
    - seccomp:unconfined
    environment:
      ETCD_ENDPOINTS: etcd:2379
      MINIO_ADDRESS: minio:9000
    volumes:
      - ${DOCKER_VOLUME_DIRECTORY:-.}/volumes/milvus:/var/lib/milvus
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:9091/healthz"]
      interval: 30s
      start_period: 90s
      timeout: 20s
      retries: 3
    ports:
      - "19530:19530"
      - "9091:9091"
    depends_on:
      - "etcd"
      - "minio"

networks:
  default:
    name: milvus
```

#### 测试
```bash
curl -X GET "http://localhost:9091/healthz"
```

![](https://cdn.nlark.com/yuque/0/2024/png/2639475/1731574159619-b3a80e2e-0ea5-4306-b3fd-561300657ae1.png?x-oss-process=image%2Fformat%2Cwebp)

#### attu
```bash
docker run -d --name=attu -p 8087:3000 -e MILVUS_URL=10.0.5.73:19530 zilliz/attu:latest
```

访问：[http://10.0.5.73:8087/#/](http://10.0.5.73:8087/#/)

```bash
账号信息：root/Milvus
```

![](https://cdn.nlark.com/yuque/0/2024/png/2639475/1731660935215-fddca08f-6190-49fa-8036-acf5517e8365.png)

### 集群（不行）
不行，milvus起不来

```bash
version: "3.7"

services:
  # ETCD 服务（集群协调）
  etcd:
    image: quay.io/coreos/etcd:v3.5.0
    container_name: milvus-etcd-cluster-test
    environment:
      - ALLOW_NONE_AUTHENTICATION=yes
    ports:
      - "2378:2379"
    volumes:
      - ./etcd-data:/etcd-data
    networks:
      - milvus-network

  # MinIO 服务（对象存储）
  minio:
    image: minio/minio:latest
    container_name: milvus-minio-cluster-test
    environment:
      - MINIO_ACCESS_KEY=minio
      - MINIO_SECRET_KEY=minio123
    ports:
      - "9000:9000"
    command: server /data
    volumes:
      - ./minio-data:/data
    networks:
      - milvus-network

  # Root Coordinator
  milvus-root-coord:
    image: milvusdb/milvus:v2.3.0
    container_name: milvus-root-coord
    environment:
      - MILVUS_ETCD_ENDPOINTS=http://etcd:2379
      - MILVUS_MINIO_ADDRESS=minio:9000
      # - MILVUS_MINIO_BUCKET=milvus-bucket
      - MINIO_ACCESS_KEY=minio
      - MINIO_SECRET_KEY=minio123
      - MILVUS_DEPLOY_MODE=cluster
    ports:
      - "19530:19530"  # 根查询端口
    networks:
      - milvus-network
    depends_on:
      - etcd
      - minio

  # Query Coordinator
  milvus-query-coord:
    image: milvusdb/milvus:v2.3.0
    container_name: milvus-query-coord
    environment:
      - MILVUS_ETCD_ENDPOINTS=http://etcd:2379
      - MILVUS_DEPLOY_MODE=cluster
    networks:
      - milvus-network

  # Query Node（多个实例）
  milvus-query-node-1:
    image: milvusdb/milvus:v2.3.0
    container_name: milvus-query-node-1
    environment:
      - MILVUS_ETCD_ENDPOINTS=http://etcd:2379
      - MILVUS_DEPLOY_MODE=cluster
    networks:
      - milvus-network

  milvus-query-node-2:
    image: milvusdb/milvus:v2.3.0
    container_name: milvus-query-node-2
    environment:
      - MILVUS_ETCD_ENDPOINTS=http://etcd:2379
      - MILVUS_DEPLOY_MODE=cluster
    networks:
      - milvus-network

  # Data Node（多个实例）
  milvus-data-node-1:
    image: milvusdb/milvus:v2.3.0
    container_name: milvus-data-node-1
    environment:
      - MILVUS_ETCD_ENDPOINTS=http://etcd:2379
      - MILVUS_DEPLOY_MODE=cluster
    networks:
      - milvus-network

  milvus-data-node-2:
    image: milvusdb/milvus:v2.3.0
    container_name: milvus-data-node-2
    environment:
      - MILVUS_ETCD_ENDPOINTS=http://etcd:2379
      - MILVUS_DEPLOY_MODE=cluster
    networks:
      - milvus-network

  # Index Node
  milvus-index-node:
    image: milvusdb/milvus:v2.3.0
    container_name: milvus-index-node
    environment:
      - MILVUS_ETCD_ENDPOINTS=http://etcd:2379
      - MILVUS_DEPLOY_MODE=cluster
    networks:
      - milvus-network

  # Attu（Milvus UI）
  attu:
    image: zilliz/attu:latest
    container_name: milvus-attu
    environment:
      - SERVER_PORT=8000  # 设置 Attu 端口
    ports:
      - "8087:8000"  # 将8000端口映射到主机
    networks:
      - milvus-network
    depends_on:
      - milvus-root-coord

networks:
  milvus-network:
    driver: bridge

```



