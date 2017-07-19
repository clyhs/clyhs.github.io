---
title: GO编程学习二:流程与函数
date: 2017-07-19 15:25:09 
category: go
tags: go
---
## 流程和函数
### 流程控制
#### if
Go里面if条件判断语句中不需要括号，如下代码所示
```go
if x > 10 {
    fmt.Println("x is greater than 10")
} else {
    fmt.Println("x is less than 10")
}
```
Go的if允许声明一个变量，这个变量的作用域只能在该条件逻辑块内，其
他地方就不起作用了，如下所示
```go
// 计算获取值x,然后根据x返回的大小，判断是否大于10。
if x := computedValue(); x > 10 {
    fmt.Println("x is greater than 10")
} else {
    fmt.Println("x is less than 10")
}
```
//这个地方如果这样调用就编译出错了，因为x是条件里面的变量
fmt.Println(x)

#### goto
Go有goto语句——请明智地使用它。用goto跳转到必须在当前函数内定义的标签。例如假设这样一个循环：
```go
func myFunc() {
    i := 0
Here: //这行的第一个词，以冒号结束作为标签
    println(i)
    i++
    goto Here //跳转到Here去
}
```
标签名是大小写敏感的

#### for
Go的语法如下：
```go
for expression1; expression2; expression3 {
    //...
}
```
看看下面的例子吧：
```go
package main
import "fmt"
func main(){
    sum := 0;
    for index:=0; index < 10 ; index++ {
        sum += index
    }
    fmt.Println("sum is equal to ", sum)
}
// 输出：sum is equal to 45
```
有些时候忽略expression1和expression3：
```go
sum := 1
for ; sum < 1000; {
    sum += sum
}

其中;也可以省略

sum := 1
for sum < 1000 {
    sum += sum
}
```
在循环里面有两个关键操作break和continue ,break操作是跳出当前循环，continue是跳过本次循环。
```go
for配合range可以用于读取slice和map的数据：
for k,v:=range map {
    fmt.Println("map's key:",k)
    fmt.Println("map's val:",v)
}
```
由于 Go 支持 “多值返回”, 而对于“声明而未被调用”的变量, 编译器会报错, 在这种情况下, 可以使用"_"来丢弃
不需要的返回值 例如
```go
for _, v := range map{
    fmt.Println("map's val:", v)
}
```

#### switch
有些时候你需要写很多的if-else来实现一些逻辑处理，这个时候代码看上去就很丑很冗长，而且也不易于以后的维
护，这个时候switch就能很好的解决这个问题。它的语法如下
```go
switch sExpr {
case expr1:
    some instructions
case expr2:
    some other instructions
case expr3:
    some other instructions
default:
    other code
}
```
sExpr和expr1、expr2、expr3的类型必须一致。Go的switch非常灵活，表达式不必是常量或整数，执行的过程

### 函数
函数是Go里面的核心设计，它通过关键字func来声明，它的格式如下：
```go
func funcName(input1 type1, input2 type2) (output1 type1, output2 type2) {
//这里是处理逻辑代码
//返回多个值
    return value1, value2
}
```
上面的代码我们看出
关键字func用来声明一个函数funcName
函数可以有一个或者多个参数，每个参数后面带有类型，通过,分隔
函数可以返回多个值
上面返回值声明了两个变量output1和output2，如果你不想声明也可以，直接就两个类型
如果只有一个返回值且不声明返回值变量，那么你可以省略 包括返回值 的括号
如果没有返回值，那么就直接省略最后的返回信息
如果有返回值， 那么必须在函数的外层添加return语句
下面我们来看一个实际应用函数的例子（用来计算Max值）
```go
package main
import "fmt"
// 返回a、b中最大值.
func max(a, b int) int {
    if a > b {
        return a
    }
    return b
}
func main() {
    x := 3
    y := 4
    z := 5
    max_xy := max(x, y) //调用函数max(x, y)
    max_xz := max(x, z) //调用函数max(x, z)
    fmt.Printf("max(%d, %d) = %d\n", x, y, max_xy)
    fmt.Printf("max(%d, %d) = %d\n", x, z, max_xz)
    fmt.Printf("max(%d, %d) = %d\n", y, z, max(y,z)) // 也可在这直接调用它
}
```
上面这个里面我们可以看到max函数有两个参数，它们的类型都是int，那么第一个变量的类型可以省略（即 a,b
int,而非 a int, b int)，默认为离它最近的类型，同理多于2个同类型的变量或者返回值。同时我们注意到它的返
回值就是一个类型，这个就是省略写法。
#### 多个返回值
我们直接上代码看例子
```go
package main
import "fmt"
//返回 A+B 和 A*B
func SumAndProduct(A, B int) (int, int) {
    return A+B, A*B
}
func main() {
    x := 3
    y := 4
    xPLUSy, xTIMESy := SumAndProduct(x, y)
    fmt.Printf("%d + %d = %d\n", x, y, xPLUSy)
    fmt.Printf("%d * %d = %d\n", x, y, xTIMESy)
}
```
上面的例子我们可以看到直接返回了两个参数，当然我们也可以命名返回参数的变量，这个例子里面只是用了两个类
型，我们也可以改成如下这样的定义，然后返回的时候不用带上变量名，因为直接在函数里面初始化了。但如果你的
函数是导出的(首字母大写)，官方建议：最好命名返回值，因为不命名返回值，虽然使得代码更加简洁了，但是会造
成生成的文档可读性差。
```go
func SumAndProduct(A, B int) (add int, Multiplied int) {
   add = A+B
   Multiplied = A*B
   return
}
```

