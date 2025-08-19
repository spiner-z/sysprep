# kubeflow

## 架构

### kubeflow 生态

<img width="1342" height="1443" alt="image" src="https://github.com/user-attachments/assets/6dded513-6b65-4191-a022-772f566aee9b" />

### ML 生命周期

<img width="3020" height="1421" alt="image" src="https://github.com/user-attachments/assets/ac0d52c2-05da-45fa-9e82-d8438503fb39" />

- **Data prepatation**: 执行特征工程，提取 ML 特征。准备用于模型开发的训练数据。通常使用数据处理工具：Spark、Dask、Flink 或 Ray
- **Model development**: 模型开发、微调，如：BERT 或 Llama
- **Model Optimization**: 使用各种 AutoML 算法（如神经架构搜索和模型压缩）优化模型超参数
- **Model training**: 训练的结果，可以存放在模型注册表
- **Model serving**: 在线或批量推理

生产/开发阶段图：
<img width="3330" height="1610" alt="image" src="https://github.com/user-attachments/assets/4ac49676-eb02-4fd6-95cb-bc89b8a654f5" />

生命周期中的组件：
<img width="3788" height="1683" alt="image" src="https://github.com/user-attachments/assets/ec788de7-5cbd-43e4-90c6-6ec86f9395e7" />

- [Kubeflow Spark Operator](https://github.com/kubeflow/spark-operator) 用于数据准备和特征工程
- Kubeflow notebooks 可用于模型开发
- Kubeflow Katib 可用于模型优化和超参数调优，支持各种 AutoML 算法，如：网格搜索、随机搜索、贝叶斯优化
- Kubeflow trainer 用于大规模分布式训练或 LLM 微调
- Kubeflow model register 可用于存储 ML 元数据、模型工件，并准备模型用于生产服务
- Kserve 用于模型服务步骤中的在线和批量推理，具备弹性伸缩等能力
- Feast 可用作特征存储，并管理离线和在线特征
- Kubeflow pipeline 用于构建、部署和管理 ML 生命周期中的每个步骤

大多数 Kubeflow 组件可以独立部署到自己的 AI/ML 平台；也可以部署完整的 Kubeflow 平台。

## 安装

详见：https://kubeflow.org.cn/docs/started/installing-kubeflow/

[示例](https://kubeflow.org.cn/docs/started/kubeflow-examples/) 包括一个 MNIST 图像分类

## 组件

### Central Dashboard

### Kubeflow Notebooks

### Kubeflow Pipelines

### Kubeflow Trainer

### Katib

### Model Registry

### Spark Operator

## 外部插件

### Kserve

### Feast

### Elyra







