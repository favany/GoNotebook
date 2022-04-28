# 匿名结构体

匿名结构体多用于一些临时场景

匿名结构体的声明：

```go
var anonymousStruct struct {
	name string
	age  int
}
```

匿名结构体内部的赋值

```go
anonymousStruct.name = "Echo"
anonymousStruct.age = 23
```

#### 完整代码

```go
package main

import "fmt"

// 匿名结构体：多用于一些临时场景

func main() {

	var anonymousStruct struct {
		name string
		age  int
	}

	anonymousStruct.name = "Echo"
	anonymousStruct.age = 23

	fmt.Printf("type:%T value:%v", anonymousStruct, anonymousStruct)
	// type:struct { name string; age int } value:{Meggy 23}

}
```
