# Go语言切片对下层数组的操作

今天在看[Go语言之旅](https://tour.golang.org/moretypes/8)，
文中用例子说明了切片实际上是对底层数组的操作。

执行
```golang
package main

import "fmt"

func main() {
	names := [4]string{
		"John",
		"Paul",
		"George",
		"Ringo",
	}
	fmt.Println(names)

	a := names[0:2]
	b := names[1:3]
	fmt.Println(a, b)

	b[0] = "XXX"
	fmt.Println(a, b)
	fmt.Println(names)
}
```
会得到结果
```
[John Paul George Ringo]
[John Paul] [Paul George]
[John XXX] [XXX George]
[John XXX George Ringo]
```

## 追加一个元素

这时我突发奇想，如果往切片中追加元素，会发生什么呢？
所以写了一个例子，执行
```golang
package main

import "fmt"

func main() {
	names := [4]string{
		"John",
		"Paul",
		"George",
		"Ringo",
	}
	fmt.Println(names)

	a := names[0:2]
	b := names[1:3]
	fmt.Println(a, b)

	b[0] = "XXX"
	fmt.Println(a, b)
	fmt.Println(names)

	a = append(a, "YYY")
	fmt.Println(a, b)
	fmt.Println(names)
}
```
得到了结果
```
[John Paul George Ringo]
[John Paul] [Paul George]
[John XXX] [XXX George]
[John XXX George Ringo]
[John XXX YYY] [XXX YYY]
[John XXX YYY Ringo]
```

意料之中，新元素被写到了names[2]的位置。

## 一次追加三个元素

继续探究，如果追加的元素太多，超出了原切片的容量，又会发生什么呢？
执行
```golang
package main

import "fmt"

func main() {
	names := [4]string{
		"John",
		"Paul",
		"George",
		"Ringo",
	}
	fmt.Println(names)

	a := names[0:2]
	b := names[1:3]
	fmt.Println(a, b)

	b[0] = "XXX"
	fmt.Println(a, b)
	fmt.Println(names)

	a = append(a, "YYY", "ZZZ", "AAA")
	fmt.Println(a, b)
	fmt.Println(names)
}
```
得到结果
```
[John Paul George Ringo]
[John Paul] [Paul George]
[John XXX] [XXX George]
[John XXX George Ringo]
[John XXX YYY ZZZ AAA] [XXX George]
[John XXX George Ringo]
```

可以发现，a切片的容量本来为下层数组的长度，也就是4，
a切片中已经有了2个元素，此时直接追加3个元素，并没有将names数组的2、3位置
进行覆盖，跟预想的结果有些不一样。

## 三次追加一个元素

如果把上面的例子修改一下，从一次直接追加三个元素，改为每次追加一个元素，分三次追加：
```golang
package main

import "fmt"

func main() {
	names := [4]string{
		"John",
		"Paul",
		"George",
		"Ringo",
	}
	fmt.Println(names)

	a := names[0:2]
	b := names[1:3]
	fmt.Println(a, b)

	b[0] = "XXX"
	fmt.Println(a, b)
	fmt.Println(names)

	a = append(a, "YYY")
	a = append(a, "ZZZ")
	a = append(a, "AAA")
	fmt.Println(a, b)
	fmt.Println(names)
}
```
得到结果
```
[John Paul George Ringo]
[John Paul] [Paul George]
[John XXX] [XXX George]
[John XXX George Ringo]
[John XXX YYY ZZZ AAA] [XXX YYY]
[John XXX YYY ZZZ]
```

与上一个例子的结果比较，可以发现Go在给切片追加元素的时候，
是先检查容量是否足够容纳要追加的所有元素，如果不够会分配新数组，然后再进行追加；
而不是将要追加的所有元素一个一个地写到底层数组里，等填满了底层数组再进行扩充。

## 解压缩切片

如果将一个切片追加到a末尾：
```golang
package main

import "fmt"

func main() {
	names := [4]string{
		"John",
		"Paul",
		"George",
		"Ringo",
	}
	fmt.Println(names)

	a := names[0:2]
	b := names[1:3]
	fmt.Println(a, b)

	b[0] = "XXX"
	fmt.Println(a, b)
	fmt.Println(names)

	a = append(a, []string{"YYY", "ZZZ", "AAA"}...)
	fmt.Println(a, b)
	fmt.Println(names)
}
```
可以得到结果
```
[John Paul George Ringo]
[John Paul] [Paul George]
[John XXX] [XXX George]
[John XXX George Ringo]
[John XXX YYY ZZZ AAA] [XXX George]
[John XXX George Ringo]
```
和直接追加3个元素是一样的。
