### 安装
参考：[Install Milvus Cluster with Helm | Milvus Documentation](https://milvus.io/docs/install_cluster-helm.md)

```bash
# 添加 Milvus Helm 存储库
helm repo add zilliztech https://zilliztech.github.io/milvus-helm
# 从存储库中获取 Milvus 
helm repo update
# upgrade existing helm release
helm upgrade my-release zilliztech/milvus

# 在线安装
helm install my-release milvus/milvus --set replica.number=2,resources.requests.memory=4Gi,resources.requests.cpu=2 --namespace kate-test8 --create-namespace
# 安装命令
helm install my-release milvus/milvus --namespace kate-test --create-namespace

```

### 测试
```bash
# 检查端口
kubectl get pod my-milvus-proxy-7b59bc5788-szd25 -n kate-test --template='{{(index (index .spec.containers 0).ports 0).containerPort}}{{"\n"}}'

# 转发
kubectl port-forward svc/my-milvus 9091:9091 -n kate-test

# 测试访问
curl http://localhost:9091/healthz
```

![](https://cdn.nlark.com/yuque/0/2024/png/2639475/1731313783938-72486938-0e83-43e6-8515-1c0c8a41d3ea.png)

![](https://cdn.nlark.com/yuque/0/2024/png/2639475/1731313796271-5ed0b85f-dca8-4739-9792-eec21dc0c6fa.png)

### attu
安装attu的话把svc的type改成NodePort类型

![](https://cdn.nlark.com/yuque/0/2024/png/2639475/1731652534629-76b7b253-3ed3-40e3-86b8-da44d6e6016f.png)

跑起来

```bash
netstat -tuln | grep "端口号"

docker run -d --name=attu -p 8089:3000 -e MILVUS_URL=10.0.5.73:32269 zilliz/attu:latest
```

登录attu：[http://10.0.5.73:8089/#/databases/default/colletions](http://10.0.5.73:8089/#/databases/default/colletions)

![](https://cdn.nlark.com/yuque/0/2024/png/2639475/1731642241140-300e82ce-a053-4145-b03c-261429f48891.png)

![](https://cdn.nlark.com/yuque/0/2024/png/2639475/1731642309680-82cfcfb4-471c-4d8b-b16a-b35022beee71.png)



