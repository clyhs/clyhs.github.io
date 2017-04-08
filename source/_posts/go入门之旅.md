---
title: go入门之旅
date: 2017-04-07 18:45:41
category: "go"
tags: "go"
---
# go入门
## go简介
> Go语言是由谷歌开发出来的一种新的语言。

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
### 编写一个HTTP服务
```
# cd $GOPATH/src/github.com/clyhs
# mkdir helloworld
# cd helloworld
# vi main.go

package main
import (
    "fmt"
    "net/http"
    "strings"
    "log"
)
func sayhelloName(w http.ResponseWriter, r *http.Request) {
    r.ParseForm() //解析参数，默认是不会解析的
    fmt.Println(r.Form) //这些信息是输出到服务器端的打印信息
    fmt.Println("path", r.URL.Path)
    fmt.Println("scheme", r.URL.Scheme)
    fmt.Println(r.Form["url_long"])
    for k, v := range r.Form {
        fmt.Println("key:", k)
        fmt.Println("val:", strings.Join(v, ""))
    }
    fmt.Fprintf(w, "Hello astaxie!") //这个写入到w的是输出到客户端的
}
func main() {
    http.HandleFunc("/", sayhelloName) //设置访问的路由
    err := http.ListenAndServe(":8080", nil) //设置监听的端口
    if err != nil {
        log.Fatal("ListenAndServe: ", err)
    }
}

# go build
# ./helloworld
```
在浏览器里输入http://localhost:8080
```
输出`Hello astaxie!`
```
