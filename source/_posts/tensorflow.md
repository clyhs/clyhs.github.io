---
title: tensorflow安装部署
date: 2023-08-11 22:45:25
category: 人工智能
tags: tensorflow
---

# tensorflow

## install 

### Anaconda3

```
yum install -y bzip2

wget https://repo.continuum.io/archive/Anaconda3-5.3.0-Linux-x86_64.sh

bash Anaconda3-5.3.0-Linux-x86_64.sh

source ~/.bachrc

# 创建一个名为py3，版本为python3.6得虚拟环境
conda create --name py3 python=3.6

# 列出所有得虚拟环境
conda env list

#**py3这个虚拟环境
source activate py3

#取消**
source deactivate

```

### tensorflow

```
pip install tensorflow==1.13.1
```

### deepspeech

```
pip3 install deepspeech
wget https://github.com/mozilla/DeepSpeech/releases/download/v0.4.1/deepspeech-0.4.1-models.tar.gz
wget https://github.com/mozilla/DeepSpeech/releases/download/v0.4.1/audio-0.4.1.tar.gz

deepspeech --model models/output_graph.pbmm --alphabet models/alphabet.txt --lm models/lm.binary --trie models/trie --audio audio/4507-16021-0012.wav
```

