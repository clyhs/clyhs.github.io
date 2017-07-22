---
title: GO编程学习七-表单处理
date: 2017-07-22 23:56:35
category: go
tags: go
---
### 处理表单的输入
先来看一个表单递交的例子，我们有如下的表单内容，命名成文件login.gtpl(放入当前新建项目的目录里面)
```html
<html>
<head>
<title></title>
</head>
<body>
<form action="http://127.0.0.1:9090/login" method="post">
用户名:<input type="text" name="username">
密码:<input type="password" name="password">
<input type="submit" value="登陆">
</form>
</body>
</html>
```
上面递交表单到服务器的/login，当用户输入信息点击登陆之后，会跳转到服务器的路由login里面，我们首先要
判断这个是什么方式传递过来，POST还是GET呢？
http包里面有一个很简单的方式就可以获取，我们在前面web的例子的基础上来看看怎么处理login页面的form数据

http包里面有一个很简单的方式就可以获取，我们在前面web的例子的基础上来看看怎么处理login页面的form数据
```go
package main
import (
    "fmt"
    "html/template"
    "log"
    "net/http"
    "strings"
)
func sayhelloName(w http.ResponseWriter, r *http.Request) {
    r.ParseForm() //解析url传递的参数，对于POST则解析响应包的主体（request body）
    //注意:如果没有调用ParseForm方法，下面无法获取表单的数据
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
func login(w http.ResponseWriter, r *http.Request) {
    fmt.Println("method:", r.Method) //获取请求的方法
    if r.Method == "GET" {
        t, _ := template.ParseFiles("login.gtpl")
        t.Execute(w, nil)
    } else {
    //请求的是登陆数据，那么执行登陆的逻辑判断
        r.ParseForm()
        fmt.Println("username:", r.Form["username"])
        fmt.Println("password:", r.Form["password"])
    }
}
func main() {
    http.HandleFunc("/", sayhelloName) //设置访问的路由
    http.HandleFunc("/login", login) //设置访问的路由
    err := http.ListenAndServe(":9090", nil) //设置监听的端口
    if err != nil {
        log.Fatal("ListenAndServe: ", err)
    }
}
```
通过上面的代码我们可以看出获取请求方法是通过r.Method来完成的，这是个字符串类型的变量，返回GET, POST,
PUT等method信息。
login函数中我们根据r.Method来判断是显示登录界面还是处理登录逻辑。当GET方式请求时显示登录界面，其他方
式请求时则处理登录逻辑，如查询数据库、验证登录信息等。
当我们在浏览器里面打开http://127.0.0.1:9090/login的时候，出现如下界面
![img](https://clyhs.github.io/images/go/go_03.png)
我们输入用户名和密码之后发现在服务器端是不会打印出来任何输出的，为什么呢？默认情况下，Handler里面是不
会自动解析form的，必须显式的调用r.ParseForm()后，你才能对这个表单数据进行操作。我们修改一下代码，在
fmt.Println("username:", r.Form["username"])之前加一行r.ParseForm(),重新编译，再次测试输入
递交，现在是不是在服务器端有输出你的输入的用户名和密码了。
r.Form里面包含了所有请求的参数，比如URL中query-string、POST的数据、PUT的数据，所有当你在URL的querystring
字段和POST冲突时，会保存成一个slice，里面存储了多个值，Go官方文档中说在接下来的版本里面将会把
POST、GET这些数据分离开来。
