# goroutine

最简单的 `goroutine`:

```go
package main

import (
	"fmt"
	"time"
)

// goroutine

func hello() {
	fmt.Println("Hello, Gophist!")
}

func main() { // 开启一个主 goroutine 去执行 main 函数
	go hello() // 开启了一个 goroutine 去执行 hello 这个函数
	fmt.Println("Hello, main!")
	time.Sleep(time.Second) // 等待一秒
	// 这句需要写 如果只写一个go语句, 后面没有可以运行程序的话就会出现一个尴尬的问题,
	// 主进程直接关闭, 对应goroutine也直接关闭,导致函数没有运行
}

```

Go 的操作系统线程和 `goroutine` 的关系：

1. 一个操作系统线程对应用户态多个 `goroutine`
2. go程序可以同时使用多个操作系统线程
3. goroutine 和 系统线程是多对多的关系，即 m 对 n
