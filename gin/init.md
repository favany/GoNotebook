# Gin 项目初始化

```
go mod init getPath
```

初始化代码：

```go
package main

import (
	"github.com/gin-gonic/gin"
)

func main() {
	r := gin.Default()

	r.GET("/", func(c *gin.Context) {

	})

	r.Run(":9000")
}

```

补全需要的库：

```
go mod tidy
```
