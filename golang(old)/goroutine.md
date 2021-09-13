goroutine非常清亮，一个新创建的goroutine被赋予了几千字节，当它不运行时，go语言的运行时会自动增加（缩小）存储堆栈的内存，允许许多goroutine存在适当的内存中。每个函数调用CPU的开销平均为3个廉价指令。

计算gouroutine的创建前后分配的内存数量：

```go
memConsumed := func()uint64 {
        runtime.GC()
        var s runtime.MemStats
        runtime.ReadMemStats(&s)
        return s.Sys
    }
    var c <-chan interface{}
    var wg sync.WaitGroup
    noop := func() {
        wg.Done()
        <- c
    }
    const numGouroutines = 1e4
    wg.Add(numGouroutines)
    before := memConsumed()
    for i := numGouroutines; i > 0; i-- {
        go noop()
    }
    after := memConsumed()
	fmt.Printf("%.3f kb", float64(after - before) / numGouroutines / 1000)
```

运行结果：

windows下：

![image-20210704152815497](http://akatsuke.com/image-20210704152815497.png)

ubuntu下：

![image-20210704153518646](http://akatsuke.com/image-20210704153518646.png)