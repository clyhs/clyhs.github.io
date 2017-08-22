---
title: GO编程学习十-并发模型管道和取消
date: 2017-08-22 15:16:45
category: go
tags: go
---
## 简介

Golang的原子并发特性使得它很容易构造流数据管道，这使得Golang可有效的使用I/O和多CPU特性。本文提出一些关于管道的示例，在这个过程中突出了操作失败的微妙之处和介绍处理失败的具体技术。

### 什么是管道

在Golang对于管道没有明确的定义；它只是许多种并发程序中的一种。管道是通道连接的一系列阶段， 每个阶段是一组goroutine运行相同的功能。在每个阶段，goroutine运行步骤为：

* 从上游经过入境通道接受值

* 对数据执行一些功能操作，通常会产生新的值

* 从下游经过出境通道发送值

除了开始和最后阶段只有一个入境通道或者一个出境通道外，其他每个阶段有任意数量的入境通道和出境通道，。开始阶段有时又称为源或者生产者；最后一个阶段又称为sink或者消费者。

### 平方数
一个通道有三个阶段。

第一阶段：gen，以从列表读出整数的方式转换整数列表到一个通道。gen函数开始goroutine后， 在通道上发送整数并且在在所有的值被发送完后将通道关闭：

```go
func gen(nums ...int) <-chan int {
    out := make(chan int)
    go func() {
        for _, n := range nums {
            out <- n
        }
        close(out)
    }()
    return out
}
```

第二阶段：sq，从通道接受整数，然后将接受到的每个整数值的平方后返回到一个通道 。在入境通道关闭和发送所有下行值的阶段结束后，关闭出口通道：

```go
func sq(in <-chan int) <-chan int {
    out := make(chan int)
    go func() {
        for n := range in {
            out <- n * n
        }
        close(out)
    }()
    return out
}
```

main函数建立了通道并运行最后一个阶段：它接受来自第二阶段的值并打印出每个值，直到通道关闭：
```go
func main() {
    // Set up the pipeline.
    c := gen(2, 3)
    out := sq(c)

    // Consume the output.
    fmt.Println(<-out) // 4
    fmt.Println(<-out) // 9
}
```

由于sq有相同类型的入境和出境通道，我们可以写任意次。我们也可以重写main函数，像其他阶段一样做一系列循环 ：

```go
func main() {
    // Set up the pipeline and consume the output.
    for n := range sq(sq(gen(2, 3))) {
        fmt.Println(n) // 16 then 81
    }
}
```

### 扇出，扇入

* 扇出（fan-out）：多个函数能从相同的通道中读数据，直到通道关闭；这提供了一种在一组“人员”中分发任务的方式，使得CPU和I/O的并行处理.

* 扇入（fan-in）：一个函数能从多个输入中读取并处理数据，而这多个输入通道映射到一个单通道，该单通道随着所有输入的结束而关闭。

我们可以改变通道去运行两个sq实例，每个实例从相同的输入通道读取数据。我们引入了一个新函数merge去扇入结果：
```go
func main() {
    in := gen(2, 3)

    // Distribute the sq work across two goroutines that both read from in.
    c1 := sq(in)
    c2 := sq(in)

    // Consume the merged output from c1 and c2.
    for n := range merge(c1, c2) {
        fmt.Println(n) // 4 then 9, or 9 then 4
    }
}

```

merge函数通过为每一个入境通道开启一个goroutine去复制数值到唯一的出境通道，从而实现了转换通道列表到一个单通道 。一旦所有的output goroutine启动，所有在通道上的发送完成后merge函数开启一个以上的goroutine用于关闭出境通道。

在一个关闭的通道上发送没有意义，所以在关闭之前确保所有的发送完成是重要的。sync.WaitGroup类型提供了一个简单的方法去组织同步：

```go
func merge(cs ...<-chan int) <-chan int {
    var wg sync.WaitGroup
    out := make(chan int)

    // Start an output goroutine for each input channel in cs.  output
    // copies values from c to out until c is closed, then calls wg.Done.
    output := func(c <-chan int) {
        for n := range c {
            out <- n
        }
        wg.Done()
    }
    wg.Add(len(cs))
    for _, c := range cs {
        go output(c)
    }

    // Start a goroutine to close out once all the output goroutines are
    // done.  This must start after the wg.Add call.
    go func() {
        wg.Wait()
        close(out)
    }()
    return out
}
```

### 短暂停止

管道函数模型：

* 当所有的发送操作结束后， 阶段关闭他们的出境通道。

* 阶段持续接收来自入境通道的值，直到那些通道关闭。

这个模型允许每一个接收阶段通过range循环的写数据，确保一旦所有向下游发送的值发送成功，所有的goroutine退出。 

但在一个真实的管道上，阶段并不总是接收所有的入境值。有时设计是这样的：接收者可能只需要一个子集值就能取得进展。更多时候是一个阶段早早的退出，因为一个入境值代表一个早期阶段的错误。 在这两种情况下接收者不应该等待剩余的值到达，我们想要早期阶段停止产生后续阶段不需要的值。


在我们的示例中，如果一个阶段不能处理所有的入境值，那么试图发送这些值得goroutine将无限期的阻塞：
```go
// Consume the first value from output.
    out := merge(c1, c2)
    fmt.Println(<-out) // 4 or 9
    return
    // Since we didn't receive the second value from out,
    // one of the output goroutines is hung attempting to send it.
}
```

这是一个资源锁：goroutine消耗内存和运行资源，并且在goroutine栈中的堆引用防止数据被回收。Goroutine不能垃圾回收；它们必须自己退出。

当下游阶段在接收所有的入境值失败后，我们需要安排管道的上游阶段退出。一种实现方法是将出境通道改为一个缓冲区。该缓冲区能保存固定数量的值；如果缓冲区有空闲就立即发送操作完成信号：
```go
c := make(chan int, 2) // buffer size 2
c <- 1  // succeeds immediately
c <- 2  // succeeds immediately
c <- 3  // blocks until another goroutine does <-c and receives 1
```

当在通道创建就预先知道待发送的数值个数时，通过使用缓冲区可以简化代码。例如，我们可以重写  gen 来将整数列表复制到带有缓冲区的通道中，也可以避免创建一个新的 goroutine：
```go
func gen(nums ...int) <-chan int {
    out := make(chan int, len(nums))
    for _, n := range nums {
        out <- n
    }
    close(out)
    return out
}

```
回到管道中处于阻塞状态的 goroutine，我们可以考虑为 merge 返回的出境通道增加一个缓冲区：
```go
func merge(cs ...<-chan int) <-chan int {
    var wg sync.WaitGroup
    out := make(chan int, 1) // enough space for the unread inputs
    // ... the rest is unchanged ...


```

尽管它修正了程序中 goroutine 的阻塞问题，但却不能称为好代码。在这里，缓冲区大小选取为 1，取决于预知 merge 将会接收的数值个数及下游各阶段将会消费的数值个数。这很脆弱：如果我们给 gen 多传了一个数值，或者下游阶段少读了一些数值，goroute 的阻塞问题会再次出现。

作为代替，我们需要为下游各阶段提供一种手段，来向发送方表明指明它们将停止接收数据的输入。

### 显式取消

当main没有接受完out所有的值就决定退出时，它必须告知上游状态(upstream stage)的goroutines，让它丢弃正在发送中的数据。通过在一个叫做done的channel上发送数据，即可实现。例子里有两个受阻的发送方，所以发送的值有两组：

```go
func main() {
    in := gen(2, 3)

    // Distribute the sq work across two goroutines that both read from in.
    c1 := sq(in)
    c2 := sq(in)

    // Consume the first value from output.
    done := make(chan struct{}, 2)
    out := merge(done, c1, c2)
    fmt.Println(<-out) // 4 or 9

    // Tell the remaining senders we're leaving.
    done <- struct{}{}
    done <- struct{}{}
}
```

使用select语句，让发送中的goroutines取代了发送操作。这条语句既可以处理在发送out的情形，也可以处理从done中接受一个值的情况。done的值类型是空结构，因为它的数值并不重要:它是一个接受事件，表明out的发送应该被丢弃。output goroutines继续在channel c内循环运行,而不会阻塞上游状态(upstream stage)：
```go
func merge(done <-chan struct{}, cs ...<-chan int) <-chan int {
    var wg sync.WaitGroup
    out := make(chan int)

    // Start an output goroutine for each input channel in cs.  output
    // copies values from c to out until c is closed or it receives a value
    // from done, then output calls wg.Done.
    output := func(c <-chan int) {
        for n := range c {
            select {
            case out <- n:
            case <-done:
            }
        }
        wg.Done()
    }
    // ... the rest is unchanged ...

```

但是这种方法有个问题：下游的接收者需要知道潜在会被阻塞的上游发送者的数量。追踪这些数量不仅枯燥，还容易出错。


我们要有一个方法告知一个未知的、无限数量的go程序向下游发送它们的值。在GO里面我们通过关闭一个通道来实现，因为一个在已关闭通道上的接收操作总能立即执行，并返回该元素类型的零值。

这意味着main函数只需关闭“done”通道就能开启所有发送者。close实际上是传给发送者的一个广播信号。我们扩展每一个管道函数接收“done”参数并通过一个“defer”语句触发“close”，这样所有来自main的返回路径都会以信号通知管道退出。

```go
func main() {
    // Set up a done channel that's shared by the whole pipeline,
    // and close that channel when this pipeline exits, as a signal
    // for all the goroutines we started to exit.
    done := make(chan struct{})
    defer close(done)

    in := gen(done, 2, 3)

    // Distribute the sq work across two goroutines that both read from in.
    c1 := sq(done, in)
    c2 := sq(done, in)

    // Consume the first value from output.
    out := merge(done, c1, c2)
    fmt.Println(<-out) // 4 or 9

    // done will be closed by the deferred call.
}
```

管道里的每个状态现在都可以随意的提早退出了：sq可以在它的循环中退出，因为我们知道如果done已经被关闭了，也会关闭上游的gen状态。sq通过defer语句，保证不管从哪个返回路径，它的outchannel 都会被关闭。

```go
func sq(done <-chan struct{}, in <-chan int) <-chan int {
    out := make(chan int)
    go func() {
        defer close(out)
        for n := range in {
            select {
            case out <- n * n:
            case <-done:
                return
            }
        }
    }()
    return out
}
```
> http://studygolang.com/articles/1143
























