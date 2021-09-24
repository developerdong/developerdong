# Go语言中方法的类型

Go语言中开发者可以给某个类型定义方法，
根据Go语言规范中[所述](https://golang.org/ref/spec#:~:text=The%20type%20of%20a%20method%20is%20the%20type%20of%20a%20function%20with%20the%20receiver%20as%20first%20argument.)，
方法的类型实际上就是将receiver作为第一个参数的函数的类型。

但是需要注意的是，`type.method`和`var.method`是不一样的，
执行下面这个例子：

```golang
package main

import (
        "fmt"
        "math"
)

type Point struct {
        x, y float64
}

func (p *Point) Length() float64 {
        return math.Sqrt(p.x*p.x + p.y*p.y)
}

func (p *Point) Scale(factor float64) {
        p.x *= factor
        p.y *= factor
}

func main() {
        point := new(Point)
        fmt.Printf("%T\n", point.Scale)
        fmt.Printf("%T", (*Point).Scale)
}
```

可以得到输出：

```
func(float64)
func(*main.Point, float64)
```

可以发现，`(*Point).Scale`的类型和`point.Scale`的类型是不同的。
`(*Point).Scale`的类型如Go规范所述，将`*Point`作为第一个参数；
而`point.Scale`的类型没有将`*Point`本身作为参数。
实际上在`point.Scale`方法执行时，方法内部的`p`固定为`point`，
在这个例子中，只需要传`factor`参数，而`p.x`和`p.y`都是0。

如果某个地方需要`func(float64)`类型的函数作为参数，
那么传`point.Scale`是合法的；
如果某个地方需要`func(*main.Point, float64)`类型的函数作为参数，
那么传`(*Point).Scale`也是合法的。