---
title: fastchat
date: 2023-08-30 09:57:00 
category: 人工智能
tags: fastchat
---

# fastchat

## 安装

```
conda create --name fastchat python=3.10
pip install fschat
pip3 install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cpu

```

*源码*
 git clone https://github.com/lm-sys/FastChat.git
 cd FastChat



## 合并模型

首先需要下载 Vicuna模型的 delta 参数。

```
git clone https://huggingface.co/lmsys/vicuna-7b-delta-v1.1
```

注意git默认没有lfs支持，clone下来是没有二进制的模型文件的，所以要从[https://huggingface.co/lmsys/vicuna-7b-delta-v1.1/tree/main](https://link.zhihu.com/?target=https%3A//huggingface.co/lmsys/vicuna-7b-delta-v1.1/tree/main)下载那两个bin文件，放到vicuna-7b-delta-v1.1目录下。

```bash
git lfs pull
git lfs install
```



其次把 LLaMA 参数与 Vicuna 的 delta 参数进行合并，得到 Vicuna 模型参数。

```
python -m fastchat.model.apply_delta
	--base-model-path ./output
	--target-model-path ./vicuna-7b
	--delta-path ./vicuna-7b-delta-v1.1
# 如下：

python3 -m fastchat.model.apply_delta \
--base-model-path /path/to/llama-7b \
--target-model-path /path/to/output/vicuna-7b \
--delta-path lmsys/vicuna-7b-delta-v1.1

python3 -m fastchat.model.apply_delta \
--base-model-path /path/to/llama-13b \
--target-model-path /path/to/output/vicuna-13b \
--delta-path lmsys/vicuna-13b-delta-v1.1
	
	
```

## 运行模型

```
python -m fastchat.serve.cli --model-path xxx
```

模型可以到https://huggingface.co/lmsys下载

lmsys/fastchat-t5-3b-v1.0

vicuna-7b-v1.3



模型可以在 GPU 上运行。

```
python -m fastchat.serve.cli --model-path ./vicuna-7b
```

若想让模型运行在CPU上，则使用`--device` 参数。

python -m fastchat.serve.cli --model-path ./vicuna-7b --device cpu

使用 `--load-8bit` 参数减小占用的内存。

```
python -m fastchat.serve.cli --model-path ./vicuna-7b --load-8bit
```



## webui

FastChat还提供了web界面可以使用，具体流程如下

#### **启动控制器**

```
python3 -m fastchat.serve.controller

```

#### **启动模型工作者**

```javascript
python3 -m fastchat.serve.model_worker --model-path lmsys/vicuna-7b-v1.3
```

为确保您的模型工作者已正确连接到控制器，请使用以下命令发送测试消息：

```javascript
python3 -m fastchat.serve.test_message --model-name vicuna-7b-v1.3
```

#### **启动Gradio Web服务器**

```javascript
python3 -m fastchat.serve.gradio_web_server
```



*备注：pip 源慢*

1.清华大学：https://pypi.tuna.tsinghua.edu.cn/simple/     （常用）

2.豆瓣：https://pypi.douban.com/simple/

3.阿里云：https://mirrors.aliyun.com/pypi/simple/

4.中国科学技术大学：https://pypi.mirrors.ustc.edu.cn/simple/