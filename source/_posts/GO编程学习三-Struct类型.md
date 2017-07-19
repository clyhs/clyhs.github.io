---
title: GO编程学习三-Struct类型
date: 2017-07-19 16:16:52
category: go
tags: go 
---
## struct
Go语言中，也和C或者其他语言一样，我们可以声明新的类型，作为其它类型的属性或字段的容器。例如，我们可以
创建一个自定义类型person代表一个人的实体。这个实体拥有属性：姓名和年龄。这样的类型我们称之struct。如
下代码所示:
```go
type person struct {
    name string
    age int
}

var P person // P现在就是person类型的变量了

P.name = "Astaxie" // 赋值"Astaxie"给P的name属性.
P.age = 25 // 赋值"25"给变量P的age属性
fmt.Printf("The person's name is %s", P.name) // 访问P的name属性.
```
除了上面这种P的声明使用之外，还有两种声明使用方式
* 1.按照顺序提供初始化值
P := person{"Tom", 25}
* 2.通过field:value的方式初始化，这样可以任意顺序
P := person{age:24, name:"Tom"}

下面我们看一个完整的使用struct的例子
```go 
package main
import "fmt"
// 声明一个新的类型
type person struct {
    name string
    age int
}
// 比较两个人的年龄，返回年龄大的那个人，并且返回年龄差
// struct也是传值的
func Older(p1, p2 person) (person, int) {
    if p1.age>p2.age { // 比较p1和p2这两个人的年龄
        return p1, p1.age-p2.age
    }
    return p2, p2.age-p1.age
}
func main() {
    var tom person
    // 赋值初始化
    tom.name, tom.age = "Tom", 18
    // 两个字段都写清楚的初始化
    bob := person{age:25, name:"Bob"}
    // 按照struct定义顺序初始化值
    paul := person{"Paul", 43}
    tb_Older, tb_diff := Older(tom, bob)
    tp_Older, tp_diff := Older(tom, paul)
    bp_Older, bp_diff := Older(bob, paul)
    fmt.Printf("Of %s and %s, %s is older by %d years\n",
    tom.name, bob.name, tb_Older.name, tb_diff)
    fmt.Printf("Of %s and %s, %s is older by %d years\n",
    tom.name, paul.name, tp_Older.name, tp_diff)
    fmt.Printf("Of %s and %s, %s is older by %d years\n",
    bob.name, paul.name, bp_Older.name, bp_diff)
}
```

#### struct的匿名字段
我们上面介绍了如何定义一个struct，定义的时候是字段名与其类型一一对应，实际上Go支持只提供类型，而不写字
段名的方式，也就是匿名字段，也称为嵌入字段。
当匿名字段是一个struct的时候，那么这个struct所拥有的全部字段都被隐式地引入了当前定义的这个struct。
让我们来看一个例子，让上面说的这些更具体化
```go
package main
import "fmt"
type Human struct {
    name string
    age int
    weight int
}
type Student struct {
    Human // 匿名字段，那么默认Student就包含了Human的所有字段
    speciality string
}

func main() {
    // 我们初始化一个学生
    mark := Student{Human{"Mark", 25, 120}, "Computer Science"}
    // 我们访问相应的字段
    fmt.Println("His name is ", mark.name)
    fmt.Println("His age is ", mark.age)
    fmt.Println("His weight is ", mark.weight)
    fmt.Println("His speciality is ", mark.speciality)
    // 修改对应的备注信息
    mark.speciality = "AI"
    fmt.Println("Mark changed his speciality")
    fmt.Println("His speciality is ", mark.speciality)
    // 修改他的年龄信息
    fmt.Println("Mark become old")
    mark.age = 46
    fmt.Println("His age is", mark.age)
    // 修改他的体重信息
    fmt.Println("Mark is not an athlet anymore")
    mark.weight += 60
    fmt.Println("His weight is", mark.weight)
}
```