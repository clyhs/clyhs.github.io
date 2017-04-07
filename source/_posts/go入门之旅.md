---
title: go入门之旅
date: 2017-04-07 18:45:41
category: "go"
tags: "go"
---
### 系统版本
```
centos7
```
### go安装
```
# yum install go
```
### 查看go版本
```
# go version
# go version go1.6.3 linux/amd4
```
### 配置环境变量
```
# cd ~
# mkdir go
```
编辑profile

```
# vi /etc/profile
# export GOPATH=$HOME/go
# export PATH=$GOPATH/bin:$PATH
# source /etc/profile
# echo $GOPATH
/root/go 
```
### 新建一个项目
```
# mkdir -p $GOPATH/src/github.com/clyhs(这里是你的用户名)
# cd $GOPATH/src/github.com/clyhs
# git clone https://github.com/clyhs/hello-go.git
# cd hello-go
```
### 编写`main.go`
~~~go
package main
func main() {
    println("hello!")
}
~~~
调用 go build编译当前目录
```
# go build
# ./hello-go
hello!
```
### 提交到git仓库
```
# git add .
# git commit -m "first commit"
# git push -u origin master
```
### 在$GOPATH/bin生成hello-go文件
```
go get github.com/clyhs/hello-go
```
