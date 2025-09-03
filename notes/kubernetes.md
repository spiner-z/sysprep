# Kubernetes

## Interview picks

### 一个 Pod 从创建到最终被调度执行的完整流程

1. 通过 `kubectl apply` 等方式发送 yaml 到 API Server
2. 经过 API Server 认证，在 etcd 中存储 Pod 对象 （此时 Pod 的 NodeName为空，状态为 Pending）
3. kube-scheduler 持续监听（By Informer） API Server 的 Pending Pods 列表，将新 Pod 加入其调度队列
4. 调度器对每个 Pod 执行调度循环：

   - Filter
   - Score
   - Reserve
   - Permit
   - Bind

5. 目标节点上 Kubelet 监听（By Informer） APIserver，发现 nodeName 是自己且尚未运行，则：拉取镜像、创建容器、启动容器

**当有多个调度器时**

Pod 的 `.spec.schedulerName` 决定有哪个调度器负责，每个调度器都有自己独立的调度队列

不同调度器调度自己队列中的 Pod 是并发和竞争的。由哪个调度器成功调度，取决于谁的 Bind 请求最先被 APIserver 接受并传入 ETCD

> 一般不同调度器还是设置不同的资源池，避免相互干扰

### Pod 是如何挂载并使用 GPU 的

NVIDIA Device Plugin（通常以 DaemonSet 方式运行），会与节点上的 kubelet 建立 gRPC 连接，通过 ListAndWatch 接口向 kubelet 定期汇报可用 GPU 列表。

调度阶段：发现 Pod 的资源声明里有 nvidia.com/gpu: 1，就会筛选有空余 GPU 的节点

检测到容器需要 GPU，于是调用 Device Plugin 的 Allocate() 接口为容器分配具体设备：设备文件（如 /dev/nvidia0）以及相关驱动依赖（如 /usr/lib/nvidia/... 驱动目录）


