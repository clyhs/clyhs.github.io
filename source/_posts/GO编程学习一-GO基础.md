---
title: GO编程学习一:GO基础
date: 2017-07-19 15:16:48 
category: go
tags: go
---
## 你好，Go
### 程序
这就像一个传统，在学习大部分语言之前，你先学会如何编写一个可以输出hello world的程序。
准备好了吗？Let's Go!
```go
package main
import "fmt"
func main() {
fmt.Printf("Hello, world \n")
}
```
输出如下：
```go
Hello, world 
```

### 详解
首先我们要了解一个概念，Go程序是通过package来组织的
package <pkgName>（在我们的例子中是package main）这一行告诉我们当前文件属于哪个包，而包名main则
告诉我们它是一个可独立运行的包，它在编译后会产生可执行文件。除了main包之外，其它的包最后都会生成*.a文
件（也就是包文件）并放置在$GOPATH/pkg/$GOOS_$GOARCH中（以Mac为例就是
$GOPATH/pkg/darwin_amd64）。
每一个可独立运行的Go程序，必定包含一个package main，在这个main包中必定包含一个入口函数main，而这
个函数既没有参数，也没有返回值。
为了打印Hello, world...，我们调用了一个函数Printf，这个函数来自于fmt包，所以我们在第三行中导入了
系统级别的fmt包：import "fmt"。