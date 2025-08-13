# Interview Experiences

## Target
快手大模型

快手容器云

滴滴

腾讯混元

百度云原生调度

小红书云原生调度

## Overview

Golang 协程原理

进程线程协程的区别

Linux SIG 有哪些

GMP 模型

Kubernetes 优雅退出的流程

XID 故障

GPU 共享机制

HAMI 底层原理

MPU

## Experiences

### 快手 Kstar 容器云研发

[【投递】](https://campus.kuaishou.cn/recruit/campus/e/#/campus/job-info/9503)

【要求】Docker、Kubernetes、Istio等使用或二次开发经验

【面经】

- raft实现

- mvcc

- Kafka 可靠传输

### 滴滴 k8s开发工程师

[【投递】](https://campus.didiglobal.com/campus_apply/didiglobal/96064#/job/c97641b7-4417-495a-a083-c84e20f9b79d)

【描述】k8s 调度；k8s GPU 集群、k8s master 建设

【要求】熟悉但不限于k8s/kubelet/docker/kafka/rocketmq/zookeeper/codis/redis等任意一种开源产品源代码

【面经】

- 内存分配算法
- Java gc 策略

【代码】

- go，俩协程轮流打印1-100
- LRU
- 环形队列，多生产者消费者：阻塞有锁版 + 无锁版；CAS 原理
- 一个数组找出最大和最小值，优化比较次数

### 虾皮 ai infra

【描述】k8s, ray; nccl, cuda 变成, 训练推理框架

【面经】

- pod 从提交到拉起的全流程（存etcd、调度、kubelet监听、containerd拉起）
- k8s controller
- k8s informer
- k8s 流量相关：发布了新版，怎么拉起新的，流量怎么切过去
- k8s调度框架：三个调度队列，pod 怎么流转
- ray：ray组件、调度器结构、单节点 OOM 了怎么处理；参数依赖：加入b节点依赖a的参数，a挂了怎么办
- B+树实现
- 索引失效
- 索引覆盖、回表
- redis 基础数据结构，zset实现
- 用户态内核态
- Linux 基础命令：查端口占用、文件字符串查找
- 线程池相关、创建参数

【代码】

- 反转链表
- 最长有效括号

## Positions

### 字节 AML 机器学习系统调度编排

[【投递】](https://jobs.bytedance.com/campus/position/7529810995080202514/detail)

【描述】（PS： 岗位描述较详细）

- k8s 调度框架
- AutoScaling
- 多集群混合调度；
- 抢占、驱逐
- 混部、超卖

【要求】熟悉分布式系统原理

### 字节 AML 大模型推理系统研发

[【投递】](https://jobs.bytedance.com/campus/position/7532039453444835602/detail)

【描述】火山方舟MaaS推理系统工程研发：多角色分离式推理、分布式KV Cache系统、异构推理、弹性计算、多租户混布推理

【要求】

- CUDA
- GPU性能分析
- 分布式系统/大规模异构推理前沿进展
- PD分离、KV Cache系统、多机推理
- 分布式网络通信优化经验：分布式通信算子实现原理、RDMA原理
- k8s, ray



