---
title: shipyard编译入门
date: 2017-04-10 15:38:37
category: shipyard
tags: [shipyard,docker]
---
# shipyard
## shipyard简介
Shipyard是一款开源的图形化的Docker管理工具

------

### 环境要求

所以整个项目需要的环境如下:`centos7`
* go环境
* godep工具
* nvm
* bower
* node.js

### 获取shipyard源码
```
# cd $GOPATH/src/github.com/you-username/
# git clone https://github.com/shipyard/shipyard.git

```
### liteide
通过LiteIDE或者其它工具打开源码，进行BUILD，提示相关的缺少的依赖需要手动下载到本地
比如：提示缺 github.com/xxx/xxx
手动下载
｀｀｀
＃ go get github.com/xxx/xxx
｀｀｀
总共有10个左右依赖下载，直到BUILD通过
### 进入shipyard的controller
```
# godep go build-a -tags “netgo static_build” -installsuffix netgo
```
### UI编译
进入到controller/static目录，然后直接输入如下命令回车即可
```
# bower -s install–allow-root -p | xargs echo > /dev/null
```
编译完成之后会在static目录下生成一个bower_components目录

### 构建本地docker镜像
```
# docker build –t shipclub/shipyard .
```