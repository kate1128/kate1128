#  环境
![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1747823051490-53920938-67cb-4f3f-99aa-0084dbe39e0a.png)

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1747823051647-2cc45e30-4bbd-4c1b-8b15-47f272b4338f.png)

# 安装 hami
## download文件
版本：2.5.1

地址：[https://github.com/Project-HAMi/HAMi/releases/tag/v2.5.1](https://github.com/Project-HAMi/HAMi/releases/tag/v2.5.1)

## 参考文档
昇腾910b支持文档：[https://github.com/Project-HAMi/HAMi/blob/master/docs/ascend910b-support_cn.md](https://github.com/Project-HAMi/HAMi/blob/master/docs/ascend910b-support_cn.md)

2.5.1版本：[https://github.com/Project-HAMi/HAMi/blob/v2.5.1/docs/ascend910b-support_cn.md](https://github.com/Project-HAMi/HAMi/blob/v2.5.1/docs/ascend910b-support_cn.md)

## 修改配置
`charts/hami/values.yaml`

```yaml
-----折叠------

devices:
  mthreads:
    enabled: false
    customresources:
      - mthreads.com/vgpu
  ascend:
   # 打开
    enabled: true
    image: "ascend-device-plugin:master"
    imagePullPolicy: IfNotPresent
    extraArgs: []
    nodeSelector:
      ascend: "on"
    tolerations: []
    customresources: 
       # 有其他型号也要加上，比如910B2
      - huawei.com/Ascend910A
      - huawei.com/Ascend910A-memory
      - huawei.com/Ascend910B
      - huawei.com/Ascend910B-memory
      - huawei.com/Ascend910B4
      - huawei.com/Ascend910B4-memory
      - huawei.com/Ascend310P
      - huawei.com/Ascend310P-memory
```

## 修改分区（可不改）
官方默认的：[https://github.com/Project-HAMi/HAMi/blob/v2.5.1/charts/hami/templates/scheduler/device-configmap.yaml](https://github.com/Project-HAMi/HAMi/blob/v2.5.1/charts/hami/templates/scheduler/device-configmap.yaml)

如果满足要求就不用加



如果不满足

查一下算力切分的模板信息

```bash
npu-smi info -t template-info
```

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1747823051797-46c8be06-6dee-41d8-8cae-055cc3b7451a.png)

或者是从文档去找：[https://www.hiascend.com/document/detail/zh/mindx-dl/600/clusterscheduling/clusterscheduling/cpaug/mxdlug_021.html](https://www.hiascend.com/document/detail/zh/mindx-dl/600/clusterscheduling/clusterscheduling/cpaug/mxdlug_021.html)



把对应的分区信息加上

在`charts/hami`目录下创建`files`目录，然后新增文件：`device-config.yaml`

一样规格的可以不加（因为加了发现和没加一样）

```yaml
vnpus:
- chipName: 910B
  commonWord: Ascend910A
  resourceName: huawei.com/Ascend910A
  resourceMemoryName: huawei.com/Ascend910A-memory
  memoryAllocatable: 32768
  memoryCapacity: 32768
  aiCore: 30
  templates:
    - name: vir02
      memory: 2184
      aiCore: 2
    - name: vir04
      memory: 4369
      aiCore: 4
    - name: vir08
      memory: 8738
      aiCore: 8
    - name: vir16
      memory: 17476
      aiCore: 16
- chipName: 910B2
  commonWord: Ascend910B2
  resourceName: huawei.com/Ascend910B2
  resourceMemoryName: huawei.com/Ascend910B2-memory
  memoryAllocatable: 65536
  memoryCapacity: 65536
  aiCore: 24
  aiCPU: 6
  templates:
    - name: vir03_1c_8g
      memory: 8192
      aiCore: 3
      aiCPU: 1
    - name: vir06_1c_16g
      memory: 16384
      aiCore: 6
      aiCPU: 1
    - name: vir12_3c_32g
      memory: 32768
      aiCore: 12
      aiCPU: 3  
- chipName: 910B3
  commonWord: Ascend910B
  resourceName: huawei.com/Ascend910B
  resourceMemoryName: huawei.com/Ascend910B-memory
  memoryAllocatable: 65536
  memoryCapacity: 65536
  aiCore: 20
  aiCPU: 7
  templates:
    - name: vir05_1c_16g
      memory: 16384
      aiCore: 5
      aiCPU: 1
    - name: vir10_3c_32g
      memory: 32768
      aiCore: 10
      aiCPU: 3
- chipName: 910B4
  commonWord: Ascend910B4
  resourceName: huawei.com/Ascend910B4
  resourceMemoryName: huawei.com/Ascend910B4-memory
  memoryAllocatable: 32768
  memoryCapacity: 32768
  aiCore: 20
  aiCPU: 7
  templates:
    - name: vir10_3c_16g
      memory: 16384
      aiCore: 10
      aiCPU: 3
    - name: vir10_4c_16g_m
      memory: 16384
      aiCore: 10
      aiCPU: 4
    - name: vir10_3c_16g_nm
      memory: 16384
      aiCore: 10
      aiCPU: 3
    - name: vir05_1c_8g
      memory: 8192
      aiCore: 5
      aiCPU: 1
- chipName: 310P3
  commonWord: Ascend310P
  resourceName: huawei.com/Ascend310P
  resourceMemoryName: huawei.com/Ascend310P-memory
  memoryAllocatable: 21527
  memoryCapacity: 24576
  aiCore: 8
  aiCPU: 7
  templates:
    - name: vir01
      memory: 3072
      aiCore: 1
      aiCPU: 1
    - name: vir02
      memory: 6144
      aiCore: 2
      aiCPU: 2
    - name: vir04
      memory: 12288
      aiCore: 4
      aiCPU: 4
```

## 安装
```bash
helm install hami ./hami -n hami-system
```

# 安装 ascend-device-plugin
参考地址：[https://github.com/Project-HAMi/ascend-device-plugin/tree/v1.0.1](https://github.com/Project-HAMi/ascend-device-plugin/tree/v1.0.1)

## 直接执行
```bash
kubectl apply -f ascend-device-plugin-2.4.1.yaml
```

也可以改成自己想装的名称空间，注意挂载的文件名要一致：`device-config.yaml`

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: hami-ascend
rules:
  - apiGroups: [""]
    resources: ["pods"]
    verbs: ["get", "list", "update", "watch", "patch"]
  - apiGroups: [""]
    resources: ["nodes"]
    verbs: ["get", "update", "patch"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: hami-ascend
subjects:
  - kind: ServiceAccount
    name: hami-ascend
    namespace: hami-system
roleRef:
  kind: ClusterRole
  name: hami-ascend
  apiGroup: rbac.authorization.k8s.io
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: hami-ascend
  namespace: hami-system
  labels:
    app.kubernetes.io/component: "hami-ascend"
---
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: hami-ascend-device-plugin
  namespace: hami-system
  labels:
    app.kubernetes.io/component: hami-ascend-device-plugin
spec:
  selector:
    matchLabels:
      app.kubernetes.io/component: hami-ascend-device-plugin
      hami.io/webhook: ignore
  template:
    metadata:
      labels:
        app.kubernetes.io/component: hami-ascend-device-plugin
        hami.io/webhook: ignore
    spec:
      priorityClassName: "system-node-critical"
      serviceAccountName: hami-ascend
      containers:
        - image: projecthami/ascend-device-plugin:v1.0.1
          imagePullPolicy: IfNotPresent
          name: device-plugin
          resources:
            requests:
              memory: 500Mi
              cpu: 500m
            limits:
              memory: 500Mi
              cpu: 500m
          args:
            - --config_file
            - /device-config.yaml
          securityContext:
            privileged: true
            readOnlyRootFilesystem: false
          volumeMounts:
            - name: device-plugin
              mountPath: /var/lib/kubelet/device-plugins
            - name: pod-resource
              mountPath: /var/lib/kubelet/pod-resources
            - name: hiai-driver
              mountPath: /usr/local/Ascend/driver
              readOnly: true
            - name: log-path
              mountPath: /var/log/mindx-dl/devicePlugin
            - name: tmp
              mountPath: /tmp
            - name: ascend-config
              mountPath: /device-config.yaml
              subPath: device-config.yaml
              readOnly: true
          env:
            - name: NODE_NAME
              valueFrom:
                fieldRef:
                  fieldPath: spec.nodeName
      volumes:
        - name: device-plugin
          hostPath:
            path: /var/lib/kubelet/device-plugins
        - name: pod-resource
          hostPath:
            path: /var/lib/kubelet/pod-resources
        - name: hiai-driver
          hostPath:
            path: /usr/local/Ascend/driver
        - name: log-path
          hostPath:
            path: /var/log/mindx-dl/devicePlugin
            type: Directory
        - name: tmp
          hostPath:
            path: /tmp
        - name: ascend-config
          configMap:
            name: hami-scheduler-device
      nodeSelector:
        ascend: "on"
```

## 打标签
```bash
kubectl label node xxx ascend=on
```

# 检查
装完之后状态正常就ok

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1747823051988-bfc574bd-9e99-4054-9e57-b0bd995734ca.png)

读配置文件对的：`device-config.yaml`

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1747823052182-7687fc09-5455-4ec8-8e11-d38ac53e986d.png)

然后可以看下`node`的`yaml`，有注册对应的卡，粘一个当示例

```yaml
poddevices={"Ascend910B4":[[{"Idx":0,"UUID":"Ascend910B4-6","Type":"Ascend910B4","Usedmem":8192,"Usedcores":0}]]}
```

```yaml
(base) root@wenxue-master:~/hami251# kubectl get node wenxue-master -o yaml
apiVersion: v1
kind: Node
metadata:
  annotations:
    baseDeviceInfos: '{"Ascend910-0":{"IP":"10.20.0.100","SuperDeviceID":4290772992},"Ascend910-1":{"IP":"10.20.0.101","SuperDeviceID":4291035137},"Ascend910-2":{"IP":"10.20.0.102","SuperDeviceID":4291297282},"Ascend910-4":{"IP":"10.20.0.104","SuperDeviceID":4291821572},"Ascend910-5":{"IP":"10.20.0.105","SuperDeviceID":4292083717},"Ascend910-5c.1cpu.8g-148-3":{"IP":"10.20.0.103","SuperDeviceID":4291559427},"Ascend910-5c.1cpu.8g-196-6":{"IP":"10.20.0.106","SuperDeviceID":4292345862},"Ascend910-7":{"IP":"10.20.0.107","SuperDeviceID":4292608007}}'
    flannel.alpha.coreos.com/backend-data: '{"VNI":1,"VtepMAC":"5a:12:30:58:e0:84"}'
    flannel.alpha.coreos.com/backend-type: vxlan
    flannel.alpha.coreos.com/kube-subnet-manager: "true"
    flannel.alpha.coreos.com/public-ip: 10.0.6.149
    hami.io/node-handshake: Deleted_2025-05-20 02:31:38
    hami.io/node-handshake-Ascend310P: Deleted_2025-05-20 05:54:27
    hami.io/node-handshake-Ascend910A: Deleted_2025-05-20 05:54:27
    hami.io/node-handshake-Ascend910B: Deleted_2025-05-20 05:54:27
    hami.io/node-handshake-Ascend910B2: Deleted_2025-05-21 02:25:40
    hami.io/node-handshake-Ascend910B4: Requesting_2025-05-21 10:06:33
    hami.io/node-handshake-dcu: Deleted_2025-05-20 02:31:38
    hami.io/node-register-Ascend910B4: '[{"id":"Ascend910B4-0","count":4,"devmem":32768,"devcore":20,"type":"Ascend910B4","health":true},{"index":1,"id":"Ascend910B4-1","count":4,"devmem":32768,"devcore":20,"type":"Ascend910B4","health":true},{"index":2,"id":"Ascend910B4-2","count":4,"devmem":32768,"devcore":20,"type":"Ascend910B4","health":true},{"index":3,"id":"Ascend910B4-3","count":4,"devmem":32768,"devcore":20,"type":"Ascend910B4","health":true},{"index":4,"id":"Ascend910B4-4","count":4,"devmem":32768,"devcore":20,"type":"Ascend910B4","health":true},{"index":5,"id":"Ascend910B4-5","count":4,"devmem":32768,"devcore":20,"type":"Ascend910B4","health":true},{"index":6,"id":"Ascend910B4-6","count":4,"devmem":32768,"devcore":20,"type":"Ascend910B4","health":true},{"index":7,"id":"Ascend910B4-7","count":4,"devmem":32768,"devcore":20,"type":"Ascend910B4","health":true}]'
    kubeadm.alpha.kubernetes.io/cri-socket: /var/run/dockershim.sock
    node.alpha.kubernetes.io/ttl: "0"
    volumes.kubernetes.io/controller-managed-attach-detach: "true"
  creationTimestamp: "2025-04-18T09:13:27Z"
  labels:
    accelerator: huawei-Ascend910
    accelerator-type: module
    ascend: "on"
    beta.kubernetes.io/arch: arm64
    beta.kubernetes.io/os: linux
    host-arch: huawei-arm
    kubernetes.io/arch: arm64
    kubernetes.io/hostname: wenxue-master
    kubernetes.io/os: linux
    masterselector: dls-master-node
    node-role.kubernetes.io/control-plane: ""
    node-role.kubernetes.io/master: ""
    node-role.kubernetes.io/worker: worker
    node.kubernetes.io/exclude-from-external-load-balancers: ""
    node.kubernetes.io/npu.chip.name: 910B4
    nodeDEnable: "on"
    servertype: Ascend910B-20
    test: wenxue
    workerselector: dls-worker-node
  name: wenxue-master
  resourceVersion: "4583455"
  uid: 568c465b-aea7-4d48-bc26-439281970938
spec:
  podCIDR: 10.244.0.0/24
  podCIDRs:
  - 10.244.0.0/24
status:
  addresses:
  - address: 10.0.6.149
    type: InternalIP
  - address: wenxue-master
    type: Hostname
  allocatable:
    cpu: "256"
    ephemeral-storage: "421372373524"
    huawei.com/Ascend910: "6"
    huawei.com/Ascend910-5c.1cpu.8g: "2"
    huawei.com/Ascend910B4: "32"
    hugepages-1Gi: "0"
    hugepages-2Mi: "0"
    hugepages-32Mi: "0"
    hugepages-64Ki: "0"
    memory: 526904468Ki
    pods: "110"
  capacity:
    cpu: "256"
    ephemeral-storage: 457218288Ki
    huawei.com/Ascend910: "6"
    huawei.com/Ascend910-5c.1cpu.8g: "2"
    huawei.com/Ascend910B4: "32"
    hugepages-1Gi: "0"
    hugepages-2Mi: "0"
    hugepages-32Mi: "0"
    hugepages-64Ki: "0"
    memory: 527006868Ki
    pods: "110"
  conditions:
  - lastHeartbeatTime: "2025-05-09T01:18:42Z"
    lastTransitionTime: "2025-05-09T01:18:42Z"
    message: Flannel is running on this node
    reason: FlannelIsUp
    status: "False"
    type: NetworkUnavailable
  - lastHeartbeatTime: "2025-05-21T10:03:58Z"
    lastTransitionTime: "2025-04-18T09:13:22Z"
    message: kubelet has sufficient memory available
    reason: KubeletHasSufficientMemory
    status: "False"
    type: MemoryPressure
  - lastHeartbeatTime: "2025-05-21T10:03:58Z"
    lastTransitionTime: "2025-04-18T09:13:22Z"
    message: kubelet has no disk pressure
    reason: KubeletHasNoDiskPressure
    status: "False"
    type: DiskPressure
  - lastHeartbeatTime: "2025-05-21T10:03:58Z"
    lastTransitionTime: "2025-04-18T09:13:22Z"
    message: kubelet has sufficient PID available
    reason: KubeletHasSufficientPID
    status: "False"
    type: PIDPressure
  - lastHeartbeatTime: "2025-05-21T10:03:58Z"
    lastTransitionTime: "2025-04-23T08:23:12Z"
    message: kubelet is posting ready status. AppArmor enabled
    reason: KubeletReady
    status: "True"
    type: Ready
  daemonEndpoints:
    kubeletEndpoint:
      Port: 10250
  images:
  - names:
    - harbor.dcclouds.com/dcg/wuhan/tai/new-model-server:npu-910b-latest
    sizeBytes: 39347225998
  - names:
    - harbor.dcclouds.com/dcg/wuhan/tai/emb-model-server:NPU_910_latest
    sizeBytes: 27716520931
  - names:
    - image.sourcefind.cn:5000/dcu/admin/base/custom@sha256:522c3f0c94afa5b3f72ed223b3da4d65aca9c695ab566ceeabecb69f78a6949f
    - image.sourcefind.cn:5000/dcu/admin/base/custom:vllm0.8.4-ubuntu22.04-dtk25.04-rc7-das1.5-py3.10-20250429-dev-qwen3-only
    sizeBytes: 22310389689
  - names:
    - harbor.dcclouds.com/dcg/wuhan/tai/ocr-model-server:npu-910b4-latest-1
    sizeBytes: 22293929635
  - names:
    - mindie:2.0.T9.B020-800I-A2-py3.11-openeuler24.03-lts-aarch64
    sizeBytes: 14982347726
  - names:
    - swr.cn-south-1.myhuaweicloud.com/ascendhub/mindie@sha256:9001be1af7b1ebf659814b406ef7a3d5ae9a9aa40b7bfda51a4ea1d8d8bcb46f
    - swr.cn-south-1.myhuaweicloud.com/ascendhub/mindie:2.0.RC1-300I-Duo-py311-openeuler24.03-lts
    sizeBytes: 14856689220
  - names:
    - quay.io/ascend/vllm-ascend@sha256:ff0b3b9131bba0aa1aed44bf0289da2bc8fb8ae74539190bad5c1d8e2a6e715e
    - quay.io/ascend/vllm-ascend:main
    sizeBytes: 14544071842
  - names:
    - quay.io/ascend/vllm-ascend@sha256:5043e8307cb6ec3f771a31a1836fafa3084a59153a6ba4f071bea12400782078
    - quay.io/ascend/vllm-ascend:v0.8.4rc1
    sizeBytes: 14418946917
  - names:
    - quay.io/ascend/vllm-ascend@sha256:57db23b4034a5e96ee2468b3b41f7429dcc12a27406899d55a08f1fb991e8391
    - quay.io/ascend/vllm-ascend:v0.8.4rc2-openeuler
    sizeBytes: 14183267797
  - names:
    - harbor.dcclouds.com/dcg/wuhan/tai/llm-finetune/qwen_nodomain@sha256:d413e940524a624a44dddbd67830c7d1d24e0ef61b8a63d3d7d83d4c2bad0e76
    - harbor.dcclouds.com/dcg/wuhan/tai/llm-finetune/qwen_nodomain:latest-910b
    sizeBytes: 13976899661
  - names:
    - harbor.dcclouds.com/dcg/wuhan/tai/llm-finetune/deepseekv3_nodomain@sha256:a20dd8def933e7e430e9804d8765af9ba0fcd6a0ed748b5117a4f7f47329df25
    - harbor.dcclouds.com/dcg/wuhan/tai/llm-finetune/deepseekv3_nodomain:latest-910b
    sizeBytes: 13959129076
  - names:
    - quay.io/ascend/vllm-ascend@sha256:2000312f2b616573f0ca029ad5e4b4e1b7e22e627133082fb8a1a176e388f558
    - quay.io/ascend/vllm-ascend:v0.7.3rc2
    sizeBytes: 13950386668
  - names:
    - swr.cn-south-1.myhuaweicloud.com/ascendhub/mindie:2.0.T3-800I-A2-py311-openeuler24.03-lts
    sizeBytes: 13916548901
  - names:
    - mindie:2.0.T3-800I-A2-py311-openeuler24.03-wx
    sizeBytes: 13916508693
  - names:
    - mindie:2.0.T3-800I-A2-py311-openeuler24.03
    sizeBytes: 13916507213
  - names:
    - vllm-ascend-dev-image:latest
    sizeBytes: 13741049893
  - names:
    - swr.cn-south-1.myhuaweicloud.com/ascendhub/mindie@sha256:86d7aa799c4a094912cf826965b2cf42782c1babed030725397c834b91c16534
    - swr.cn-south-1.myhuaweicloud.com/ascendhub/mindie:1.0.RC3-300I-Duo-arm64
    sizeBytes: 13678534592
  - names:
    - swr.cn-south-1.myhuaweicloud.com/ascendhub/ascend-pytorch:24.0.RC3-A2-2.1.0-openeuler20.03
    sizeBytes: 11270633547
  - names:
    - quay.io/ascend/cann@sha256:be138da007846ea253f773ecb6a18cdebd1f1ca483e5631826d619d7f696c54e
    - quay.io/ascend/cann:8.0.0-910b-ubuntu22.04-py3.10
    sizeBytes: 11246313026
  - names:
    - 10.0.6.149:8081/dcg/wuhan/tai/ai-backend@sha256:07403ff1945389d8d48f720802b23c5698944f3a181ef6e283a97e609b2db3c3
    - 10.0.6.149:8081/dcg/wuhan/tai/ai-backend:latest-arm
    - harbor.dcclouds.com/dcg/wuhan/tai/ai-backend:release33-arm
    sizeBytes: 6001437242
  - names:
    - firecrawl-api:latest
    sizeBytes: 1668865755
  - names:
    - firecrawl-worker:latest
    sizeBytes: 1668865755
  - names:
    - milvusdb/milvus@sha256:fe39aa0fa006be4825a177b6b41393465deece910351c33a58170444265c63d8
    - milvusdb/milvus:v2.5.6
    sizeBytes: 1564042908
  - names:
    - firecrawl-playwright-service:latest
    sizeBytes: 1337817712
  - names:
    - nvcr.io/nvidia/k8s/dcgm-exporter@sha256:98781424e83e14e8855aa6881e5ca8e68c81fdc75c82dd1bb3fe924349aee9d4
    - nvcr.io/nvidia/k8s/dcgm-exporter:3.3.7-3.5.0-ubuntu22.04
    sizeBytes: 1134111292
  - names:
    - mysql@sha256:7839322bd6c3174a699586c3ea36314c59b61b4ce01b4146951818b94aef5fd7
    - mysql:latest
    sizeBytes: 876234535
  - names:
    - projecthami/hami-webui-fe-oss@sha256:2f0e3dd239274248703a3d28dbdab873a1bcd902da21867e220975638e560465
    - projecthami/hami-webui-fe-oss:v1.0.3
    sizeBytes: 811192519
  - names:
    - projecthami/hami-webui-fe-oss@sha256:f4cdeb32ede9abde98de73b91c50fbf2fdc9875792e0c8c545e252a435cf3a2b
    - projecthami/hami-webui-fe-oss:main
    sizeBytes: 797348373
  - names:
    - bitnami/kafka@sha256:55df55bfc7ed5980447387620afa3498eab3985a4d8c731013d82b3fa8b43bff
    - bitnami/kafka:3.9.0
    sizeBytes: 467749968
  - names:
    - goharbor/trivy-adapter-photon:v2.10.1-aarch64
    sizeBytes: 455021504
  - names:
    - langgenius/dify-sandbox@sha256:df44a051615d28c463af893687f7e049dcf66506e482433337a16ec682290b39
    - langgenius/dify-sandbox:0.2.1
    sizeBytes: 433725201
  - names:
    - grafana/grafana:10.4.1
    sizeBytes: 422720806
  - names:
    - registry.cn-hangzhou.aliyuncs.com/google_containers/etcd@sha256:9ce33ba33d8e738a5b85ed50b5080ac746deceed4a7496c550927a7a19ca3b6d
    - registry.aliyuncs.com/google_containers/etcd:3.5.0-0
    - registry.cn-hangzhou.aliyuncs.com/google_containers/etcd:3.5.0-0
    sizeBytes: 363630426
  - names:
    - projecthami/hami@sha256:9568f988d9812c0a684d4d389fbc8473f6a36d31b4b7d97164fe7a901497a870
    - projecthami/hami:v2.5.1
    sizeBytes: 361382476
  - names:
    - registry.aliyuncs.com/google_containers/etcd:3.4.9-1
    sizeBytes: 312285450
  - names:
    - goharbor/harbor-db:v2.10.1-aarch64
    sizeBytes: 306978225
  - names:
    - goharbor/harbor-db-base:v2.10.1-aarch64
    sizeBytes: 306965010
  - names:
    - rabbitmq@sha256:cb5f2b8953eeb4ba986b9009ab2fec270d1c2d2999af3f4f2ced3fb86d12661e
    - rabbitmq:3-management
    sizeBytes: 270876699
  - names:
    - minio/minio@sha256:6d770d7f255cda1f18d841ffc4365cb7e0d237f6af6a15fcdb587480cd7c3b93
    - minio/minio:RELEASE.2023-03-20T20-16-18Z
    sizeBytes: 266795100
  - names:
    - postgres@sha256:ef9d1517df69c4d27dbb9ddcec14f431a2442628603f4e9daa429b92ae6c3cd1
    - postgres:15-alpine
    sizeBytes: 264729492
  - names:
    - quay.io/prometheus/prometheus:v2.51.2
    sizeBytes: 255890906
  - names:
    - goharbor/redis-photon:v2.10.1-aarch64
    sizeBytes: 239881163
  - names:
    - goharbor/harbor-redis-base:v2.10.1-aarch64
    sizeBytes: 239765365
  - names:
    - swr.cn-south-1.myhuaweicloud.com/ascendhub/vc-scheduler@sha256:aa518999201502e15e7e55f9d7513812c7b68c4b81a4c80e47c8b6c54fc1e9b7
    - volcanosh/vc-scheduler:v1.7.0
    - swr.cn-south-1.myhuaweicloud.com/ascendhub/vc-scheduler:v1.7.0-v6.0.0
    sizeBytes: 226894527
  - names:
    - goharbor/prepare:v2.10.1-aarch64
    sizeBytes: 214968592
  - names:
    - goharbor/harbor-prepare-base:v2.10.1-aarch64
    sizeBytes: 211964950
  - names:
    - openeuler-24.03-lts-sp1:latest
    sizeBytes: 204629377
  - names:
    - goharbor/harbor-log:v2.10.1-aarch64
    sizeBytes: 194954514
  - names:
    - goharbor/harbor-log-base:v2.10.1-aarch64
    sizeBytes: 194950600
  - names:
    - goharbor/harbor-portal:v2.10.1-aarch64
    sizeBytes: 192198013
  nodeInfo:
    architecture: arm64
    bootID: a778477f-f1f5-4c84-a7ce-ee45347b78cc
    containerRuntimeVersion: docker://20.10.8
    kernelVersion: 5.15.0-25-generic
    kubeProxyVersion: v1.22.0
    kubeletVersion: v1.22.0
    machineID: c0b85b24848a4e6cbf03d1bda59e83c6
    operatingSystem: linux
    osImage: Ubuntu 22.04 LTS
    systemUUID: a6160f96-b04f-9f5a-ee11-9baca8281b45
```

# 测试
随便找个镜像测一下，在容器里执行`npu-smi info`看看

！！！注意这里必须是 `Ascend910B4`，`Ascend910B`不行，以`node`注册的`type`为准

具体分区是按照上面配置的分区规格来的

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: gpu-pod-910b4
  namespace: hami-system
spec:
  containers:
  - image: harbor.dcclouds.com/dcg/wuhan/tai/emb-model-server:NPU_910_latest
    command: ["bash", "-c", "sleep 86400"]
    imagePullPolicy: IfNotPresent
    name: emb-model-server
    ports:
    - containerPort: 8000
      protocol: TCP
    resources:
      limits:
        huawei.com/Ascend910B4: 1 # requesting 1 NPU
        huawei.com/Ascend910B4-memory: 2000 # requesting 2000m device memory
  schedulerName: hami-scheduler
```

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1747823052309-57026331-f23f-4769-b708-6f934cf7785f.png)

有环境的话可以直接跑一下，粘一个示例

```yaml
(base) root@wenxue-master:~/hami251# kubectl get pod emb-model-server -o yaml
apiVersion: v1
kind: Pod
metadata:
  annotations:
    ascend.kubectl.kubernetes.io/ascend-910-configuration: '{"pod_name":"emb-model-server","server_id":"10.0.6.149","super_pod_id":-1,"devices":[{"device_id":"4","device_ip":"10.20.0.104","super_device_id":"4291821572"}]}'
    distributed-job: "false"
    huawei.com/Ascend910: Ascend910-4
    huawei.com/AscendReal: Ascend910-4
    huawei.com/kltDev: Ascend910-2
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","kind":"Pod","metadata":{"annotations":{"huawei.com/Ascend910":"Ascend910-1"},"labels":{"app":"emb-model-server","fault-scheduling":"grace","host-arch":"huawei-arm","ring-controller.atlas":"ascend-910b"},"name":"emb-model-server","namespace":"default"},"spec":{"containers":[{"env":[{"name":"ASCEND_VISIBLE_DEVICES","valueFrom":{"fieldRef":{"fieldPath":"metadata.annotations['huawei.com/Ascend910']"}}}],"image":"harbor.dcclouds.com/dcg/wuhan/tai/emb-model-server:NPU_910_latest","imagePullPolicy":"IfNotPresent","name":"emb-model-server","ports":[{"containerPort":8000,"protocol":"TCP"}],"resources":{"limits":{"huawei.com/Ascend910":1},"requests":{"huawei.com/Ascend910":1}},"volumeMounts":[{"mountPath":"/var/log/npu/conf/slog/","name":"slog"},{"mountPath":"/etc/localtime","name":"localtime"},{"mountPath":"/usr/local/bin","name":"npu-bin"},{"mountPath":"/dev/shm","name":"dshm"}]}],"dnsPolicy":"ClusterFirst","nodeSelector":{"host-arch":"huawei-arm"},"restartPolicy":"Always","schedulerName":"volcano","securityContext":{},"terminationGracePeriodSeconds":30,"volumes":[{"emptyDir":{"medium":"Memory"},"name":"dshm"},{"hostPath":{"path":"/var/log/npu/conf/slog/"},"name":"slog"},{"hostPath":{"path":"/etc/localtime"},"name":"localtime"},{"hostPath":{"path":"/usr/local/bin"},"name":"npu-bin"}]}}
    predicate-time: "18446744073709551615"
    scheduling.k8s.io/group-name: podgroup-239285b8-d8a0-4999-b43f-6a3cbc611618
  creationTimestamp: "2025-05-21T09:58:28Z"
  labels:
    app: emb-model-server
    fault-scheduling: grace
    host-arch: huawei-arm
    ring-controller.atlas: ascend-910b
  name: emb-model-server
  namespace: default
  resourceVersion: "4582807"
  uid: 239285b8-d8a0-4999-b43f-6a3cbc611618
spec:
  containers:
  - env:
    - name: ASCEND_VISIBLE_DEVICES
      valueFrom:
        fieldRef:
          apiVersion: v1
          fieldPath: metadata.annotations['huawei.com/Ascend910']
    image: harbor.dcclouds.com/dcg/wuhan/tai/emb-model-server:NPU_910_latest
    imagePullPolicy: IfNotPresent
    name: emb-model-server
    ports:
    - containerPort: 8000
      protocol: TCP
    resources:
      limits:
        huawei.com/Ascend910: "1"
      requests:
        huawei.com/Ascend910: "1"
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /var/log/npu/conf/slog/
      name: slog
    - mountPath: /etc/localtime
      name: localtime
    - mountPath: /usr/local/bin
      name: npu-bin
    - mountPath: /dev/shm
      name: dshm
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: kube-api-access-plpsz
      readOnly: true
  dnsPolicy: ClusterFirst
  enableServiceLinks: true
  nodeName: wenxue-master
  nodeSelector:
    host-arch: huawei-arm
  preemptionPolicy: PreemptLowerPriority
  priority: 0
  restartPolicy: Always
  schedulerName: volcano
  securityContext: {}
  serviceAccount: default
  serviceAccountName: default
  terminationGracePeriodSeconds: 30
  tolerations:
  - effect: NoExecute
    key: node.kubernetes.io/not-ready
    operator: Exists
    tolerationSeconds: 300
  - effect: NoExecute
    key: node.kubernetes.io/unreachable
    operator: Exists
    tolerationSeconds: 300
  volumes:
  - emptyDir:
      medium: Memory
    name: dshm
  - hostPath:
      path: /var/log/npu/conf/slog/
      type: ""
    name: slog
  - hostPath:
      path: /etc/localtime
      type: ""
    name: localtime
  - hostPath:
      path: /usr/local/bin
      type: ""
    name: npu-bin
  - name: kube-api-access-plpsz
    projected:
      defaultMode: 420
      sources:
      - serviceAccountToken:
          expirationSeconds: 3607
          path: token
      - configMap:
          items:
          - key: ca.crt
            path: ca.crt
          name: kube-root-ca.crt
      - downwardAPI:
          items:
          - fieldRef:
              apiVersion: v1
              fieldPath: metadata.namespace
            path: namespace
status:
  conditions:
  - lastProbeTime: null
    lastTransitionTime: "2025-05-21T09:58:33Z"
    status: "True"
    type: Initialized
  - lastProbeTime: null
    lastTransitionTime: "2025-05-21T10:00:40Z"
    status: "True"
    type: Ready
  - lastProbeTime: null
    lastTransitionTime: "2025-05-21T10:00:40Z"
    status: "True"
    type: ContainersReady
  - lastProbeTime: null
    lastTransitionTime: "2025-05-21T09:58:33Z"
    status: "True"
    type: PodScheduled
  containerStatuses:
  - containerID: docker://8f7fd5d82231c364bbf2b18a5be940d9d5303924bbc5fc41e0960c2d47458767
    image: harbor.dcclouds.com/dcg/wuhan/tai/emb-model-server:NPU_910_latest
    imageID: docker://sha256:f75cd86654557cc971e7b23d50c7c6422957ac1258821eff362ea5181d6372a5
    lastState:
      terminated:
        containerID: docker://5b93089ec270bfc1257413ed4a182cfd99f166a656df182924725c5c929cae20
        exitCode: 139
        finishedAt: "2025-05-21T09:59:54Z"
        reason: Error
        startedAt: "2025-05-21T09:59:48Z"
    name: emb-model-server
    ready: true
    restartCount: 4
    started: true
    state:
      running:
        startedAt: "2025-05-21T10:00:39Z"
  hostIP: 10.0.6.149
  phase: Running
  podIP: 10.244.0.172
  podIPs:
  - ip: 10.244.0.172
  qosClass: BestEffort
  startTime: "2025-05-21T09:58:33Z"
```

# 常用命令
## 查询算力切分模板
```bash
npu-smi info -t template-info
```

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1747823052425-ae16caca-f5f3-4df9-a2ac-19372fb7af1a.png)

## 查询指定芯片的vNPU
```bash
npu-smi info -t info-vnpu -i 4 -c 0
```

![](https://cdn.nlark.com/yuque/0/2025/jpeg/2639475/1747823052742-daac9d0e-3f8d-4f68-9b0f-399e1b4e3e3e.jpeg)

## 查询所有芯片的进程占用内存
```bash
npu-smi info -t proc-mem -i 7
```

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1747823053028-87ea28be-e67d-45d8-ba9f-11e7637035f3.png)

## 查询指定芯片的进程占用内存
```bash
npu-smi info -t proc-mem -i 2 -c 0
```

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1747823053178-920be478-e72c-40a7-b96b-6bfe01b31cf3.png)

# 常见问题
hami常见问题解答：[https://github.com/Project-HAMi/HAMi/issues/646](https://github.com/Project-HAMi/HAMi/issues/646)

容器内npu-smi info报错：[https://github.com/Project-HAMi/HAMi/issues/615](https://github.com/Project-HAMi/HAMi/issues/615)

异构资源池化的理解：[https://github.com/Project-HAMi/HAMi/issues/957](https://github.com/Project-HAMi/HAMi/issues/957)

已经支持的gpu：[https://github.com/Project-HAMi/HAMi/issues/598](https://github.com/Project-HAMi/HAMi/issues/598)

910b已开源：[https://github.com/Project-HAMi/HAMi/issues/643](https://github.com/Project-HAMi/HAMi/issues/643)

申请虚拟资源失败：[https://github.com/Project-HAMi/HAMi/issues/927](https://github.com/Project-HAMi/HAMi/issues/927)

华为命令行：[https://support.huawei.com/enterprise/zh/doc/EDOC1100288570/d4ec2c12](https://support.huawei.com/enterprise/zh/doc/EDOC1100288570/d4ec2c12)

查询命令：[https://support.huawei.com/enterprise/zh/doc/EDOC1100288570/3b21d0f6](https://support.huawei.com/enterprise/zh/doc/EDOC1100288570/3b21d0f6)

类似问题：[https://www.hiascend.com/forum/thread-02108181989912857060-1-1.html](https://www.hiascend.com/forum/thread-02108181989912857060-1-1.html)

类似问题参考：[https://developer.huawei.com/home/forum/ascend/thread-0295152424292444039-1-1.html](https://developer.huawei.com/home/forum/ascend/thread-0295152424292444039-1-1.html)

vnpu上跑的进程在宿主机上查不到

![](https://cdn.nlark.com/yuque/0/2025/jpeg/2639475/1747823053370-5d880826-060a-4331-b6d4-8871e36f92ae.jpeg)

![](https://cdn.nlark.com/yuque/0/2025/jpeg/2639475/1747823053581-c5a73d3d-c0b2-4d65-a129-bf02ccceb669.jpeg)

容器里有显示

![](https://cdn.nlark.com/yuque/0/2025/jpeg/2639475/1747823053745-37c3b569-88c0-466e-aca6-b1ffd155759f.jpeg)

容器外无

![](https://cdn.nlark.com/yuque/0/2025/jpeg/2639475/1747823054094-4d630a11-2ba0-41e6-87f7-f66990191cee.jpeg)

![](https://cdn.nlark.com/yuque/0/2025/jpeg/2639475/1747823054453-d2dcd522-9d6d-4f78-aa46-dfb1b5ed6655.jpeg)

