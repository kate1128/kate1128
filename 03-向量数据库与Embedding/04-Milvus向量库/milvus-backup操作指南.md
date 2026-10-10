异机备份恢复的操作指南

### 安装 milvus backup
文档：[https://milvus.io/docs/v2.3.x/milvus_backup_overview.md](https://milvus.io/docs/v2.3.x/milvus_backup_overview.md)

安装地址：[https://github.com/zilliztech/milvus-backup/releases](https://github.com/zilliztech/milvus-backup/releases)

### 目录结构
```bash
-workspace
  -milvus-backup
  -logs
    -backup.log
    -restore.log
  -configs
    -backup.yaml #默认是backup.yaml
    -restore.yaml
```

### 备份环境
attu：[http://10.0.5.73:8086/#/](http://10.0.5.73:8086/#/)

![](https://cdn.nlark.com/yuque/0/2024/png/2639475/1733118084337-c1d4a7bb-20fd-4c68-b9a7-e15395d11de2.png)

#### 源 milvus
+ `address: 192.169.222.187  `
+ `port: 19530`
+ `user: "root" `
+ `password: "Milvus"`

#### 源 milvus 的 minio
+ `address: 192.169.153.2`
+ `port: 9000`
+ `accessKeyID: minioadmin`
+ `secretAccessKey: minioadmin`

#### 备份目的地 minio
+ `backupAddress: 192.169.153.2`
+ `backupPort: 9000`
+ `backupAccessKeyID: minioadmin`
+ `backupSecretAccessKey: minioadmin`

### 备份操作
#### 初始化 minio
```bash
$ mc alias set standalone http://192.169.153.2:9000 minioadmin minioadmin
Added `standalone` successfully.


```

#### 查看备份环境的二进制日志的位置，配置到`backup.yaml`
+ `bucketName: "my-release"`
+ `rootPath: "files"`

```bash
$ mc ls --versions standalone
[2024-11-19 09:24:22 UTC]     0B a-bucket/
[2024-11-19 03:53:39 UTC]     0B my-release/
[2024-11-20 09:32:43 UTC]     0B my-release/files/

$ mc ls --versions standalone/my-release/files/
[2024-11-20 09:35:01 UTC]     0B insert_log/
[2024-11-20 09:35:01 UTC]     0B stats_log/
```

#### 写`backup.yaml`
配置文件：`backup.yaml`

##### 指定 milvus
+ `address: 192.169.222.187  `
+ `port: 19530`
+ `user: "root" `
+ `password: "Milvus"`

##### milvus 的 minio
+ `address: 192.169.153.2`
+ `port: 9000`
+ `accessKeyID: minioadmin`
+ `secretAccessKey: minioadmin`
+ `bucketName: "my-release"`（根据实际情况配置）
+ `rootPath: "files"`（根据实际情况配置）

##### 备份目的地 minio
+ `backupAddress: 192.169.153.2`
+ `backupPort: 9000`
+ `backupAccessKeyID: minioadmin`
+ `backupSecretAccessKey: minioadmin`
+ `backupBucketName: "minio-backup-bucket"`
+ `backupRootPath: "backup-path"`

```bash
# Configures the system log output.
log:
  level: info # Only supports debug, info, warn, error, panic, or fatal. Default 'info'.
  console: true # whether print log to console
  file:
    rootPath: "logs/backup.log"

http:
  simpleResponse: true

# milvus proxy address, compatible to milvus.yaml
milvus:
  address: 192.169.222.187
  port: 19530
  authorizationEnabled: false
  # tls mode values [0, 1, 2]
  # 0 is close, 1 is one-way authentication, 2 is two-way authentication.
  tlsMode: 0
  user: "root"
  password: "Milvus"

# minio的相关配置，它负责Milvus的数据持久化。
minio:
  # Milvus存储配置，使它们与Milvus配置相同
  storageType: "minio" # support storage type: local, minio, s3, aws, gcp, ali(aliyun), azure, tc(tencent)
  address: 192.169.153.2 # Address of MinIO/S3, source ip
  port: 9000   # Port of MinIO/S3
  accessKeyID: minioadmin  # accessKeyID of MinIO/S3
  secretAccessKey: minioadmin # MinIO/S3 encryption string
  useSSL: false # Access to MinIO/S3 with SSL
  useIAM: false
  iamEndpoint: ""
  bucketName: "my-release" # Milvus MinIO/S3中的桶名，与Milvus实例保持一致
  rootPath: "files" #  Milvus在MinIO/S3中的存储根路径，使其与您的Milvus实例相同

  # Backup storage configs，用于存放备份数据的存储
  backupStorageType: "minio" # support storage type: local, minio, s3, aws, gcp, ali(aliyun), azure, tc(tencent)
  backupAddress: 192.169.153.2 # Address of MinIO/S3, target ip
  backupPort: 9000   # Port of MinIO/S3
  backupAccessKeyID: minioadmin  # accessKeyID of MinIO/S3
  backupSecretAccessKey: minioadmin # MinIO/S3 encryption string
  backupBucketName: "minio-backup-bucket" #  #存放备份数据的桶名. Backup data will store to backupBucketName/backupRootPath
  backupRootPath: "backup-path" #  #存放备份数据的根路径. Backup data will store to backupBucketName/backupRootPath

  # 如果需要在两个不同存储系统之间备份或恢复数据，不支持客户端直接复制。
  # 将此选项设置为 true 以启用通过 Milvus Backup 进行数据传输。
  # 注意：如果 `minio.storageType` 和 `minio.backupStorageType` 不同，此选项将自动设置为 true。
  # 但是，如果它们相同但属于不同的服务，则必须手动将此选项设置为“true”
  # If you need to back up or restore data between two different storage systems, direct client-side copying is not supported.
  # Set this option to true to enable data transfer through Milvus Backup.
  # Note: This option will be automatically set to true if `minio.storageType` and `minio.backupStorageType` differ.
  # However, if they are the same but belong to different services, you must manually set this option to `true`.
  crossStorage: "true"

backup:
  # 最大分段组大小，以字节为单位。默认为 2GB。
  maxSegmentGroupSize: 2G

  # 并行性配置
  parallelism:
    # collection level parallelism to backup 备份的集合级并行度
    backupCollection: 4
    # thread pool to copy data. reduce it if blocks your storage's network bandwidth 用于复制数据的线程池。如果阻塞存储的网络带宽，则减少它
    copydata: 128
    # Collection level parallelism to restore 恢复集合级并行度
    restoreCollection: 2

  # keep temporary files during restore, only use to debug 恢复期间保留临时文件，仅用于调试
  keepTempFiles: false

  # Pause GC during backup through Milvus Http API. 通过 Milvus Http API 备份时暂停 GC
  gcPause:
    enable: true
    seconds: 7200
    address: http://192.169.222.187:9091


```

#### 执行备份任务
备份任务名：`backup_standalone`

```bash
$ ./milvus-backup create -n backup_standalone --config=/root/kate/workspace/configs/backup.yaml

```

#### 执行后查看备份目录
`backup_standalone`目录下存在`binlogs`日志

```bash
$ mc ls --versions standalone
[2024-11-19 09:24:22 UTC]     0B a-bucket/
[2024-11-20 09:33:58 UTC]     0B minio-backup-bucket/
[2024-11-20 09:34:08 UTC]     0B minio-backup-bucket/backup-path/
[2024-11-19 03:53:39 UTC]     0B my-release/
[2024-11-20 09:34:08 UTC]     0B my-release/files/

$ mc ls --versions standalone/minio-backup-bucket/backup-path/backup_standalone/
[2024-11-20 09:35:45 UTC]     0B binlogs/
[2024-11-20 09:35:45 UTC]     0B meta/
```

#### 查看备份配置
```bash
./milvus-backup list
./milvus-backup get -n backup_standalone
```

### 恢复环境
attu：[http://10.0.5.73:8085/#/databases/database2/collection/overview](http://10.0.5.73:8085/#/databases/database2/collection/overview)

![](https://cdn.nlark.com/yuque/0/2024/png/2639475/1733118204424-ed32dbbe-4e46-40a8-8af2-fae2cc292526.png)

#### 目的地 milvus
+ `address: 192.169.20.106  `
+ `port: 19530`
+ `user: "root" `
+ `password: "Milvus"`

#### 目的地 milvus 的 minio
+ `address: 192.169.55.81`
+ `port: 9000`
+ `accessKeyID: minioadmin`
+ `secretAccessKey: minioadmin`
+ `bucketName: "my-release"`（根据实际情况配置）
+ `rootPath: "files"`（根据实际情况配置）

#### 恢复数据来源 minio
+ `backupAddress: 192.169.55.81`
+ `backupPort: 9000`
+ `backupAccessKeyID: minioadmin`
+ `backupSecretAccessKey: minioadmin`
+ `backupBucketName: "minio-backup-bucket"`
+ `backupRootPath: "backup-path"`

### 恢复操作
#### 初始化 minio
```bash
$ mc alias set distributed http://192.169.55.81:9000 minioadmin minioadmin
Added `distributed` successfully.
```

#### 查看恢复目的地目录，把这个配置到`restore.yaml`
+ `bucketName: "my-release"`
+ `rootPath: "files"`

```bash
$ mc ls --versions distributed
[2024-11-19 06:49:25 UTC]     0B my-release/
```

#### 创建恢复目录
```bash
$ mc mb distributed/minio-backup-bucket
Bucket created successfully `distributed/minio-backup-bucket`.

$ mc ls --versions distributed
[2024-11-20 09:38:06 UTC]     0B minio-backup-bucket/
[2024-11-19 06:49:25 UTC]     0B my-release/
```

#### 将备份数据导入到恢复目的地
```bash
# 直接同步备份库
$ mc mirror standalone/minio-backup-bucket distributed/minio-backup-bucket
...andalone02/meta/segment_meta.json: 133.12 KiB / 133.12 KiB ┃▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓┃ 764.69 KiB/s 0s

# 或者将刚才的任务目录导过来
$ mc cp --recursive standalone/minio-backup-bucket/backup-path/backup_standalone/ distributed/minio-backup-bucket/backup-path/backup_standalone/


```

#### 查看数据是否已存在
```bash
$ mc ls --versions distributed
[2024-11-20 09:38:06 UTC]     0B minio-backup-bucket/
[2024-11-20 09:38:49 UTC]     0B minio-backup-bucket/backup-path/
[2024-11-19 06:49:25 UTC]     0B my-release/

$ mc ls --versions distributed/minio-backup-bucket/backup-path/
[2024-11-20 09:38:54 UTC]     0B backup_standalone/

$ mc ls --versions distributed/minio-backup-bucket/backup-path/backup_standalone/
[2024-11-20 09:39:00 UTC]     0B binlogs/
[2024-11-20 09:39:00 UTC]     0B meta/
```

#### 写`restore.yaml`
配置文件：`restore.yaml`

##### 目的地 milvus
+ `address: 192.169.20.106  `
+ `port: 19530`
+ `user: "root" `
+ `password: "Milvus"`

##### 目的地 milvus 的 minio
+ `address: 192.169.55.81`
+ `port: 9000`
+ `accessKeyID: minioadmin`
+ `secretAccessKey: minioadmin`
+ `bucketName: "my-release"`（根据实际情况配置）
+ `rootPath: "files"`（根据实际情况配置）

##### 恢复数据来源 minio
+ `backupAddress: 192.169.55.81`
+ `backupPort: 9000`
+ `backupAccessKeyID: minioadmin`
+ `backupSecretAccessKey: minioadmin`
+ `backupBucketName: "minio-backup-bucket"`
+ `backupRootPath: "backup-path"`

```bash
# Configures the system log output.
log:
  level: info # Only supports debug, info, warn, error, panic, or fatal. Default 'info'.
  console: true # whether print log to console
  file:
    rootPath: "logs/restore.log"

http:
  simpleResponse: true

# milvus proxy address, compatible to milvus.yaml
milvus:
  address: 192.169.20.106
  port: 19530
  authorizationEnabled: false
  # tls mode values [0, 1, 2]
  # 0 is close, 1 is one-way authentication, 2 is two-way authentication.
  tlsMode: 0
  user: "root"
  password: "Milvus"

# minio的相关配置，它负责Milvus的数据持久化。
minio:
  # Milvus存储配置，使它们与Milvus配置相同
  storageType: "minio" # support storage type: local, minio, s3, aws, gcp, ali(aliyun), azure, tc(tencent)
  address: 192.169.55.81 # Address of MinIO/S3, source ip
  port: 9000   # Port of MinIO/S3
  accessKeyID: minioadmin  # accessKeyID of MinIO/S3
  secretAccessKey: minioadmin # MinIO/S3 encryption string
  useSSL: false # Access to MinIO/S3 with SSL
  useIAM: false
  iamEndpoint: ""
  bucketName: "my-release" # Milvus MinIO/S3中的桶名，与Milvus实例保持一致
  rootPath: "files" #  Milvus在MinIO/S3中的存储根路径，使其与您的Milvus实例相同

  # Backup storage configs，用于存放备份数据的存储
  backupStorageType: "minio" # support storage type: local, minio, s3, aws, gcp, ali(aliyun), azure, tc(tencent)
  backupAddress: 192.169.55.81 # Address of MinIO/S3, target ip
  backupPort: 9000   # Port of MinIO/S3
  backupAccessKeyID: minioadmin  # accessKeyID of MinIO/S3
  backupSecretAccessKey: minioadmin # MinIO/S3 encryption string
  backupBucketName: "minio-backup-bucket" #  #存放备份数据的桶名. Backup data will store to backupBucketName/backupRootPath
  backupRootPath: "backup-path" #  #存放备份数据的根路径. Backup data will store to backupBucketName/backupRootPath

  # 如果需要在两个不同存储系统之间备份或恢复数据，不支持客户端直接复制。
  # 将此选项设置为 true 以启用通过 Milvus Backup 进行数据传输。
  # 注意：如果 `minio.storageType` 和 `minio.backupStorageType` 不同，此选项将自动设置为 true。
  # 但是，如果它们相同但属于不同的服务，则必须手动将此选项设置为“true”
  # If you need to back up or restore data between two different storage systems, direct client-side copying is not supported.
  # Set this option to true to enable data transfer through Milvus Backup.
  # Note: This option will be automatically set to true if `minio.storageType` and `minio.backupStorageType` differ.
  # However, if they are the same but belong to different services, you must manually set this option to `true`.
  crossStorage: "true"

backup:
  # 最大分段组大小，以字节为单位。默认为 2GB。
  maxSegmentGroupSize: 2G

  # 并行性配置
  parallelism:
    # collection level parallelism to backup 备份的集合级并行度
    backupCollection: 4
    # thread pool to copy data. reduce it if blocks your storage's network bandwidth 用于复制数据的线程池。如果阻塞存储的网络带宽，则减少它
    copydata: 128
    # Collection level parallelism to restore 恢复集合级并行度
    restoreCollection: 2

  # keep temporary files during restore, only use to debug 恢复期间保留临时文件，仅用于调试
  keepTempFiles: false

  # Pause GC during backup through Milvus Http API. 通过 Milvus Http API 备份时暂停 GC
  gcPause:
    enable: true
    seconds: 7200
    address: http://192.169.20.106:9091


```

#### 执行恢复任务
```bash
# -s 重命名
# --restore_index 恢复索引
# --config 指定配置文件
$ ./milvus-backup restore -n backup_standalone -s _restore --restore_index --config=/root/kate/workspace/configs/restore.yaml
```

#### 查看恢复目录
```bash
$ mc ls --versions distributed
[2024-11-20 09:38:06 UTC]     0B minio-backup-bucket/
[2024-11-20 10:07:02 UTC]     0B minio-backup-bucket/backup-path/
[2024-11-19 06:49:25 UTC]     0B my-release/
[2024-11-20 10:07:02 UTC]     0B my-release/files/

$ mc ls --versions distributed/my-release/files/
[2024-11-20 10:07:07 UTC]     0B insert_log/
[2024-11-20 10:07:07 UTC]     0B stats_log/
```



