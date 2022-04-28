# go-redis的基本使用

```go
package main

import (
	"fmt"
	"github.com/go-redis/redis"
)

// 声明一个全局的rdb变量
var rdb *redis.Client

func initClient() (err error) {
	rdb = redis.NewClient(&redis.Options{
		Addr:     "localhost:6379",
		Password: "",
		DB:       0,   // use default DB
		PoolSize: 100, //连接池大小
	})

	_, err = rdb.Ping().Result()
	return err
}

func goRedis() {
	err := rdb.Set("score", 100, 0).Err()
	if err != nil {
		fmt.Printf("set score failed, err:%v\n", err)
		return
	}

	val, err := rdb.Get("score").Result()
	if err != nil {
		fmt.Printf("get score failed, err:%v\n", err)
		return
	}
	fmt.Println("score", val)

	val2, err := rdb.Get("name").Result()
	if err != redis.Nil {
		fmt.Println("name does not exist")
	} else if err != nil {
		fmt.Printf("get name failed, err:%v\n", err)
		return
	} else {
		fmt.Println("name", val2)
	}

}

func hget() {
	v, err := rdb.HGetAll("user").Result()
	if err != nil {
		// redis.Nil
		// 其他错误
		fmt.Printf("hgetall failed, err:%v\n", err)
		return
	}
	fmt.Println(v)

	v2 := rdb.HMGet("user", "name", "age").Val()
	fmt.Println(v2)

	v3 := rdb.HGet("user", "age").Val()
	fmt.Println(v3)
}

// 排行榜 差集 交集
func zset() {
	zsetkey := "language_rank"
	languages := []redis.Z{
		redis.Z{
			Score:  99.0,
			Member: "Go",
		},
		redis.Z{
			Score:  80.0,
			Member: "Java",
		},
		redis.Z{
			Score:  88.0,
			Member: "Python",
		},
		redis.Z{
			Score:  86.0,
			Member: "JavaScript",
		},
		redis.Z{
			Score:  80.0,
			Member: "C/C++",
		},
	}

	// ZADD
	num, err := rdb.ZAdd(zsetkey, languages...).Result()
	if err != nil {
		fmt.Printf("zadd failed, err:%v\n", err)
		return
	}
	fmt.Printf("zadd %d succ.\n", num)

	// 把 Go 的分数加10
	newScore, err := rdb.ZIncrBy(zsetkey, 10, "Go").Result()
	if err != nil {
		fmt.Printf("zincrby failed, err:%v\n", err)
		return
	}
	fmt.Printf("Go'score is %f now.\n", newScore)

	// 取分数最高的3个
	ret, err := rdb.ZRevRangeWithScores(zsetkey, 0, 2).Result()
	if err != nil {
		fmt.Printf("zrevrange failed, err:%v\n", err)
		return
	}
	for _, z := range ret {
		fmt.Println(z.Member, z.Score)
	}

	// 取90～110分
	op := redis.ZRangeBy{
		Min: "90",
		Max: "110",
	}
	ret, err = rdb.ZRangeByScoreWithScores(zsetkey, op).Result()
	if err != nil {
		fmt.Printf("ZRangeByScoreWithScores failed, err:%v\n", err)
		return
	}
	for _, z := range ret {
		fmt.Println(z.Member, z.Score)
	}

}

func main() {
	if err := initClient(); err != nil {
		fmt.Printf("init redis client failed, err:%v\n", err)
		return
	}
	fmt.Println("connect redis success")
	// 程序退出时释放相关资源
	defer rdb.Close()

	//goRedis()
	//hget()
	zset()
}
```
