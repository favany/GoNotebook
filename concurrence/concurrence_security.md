# 并发安全和锁

## Go 并发：互斥锁、读写互斥锁和并发安全

**作者：刘俊 Bingo Gophist**

<details>

<summary>参考文档</summary>

* Go语言中的并发编程 [https://liwenzhou.com/posts/Go/14\_concurrence/](https://liwenzhou.com/posts/Go/14\_concurrence/)
* Go语言并发编程：互斥锁 [https://blog.51cto.com/u\_15060545/4177914](https://blog.51cto.com/u\_15060545/4177914)
* Go语言基础之并发同步与锁 [https://www.bilibili.com/video/BV1ZJ411W7jG?p=25](https://www.bilibili.com/video/BV1ZJ411W7jG?p=25)
* go语言:sync.Once的用法 [https://studygolang.com/articles/5711](https://studygolang.com/articles/5711)

</details>

在 Go 中，可能会存在多个 `Goroutine` 同时操作一个资源，会发生竞态问题。类比现实生活中，有十字路口，被多个方向的汽车竞争；多人同时要上一辆地铁。

比如，下面这个例子：

[点这里，在线试试！](https://go.dev/play/p/-PF7GfiUjRf)

```go
package main

import (
	"fmt"
	"sync"
)

var x int64
var wg sync.WaitGroup

func add() {
	for i := 0; i < 500000; i++ {
		x = x + 1
	}
	wg.Done()
}

func main() {
	wg.Add(2)
	go add()
	go add()
	wg.Wait()
	fmt.Println(x)
}
```

我们用两个 `goroutine` 去累加变量x的值，这两个goroutine 在访问和修改 x 变量时，会存在数据竞争（比如同时拿到了同一个值，就会造成同样的 +1 操作，进行了两遍），导致最后的结果和预期不符。

![](<../.gitbook/assets/未命名文件 (3).png>)

Go 的同步工具主要由 sync 包提供，互斥锁 (Mutex) 与读写锁 (RWMutex) 就是sync 包中的方法。

### 互斥锁 Mutex

互斥锁可以用来保护一个临界区，保证同一时刻只有一个线程 `goroutine` 处于该临界区内，其它的 `goroutine` 则在等待锁。当互斥锁释放后，等待的 `goroutine` 才可以获取锁进入临界区。多个 `goroutine` 同时等待一个锁时，唤醒的策略时随机的。

主要包括锁定 `lock()` 和 `unlock()` ，首先对进入临界区的 `goroutine`进行锁定，离开时进行解锁。

```go
package main

import (
	"fmt"
	"sync"
)

var (
	x    int64
	wg   sync.WaitGroup
	lock sync.Mutex // 互斥锁
)

func accumulate() {
	for i := 0; i < 500000; i++ {
		lock.Lock() // 加锁
		x = x + 1
		lock.Unlock() // 释放锁
	}
	wg.Done()
}

func main() {
	wg.Add(2)
	go accumulate()
	go accumulate()
	wg.Wait()
	fmt.Println(x)
}
```

[点这里，在线试试！](https://go.dev/play/p/eCfGhgKChva)

![](<../.gitbook/assets/未命名文件 (2) (1).png>)

使用互斥锁 (Mutex)时要注意以下几点：

* 不要重复锁定互斥锁，否则会阻塞，也可能会导致死锁（deadlock）；
* 要对互斥锁进行解锁，这也是为了避免重复锁定； 不要对未锁定或者已解锁的互斥锁解锁；
* 不要在多个函数之间直接传递互斥锁，sync.Mutex类型属于值类型，将它传给一个函数时，会产生一个副本，在函数中对锁的操作不会影响原锁。

总之，一个互斥锁只用来保护一个临界区，加锁后记得解锁，对于每一个锁定操作，都要有且只有一个对应的解锁操作，也就是加锁和解锁要成对出现，最保险的做法是使用defer语句解锁。

互斥锁通常用于读锁和写锁差不多的情况，而现实使用场景中，更多的场景是读多写少的。如果在这种情况下，读和写都加锁，会大幅影响性能。

### 读写互斥锁

而由此读写互斥锁应运而生了。

读写互斥锁能实现：在读锁占用的情况下，阻止写，但不阻止读。

也就是说，如果多个协程 `goroutine` 只涉及读，则可同时获取读锁 `RLock()` ，多个 goroutine 可同时进行；而写锁 `Lock()` 则和互斥锁一样，会阻止任何其他 goroutine（无论读和写）进来，整个锁相当于由一个 协程 `goroutine` 独占，离开时才解锁，让下一个协程 `goroutine` 开始。

以下案例是互斥锁和读写互斥锁 示例的对比：

* 读写互斥锁 用时约 662ms [点这里，在线试试！](https://go.dev/play/p/7utMTVVsbMr)

```go
package main

import (
	"fmt"
	"sync"
	"time"
)

// 读写互斥锁

var (
	x      int64
	wg     sync.WaitGroup
	rwlock sync.RWMutex
)

func read() {
	rwlock.RLock()
	time.Sleep(time.Millisecond) // 模拟读锁
	rwlock.RUnlock()
	wg.Done()
}

func write() {
	rwlock.Lock()
	time.Sleep(time.Millisecond * 10) // 模拟写锁
	rwlock.Unlock()
	wg.Done()
}

func main() {
	start := time.Now()
	for i := 0; i < 6666; i++ {
		wg.Add(1)
		go read()
	}

	for i := 0; i < 66; i++ {
		wg.Add(1)
		go write()
	}

	wg.Wait()
	end := time.Now()
	fmt.Println(end.Sub(start))
}
```

* 互斥锁 用时约 7.326s （实现的源码在这里👉） [点这里，在线试试！](https://go.dev/play/p/C0RHz2YjLcx)

### 并发安全的 map

如果按通常的方法去并发地修改一个 map ，例如： [点这里，在线试试！](https://go.dev/play/p/D\_qT9Hr4ryU)

```go
package main

import (
	"fmt"
	"sync"
)

var (
	wg sync.WaitGroup
)

var m = make(map[int]int)

func get(key int) int {
	return m[key]
}

func set(key int, value int) {
	m[key] = value
}

func main() {
	for i := 0; i < 2; i++ {
		wg.Add(1)
		go func(i int) {
			set(i, i+100)                            // 设置 map 键值对
			fmt.Printf("key:%v value:%v", i, get(i)) // 打印键值对
			wg.Done()
		}(i)
	}
	wg.Wait()
}
```

则会报如下错误：

```
fatal error: concurrent map writes
```

可见，Go 原生的 map 不能保证并发安全。

可以用 sync.Map 代替 Map实现并发安全，也可以用读写互斥锁来实现。值得注意的是，sync.Map 和 原生的 map 在用法上是不一致的。 `sync.Map` 内置了诸如 `Store` 、`Load` 、`LoadOrStore` 、`Delete` 、`Range` 等操作方法。

* sync.Map 的代码实现

[点这里，在线试试！](https://go.dev/play/p/LQffvYpFs2x)

```go
package main

import (
	"fmt"
	"sync"
)

// sync.Map 并发安全的map

var (
	wg sync.WaitGroup
	m2 sync.Map
)

func main() {
	for i := 0; i < 66; i++ {
		wg.Add(1)
		go func(i int) {
			m2.Store(i, i+100) // 设置 map 键值对
			value, _ := m2.Load(i)
			fmt.Printf("key:%v value:%v", i, value) // 打印键值对
			wg.Done()
		}(i)
	}
	wg.Wait()
}
```

* 用读写互斥锁和 Go 原生 map 实现并发安全的 map

[点这里，在线试试！](https://go.dev/play/p/-HLWe9t9une)

```go
package main

import (
	"fmt"
	"sync"
)

var (
	wg     sync.WaitGroup
	rwlock sync.RWMutex
)

var m = make(map[int]int)

func get(key int) int {
	rwlock.RLock()
	ret := m[key]
	rwlock.RUnlock()
	return ret
}

func set(key int, value int) {
	rwlock.Lock()
	m[key] = value
	rwlock.Unlock()
}

func main() {
	for i := 0; i < 666; i++ {
		wg.Add(1)
		go func(i int) {
			set(i, i+100)                              // 设置 map 键值对
			fmt.Printf("key:%v value:%v  ", i, get(i)) // 打印键值对
			wg.Done()
		}(i)
	}
	wg.Wait()
}
```

**sync.Once** 是 Golang package 中使方法只执行一次的对象实现，作用与 **init** 函数类似。但也有所不同。

* **init** 函数是在文件包首次被加载的时候执行，且只执行一次
* **sync.Onc** 是在代码运行中需要的时候执行，且只执行一次

当一个函数不希望程序在一开始的时候就被执行的时候，我们可以使用 **sync.Once** 。

[点这里，在线试试！](https://go.dev/play/p/aCfNnP-vQ7A)

```go
package main

import (
	"fmt"
	"sync"
)

func main() {
	var once sync.Once
	for i := 0; i < 10; i++ {
		once.Do(RunOnce)
		fmt.Println("Run RunOnce finished.")
	}
	for i := 0; i < 10; i++ {
		go func() {
			once.Do(goRunOnce)
			fmt.Println("Run goRunOnce finished.")
		}()
	}
}

func RunOnce() {
	fmt.Println("in RunOnce")
}

func goRunOnce() {
	fmt.Println("in goRunOnce")
}
```
