# milvus 存储
![画板](https://cdn.nlark.com/yuque/0/2024/jpeg/2639475/1733278888804-123e80d6-4d56-432d-a4ef-d4ed6a804231.jpeg)

# milvus-backup
已知：milvus 是在 etcd 和 minio 中存储数据的

地址：

[GitHub1s](https://github1s.com/milvus-io/milvus/blob/master/internal/metastore/kv/rootcoord/rootcoord_constant.go)

可以看下 etcd 的索引

```go
const (
    // ComponentPrefix prefix for rootcoord component
    ComponentPrefix = "root-coord"

    DatabaseMetaPrefix       = ComponentPrefix + "/database"
    DBInfoMetaPrefix         = DatabaseMetaPrefix + "/db-info"
    CollectionInfoMetaPrefix = DatabaseMetaPrefix + "/collection-info"

    // CollectionMetaPrefix prefix for collection meta
    CollectionMetaPrefix = ComponentPrefix + "/collection"

    PartitionMetaPrefix = ComponentPrefix + "/partitions"
    AliasMetaPrefix     = ComponentPrefix + "/aliases"
    FieldMetaPrefix     = ComponentPrefix + "/fields"
    FunctionMetaPrefix  = ComponentPrefix + "/functions"

    // CollectionAliasMetaPrefix210 prefix for collection alias meta
    CollectionAliasMetaPrefix210 = ComponentPrefix + "/collection-alias"

    SnapshotsSep   = "_ts"
    SnapshotPrefix = "snapshots"
    Aliases        = "aliases"

    // CommonCredentialPrefix subpath for common credential
    /* #nosec G101 */
    CommonCredentialPrefix = "/credential"

    // UserSubPrefix subpath for credential user
    UserSubPrefix = CommonCredentialPrefix + "/users"

    // CredentialPrefix prefix for credential user
    CredentialPrefix = ComponentPrefix + UserSubPrefix

    // RolePrefix prefix for role
    RolePrefix = ComponentPrefix + CommonCredentialPrefix + "/roles"

    // RoleMappingPrefix prefix for mapping between user and role
    RoleMappingPrefix = ComponentPrefix + CommonCredentialPrefix + "/user-role-mapping"

    // GranteePrefix prefix for mapping among role, resource type, resource name
    GranteePrefix = ComponentPrefix + CommonCredentialPrefix + "/grantee-privileges"

    // GranteeIDPrefix prefix for mapping among privilege and grantor
    GranteeIDPrefix = ComponentPrefix + CommonCredentialPrefix + "/grantee-id"

    // PrivilegeGroupPrefix prefix for privilege group
    PrivilegeGroupPrefix = ComponentPrefix + "/privilege-group"
)
```

查看元数据的命令

```powershell
etcdctl --endpoints=http://10.1.32.114:2379 get "wenxue-release/meta/root-coord" --prefix

etcdctl --endpoints=http://10.1.32.114:2379 get "wenxue-release/meta/root-coord/credential" --prefix
```

milvus-backup 备份的一个脚本

```go
#!/bin/bash
DATE=`date +%F`
# 备份文件日志路径
MilvusBackLogs=/DATA/milvus/logs/backup_logs
MilvusBackUpFiles=/DATA/milvus/volumes/minio/a-bucket/backup

# set a new backup file 切换对应的目录并执行备份命令
cd /DATA/milvus

if [ ! -d ${MilvusBackLogs} ];then
mkdir -p ${MilvusBackLogs}
fi

if [ ! -d ${MilvusBackUpFiles} ];then
mkdir -p ${MilvusBackUpFiles}
fi

# Milvus-Backup
./milvus-backup create -n MilvusBackup_${DATE}  > ${MilvusBackLogs}/MilvusBackup_${DATE}.log 2>&1

# 清理 delete timeout file for backup_logs/backupFiles
cd ${MilvusBackLogs}
/usr/bin/find ./ -name 'MilvusBackup_*' -mtime +3|xargs rm -f ;
# 清理 delete timeout file for backup_logs/backupFiles
cd ${MilvusBackUpFiles}
/usr/bin/find ./ -name 'MilvusBackup_*' -mtime +3|xargs rm -f ;

# 发送
scp ${MilvusBackUpFiles}/MilvusBackup_${DATE} root@172.16.101.209:/DATA/milvus/test
```

