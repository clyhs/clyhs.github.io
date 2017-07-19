---
title: GO编程学习四-面向对象
date: 2017-07-19 16:32:54
category: go
tags: go 
---
## method
现在假设有这么一个场景，你定义了一个struct叫做长方形，你现在想要计算他的面积，那么按照我们一般的思路应
该会用下面的方式来实现
```go
package main
import (
"fmt"
"math"
)
type Rectangle struct {
    width, height float64
}
type Circle struct {
    radius float64
}
func (r Rectangle) area() float64 {
    return r.width*r.height
}
func (c Circle) area() float64 {
    return c.radius * c.radius * math.Pi
}
func main() {
    r1 := Rectangle{12, 2}
    r2 := Rectangle{9, 4}
    c1 := Circle{10}
    c2 := Circle{25}
    fmt.Println("Area of r1 is: ", r1.area())
    fmt.Println("Area of r2 is: ", r2.area())
    fmt.Println("Area of c1 is: ", c1.area())
    fmt.Println("Area of c2 is: ", c2.area())
}
```

## method继承
前面一章我们学习了字段的继承，那么你也会发现Go的一个神奇之处，method也是可以继承的。如果匿名字段实现了
一个method，那么包含这个匿名字段的struct也能调用该method。让我们来看下面这个例子
```go
package main
import "fmt"
type Human struct {
    name string
    age int
    phone string
}
type Student struct {
    Human //匿名字段
    school string
}
type Employee struct {
    Human //匿名字段
    company string
}
//在human上面定义了一个method
func (h *Human) SayHi() {
    fmt.Printf("Hi, I am %s you can call me on %s\n", h.name, h.phone)
}
func main() {
    mark := Student{Human{"Mark", 25, "222-222-YYYY"}, "MIT"}
    sam := Employee{Human{"Sam", 45, "111-888-XXXX"}, "Golang Inc"}
    mark.SayHi()
    sam.SayHi()
}
```
## method重写
上面定义一个method，重写了匿名字段的方法。请看下面的例子
```go 
package main
import "fmt"
type Human struct {
    name string
    age int
    phone string
}
type Student struct {
    Human //匿名字段
    school string
}
type Employee struct {
    Human //匿名字段
    company string
}
//Human定义method
func (h *Human) SayHi() {
    fmt.Printf("Hi, I am %s you can call me on %s\n", h.name, h.phone)
}
//Employee的method重写Human的method
func (e *Employee) SayHi() {
    fmt.Printf("Hi, I am %s, I work at %s. Call me on %s\n", e.name,e.company, e.phone) //Yes you can split into 2 lines here.
}
func main() {
    mark := Student{Human{"Mark", 25, "222-222-YYYY"}, "MIT"}
    sam := Employee{Human{"Sam", 45, "111-888-XXXX"}, "Golang Inc"}
    mark.SayHi()
    sam.SayHi()
}
```