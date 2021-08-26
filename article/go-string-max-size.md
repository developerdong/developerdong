# Go语言中string的最大长度

我在上网的时候，偶然在StackOverflow上浏览到这样一个[问题](https://stackoverflow.com/questions/40675029/is-there-any-string-length-limit-in-golangs-string-map-key)，
问的是go语言中string的最大长度限制是多少。

按照回答中所说，因为Go语言内置的len函数返回的是int，
而len函数是可以对string类型进行操作的，
所以string长度的最大理论限制应该是int的最大值，
在32位机上是`math.MaxInt32`，在64位机上是`math.MaxInt64`。

虽然这个回答听起来很有道理，但是并没有在源码中得到证实，
直到我在看源码时看到了[这行代码](https://github.com/golang/go/blob/e1fcf8857e1b3e076cc3a6fad1860afe0d6c2ca6/src/strings/strings.go#L530)。
以及代码注释中引用的[这个问题](golang.org/issue/16237)。
此时已经可以比较确定地说，string的最大长度就是int的最大值（如果内存无限的话）。