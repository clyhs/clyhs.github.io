---
title: bert模型测试
date: 2023-08-18 23:15:53
category: 人工智能
tags: 大模型
---

# bert模型测试

## 安装bert模型

### 安装bert

```
conda create --name milvus python=3.6
conda activate milvus

pip install bert-serving-server
pip install bert-serving-client
```

### 安装tensorflow

```
pip install tensorflow==1.13.1
```

### 启动 bert

下载模型：https://storage.googleapis.com/bert_models/2018_11_03/chinese_L-12_H-768_A-12.zip

```
BERT-serving-start -model_dir chinese_L-12_H-768_A-12/ -num_worker=4 -max_seq_len=20

使用Client客户端获取sentence encodes（语句编码）

① 打开一个新的命令行窗口，按照之前的操作方式进入到虚拟环境bert_env下；

② 进入python环境；

③ 输入以下命令：

from bert_serving.client import BertClient #导入客户端

bc = BertClient() # 创建客户端对象

result = bc.encode(['你好', '我来自中国', '中国是一个历史悠久、文化底蕴深厚的国家'])

result.shape # 查看输出结果shape
```

```

```

