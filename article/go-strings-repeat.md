# Go语言官方库strings.Repeat函数源码阅读

我在工作中用到了strings包的Repeat函数，所以顺便就阅读了一下源码。
我阅读时Repeat函数的源码如下：

```golang
// Repeat returns a new string consisting of count copies of the string s.
//
// It panics if count is negative or if
// the result of (len(s) * count) overflows.
func Repeat(s string, count int) string {
	if count == 0 {
		return ""
	}

	// Since we cannot return an error on overflow,
	// we should panic if the repeat will generate
	// an overflow.
	// See Issue golang.org/issue/16237
	if count < 0 {
		panic("strings: negative Repeat count")
	} else if len(s)*count/count != len(s) {
		panic("strings: Repeat count causes overflow")
	}

	n := len(s) * count
	var b Builder
	b.Grow(n)
	b.WriteString(s)
	for b.Len() < n {
		if b.Len() <= n/2 {
			b.WriteString(b.String())
		} else {
			b.WriteString(b.String()[:n-b.Len()])
			break
		}
	}
	return b.String()
}
```

## 分配足够的内存

首先，函数中对参数进行了检查，其中包括对结果是否溢出的检查。
然后声明了一个`Builder`，一次性分配了需要的内存，最后再进行写入。
分配空间时使用的`Builder.Grow`方法代码如下：

```golang
// Grow grows b's capacity, if necessary, to guarantee space for
// another n bytes. After Grow(n), at least n bytes can be written to b
// without another allocation. If n is negative, Grow panics.
func (b *Builder) Grow(n int) {
	b.copyCheck()
	if n < 0 {
		panic("strings.Builder.Grow: negative count")
	}
	if cap(b.buf)-len(b.buf) < n {
		b.grow(n)
	}
}
```

首先会对Builder是否为另一个Builder的拷贝进行检查。
因为Builder的底层实际上是一个字节切片，
所以如果一个非空的Builder被值拷贝的话，
那么两个Builder变量实际上指向同一个字节切片，
这样会出问题（当然对同一个Builder并发写也是不行的）。
拷贝检查是通过Builder内部的`addr`指针来完成的，
Builder的每个写入方法都会调用`copyCheck`方法进行检查，
而copyCheck方法会将addr初始化为Builder本身的地址，
也就是说，一个Builder一旦被写入，那么addr就不再是nil，
而是指向Builder本身。
如果后面它被复制的话，那么新Builder变量的addr指向的还是旧Builder，在copyCheck时就会触发panic。

拷贝检查之后，就是参数的检查和剩余容量的检查，
如果容量不够，则进行实际的扩展，`grow`方法代码如下：

```golang
// grow copies the buffer to a new, larger buffer so that there are at least n
// bytes of capacity beyond len(b.buf).
func (b *Builder) grow(n int) {
	buf := make([]byte, len(b.buf), 2*cap(b.buf)+n)
	copy(buf, b.buf)
	b.buf = buf
}
```

其中分配空间时并不是只分配了需要的空间`len+n-cap`，
而是会分配额外的空间，达到2*cap+n，
这种分配策略比[默认的切片扩容策略](go-slice-expand.md)要激进，
虽然会浪费更多空间，不过会减少内存分配次数。

## 重复写入字符串

写入这里，我以为会循环count次，每次将字符串s追加到后面，
但实际上不是，所以这次又学习到了，果然学无止境。
原生库这种实现方式将复杂度从O(N)降低到了O(logN)，
减少了内存复制的次数。