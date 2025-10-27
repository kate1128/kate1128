[@凯佳er](undefined/qiaokate)

| 2024年11月11日 | 单机、集群部署 |
| --- | --- |
| 2024年11月12日 | mc、juice 单桶/文件 迁移 |
| 2024年11月13日 | mc多桶迁移 |


# 部署方式
## 单机
```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update

helm search repo minio

helm pull bitnami/minio --version 14.7.0

#helm 安装
helm install minio minio-14.7.0.tgz \
 --namespace kate-test3 --create-namespace \
 --set global.storageClass=nfs-client \
 --set auth.rootUser=admin \
 --set auth.rootPassword="password01" \
 --set service.type=NodePort \
 --wait
```

### 访问地址
访问地址：[http://10.0.5.73:31650/tools/metrics](http://10.0.5.73:31650/tools/metrics)

![](https://cdn.nlark.com/yuque/0/2024/png/2639475/1731310762990-ff7f202e-8896-40ec-b0ea-ce233e65c03c.png)

![](https://cdn.nlark.com/yuque/0/2024/png/2639475/1731310848735-5f0383e8-4721-471f-890c-a0bd1eeea3bb.png)

### 密钥
[credentials.json](https://www.yuque.com/attachments/yuque/0/2024/json/2639475/1734843591814-6e342f12-8254-4c74-a667-50c55a64fd7d.json)

```bash
access-key： ZvOibS01jZrYLobgDEuM
secret-key： 02yHO0gqgRn9vWT8DhzuS6s9OLGnF8UnnRW0F1XU
```

## 集群
```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update

helm search repo minio

helm pull bitnami/minio --version 14.7.0

vi values_test.yaml
mode: distributed


#helm 安装
helm install minio ./minio-14.7.0 \
 --namespace kate-test7 --create-namespace \
 --set global.storageClass=nfs-client \
 --set auth.rootUser=admin \
 --set auth.rootPassword="password01" \
 --set service.type=NodePort \
 --wait

 # -f values-test.yaml
```

### 访问地址
访问地址：[http://10.0.5.73:32278/tools/metrics](http://10.0.5.73:32278/tools/metrics)

![](https://cdn.nlark.com/yuque/0/2024/png/2639475/1731310410771-55648a98-c05f-4648-90b8-9f02eda28a7b.png)

![](https://cdn.nlark.com/yuque/0/2024/png/2639475/1731310496993-65c60fba-e4cf-4a8e-83e6-9fc0e57905e5.png)

### 密钥
[credentials (1).json](https://www.yuque.com/attachments/yuque/0/2024/json/2639475/1734843591891-4dfda022-96a1-44f4-b8db-7f8f9668b2ea.json)

```bash
access-key： OD4zyJJDbF0NVQxgxnS6
secret-key： t9Q1YCnpUQMtA4jQgNMKxcdGFf0kiJ0L0jfEvLci
```

# 工具选择
## mc
### 安装mc
```bash
wget https://dl.min.io/client/mc/release/linux-amd64/mc
```

![](https://cdn.nlark.com/yuque/0/2024/png/2639475/1731316322116-6b08350a-4d35-4ac4-8aa6-87e73d3ea052.png)

```bash
chmod a+x mc

mc --version
```

![](https://cdn.nlark.com/yuque/0/2024/png/2639475/1731316344754-86877f82-1c99-43dc-bdc1-16d60d00deab.png)

### mc cp
#### 备份
```bash
kubectl get pods,svc -n kate-test3 -o wide
```

![](https://cdn.nlark.com/yuque/0/2024/png/2639475/1731316391255-515d8b21-7fd1-46e5-8bf8-b67b1a920fd6.png)

```bash
# 配置 mc 连到单个 Pod 的 minio
# 参数：podIp  port  access-key secret-key
mc alias set local http://192.168.20.61:9000 ZvOibS01jZrYLobgDEuM 02yHO0gqgRn9vWT8DhzuS6s9OLGnF8UnnRW0F1XU

# 创建一个备份存储桶
mc mb local/backup

# 把数据复制到备份存储桶
mc cp --recursive local/kate-test local/backup/kate-test
```

![](https://cdn.nlark.com/yuque/0/2024/png/2639475/1731316440517-0067f158-7ffb-4ac1-8c7b-6937b641f7eb.png)

![](https://cdn.nlark.com/yuque/0/2024/png/2639475/1731316698286-e37b1bee-2eca-4c10-956d-21870b428fdb.png)

#### 恢复
```bash
kubectl get pod,svc -n kate-test7 -o wide
```

![](https://cdn.nlark.com/yuque/0/2024/png/2639475/1731316536529-5de341b5-e7ba-408e-8726-7bcbaec5da73.png)

```bash
# 配置 mc 连到单个 Pod 的 minio
# 参数：podIp  port  access-key secret-key
mc alias set cluster http://192.168.20.16:9000 OD4zyJJDbF0NVQxgxnS6 t9Q1YCnpUQMtA4jQgNMKxcdGFf0kiJ0L0jfEvLci

# 将备份数据复制到 StatefulSet 的 minio
# 可以选择将数据复制到每个pod，或者复制到一个pod，分发给其他pod
mc cp --recursive local/backup/kate-test cluster/restore
```

![](https://cdn.nlark.com/yuque/0/2024/png/2639475/1731316575373-13cc3dfc-33a2-4143-aa9e-f53078d3b7de.png)

![](https://cdn.nlark.com/yuque/0/2024/png/2639475/1731316662372-286df52c-b8af-4230-9b61-6631e6c69344.png)



### mc mirror
#### 单bucket迁移
```bash
# local
mc alias set local http://192.168.20.61:9000 ZvOibS01jZrYLobgDEuM 02yHO0gqgRn9vWT8DhzuS6s9OLGnF8UnnRW0F1XU

# cluster
mc alias set cluster http://192.168.20.16:9000 OD4zyJJDbF0NVQxgxnS6 t9Q1YCnpUQMtA4jQgNMKxcdGFf0kiJ0L0jfEvLci

# 创建桶
mc mb cluster/backup2

# 异minio迁移
mc mirror --overwrite --remove sourceMinIOTest/backup targetMinIOTest/backup2

# 同一个minio
mc mirror --overwrite --remove local/kate-overwrite local/backup2/backup-bucket

```

![](https://cdn.nlark.com/yuque/0/2024/png/2639475/1731394015893-01aad864-79a1-4022-9455-e62a860e2cfd.png)

同一个minio，由kate-overwrite备份到backup2，只要kate-overwrite有新增，迁移就会完全覆盖backup2里的内容

![](https://cdn.nlark.com/yuque/0/2024/png/2639475/1731394055826-8d2cca5e-ad54-4f54-876b-063784dfbf70.png)

![](https://cdn.nlark.com/yuque/0/2024/png/2639475/1731394279064-c89e45f9-c91b-49a8-a178-1c25e1c52ce7.png)

如果目标端的桶backup2新增，再同步数据不变化

![](https://cdn.nlark.com/yuque/0/2024/png/2639475/1731394390950-6c666199-d687-473f-9635-9c3a7021a70f.png)

如果源端桶里数据新增，那backup2里与源不符合的数据就被删除了

![](https://cdn.nlark.com/yuque/0/2024/png/2639475/1731394427078-b05e82a1-5fbd-4cba-87fc-fc62a2f1dcd8.png)

#### 多桶
```bash
# 设置 sourceMinIO
mc alias set sourceMinIO http://192.168.20.44:9000 ZvOibS01jZrYLobgDEuM 02yHO0gqgRn9vWT8DhzuS6s9OLGnF8UnnRW0F1XU

# 设置 targetMinIO
mc alias set targetMinIO http://192.168.20.40:9000 OD4zyJJDbF0NVQxgxnS6 t9Q1YCnpUQMtA4jQgNMKxcdGFf0kiJ0L0jfEvLci

# sourceMinIO所有桶迁移到
mc mirror --watch sourceMinIO/ targetMinIO/
```

![](https://cdn.nlark.com/yuque/0/2024/png/2639475/1731467864071-6544f1e7-359e-4f97-8778-8ffde4a3f7a2.png)

![](https://cdn.nlark.com/yuque/0/2024/png/2639475/1731467823539-4921ad34-f392-4e7c-93ed-2bb26987bacf.png)

![](https://cdn.nlark.com/yuque/0/2024/png/2639475/1731467787852-66b10113-6313-4073-8dd6-bf372d67c553.png)

### mc replicate
#### 单桶迁移
```bash
# 设置 sourceMinIO
mc alias set sourceMinIO http://192.168.20.44:9000 ZvOibS01jZrYLobgDEuM 02yHO0gqgRn9vWT8DhzuS6s9OLGnF8UnnRW0F1XU

# 设置 targetMinIO
mc alias set targetMinIO http://192.168.20.40:9000 OD4zyJJDbF0NVQxgxnS6 t9Q1YCnpUQMtA4jQgNMKxcdGFf0kiJ0L0jfEvLci

# 设置版本可用
mc version enable sourceMinIO/replicate

mc version enable targetMinIO/replicate

# 给一个复制任务
mc replicate add sourceMinIO/replicate --remote-bucket targetMinIO/replicate

# 查看minio实例的桶
mc ls sourceMinIO | awk '{print $5}'

mc ls targetMinIO | awk '{print $5}'
```

![](https://cdn.nlark.com/yuque/0/2024/png/2639475/1731478019483-3d7fbb0c-1df5-43e0-a212-be369dd9c581.png)

源Minio

![](https://cdn.nlark.com/yuque/0/2024/png/2639475/1731478038266-6403081f-aa8e-471f-a4c2-53daac27c735.png)

目标Minio

![](https://cdn.nlark.com/yuque/0/2024/png/2639475/1731478052203-af3da8ce-b625-481d-bad8-81eac83804b3.png)

#### 源端新增
![](https://cdn.nlark.com/yuque/0/2024/png/2639475/1731480983611-892d4a61-f995-46dd-8272-fb271743fcb2.png)

目标端也立刻新增

![](https://cdn.nlark.com/yuque/0/2024/png/2639475/1731480966667-97a6f8a9-e6a9-44d0-86cc-3b455950f2e0.png)

#### 源端删除
![](https://cdn.nlark.com/yuque/0/2024/png/2639475/1731480475526-901b249c-dae1-4d6d-a9e7-386aa67fbd2e.png)

![](https://cdn.nlark.com/yuque/0/2024/png/2639475/1731480634929-7c082952-592a-4f6d-822b-f2b603b4a6ab.png)

目的端也是删除状态

![](https://cdn.nlark.com/yuque/0/2024/png/2639475/1731480810119-96fb5641-77c7-4b56-8a15-33bed3cada2b.png)

![](https://cdn.nlark.com/yuque/0/2024/png/2639475/1731480570535-2320bb04-6975-468e-be3a-76c06b118523.png)

## juicesync
### 安装 `JuiceSync`
```bash
curl -sSL https://d.juicefs.com/install | sh -
```

![](https://cdn.nlark.com/yuque/0/2024/png/2639475/1731402347450-1cfaddf5-eda0-4658-baf4-e8020b57a384.png)

### 本地文件迁移到目标minio
```bash
juicefs sync ./example minio://ZvOibS01jZrYLobgDEuM:02yHO0gqgRn9vWT8DhzuS6s9OLGnF8UnnRW0F1XU@192.168.20.61:9000/backup
```

![](https://cdn.nlark.com/yuque/0/2024/png/2639475/1731397964613-59dacc7c-a439-409b-ac98-83a36a843e69.png)

### 单bucket迁移
```bash
juicefs sync minio://ZvOibS01jZrYLobgDEuM:02yHO0gqgRn9vWT8DhzuS6s9OLGnF8UnnRW0F1XU@192.168.20.61:9000/backup/ minio://OD4zyJJDbF0NVQxgxnS6:t9Q1YCnpUQMtA4jQgNMKxcdGFf0kiJ0L0jfEvLci@192.168.20.16:9000/restore-overwrite/

```

![](https://cdn.nlark.com/yuque/0/2024/png/2639475/1731398399263-d28a749a-7adb-44ce-8ab7-6d12c4380ba1.png)

![](https://cdn.nlark.com/yuque/0/2024/png/2639475/1731398375994-aabd5973-4e2e-4c82-8757-d949df7768d1.png)

### 多桶（不行）
`juicefs sync`和`juicesync`都没有直接迁移多桶的命令，以下执行不通

```bash
juicefs sync --multi-buckets minio://ACCESS_KEY:SECRET_KEY@source-endpoint/ minio://ACCESS_KEY:SECRET_KEY@destination-endpoint/

# eg
juicefs sync --multi-buckets minio://ZvOibS01jZrYLobgDEuM:02yHO0gqgRn9vWT8DhzuS6s9OLGnF8UnnRW0F1XU@192.168.20.61:9000/ minio://OD4zyJJDbF0NVQxgxnS6:t9Q1YCnpUQMtA4jQgNMKxcdGFf0kiJ0L0jfEvLci@192.168.20.16:9000/

# eg
juicesync --multi-buckets minio://ZvOibS01jZrYLobgDEuM:02yHO0gqgRn9vWT8DhzuS6s9OLGnF8UnnRW0F1XU@192.168.20.61:9000/ minio://OD4zyJJDbF0NVQxgxnS6:t9Q1YCnpUQMtA4jQgNMKxcdGFf0kiJ0L0jfEvLci@192.168.20.16:9000/
```

## 总结
+ `mc cp` 可以单向复制文件或目录，指定某个桶把数据备份到另一个桶。

```bash
mc cp --recursive 别名/桶名/文件名 别名/桶名/文件名
```

+ `mc mirror` 可以同步两个存储桶内容，也可以用--watch去实时同步全部数据。

```bash
mc mirror --overwrite --remove 别名/桶名 别名/桶名

mc mirror --watch sourceMinIO/ targetMinIO/
```

+ `mc replicate` 可以配置数据复制，它可以在两个存储之间持续的做数据复制。

```bash
mc replicate add 别名/桶名 --remote-bucket 别名/桶名
```

+ `juicefs sync`/ `juicesync`这两个使用上几乎一致

```bash
# minio的格式
minio://access-key:secret-key@ip:端口/桶名/

# 同步命令
juicefs sync minio://ZvOibS01jZrYLobgDEuM:02yHO0gqgRn9vWT8DhzuS6s9OLGnF8UnnRW0F1XU@192.168.20.61:9000/backup/ minio://OD4zyJJDbF0NVQxgxnS6:t9Q1YCnpUQMtA4jQgNMKxcdGFf0kiJ0L0jfEvLci@192.168.20.16:9000/restore-overwrite/

```

# <font style="color:rgb(28, 30, 33);">迁移</font>
## 实时增量同步
`backup_script.sh`

```bash
#!/bin/bash

# 设置源和目标 MinIO 别名
# 设置 sourceMinIO
mc alias set sourceMinIO http://192.168.20.44:9000 ZvOibS01jZrYLobgDEuM 02yHO0gqgRn9vWT8DhzuS6s9OLGnF8UnnRW0F1XU

# 设置 targetMinIO
mc alias set targetMinIO http://192.168.20.40:9000 OD4zyJJDbF0NVQxgxnS6 t9Q1YCnpUQMtA4jQgNMKxcdGFf0kiJ0L0jfEvLci

# 备份所有 bucket
mc mirror --watch sourceMinIO/ targetMinIO/

echo "Backup completed at $(date)"
```



![](https://cdn.nlark.com/yuque/0/2024/png/2639475/1731479157532-9fd0c6d2-3798-49e4-9128-9512f0fc023b.png)

![](https://cdn.nlark.com/yuque/0/2024/png/2639475/1731479191011-ce51d70e-6931-4b0b-a2d0-30eea1e7af09.png)

![](https://cdn.nlark.com/yuque/0/2024/png/2639475/1731479206252-508a00d3-e984-4bc0-8df3-d8d37307556c.png)

## 定时任务
```bash
crontab -e

# 添加以下行以每天凌晨 2 点运行备份脚本
0 2 * * * /path/to/backup_script.sh >> /path/to/backup_log.txt 2>&1
```

## 复制任务
`configure_replication.sh`

```bash
#!/bin/bash

# 获取所有 bucket 列表
buckets=$(mc ls sourceMinIO | awk '{print $5}')

# 为每个 bucket 配置异地复制
for bucket in $buckets; do
    # mc replicate add sourceMinIO/replicate --remote-bucket targetMinIO/replicate
    mc version enable sourceMinIO/$bucket
    mc replicate add sourceMinIO/$bucket --remote-bucket targetMinIO/$bucket 
    echo "Replication configured for bucket: $bucket"
done
```

```bash
bash configure_replication.sh
```

## 数据一致性检查
```bash
#!/bin/bash

# 检查源和目标 MinIO 集群中的 bucket 列表是否一致
source_buckets=$(mc ls sourceMinIO | awk '{print $5}')
target_buckets=$(mc ls targetMinIO | awk '{print $5}')

if [ "$source_buckets" == "$target_buckets" ]; then
    echo "Bucket lists are consistent between source and target."
else
    echo "Bucket lists are inconsistent. Please investigate."
fi

# 检查每个 bucket 中的数据一致性
for bucket in $source_buckets; do
    source_hash=$(mc find sourceMinIO/$bucket --exec "md5sum {}" | sort | md5sum)
    target_hash=$(mc find targetMinIO/$bucket --exec "md5sum {}" | sort | md5sum)

    if [ "$source_hash" == "$target_hash" ]; then
        echo "Data in bucket $bucket is consistent."
    else
        echo "Data in bucket $bucket is inconsistent. Please investigate."
    fi
done

```

## 检查bucket是否存在
```bash
#!/bin/bash

# Define the MinIO instance alias and the bucket name
MINIO_INSTANCE="myminio"
BUCKET_NAME="bucketA"

# Function to check if the bucket exists
check_bucket() {
    # Use mc ls to list the buckets and grep for the desired bucket name
    if mc ls "$MINIO_INSTANCE" | grep -q "$BUCKET_NAME"; then
        echo "Bucket '$BUCKET_NAME' exists."
        return 0  # Bucket exists
    else
        echo "Bucket '$BUCKET_NAME' does not exist."
        return 1  # Bucket does not exist
    fi
}

# Call the function to check if the bucket exists
if check_bucket; then
    echo "Bucket already exists, exiting."
else
    # If the bucket does not exist, create it
    echo "Creating bucket '$BUCKET_NAME'..."
    mc mb "$MINIO_INSTANCE"/"$BUCKET_NAME"
    if [ $? -eq 0 ]; then
        echo "Bucket '$BUCKET_NAME' created successfully."
    else
        echo "Failed to create bucket '$BUCKET_NAME'."
    fi
fi

```

# 常用命令
```bash
# 查看mc config
mc config host list

# 检查特定对象的版本信息
mc stat <alias>/<bucket-name>/<object-name>
mc stat targetMinIO/replicate
mc stat targetMinIO/replicate/debug.log

# 检查存储桶的版本控制状态
mc ls --versions <alias>/<bucket-name>
mc ls --versions sourceMinIO/replicate

# 查看桶
mc ls sourceMinIO | awk '{print $5}'

# 查看桶
mc ls targetMinIO | awk '{print $5}'

# 删除桶
mc rb targetMinIOTest/backup2 --force

# 创建桶
mc mb targetMinIO/backup2/
```

# 执行脚本
```bash
#!/bin/bash

# 设置源和目标 MinIO 别名
# 设置 sourceMinIO
mc alias set sourceMinIOTest http://10.0.5.73:30272 ZvOibS01jZrYLobgDEuM 02yHO0gqgRn9vWT8DhzuS6s9OLGnF8UnnRW0F1XU

# 设置 targetMinIO
mc alias set targetMinIOTest http://10.0.5.73:30370 OD4zyJJDbF0NVQxgxnS6 t9Q1YCnpUQMtA4jQgNMKxcdGFf0kiJ0L0jfEvLci

# 获取所有 bucket 列表
buckets=$(mc ls sourceMinIOTest | awk '{print $5}')

# 备份所有 bucket
for bucket in $buckets; do
  echo "Checking if bucket $bucket exists in target..."

  # 如果 target 中不存在 bucket，则创建它
  if ! mc ls targetMinIOTest/$bucket > /dev/null 2>&1; then
    mc mb targetMinIOTest/$bucket
    echo "Created bucket: $bucket in targetMinIO"
  fi

  # 同步数据
  mc mirror --overwrite --remove sourceMinIOTest/$bucket targetMinIOTest/$bucket
  echo "Backup completed for bucket: $bucket at $(date)"
done

```

# 参考
1. mc使用参考：[https://www.cnblogs.com/evescn/p/16242023.html](https://www.cnblogs.com/evescn/p/16242023.html)
2. mc client中文文档：[https://www.minio.org.cn/docs/minio/linux/reference/minio-mc/mc-mirror.html](https://www.minio.org.cn/docs/minio/linux/reference/minio-mc/mc-mirror.html)
3. mc mirror使用参考：[https://juejin.cn/post/7388305846350446646](https://juejin.cn/post/7388305846350446646)
4. juicesync官方：[https://juicefs.com/docs/zh/community/getting-started/installation/#%E7%B1%BB-unix-%E5%AE%A2%E6%88%B7%E7%AB%AF](https://juicefs.com/docs/zh/community/getting-started/installation/#%E7%B1%BB-unix-%E5%AE%A2%E6%88%B7%E7%AB%AF)
5. minio用helm集群部署value修改指南：[https://github.com/bitnami/charts/tree/main/bitnami/minio](https://github.com/bitnami/charts/tree/main/bitnami/minio)

