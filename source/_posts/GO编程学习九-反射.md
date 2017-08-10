---
title: GO编程学习九-反射
date: 2017-08-10 22:45:25
category: go
tags: go
---
### value.go中的函数

func Append(s Value, x ...Value) Value
func AppendSlice(s, t Value) Value
func Indirect(v Value) Value
func MakeChan(typ Type, buffer int) Value
func MakeFunc(typ Type, fn func(args []Value) (results []Value)) Value
func MakeMap(typ Type) Value
func MakeSlice(typ Type, len, cap int) Value
func New(typ Type) Value
func NewAt(typ Type, p unsafe.Pointer) Value
func ValueOf(i interface{}) Value
func Zero(typ Type) Value

### value结构的方法
**Addr() Value**
    通常用于获取一个指向结构体字段或slice元素为了调用一个方法,需要一个指针接收机。
**Bool() bool**
    返回底层的值,如果v的kind不是bool则会产生恐慌
**Bytes() []byte**
    返回底层的值,如果v的底层值不是一个字节切片,则会产生恐慌 
**CanAddr() bool**
    检查v是否是可寻址的
**CanSet() bool**
    检查值是否可被设置,只有可寻址的才能被设置 
**Call(in []Value) []Value**
    反射函数的值.并调用 
**CallSlice(in []Value) []Value**
    同上
**Close()**
    关闭channel,如果不是chan则产生恐慌
**Complex() complex128**
    返回底层的值,如果值不是一个复数,则产生一个恐慌
**Elem() Value**
    返回v包含的值,多被用于设置值时的寻址操作
**Field(i int) Value**
    返回结构中索引字段的Value 
**FieldByIndex(index []int) Value**
    同上不过.提供的是一个切片
**FieldByName(name string) Value**
    通过字段名查找
**FieldByNameFunc(match func(string) bool) Value**
    通过函数名查找
**Float() float64**
    返回底层的值,如果值不是一个float,则产生一个恐慌
**Index(i int) Value**
    如果kind不是array或者sliece则差生恐慌,将其中的元素返回为Value
**Int() int64**
    返回底层的值,如果值不是一个int,则产生一个恐慌
**CanInterface() bool**
    如果接口能被使用,则返回true
**Interface() (i interface{})**
    返回V作为interface{}的当前值
**InterfaceData() [2]uintptr**
    如果kind不是一个接口则会产生恐慌
**IsNil() bool**
    如果v是一个nil,则返回true
**IsValid() bool**
    如果v代表一个值,则返回true
**Kind() Kind**
    返回v的种类
**Len() int****
    返回v的长度
**MapIndex(key Value) Value**
    如果是一个map,根据key反射其键值的Value
**MapKeys() []Value**
    返回map的所有key
**Method(i int) Value**
    按索引反射结构某个方法的值
**NumMethod() int**
    统计结构方法数量
**MethodByName(name string) Value**
    反射方法的值根据方法名
**NumField() int**
    反射一个结构的字段数
**OverflowComplex(x complex128) bool**
    覆盖复数
**OverflowFloat(x float64) bool****
    覆盖浮点数
**overflowFloat32(x float64) bool**
**OverflowInt(x int64) bool**
**OverflowUint(x uint64) bool****
**Pointer() uintptr**
    反射一个指针的值.返回一个指针的整型值
**Recv() (x Value, ok bool)**
    用于channel
**Send(x Value)**
    用于channel
**Set(x Value)**
    如果v可设置,则设置一个v的值
**SetBool(x bool)**
    如果v可设置,且是bool,则设置一个v的值
**SetBytes(x []byte)**
**SetComplex(x complex128)**
**SetFloat(x float64)**
**SetInt(x int64)**
**SetLen(n int)**
**SetMapIndex(key, val Value)**
**SetUint(x uint64)**
**SetPointer(x unsafe.Pointer)**
**SetString(x string)**
**Slice(beg, end int) Value**
    如果底层是slice.则返回值.
**String() string**
    如果狄成是字符窜.则返回字符窜
**TryRecv() (x Value, ok bool)**
    用于channel,接受
**TrySend(x Value) bool**
    用于channel,发送
**Type() Type**
    返回type
**Uint() uint64**
    如果狄成是Uint.则返回uint
**UnsafeAddr() uintptr**

### 例子
```go
package main
 
import (
    "fmt"
    "reflect"
)
 
type MyStruct struct {
    name string
}
 
func (this *MyStruct) GetName() string {
    return this.name
}
 
type IStruct interface {
    GetName() string
}
 
func main() {
    // TypeOf
    s := "this is string"
    fmt.Println(reflect.TypeOf(s)) // output: "string"
 
    // object TypeOf
    a := new(MyStruct)
    a.name = "yejianfeng"
    typ := reflect.TypeOf(a)
    fmt.Println(typ)        // output: "*main.MyStruct"
    fmt.Println(typ.Elem()) // output: "main.MyStruct"
 
    // reflect.Type Base struct
    fmt.Println(typ.NumMethod())                   // 1
    fmt.Println(typ.Method(0))                     // {GetName  func(*main.MyStruct) string <func(*main.MyStruct) string Value> 0}
    fmt.Println(typ.Name())                        // ""
    fmt.Println(typ.PkgPath())                     // ""
    fmt.Println(typ.Size())                        // 8
    fmt.Println(typ.String())                      // *main.MyStruct
    fmt.Println(typ.Elem().String())               // main.MyStruct
    fmt.Println(typ.Elem().FieldByIndex([]int{0})) // {name main string  0 [0] false}
    fmt.Println(typ.Elem().FieldByName("name"))    // {name main string  0 [0] false} true
 
    fmt.Println(typ.Kind() == reflect.Ptr)                              // true
    fmt.Println(typ.Elem().Kind() == reflect.Struct)                    // true
    fmt.Println(typ.Implements(reflect.TypeOf((*IStruct)(nil)).Elem())) // true
 
    fmt.Println(reflect.TypeOf(12.12).Bits()) // 64, 因为是float64
 
    cha := make(chan int)
    fmt.Println(reflect.TypeOf(cha).ChanDir()) // chan
 
    var fun func(x int, y ...float64) string
    var fun2 func(x int, y float64) string
    fmt.Println(reflect.TypeOf(fun).IsVariadic())  // true
    fmt.Println(reflect.TypeOf(fun2).IsVariadic()) // false
    fmt.Println(reflect.TypeOf(fun).In(0))         // int
    fmt.Println(reflect.TypeOf(fun).In(1))         // []float64
    fmt.Println(reflect.TypeOf(fun).NumIn())       // 2
    fmt.Println(reflect.TypeOf(fun).NumOut())      // 1
    fmt.Println(reflect.TypeOf(fun).Out(0))        // string
 
    mp := make(map[string]int)
    mp["test1"] = 1
    fmt.Println(reflect.TypeOf(mp).Key()) //string
 
    arr := [1]string{"test"}
    fmt.Println(reflect.TypeOf(arr).Len()) // 1
 
    fmt.Println(typ.Elem().NumField()) // 1
 
    // MethodByName, Call
    b := reflect.ValueOf(a).MethodByName("GetName").Call([]reflect.Value{})
    fmt.Println(b[0]) // output: "yejianfeng"
}
```

```go
    b := 555
    p:=reflect.ValueOf(&b)
	v := p.Elem()  //反射对象 p并不是可寻址的，但是并不希望设置p，（实际上）是 *p。为了获得 p 指向的内容，调用值上的 Elem 方法，从指针间接指向，然后保存反射值的结果叫做 v
	v.SetInt(666)
	fmt.Println(b)
    ------------------------------------
    func test(a string) string {
		return a
	}
	func main() {
		a := "sssssss"
		args := []reflect.Value{reflect.ValueOf(a)}
		c := reflect.ValueOf(test).Call(args)
		fmt.Println(c)
	}
    ------------------------------------
    type A struct {
		a int
		b byte
		c string
	}
	func main() {
		a := A{}
		fmt.Println(reflect.ValueOf(a).Field(0).Int())
	}
        
```

```go
package main

import (
	"fmt"
	"reflect"
)

type a struct {
	Name string
	Age  string
}

func (this a) Test() {
	fmt.Println("test ok.")
}
func (this *a) Get() {
	fmt.Println("get ok.")
}
func (this *a) Print(id int, name string) {
	fmt.Println("id is ", id)
	fmt.Println("name is ", name)
}

func Test() {
	fmt.Println("func test is ok.")
}
func main() {
	d := &a{Name: "aaa", Age: "bbb"}
	v := reflect.ValueOf(d)
	ele := v.Elem()
	t := ele.Type()
	fmt.Println("\n读取对象的所有属性")
	for i := 0; i < t.NumField(); i++ {
		fmt.Println(t.Field(i).Name, ele.Field(i).Interface())
	}
	fmt.Println("\n读取对象的所有方法")
	for i := 0; i < t.NumMethod(); i++ {
		fmt.Println(t.Method(i).Name)
	}
	//反射调用方法
	fmt.Println("\n测试函数调用，看来还是e保险一些，不会出错")
	v.MethodByName("Get").Call(nil)
	v.MethodByName("Test").Call(nil)
	//使用ele.MethodByName("GET").Call(nil)会抛出异常
	//但是使用ele.MethodByName("Test").Call(nil)却是正常？因为Test没有用指针
	ele.MethodByName("Test").Call(nil)
	//调用有参数的反射
	fmt.Println("\n测试传值调用函数")
	vId := reflect.ValueOf(88)
	vName := reflect.ValueOf("小汪")
	v.MethodByName("Print").Call([]reflect.Value{vId, vName})

	fmt.Println("\n测试参数赋值，必须要用ele")
	ele.FieldByName("Name").SetString("哈哈")
	fmt.Println(d)

	c := 123
	tc := reflect.ValueOf(&c).Elem()
	tc.SetInt(333)
	fmt.Println(tc, reflect.TypeOf(&c).Elem().Name())
	fmt.Println(c)

}
```

