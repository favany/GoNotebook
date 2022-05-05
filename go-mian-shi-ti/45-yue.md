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
