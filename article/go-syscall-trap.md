# Go系统调用卡死的坑

我在使用`syscall.Splice`系统调用的时候，
遇到了系统调用不返回的问题，记录下来提醒自己以后不要再犯。

```go
// go version: 1.15.5
package main

import (
	"os"
	"syscall"
)

func main() {
	wrongExample()
	rightExample()
}

func wrongExample() {
	// reader模拟TCP套接字，writer模拟管道写入端
	reader, writer := new(os.File), new(os.File)
	defer reader.Close()
	defer writer.Close()
	finished := make(chan struct{})
	go func() {
		// 在一个goroutine里不断地拷贝数据
		readerFd, writerFd := int(reader.Fd()), int(writer.Fd())
		for {
			written, err := syscall.Splice(readerFd, nil, writerFd, nil, 1<<20, 0)
			if err != nil || written == 0 {
				return
			}
		}
		finished <- struct{}{}
	}()
	// 关闭reader，试图停止拷贝数据，预期结果是syscall.Splice返回"bad file descriptor"的错误，但在高并发的情况下，syscall.Splice可能会一直卡住，不返回，程序没法结束
	reader.Close()
	<-finished
}

func rightExample() {
	// reader模拟TCP套接字，writer模拟管道写入端
	reader, writer := new(os.File), new(os.File)
	defer reader.Close()
	defer writer.Close()
	finished := make(chan struct{})
	go func() {
		// 在一个goroutine里不断地拷贝数据
		writerFd := int(writer.Fd())
		rawConn, err := reader.SyscallConn()
		if err != nil {
			finished <- struct{}{}
			return
		}
		// 在syscall.RawConn.Read方法中进行系统调用
		rawConn.Read(func(fd uintptr) (done bool) {
			written, err := syscall.Splice(int(fd), nil, writerFd, nil, 1<<20, 0)
			if err != nil || written == 0 {
				// written == 0 means end of input.
				return true
			}
			return false
		})
		finished <- struct{}{}
	}()
	// 关闭reader，试图停止拷贝数据，会导致syscall.Splice返回"use of closed file"错误，不过至少是结束了，查看net.rawConn.Read可以发现库里面做了很多其他的准备和清理工作，我们直接调用syscall.Splice缺少了这部分工作。
	reader.Close()
	<-finished
}
```

示例代码如上，如果直接调用系统调用会卡死，而用库接口就不会。
这让我想起了之前看到过的一句话：永远使用库函数，不要直接进行系统调用。
就连安卓开发者也遇到过直接进行系统调用不兼容的坑。
之前我对这句话理解不深，现在终于体会到了。
