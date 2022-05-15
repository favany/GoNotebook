# 4-5月

2022-4-29

下面这段代码输出结果正确吗？

```go
type Foo struct {
	bar string
}
func main() {
	s1 := []Foo{
		{"A"},
		{"B"},
		{"C"},
	}
	s2 := make([]*Foo, len(s1))
	for i, value := range s1 {
		s2[i] = &value
	}
	fmt.Println(s1[0], s1[1], s1[2])
	fmt.Println(s2[0], s2[1], s2[2])
}
```

```bash
输出：
{A} {B} {C}
&{A} &{B} &{C}
```

<details>

<summary>答案</summary>

参考答案及解析：s2 的输出结果错误。

s2 的输出是 `&{C} &{C} &{C}`，在前面题目我们提到过，for range 使用短变量声明(:=)的形式迭代变量时，变量 i、value 在每次循环体中都会被重用，而不是重新声明。所以 s2 每次填充的都是临时变量 value 的地址，而在最后一次循环中，value 被赋值为{c}。因此，s2 输出的时候显示出了三个 &{c}。

可行的解决办法如下：

```go
for i := range s1 {
    s2[i] = &s1[i]
}
```

</details>



2022-4-30

下面代码里的 counter 的输出值？

```go
func main() {
	var m = map[string]int{
		"A": 21,
		"B": 22,
		"C": 23,
	}
	counter := 0
	for k, v := range m {
		if counter == 0 {
			delete(m, "A")
		}
		counter++
		fmt.Println(k, v)
	}
	fmt.Println("counter is ", counter)
}
```

* A. 2
* B. 3
* C. 2 或 3

<details>

<summary>答案</summary>

参考答案：C。

解析 ： for range map 是无序的，如果第一次循环到 A，则输出 3；否则输出 2。

代码实现：[https://replit.com/@Vooce/2022-4-30#main.go](https://replit.com/@Vooce/2022-4-30#main.go)

</details>



2022-5-1

关于协程，下面说法正确是（）

* A. 协程和线程都可以实现程序的并发执行；
* B. 线程比协程更轻量级；
* C. 协程不存在死锁问题；
* D. 通过 channel 来进行协程间的通信；

<details>

<summary>答案</summary>

参考答案及解析：AD。

</details>

\
2022-5-2

关于循环语句，下面说法正确的有（）

* A. 循环语句既支持 for 关键字，也支持 while 和 do-while；
* B. 关键字 for 的基本使用方法与 C/C++ 中没有任何差异；
* C. for 循环支持 continue 和 break 来控制循环，但是它提供了一个更高级的 break，可以选择中断哪一个循环；
* D. for 循环不支持以逗号为间隔的多个赋值语句，必须使用平行赋值的方式来初始化多个变量；

<details>

<summary>答案</summary>

参考答案及解析：CD。

</details>



5-3

下面这段代码输出什么，说明原因。

```go
func main() {
	slice := []int{0,1,2,3}
	m := make(map[int]*int)

	for key,val := range slice {
		m[key] = &val
	}

	for k,v := range m {
		fmt.Println(k,"->",*v)
	}
}
```



<details>

<summary>答案</summary>

答案：

```go
0 -> 3
1 -> 3
2 -> 3
3 -> 3
```



解析：这是新手常会犯的错误写法，for range 循环的时候会**创建每个元素的副本，而不是元素的引用**，所以 m\[key] = \&val 取的都是变量 val 的地址，所以最后 map 中的所有元素的值都是变量 val 的地址，因为最后 val 被赋值为3，所有输出都是3.

正确的写法：

```go
func main() {

	slice := []int{0,1,2,3}
	m := make(map[int]*int)

	for key,val := range slice {
		value := val
		m[key] = &value
	}

	for k,v := range m {
		fmt.Println(k,"===>",*v)
	}
}
```



扩展题目

```go
type Test struct {
	name string
}

func (this *Test) Point(){
	fmt.Println(this.name)
}

func main() {

	ts := []Test{
		{"a"},
		{"b"},
		{"c"},
	}

	for _,t := range ts {
		//fmt.Println(reflect.TypeOf(t))
		defer t.Point()
	}
	
}
```

参考：https://blog.csdn.net/idwtwt/article/details/87378419



</details>



5-5

map 的 key 为什么是无序的？

在遍历 map 的时候，我们会发现，输出的 key 是无序的。为什么？

查看答案

#### 答案解析：

map 在扩容后，会发生 key 的搬迁，原来落在同一个 bucket 中的 key，搬迁后，有些 key 就要远走高飞了（bucket 序号加上了 2^B）。而遍历的过程，就是按顺序遍历 bucket，同时按顺序遍历 bucket 中的 key。搬迁后，key 的位置发生了重大的变化，有些 key 飞上高枝，有些 key 则原地不动。这样，遍历 map 的结果就不可能按原来的顺序了。

当然，如果我就一个 hard code 的 map，我也不会向 map 进行插入删除的操作，按理说每次遍历这样的 map 都会返回一个固定顺序的 key/value 序列吧。的确是这样，但是 Go 杜绝了这种做法，因为这样会给新手程序员带来误解，以为这是一定会发生的事情，在某些情况下，可能会酿成大错。

当然，Go 做得更绝，当我们在遍历 map 时，并不是固定地从 0 号 bucket 开始遍历，每次都是从一个随机值序号的 bucket 开始遍历，并且是从这个 bucket 的一个随机序号的 cell 开始遍历。这样，即使你是一个写死的 map，仅仅只是遍历它，也不太可能会返回一个固定序列的 key/value 对了。

多说一句，“迭代 map 的结果是无序的”这个特性是从 go 1.0 开始加入的。

答案解析来自：[https://golang.design/go-questions/map/unordered/](https://golang.design/go-questions/map/unordered/)



2022-5-6

下面两段代码输出什么。

```go
// 1.
func main() {
    s := make([]int, 5)
    s = append(s, 1, 2, 3)
    fmt.Println(s)
}

// 2.
func main() {
    s := make([]int, 0)
    s = append(s,1,2,3,4)
    fmt.Println(s)
}
```

<details>

<summary>答案</summary>

```
代码 1 输出：[0 0 0 0 0 1 2 3]
代码 2 输出：[1 2 3 4]
```

参考解析：这道题考的是使用 append 向 slice 添加元素，第一段代码常见的错误是 \[1 2 3]，需要注意。

</details>

5-9

以下代码是否能编译通过？

```go
package main

import "fmt"

func main() {
    m := make(map[string]int)
    
    fmt.Println(&m["qcrao"])
}
```

<details>

<summary> 答案</summary>

#### 答案解析：

这个问题，相当于问：可以对 map 的元素直接取地址吗？

以上代码编译报错：

./main.go:8:14: cannot take the address of m\["qcrao"]

即无法对 map 的 key 或 value 进行取址。

如果通过其他 hack 的方式，例如 unsafe.Pointer 等获取到了 key 或 value 的地址，也不能长期持有，因为一旦发生扩容，key 和 value 的位置就会改变，之前保存的地址也就失效了。

题目和解析来自：[https://golang.design/go-questions/map/element-address/](https://golang.design/go-questions/map/element-address/)

</details>



5-10

如何确认两个 map 是否相等？

<details>

<summary>答案</summary>

反射

```go
package main

import( 
    "fmt"
    "relflect"
)
func main() {
    var m map[string]int
    var n map[string]int

    fmt.Println(reflect.DeepEqual(m,n))
}
```

</details>



5-12

init() 函数是什么时候执行的？

<details>

<summary>答案</summary>

`init()` 函数是 Go 程序初始化的一部分。Go 程序初始化先于 main 函数，由 runtime 初始化每个导入的包，初始化顺序不是按照从上到下的导入顺序，而是按照解析的依赖关系，没有依赖的包最先初始化。

每个包首先初始化包作用域的常量和变量（常量优先于变量），然后执行包的 `init()` 函数。同一个包，甚至是同一个源文件可以有多个 `init()` 函数。`init()` 函数没有入参和返回值，不能被其他函数调用，同一个包内多个 `init()` 函数的执行顺序不作保证。

一句话总结： import –> const –> var –> `init()` –> `main()`

```go
package main

import "fmt"

func init()  {
	fmt.Println("init1:", a)
}

func init()  {
	fmt.Println("init2:", a)
}

var a = 10
const b = 100

func main() {
	fmt.Println("main:", a)
}
// 执行结果
// init1: 10
// init2: 10
// main: 10
```

答案解析来源：[init() 函数是什么时候执行的？](https://geektutu.com/post/qa-golang-2.html#Q1-init-%E5%87%BD%E6%95%B0%E6%98%AF%E4%BB%80%E4%B9%88%E6%97%B6%E5%80%99%E6%89%A7%E8%A1%8C%E7%9A%84%EF%BC%9F)



</details>



5-13

new() 与 make() 的区别？

<details>

<summary>答案</summary>

new(T) 和 make(T,args) 是 Go 语言内建函数，用来分配内存，但适用的类型不同。

new(T) 会为 T 类型的新值分配已置零的内存空间，并返回地址（指针），即类型为 \*T 的值。换句话说就是，返回一个指针，该指针指向新分配的、类型为 T 的零值。适用于值类型，如数组、结构体等。

make(T,args) 返回初始化之后的 T 类型的值，这个值并不是 T 类型的零值，也不是指针 \*T，是经过初始化之后的 T 的引用。make() 只适用于 slice、map 和 channel。

</details>



5-15

下面这段代码能否通过编译，如果可以，输出什么？

```go
func main() {
	s1 := []int{1, 2, 3}
	s2 := []int{4, 5}
	s1 = append(s1, s2)
	fmt.Println(s1)
}
```

<details>

<summary>答案</summary>

#### 答案解析：

不能通过编译。append() 的第二个参数不能直接使用 slice，需使用 … 操作符，将一个切片追加到另一个切片上：append(s1,s2…)。或者直接跟上元素，形如：append(s1,1,2,3)。

</details>
