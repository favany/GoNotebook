# MySQL 的连接和初始化

{% code title="main.go" %}
```go
package main

import (
	"database/sql"
	"fmt"
	_ "github.com/go-sql-driver/mysql" // init()
	"time"
)

// sql.DB 是表示连接的数据库对象（结构体实例），它保存了连接数据库相关的所有信息。
// 它内部维护着一个多个底层连接的连接池，它可以安全地被多个 goroutine 同时使用。
var db *sql.DB

// initMySQL 初始化 MySQL 数据库，返回 nil 则初始化成功，否则返回 error
func initMySQL() (err error) {
	// DSN: Data Source Name
	//dsn := "user:passsword@tcp(127.0.0.1:3306)/dbname"
	dsn := "root:@tcp(127.0.0.1:3306)/sql2"
	// 初始化全局的 db 对象，而不是新声明一个 db 变量
	db, err = sql.Open("mysql", dsn)
	if err != nil {
		panic(err)
	}

	// 数值需要业务具体情况来确定
	db.SetConnMaxLifetime(time.Second * 10) // 它设置了连接可重用的最大时间长度
	db.SetMaxIdleConns(200)                 // 最大连接数
	db.SetMaxOpenConns(10)                  // 最大空闲连接数

	// 尝试与数据库建立连接（校验dsn是否正确）
	err = db.Ping()
	return
}

func main() {
	// 连接 MySQL 数据库
	if err := initMySQL(); err != nil {
		fmt.Println("connect to db failed,", err)
		//return
	} else {
		fmt.Println("connect to db success!")
	}
	// Close() 用来释放数据库连接相关的资源
	defer db.Close() // 写在err 判断的下面
}
```
{% endcode %}
