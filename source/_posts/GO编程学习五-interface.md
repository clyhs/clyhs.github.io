---
title: GO编程学习五-interface
date: 2017-07-19 17:14:43
category: go
tags: go
---
## 什么是interface
简单的说，interface是一组method的组合，我们通过interface来定义对象的一组行为。
我们前面一章最后一个例子中Student和Employee都能Sayhi，虽然他们的内部实现不一样，但是那不重要，重要的是
他们都能say hi
### interface类型
interface类型定义了一组方法，如果某个对象实现了某个接口的所有方法，则此对象就实现了此接口。详细的语法
参考下面这个例子
```go
type Human struct {
    name string
    age int
    phone string
}
type Student struct {
    Human //匿名字段Human
    school string
    loan float32
}
type Employee struct {
    Human //匿名字段Human
    company string
    money float32
}
//Human对象实现Sayhi方法
func (h *Human) SayHi() {
    fmt.Printf("Hi, I am %s you can call me on %s\n", h.name, h.phone)
}
// Human对象实现Sing方法
func (h *Human) Sing(lyrics string) {
    fmt.Println("La la, la la la, la la la la la...", lyrics)
}
//Human对象实现Guzzle方法
func (h *Human) Guzzle(beerStein string) {
    fmt.Println("Guzzle Guzzle Guzzle...", beerStein)
}
// Employee重载Human的Sayhi方法
func (e *Employee) SayHi() {
    fmt.Printf("Hi, I am %s, I work at %s. Call me on %s\n", e.name,e.company, e.phone) //Yes you can split into 2 lines here.
}
//Student实现BorrowMoney方法
func (s *Student) BorrowMoney(amount float32) {
    s.loan += amount // (again and again and...)
}
//Employee实现SpendSalary方法
func (e *Employee) SpendSalary(amount float32) {
    e.money -= amount // More vodka please!!! Get me through the day!
}
// 定义interface
type Men interface {
    SayHi()
    Sing(lyrics string)
    Guzzle(beerStein string)
}
type YoungChap interface {
    SayHi()
    Sing(song string)
    BorrowMoney(amount float32)
}
type ElderlyGent interface {
    SayHi()
    Sing(song string)
    SpendSalary(amount float32)
}
```
通过上面的代码我们可以知道，interface可以被任意的对象实现。我们看到上面的Men interface被Human、Student
和Employee实现。同理，一个对象可以实现任意多个interface，例如上面的Student实现了Men和YonggChap两个
interface。

### interface值
那么interface里面到底能存什么值呢？如果我们定义了一个interface的变量，那么这个变量里面可以存实现这个
interface的任意类型的对象。例如上面例子中，我们定义了一个Men interface类型的变量m，那么m里面可以存
Human、Student或者Employee值。
因为m能够持有这三种类型的对象，所以我们可以定义一个包含Men类型元素的slice，这个slice可以被赋予实现了
Men接口的任意结构的对象，这个和我们传统意义上面的slice有所不同。
让我们来看一下下面这个例子
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
    loan float32
}
type Employee struct {
    Human //匿名字段
    company string
    money float32
}
//Human实现Sayhi方法
func (h Human) SayHi() {
    fmt.Printf("Hi, I am %s you can call me on %s\n", h.name, h.phone)
}
//Human实现Sing方法
func (h Human) Sing(lyrics string) {
    fmt.Println("La la la la...", lyrics)
}
//Employee重载Human的SayHi方法
func (e Employee) SayHi() {
    fmt.Printf("Hi, I am %s, I work at %s. Call me on %s\n", e.name,e.company, e.phone) //Yes you can split into 2 lines here.
}
// Interface Men被Human,Student和Employee实现
// 因为这三个类型都实现了这两个方法
type Men interface {
    SayHi()
    Sing(lyrics string)
}
func main() {
    mike := Student{Human{"Mike", 25, "222-222-XXX"}, "MIT", 0.00}
    paul := Student{Human{"Paul", 26, "111-222-XXX"}, "Harvard", 100}
    sam := Employee{Human{"Sam", 36, "444-222-XXX"}, "Golang Inc.", 1000}
    Tom := Employee{Human{"Sam", 36, "444-222-XXX"}, "Things Ltd.", 5000}
//定义Men类型的变量i
    var i Men
//i能存储Student
    i = mike
    fmt.Println("This is Mike, a Student:")
    i.SayHi()
    i.Sing("November rain")
//i也能存储Employee
    i = Tom
    fmt.Println("This is Tom, an Employee:")
    i.SayHi()
    i.Sing("Born to be wild")
//定义了slice Men
    fmt.Println("Let's use a slice of Men and see what happens")
    x := make([]Men, 3)
//T这三个都是不同类型的元素，但是他们实现了interface同一个接口
    x[0], x[1], x[2] = paul, sam, mike
    for _, value := range x{
    value.SayHi()
    }
}
```
通过上面的代码，你会发现interface就是一组抽象方法的集合，它必须由其他非interface类型实现，而不能自我实
现， go 通过interface实现了duck-typing:即"当看到一只鸟走起来像鸭子、游泳起来像鸭子、叫起来也像鸭子，那
么这只鸟就可以被称为鸭子"。
### 空interface
空interface(interface{})不包含任何的method，正因为如此，所有的类型都实现了空interface。空interface对于
描述起不到任何的作用(因为它不包含任何的method），但是空interface在我们需要存储任意类型的数值的时候相当
有用，因为它可以存储任意类型的数值。它有点类似于C语言的void*类型。
// 定义a为空接口
```go
var a interface{}
var i int = 5
s := "Hello world"
// a可以存储任意类型的数值
a = i
a = s
```
一个函数把interface{}作为参数，那么他可以接受任意类型的值作为参数，如果一个函数返回interface{},那么也
就可以返回任意类型的值。是不是很有用啊！
### interface函数参数
interface的变量可以持有任意实现该interface类型的对象，这给我们编写函数(包括method)提供了一些额外的思
考，我们是不是可以通过定义interface参数，让函数接受各种类型的参数。
74
举个例子：fmt.Println是我们常用的一个函数，但是你是否注意到它可以接受任意类型的数据。打开fmt的源码文
件，你会看到这样一个定义:
```go
type Stringer interface {
    String() string
}
也就是说，任何实现了String方法的类型都能作为参数被fmt.Println调用,让我们来试一试
package main
import (
"fmt"
"strconv"
)
type Human struct {
    name string
    age int
    phone string
}
// 通过这个方法 Human 实现了 fmt.Stringer
func (h Human) String() string {
    return "􀀀"+h.name+" - "+strconv.Itoa(h.age)+" years - ✆ " +h.phone+"􀀀"
}
func main() {
    Bob := Human{"Bob", 39, "000-7777-XXX"}
    fmt.Println("This Human is : ", Bob)
}
```
现在我们再回顾一下前面的Box示例，你会发现Color结构也定义了一个method：String。其实这也是实现了
fmt.Stringer这个interface，即如果需要某个类型能被fmt包以特殊的格式输出，你就必须实现Stringer这个接口。
如果没有实现这个接口，fmt将以默认的方式输出。
//实现同样的功能
fmt.Println("The biggest one is", boxes.BiggestsColor().String())
fmt.Println("The biggest one is", boxes.BiggestsColor())
注：实现了error接口的对象（即实现了Error() string的对象），使用fmt输出时，会调用Error()方法，因此不必
再定义String()方法了。

### 嵌入interface
Go里面真正吸引人的是他内置的逻辑语法，就像我们在学习Struct时学习的匿名字段，多么的优雅啊，那么相同的逻
辑引入到interface里面，那不是更加完美了。如果一个interface1作为interface2的一个嵌入字段，那么
interface2隐式的包含了interface1里面的method。
我们可以看到源码包container/heap里面有这样的一个定义
```go
type Interface interface {
    sort.Interface //嵌入字段sort.Interface
    Push(x interface{}) //a Push method to push elements into the heap
    Pop() interface{} //a Pop elements that pops elements from the heap
}
我们看到sort.Interface其实就是嵌入字段，把sort.Interface的所有method给隐式的包含进来了。也就是下面三个
方法
type Interface interface {
    // Len is the number of elements in the collection.
    Len() int
    // Less returns whether the element with index i should sort
    // before the element with index j.
    Less(i, j int) bool
    // Swap swaps the elements with indexes i and j.
    Swap(i, j int)
}
另一个例子就是io包下面的 io.ReadWriter ，他包含了io包下面的Reader和Writer两个interface。
// io.ReadWriter
type ReadWriter interface {
    Reader
    Writer
}
```
