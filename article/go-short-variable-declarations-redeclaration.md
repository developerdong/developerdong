# Go短变量重声明时的地址变化

在我阅读[Go参考规范](https://golang.org/ref/spec#Short_variable_declarations)
时，其中有一段话提到了重声明并没有引入新变量，
只是给原变量分配了新值：

> Unlike regular variable declarations, a short variable declaration may redeclare variables provided they were
> originally declared earlier in the same block (or the parameter lists if the block is the function body) with the
> same type, and at least one of the non-blank variables is new. As a consequence, redeclaration can only appear
> in a multi-variable short declaration. Redeclaration does not introduce a new variable; it just assigns a new value
> to the original.

我对这段话中的`same block`产生了一个疑问：
这里所指的代码块是否包括子代码块？为了解答这个疑惑，
我写了个例子：

```golang
package main

import (
	"fmt"
)

func func1() (int, int) {
	return 1, 2
}

func main() {
	field1, offset := func1()
	fmt.Println(&field1, &offset)
	field2, offset := func1()
	fmt.Println(&field2, &offset)
	if true {
		field3, offset := func1()
		fmt.Println(&field3, &offset)
	}
}
```

在<https://play.golang.org>上执行这段代码，
得到以下结果（两次执行结果不一定相同）：

```
0xc0000b8000 0xc0000b8008
0xc0000b8020 0xc0000b8008
0xc0000b8028 0xc0000b8030
```

可以发现每次执行，
第二次输出的offset地址和第一次输出的offset地址都是一样的，
与规范中所说相符；
而第三次输出的offset地址和前两次输出的offset地址不同，
说明子代码块中引入了新的变量，而不是变量的重声明。