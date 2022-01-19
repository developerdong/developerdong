# Go语言struct结构体的大小计算

给定下面一段代码，输出是什么？

```go
package main

import "unsafe"

type T1 struct {
	a int32
	b uint8
}

type T2 struct {
	a int32
	b struct{}
}

func main() {
	t1, t2 := T1{}, T2{}
	println(unsafe.Sizeof(t1), unsafe.Offsetof(t1.b), unsafe.Alignof(t1))
	println(unsafe.Sizeof(t2), unsafe.Offsetof(t2.b), unsafe.Alignof(t2))
}
```

使用gc/amd64架构，第一行的输出是`8 4 4`，
有经验的Go程序员肯定会解释说`a`占用4字节，`b`占用1字节，
按4对齐之后大小就是8。
但其实语言规范并没有要求结构体最后要有padding，
这完全是实现上的决定。例如，[StdSizes.Sizeof](https://go.dev/src/go/types/sizes.go)
函数计算出来的结果就是5，而不是8。
关于这一点，在[Issue 14909](https://github.com/golang/go/issues/14909)中有讨论。

第二行可能会有人说输出是`4 4 4`，因为struct{}并不占空间。
但实际结果还是`8 4 4`，这是因为如果struct中最后一个字段的大小为0，
则它还是需要占用1个字节。
这个问题在[Issue 9401](https://github.com/golang/go/issues/9401)中有说明。
