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

插入一条数据：

```sql
insert into user (name, age) values("bingo", 22)
```

查询可得到该条数据：

```sql
select * from user;
```

