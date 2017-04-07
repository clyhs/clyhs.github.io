---
title: go之web框架bee入门
date: 2017-04-07 19:22:31
category: "go"
tags: go
---
### 从GIT上获取bee包
```
# go get github.com/beego/bee
# go get github.com/astaxie/beego
```
下载完了之后去$GOPATH/bin查看是否已经生成bee文件，如果在win下是bee.exe
将$GOPATH/bin添加到环境变量PATH后面

### 利用bee来创建WEB
```
# cd $GOPATH/src/github.com/clyhs
# bee new helloweb
# cd helloweb
# go build
```
如下图（win7下面执行，先安装git）
![](https://github.com/clyhs/clyhs.github.io/blob/master/images/go/go_01.png?raw=true)
### 运行生成服务
```
# ./helloweb
```
通过浏览器访问http://localhost:8080
