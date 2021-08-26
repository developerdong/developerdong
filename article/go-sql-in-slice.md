# Go语言中执行SQL WHERE语句时如何支持IN (slice)条件

我在使用sql包查询数据库时，遇到了需要在WHERE语句中使用IN条件的情况，
例如`WHERE name IN ('Alice', 'Bob')`。
我脑海中首先冒出来的疑问是sql包的占位符`?`是否直接支持切片。
经过查询，我找到了[这个Issue](https://github.com/go-sql-driver/mysql/issues/176)，
根据问题中的回答，切片是不被支持的（应该是为了防止SQL注入）。

既然切片没有被支持，那么就需要寻找其他方法来解决这个需求。
可选的方法有使用别人写好的库，比如sqlx，
不过我只想用原生库。最终我还是找到了一个比较好的方法，
通过拼接的方式实现我的需求，代码示例如下。
这种方式可以处理切片长度为0的情况，如果切片长度为0，
那么有关这个切片的IN条件就不会被添加到SQL语句中，
对语句的其它部分没有影响。

```golang
package main

import (
	"fmt"
	"strings"
)

func main() {
	clauses := []string{"SELECT id FROM table WHERE 1"}
	params := make([]interface{}, 0)
	if names := []string{"Alice", "Bob"}; len(names) > 0 {
		clauses = append(clauses,
			fmt.Sprintf(
				"AND name IN (%s)",
				strings.Join(strings.Split(strings.Repeat("?", len(names)), ""), ", "),
			),
		)
		for _, name := range names {
			params = append(params, name)
		}
	}
	// 现在就有最终的SQL语句和相应的参数了，可以放到sql.DB.Query方法中执行
	fmt.Println(strings.Join(clauses, " ") + ";")
	fmt.Println(params)
}
```

