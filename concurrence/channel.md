# Channel

Goroutine 是 Go 程序并发的执行体，channel 是它们之间的连接。channel 可以让一个 goroutine 发送特定值到另一个 goroutine 的通信机制。



```go
var 变量 chan 元素类型
```



### Channel 操作

通道有发送（send）、接收（receive）和关闭（close）三种操作。

发送和接收都使用  `<-`  符号。

现在我们先使用以下语句定义一个通道：

```go
ch := make(chan int)
```



#### **发送**

将一个值发送到通道中。

```go
ch <- 10 // 把10发送到ch中
```



#### 接收

从一个通道中接收值。

```go
x := <- ch // 从 ch 中接收值并赋值给变量x
<- ch      // 从 ch 中接收值，忽略结果
```



#### 关闭

通过调用内置的 `close`  函数来关闭通道

```go
close(chgo)
```
