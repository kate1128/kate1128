#  参考
[Milvus_CLI Command Reference | Milvus Documentation](https://milvus.io/docs/cli_commands.md)

下载：

[Release v1.0.0-20240729 · zilliztech/milvus_cli](https://github.com/zilliztech/milvus_cli/releases/tag/v1.0.0)

版本对应

[Milvus Command-Line Interface Milvus v2.3.x documentation](https://milvus.io/docs/v2.3.x/cli_overview.md)



# 使用
```bash
milvus_cli
```

![](https://cdn.nlark.com/yuque/0/2024/png/2639475/1731897130905-bd98101b-22dc-4047-94ce-b5467ce56e19.png)

```bash
# 连接
connect

connect -uri http://10.0.5.73:31350

# collections 列表
list collections

# collection细则
show collection -c collection1

# 版本
version

# 连接list
list connections

# 数据库 list
list databases
```

![](https://cdn.nlark.com/yuque/0/2024/png/2639475/1731897458128-15deed8a-1ace-4b82-ae1c-20dd8556226b.png)

#  脚本
## 官方提供的迁移脚本
```yaml
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

## 自定义
```bash
curl --request POST   --url "http://10.0.5.73:31350/v2/vectordb/collections/list"   --header "Authorization: Bearer root:Milvus"   --header "Content-Type: application/json"   -d '{
   "dbName": "default"
  }'
```

```bash
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

# 配置 Milvus 连接信息
CLUSTER_ENDPOINT="http://10.0.5.73:31350"
MILVUS_CLI="/data/miniconda3/bin/milvus_cli"  # milvus-cli 工具路径，确保其在 PATH 中

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
done

```





