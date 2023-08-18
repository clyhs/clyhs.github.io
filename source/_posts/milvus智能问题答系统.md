---
title: milvus智能问题系统
date: 2023-08-18 16:15:53
category: 人工智能
tags: milvus
---

# milvus智能问题系统

## 安装数据库

```
wget https://github.com/milvus-io/milvus/releases/download/v2.2.12/milvus-standalone-docker-compose.yml -O docker-compose.yml
docker-compose up -d
```

## 源码部署2.2.10

### server服务

```
git clone https://github.com/milvus-io/bootcamp.git
git checkout -b v2.2.10 origin/v2.2.10

# 配置conda环境
conda create --name milvus python=3.10
# 进入 bootcamp/solutions/nlp/question_answering_system/server
# 安装依赖环境
pip install -r requirements.txt
# (pymilvus==2.2.11对应的数据库milvus版本2.2.12)
# 进入执行 server/src 
python main.py
# 启动成功
INFO:     Started server process [7576]
INFO:     Waiting for application startup.
INFO:     Application startup complete.
INFO:     Uvicorn running on http://0.0.0.0:8282 (Press CTRL+C to quit)

```

### client客户端

进入 bootcamp/solutions/nlp/question_answering_system/client

```
# 修改pubic/env-config.js
window._env_ = {
  API_URL: "http://ip:port",
  LAN: "en",
};
# 编译
npm install
# 启动
npm run start
```

*注意*

```
出错如：RuntimeError: Node-sentence-embedding/sbert-0 runs failed, error msg: Create sentence-embedding/sbert-0 operator sentence-embedding/sbert:main with args None and kws {'model_name': 'all-MiniLM-L12-v2'} failed, err: No module named 'sentence_transformers', Traceback (most recent call last):
# 安装下面
pip install sentence_transformers

# 如果遇到 TypeError: constr() got an unexpected keyword argument 'regex'
pip install pydantic==1.10.10
pip install transformers

# 如果遇到模型找不到,修改encode.py
ops.sentence_embedding.sbert(model_name='bert-base-chinese'))
```

## docker安装

**此版本对应的milvus 2.0.2**

### 安装服务qa-chatbot-server

```
export MILVUS_HOST='127.0.0.1'
export MILVUS_PORT='19530'
export MYSQL_HOST='127.0.0.1'
export MYSQL_USER='root'
export MYSQL_PWD='tiancom'
export MYSQL_DB='dn3'
export MYSQL_USER='root'
export MYSQL_PWD='****'
export MYSQL_DB='dn3'

# 启动服务
docker run -d --name qa_chatbot_server -p 8282:8000 -e "MILVUS_HOST=${MILVUS_HOST}" -e "MILVUS_PORT=${MILVUS_PORT}" -e "MYSQL_HOST=${MYSQL_HOST}" -e "MYSQL_USER=${MYSQL_USER}" -e "MYSQL_PWD=${MYSQL_PWD}" -e "MYSQL_DB=${MYSQL_DB}" -e "MYSQL_PORT=${MYSQL_PORT}"  milvusbootcamp/qa-chatbot-server:v1

# 加载模型
docker cp chinese_L-12_H-768_A-12.zip qa_chatbot_server:/app/src/models/

# 修改app/src/config.py
#MODEL_PATH = '/app/src/models/paraphrase-mpnet-base-v2'
MODEL_PATH = '/app/src/models/chinese_L-12_H-768_A-12'

# 转模型tensorflow->pytorch
git clone https://github.com/xieyufei1993/Bert-Pytorch-Chinese-TextClassification.git
export BERT_BASE_DIR=/Users/chenliyu/clyhs/workspace/milvus/chinese_L-12_H-768_A-12
python3 convert_tf_checkpoint_to_pytorch.py \
  --tf_checkpoint_path $BERT_BASE_DIR/bert_model.ckpt \
  --bert_config_file $BERT_BASE_DIR/bert_config.json \
  --pytorch_dump_path $BERT_BASE_DIR/pytorch_model.bin

#修改chinese_L-12_H-768_A-12的bert_config.json
改为config.json

#修改mysql_helpers.py
mysql_helper.py中创建mysql表部分为：
sql = "create table .... answer Text)ENGINE=InnoDB DEFAULT CHARSET=utf8;"

# 启动
docker restart qa_chatbot_server
```

打开http://192.168.2.219:8282/docs

可以看到服务

![img](https://clyhs.github.io/images/ai/qa_chatbot_server.png)

### 安装客户端

```bash
qa-chatbot-client
```

```
export API_URL='http:/192.168.2.219:8282'
docker run -d --name qa_chatbot_client -p 8283:80 -e "API_URL=${API_URL}" milvusbootcamp/qa-chatbot-client:v1
```

打开地址：http://192.168.2.219:8283

![img](https://clyhs.github.io/images/ai/qa_chatbot_client.png)

上传 [example.csv](https://clyhs.github.io/images/ai/example.csv)文件

