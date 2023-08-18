---
title: Asrt开源语音识别
date: 2023-08-09 23:15:53
category: 人工智能
tags: Asrt
---

# Asrt开源语音识别

* https://gitee.com/ailemon/ASRT_SpeechRecognition
* https://gitee.com/ailemon/ASRT_SDK_Java

## docker 部署

```
docker pull ailemondocker/asrt_service:1.3.0
docker run --rm -it -p 20001:20001 -p 20002:20002 --name asrt-server -d ailemondocker/asrt_service:1.3.0

```



