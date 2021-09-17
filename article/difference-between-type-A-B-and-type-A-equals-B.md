# Go语言type A B类型声明和type A = B类型声明的区别

我在看`src/builtin/builtin.go`源码时发现了两种类型定义，
一种是`type A B`，另一种是`type A = B`。
我很好奇这两种有什么区别，
按源码中的注释所说，`type A = B`中A是B的别名，
所以我猜想前一种A和B是不同类型，后一种A和B是相同类型，
并且写了个例子验证了我的猜想。

## 结论

- type A B中A和B是不同的类型，B只是作为A的底层类型，A和B的变量之间需要显式转化
- type A = B中A是B的别名，只是同一种类型的不同名称，A类型变量可以直接作为B类型变量使用

## 示例

```golang
package main

import (
	"fmt"
)

type byteA byte

type byteB = byte

func main() {
	var b0 byte = 1
	var u0 uint8 = 2
	var b1 byteA
	var b2 byteB
	// b1 = b0 is illegal, we must convert it explicitly
	b1 = byteA(b0)
	// byteB is an alias for byte, we can make assignment directly
	b2 = b0
	fmt.Println(b1, b2)
	// b1 = u0 is illegal
	b1 = byteA(u0)
	// byteB is an alias for byte, and byte is an alias for uint8, so byteB is the same as uint8
	b2 = u0
	fmt.Println(b1, b2)
}
```

可以看到，`byteA`类型和`byte(uint8)`类型之间需要显式转化，不然编译器会报错；而`byteB`类型和`byte(uint8)`类型之间无需转化。