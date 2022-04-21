# 数组与切片有什么异同

slice 的底层数据是数组，slice 是对数组的封装，它描述一个数组的片段。两者都可以通过下标来访问单个元素。

数组是定长的，长度定义好之后，不能再更改。在 Go 中，数组是不常见的，因为其长度是类型的一部分，限制了它的表达能力，比如 \[3]int 和 \[4]int 就是不同的类型。

而切片则非常灵活，它可以动态地扩容。切片的类型和长度无关。

数组就是一片连续的内存， slice 实际上是一个结构体，包含三个字段：长度、容量、底层数组。

{% code title="runtime/slice.go" %}
```go
type slice struct {
	array unsafe.Pointer // 元素指针
	len   int // 长度 
	cap   int // 容量
}
```
{% endcode %}

slice 的数据结构如下：

![](<../.gitbook/assets/image (4).png>)



注意，底层数组是可以被多个切片同时指向的，因此对一个切片的元素进行操作是有可能影响到其他切片的。



<details>

<summary>【引申】[3]int 和 [4]int 是同一个类型吗？</summary>

不是。因为数组的长度是类型的一部分，这是数组与切片不同的一点。

</details>



以下代码输出什么？

```go
package main

import "fmt"

func main() {
	slice := []int{0, 1, 2, 3, 4, 5, 6, 7, 8, 9}
	s1 := slice[2:5]
	s2 := s1[2:6:7]

	s2 = append(s2, 100)
	s2 = append(s2, 200)

	s1[2] = 20

	fmt.Println(s1)
	fmt.Println(s2)
	fmt.Println(slice)
}
```



`s1` 从 `slice` 索引2（闭区间）到索引5（开区间，元素真正取到索引4 ，即 `2, 3, 4`），长度为3，容量默认到数组结尾，为8。&#x20;

`s2` 从 `s1` 的索引2（闭区间）到索引6（开区间，元素真正取到索引5，即 `4, 5, 6, 7, 8`），容量到索引7（开区间，真正到索引6），为5。

![](<../.gitbook/assets/image (1).png>)

接着，向 `s2` 尾部追加一个元素 100：

`s2` 容量刚好够，直接追加。不过，这会修改原始数组对应位置的元素。这一改动，数组和 `s1` 都可以看得到。

![](<../.gitbook/assets/image (2).png>)

再次向 `s2` 追加元素200：

[https://golang.design/go-questions/slice/vs-array/](https://golang.design/go-questions/slice/vs-array/)



