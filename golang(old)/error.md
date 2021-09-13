errors中源码：

```go
//使用一个结构体把错误信息包装起来
//
type errorString struct {
    s string
}
//errorString是builtin中的error接口的简单实现
func (e *errorString) Error() string {
	return e.s 
}
//new返回的是errorString对象指针信息的原因是，即使每次返回的错误信息是一样的，但是返回的错误是不一样的
//防止两个字符串碰撞，出现error相等的情况。
func New(text string) error {
    return &errorString{text}
}
```

在go里面，支持多参数的返回，当一个函数返回有value和error的时候，必须先对error进行判断在对value进行操作，除非当我们不关心value的值的时候，我们就可以不需要判断error的值。

而go中的panic（运行时恐慌），它不能假由调用者来解决恐慌，panic意味着代码不能够继续运行了，对于真正的不可恢复的时候，比如说是索引越界，不可恢复的环境问题，栈溢出等的时候，才使用panic的方式来处理，一般情况都使用error来进行处理。

> go effective中描述panic： This is only an example but real library functions should avoid `panic`. If the problem can be masked or worked around, it's always better to let things continue to run rather than taking down the whole program. **One possible counterexample is during initialization: if the library truly cannot set itself up, it might be reasonable to panic, so to speak**

处理野生goroutine抛出的panic：

```go
func Go(x func()) {
    go func() {
        defer func () {
            if err := recover(); err != nil {
                //do something
            }
        }()
        x()
    }()
}
```

先将有error的判断return出去，最后再处理正常的逻辑，减少难看的缩进代码。

error的types

1. sentinel error：预定义的特定的error类型，使用特定的值来表示错误：

   ```go
   if err == TypeError {}
   ```

   There are even sentinel errors that signify that an error *did not* occur, like `go/build.NoGoError, and path/filepath.SkipDir from path/filepath.Walk.`

2. error types：error type是实现了error接口的自定义类型，像下面这样

   ```go
   //自定义的结构体
   type TypeError struct {
       Msg string
       File string 
       Line int
   }
   //实现了error接口的Error方法
   func (t *TypeError) Error() string {
       return fmt.Sprintf("%s, %s,:%d", t.Msg, t.File, t.Line)
   }
   func test() error{
       return &TypeError{"somethin happened", "file1", 20}
   }
   ```

   上面代码中的TypeError是一个type，所以可以使用断言转换成这个类型，然后获取更多的上下文信息：

   ```go
   func main() {
       err := test()
       switch err := err.(type) {
       case nil:
           fmt.Println("success")
       case *TypeError:
           fmt.Println("error line:", err.Line)
       default:
           fmt.Println("it`s default")
       }
   }
   ```

   error types于错误值相比较而言，它可以包装底层错误，提供更多的上下文信息，像底层代码的os.PathError,它提供了底层什么路径，什么操作出现了错误：

   ```go
   // If the file is opened successfully, the error will be nil, but when there is a problem, it will hold an os.PathError(from go effective)
   // PathError records an error and the operation and file path that caused it.
   //PathError记录了造成这个错误的操作和文件的路径
   type PathError struct {
   	Op   string
   	Path string
   	Err  error
   }
   //实现了error接口
   func (e *PathError) Error() string { return e.Op + " " + e.Path + ": " + e.Err.Error() }
   //the PathError generates a string likes this:
   //open /etc/passwx: no such file or directory
   ```

   调用者要使用类型断言和类型 *switch*，就要让自定义的 *error* 变为 public。这种模型会导致和调用者产生强耦合，从而导致 API 变得脆弱。尽量避免使用 error types，虽然错误类型比 sentinel errors 更好，因为它们可以捕获关于出错的更多上下文，但是 error types 共享 error values 许多相同的问题。因此，我的建议是避免错误类型，或者至少避免将它们作为公共 API 的一部分。

3. Opaque errors：不透明错误处理，虽然知道发生了错误，但是不知道错误的内部的信息：

   ```go
   func get() error{
       x, err := foo()
       if err != nil {
           return err
       }
      //do something
   }
   ```

   

wrap errors: you should only handler error once.

errors.New和fmt.Errorf会返回运行的堆栈信息