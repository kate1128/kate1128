# 迁移脚本
参考：

[milvus-migration/README_2X.md at main · zilliztech/milvus-migration](https://github.com/zilliztech/milvus-migration/blob/main/README_2X.md#milvus-migration-milvus2x-to-milvus2x)

```bash
#!/bin/bash
connect -uri http://10.0.5.73:31350
collections=("collection1" "collection2" "collection3")

for collection in "${collections[@]}"; do
    echo "BatchMigration==> $collection"
    ./milvus-migration start -t="$collection" -c=/{YourConfigPath}/migration.yml
done

# how to execute?
#1. chmod +x batch_collection_migration.sh
#2. ./batch_collection_migration.sh
```

# 获取所有collections
## restful
```bash
curl --request POST \
  --url "http://10.0.5.73:31350/v2/vectordb/collections/list" \
  --header "Authorization: Bearer root:Milvus" \
  --header "Content-Type: application/json" \
  -d '{
   "dbName": "default"
  }'


curl --request POST \
  --url "${CLUSTER_ENDPOINT}/v2/vectordb/collections/list" \
  --header "Authorization: Bearer ${TOKEN}" \
  --header "Content-Type: application/json" \
  -d '{
   "dbName": "_default"
  }'
```

```bash
#!/bin/bash

CLUSTER_ENDPOINT="http://10.0.5.73:31350"
TOKEN="root:Milvus"

# 获取 collections 列表
response=$(curl --request POST \
  --url "${CLUSTER_ENDPOINT}/v2/vectordb/collections/list" \
  --header "Authorization: Bearer ${TOKEN}" \
  --header "Content-Type: application/json" \
  -d '{
   "dbName": "default"
  }')

# 检查响应是否为空
if [[ -z "$response" ]]; then
    echo "未收到响应，请检查 Milvus 服务是否运行或 API 路径是否正确。"
    exit 1
fi

echo "所有 collections："
echo "$response"

```

## milvus_cli
```bash
milvus_cli <<EOF
connect -uri http://10.0.5.73:31350
list collections
EOF
```

# 批量迁移 milvus-migration
```bash
#!/bin/bash

# 配置 Milvus 连接信息
CLUSTER_ENDPOINT="http://10.0.5.73:31350"
MILVUS_CLI="/data/miniconda3/bin/milvus_cli"  # milvus-cli 工具路径，确保其在 PATH 中
ConfigPath="/root/kate/milvus_migration2.yaml"
MILVUS_MIGRATION="/root/kate"

# 连接到 Milvus 实例并获取 collections 列表
echo "正在连接到 Milvus 实例：$CLUSTER_ENDPOINT..."

collections=$(
$MILVUS_CLI <<EOF | grep -o "\['[^]]*'\]" # 提取方括号里的部分
connect -uri $CLUSTER_ENDPOINT
use database -db default
list collections
EOF
)

# 检查执行结果并输出
if [[ -z "$collections" || "$collections" == "[]" ]]; then
    echo "没有找到任何 collections。"
else
    echo "所有 collections 列表："
    echo "$collections"
fi

# 遍历 collections 列表
# 去掉方括号和引号后逐个输出
collections_clean=$(echo "$collections" | tr -d "[]'" | tr ',' '\n')

for collection in $collections_clean; do
    echo "BatchMigration==> $collection"
    ${MILVUS_MIGRATION}/milvus-migration start -t="$collection" -c=$ConfigPath
done

```

```yaml
dumper:
  worker:
    workMode: milvus2x
    reader:
      bufferSize: 500

meta:
  mode: config
  version: 2.3.0
  # collection: collection1

source:
  milvus2x:
    endpoint: 10.0.5.73:19530
    username: root
    password: Milvus

target:
  milvus2x:
    endpoint: 10.0.5.73:31350
    username: root
    password: Milvus

```

## 第一次恢复
源端：

![](https://cdn.nlark.com/yuque/0/2024/png/2639475/1731916607960-5e5102d8-e2c3-4af2-9ca8-adfd2d9a78cc.png)

目标端：

![](https://cdn.nlark.com/yuque/0/2024/png/2639475/1731916569420-6600899e-b4fc-4d39-8035-64fb3c7e5d01.png)

## 第二次恢复
源端

![](https://cdn.nlark.com/yuque/0/2024/png/2639475/1731917639341-4219f1f3-51ab-4691-b43d-54d42a8349ee.png)

目标端

![](https://cdn.nlark.com/yuque/0/2024/png/2639475/1731917618764-8ed60d02-7d5f-4d00-9324-6599b872fbaf.png)

<br/>color2
不是覆盖的，不会删除原数据

<br/>

# milvus_backup
异机验不通

参考：[https://juejin.cn/post/7370184763678294025](https://juejin.cn/post/7370184763678294025)

[https://www.cnblogs.com/hxlasky/p/18361345](https://www.cnblogs.com/hxlasky/p/18361345)

[https://devpress.csdn.net/k8s/66c9904a13e4054e7e7d4868.html](https://devpress.csdn.net/k8s/66c9904a13e4054e7e7d4868.html)

```bash
$ mc config host list

milvus_minio
  URL       : http://10.0.5.73:9000
  AccessKey : minioadmin
  SecretKey : minioadmin
  API       : s3v4
  Path      : auto

milvus_minio_k8s
  URL       : http://192.169.93.217:9000
  AccessKey : minioadmin
  SecretKey : minioadmin
  API       : s3v4
  Path      : auto

$ mc ls milvus_minio
[2024-11-14 08:48:33 UTC]     0B a-bucket/
[2024-11-15 08:44:07 UTC]     0B backup-bucket/

$ mc ls milvus_minio_k8s
[2024-11-15 06:22:21 UTC]     0B a-bucket/
[2024-11-15 02:52:42 UTC]     0B milvus-bucket/

$ mc ls --versions milvus_minio
[2024-11-14 08:48:33 UTC]     0B a-bucket/
[2024-11-18 08:09:03 UTC]     0B a-bucket/backup/
[2024-11-18 08:09:03 UTC]     0B a-bucket/files/
[2024-11-15 08:44:07 UTC]     0B backup-bucket/
[2024-11-18 08:09:03 UTC]     0B backup-bucket/backup/

$ mc ls --versions milvus_minio_k8s
[2024-11-15 06:22:21 UTC]     0B a-bucket/
[2024-11-18 08:09:14 UTC]     0B a-bucket/backup/
[2024-11-15 02:52:42 UTC]     0B milvus-bucket/
[2024-11-18 08:09:14 UTC]     0B milvus-bucket/file/

```

![](https://cdn.nlark.com/yuque/0/2024/png/2639475/1731916891395-7287b877-6d5d-4b81-84b9-7d835e3dd67f.png)



```bash
./milvus-backup restore -n test_backup01 -s _recover --restore_index=true

/opt/milvus_backup/milvus-backup restore -n bak_db_test_20240815 -d db_test --restore_index=true --config=/opt/milvus_backup/conf/backup.yaml 
```

![](https://cdn.nlark.com/yuque/0/2024/png/2639475/1731923262915-f7adde8a-b913-4b76-97ae-5f539e449599.png)

```yaml
kubectl apply -f https://github.com/jetstack/cert-manager/releases/download/v1.5.3/cert-manager.yaml

kubectl apply -f https://raw.githubusercontent.com/zilliztech/milvus-operator/main/config/samples/milvus_minimum.yaml

```

