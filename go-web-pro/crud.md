---
description: Go 原生操作 MySQL
---

# Crud

先输入以下命令，在 MySQL 中建立一张表。

```sql
CREATE TABLE `user` (
    `id` BIGINT(20) NOT NULL AUTO_INCREMENT,
    `name` VARCHAR(20) DEFAULT '',
    `age` INT(11) DEFAULT '0',
    PRIMARY KEY(`id`)
)ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8mb4;
```

然后可以查询到 该表；

```sql
show tables;
```

<img src="../.gitbook/assets/image.png" alt="" data-size="original">

插入两条数据：

```sql
insert into user (name, age) values("bingo", 22)
```

```sql
insert into user (name, age) values("violette", 23)
```

查询可得到这些数据：

```sql
select * from user;
```

![](<../.gitbook/assets/image (4).png>)



单行查询

```go
func (db *DB) QueryRow(query string, args ...interface{}) *Row go
```

```go
// queryRow 查询单条数据
func queryRow() {
	sqlStr := "select id, name, age from user where id =?"
	var u user
	// 非常重要：确保 QueryRow 之后调用 Scan 方法，否则持有的数据库连接不会被释放
	err := db.QueryRow(sqlStr, 1).Scan(&u.id, &u.name, &u.age) // 1 代表获取第一条
	if err != nil {
		fmt.Printf("scan failed, err:%v\n", err)
		return
	}
	fmt.Printf("id:%d name:%s age:%d\n", u.id, u.name, u.age)
}
```

查询结果：![](<../.gitbook/assets/image (1).png>)

多行查询

```go
// queryMultiRows 查询多条数据
func queryMultiRows() {
	sqlStr := "select id, name, age from user where id > ?"
	rows, err := db.Query(sqlStr, 0)
	if err != nil {
		fmt.Printf("query failed, err: %v\n", err)
		return
	}
	// 非常重要：关闭rows，释放持有的数据库连接
	defer rows.Close()

	// 循环读取结果集中的数据
	for rows.Next() {
		var u user
		err := rows.Scan(&u.id, &u.name, &u.age)
		if err != nil {
			fmt.Printf("scan failed, err:%v\n", err)
			return
		}
		fmt.Printf("id:%d name:%s age:%d\n", u.id, u.name, u.age)
	}
}
```

查询结果：![](<../.gitbook/assets/image (5).png>)
