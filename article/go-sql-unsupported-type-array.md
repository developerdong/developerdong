# Go语言sql库unsupported type array问题

今天在使用sql库往数据库插入数据的时候，遇到了
`sql: converting argument $13 type: unsupported type [16]uint8, a array`
的错误。我的本意是将字节数组插入数据库，
但经过查询想起来sql库只支持使用字节切片类型来插入字节数据。

这个字节数组的类型是`[16]byte`，是调用`md5.Sum()`方法返回的，
不是自己构建的，所以改不了返回值类型。
只能是通过`array[:]`的方式将数组转为切片，再插入，就不会报错了。