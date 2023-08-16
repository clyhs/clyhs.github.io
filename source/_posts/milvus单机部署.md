title: etcd入门
date: 2023-08-16 23:15:53
category: 人工智能
tags: milvus

# milvus单机部署

Docker 和 Docker Compose环境检查

* Docker 版本 > 19.03 [部署docker](https://blog.csdn.net/u013933879/article/details/118763212)
* Docker Compose 版本 > 1.25.1 [安装Compose](https://docs.docker.com/compose/install/

## 检查CPU 对 SIMD库扩展的支持

CPU需要支持以下指令集中的任意一个

- SSE4.2
- AVX
- AVX2
- AVX512

```
# 检查的命令
lscpu | grep -e sse4_2 -e avx -e avx2 -e avx512
```

## 安装单机版

docker-compose.yml

```
version: '3.5'

services:
  etcd:
    container_name: milvus-etcd
    image: quay.io/coreos/etcd:v3.5.0
    environment:
      - ETCD_AUTO_COMPACTION_MODE=revision
      - ETCD_AUTO_COMPACTION_RETENTION=1000
      - ETCD_QUOTA_BACKEND_BYTES=4294967296
    volumes:
      - ${DOCKER_VOLUME_DIRECTORY:-.}/volumes/etcd:/etcd
    command: etcd -advertise-client-urls=http://127.0.0.1:2379 -listen-client-urls http://0.0.0.0:2379 --data-dir /etcd

  minio:
    container_name: milvus-minio
    image: minio/minio:RELEASE.2020-12-03T00-03-10Z
    environment:
      MINIO_ACCESS_KEY: minioadmin
      MINIO_SECRET_KEY: minioadmin
    volumes:
      - ${DOCKER_VOLUME_DIRECTORY:-.}/volumes/minio:/minio_data
    command: minio server /minio_data
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:9000/minio/health/live"]
      interval: 30s
      timeout: 20s
      retries: 3

  standalone:
    container_name: milvus-standalone
    image: milvusdb/milvus:v2.0.2
    command: ["milvus", "run", "standalone"]
    environment:
      ETCD_ENDPOINTS: etcd:2379
      MINIO_ADDRESS: minio:9000
    volumes:
      - ${DOCKER_VOLUME_DIRECTORY:-.}/volumes/milvus:/var/lib/milvus
    ports:
      - "19530:19530"
    depends_on:
      - "etcd"
      - "minio"
  attu:
    container_name: attu
    image: zilliz/attu:v2.1.0
    environment:
      MILVUS_URL: milvus-standalone:19530
    ports:
      - "8000:3000"
    depends_on:
      - "standalone"
    

networks:
  default:
    name: milvus

```

```
docker-compose up -d
Creating milvus-minio ... done
Creating milvus-etcd  ... done
Creating milvus-standalone ... done
```

通过命令确定单节点安装完成

```
[root@centosapp1 milvus]# docker-compose ps
      Name                     Command                  State                           Ports                     
------------------------------------------------------------------------------------------------------------------
milvus-etcd         etcd -advertise-client-url ...   Up             2379/tcp, 2380/tcp                            
milvus-minio        /usr/bin/docker-entrypoint ...   Up (healthy)   0.0.0.0:9000->9000/tcp, 0.0.0.0:9001->9001/tcp
milvus-standalone   /tini -- milvus run standalone   Exit 132   
```

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
```

## 安装qa_chatbot

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

#修改chinese_L-12_H-768_A-12的bert_config.json
改为config.json

#修改mysql_helpers.py
mysql_helper.py中创建mysql表部分为：
sql = "create table .... answer Text)ENGINE=InnoDB DEFAULT CHARSET=utf8;"

# 
docker restart qa_chatbot_server
```

打开http://192.168.2.219:8282/docs

可以看到服务

### 安装客户端

```bash
qa-chatbot-client
```

```
export API_URL='http:/192.168.2.219:8282'
docker run -d --name qa_chatbot_client -p 8283:80 -e "API_URL=${API_URL}" milvusbootcamp/qa-chatbot-client:v1
```

打开地址：http://192.168.2.219:8283/
