# Go语言time.Timer的Stop方法的错误用法

写[broadcast](https://github.com/developerdong/broadcast)库的时候，
为了避免频繁创建timer，所以只创建了一个timer，
然后每次在循环中都通过Reset方法重置计时器。

按照Reset方法的官方注释，有如下用法：
```go
if !t.Stop() {
  <-t.C
}
t.Reset(d)
```

也就是说如果timer已经超时，那么此时应该将channel排空。
但这里我忽视了一点，就是channel之前可能已经在超时判断中被排空了，
这时这里的操作就会永远阻塞下去。

这个问题的解决方案是把接收操作设置成非阻塞：
```go
if !t.Stop() {
  select {
  case <-t.C:
  default:
  }
}
t.Reset(d)
```

这时即使channel已被排空，程序也不会阻塞，
可以正常往下执行。
具体内容可以参考broadcast库中的[这个提交](https://github.com/developerdong/broadcast/commit/0c8fd8fd8a128b12f6f280d7d0f9cd1752c2c6d0)。
