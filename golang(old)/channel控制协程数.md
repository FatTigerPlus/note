code;

```go
func job (a int) {
    fmt.Println(a)
    time.Sleep(time.Second)
}

func main() {
    ch := make(chan struct{}, 10)  //长度为10 的channel
    var wg sync.WaitGroup  
    for i := 0; i < 100; i++ {
        ch <- struct{}{}
        wg.Add(1)
        go func() {
            job(i)
            defer wg.Done()
            defer func() {
              <- ch
            }()
        }()
    }
    wg.Wait()
}
```

