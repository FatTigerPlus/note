### 定时器

go的定时器分两种：

1. 一次性的定时器（Timer）：定时器只计时一次，计时结束便停止运行
2. 周期性的定时器（Ticker）：定时器周期性的进行计时，除非主动停止，否则将永久运行。

#### Timer的用法

>  **Timer只运行一次**

Timer的在源码里面的定义的数据结构（src/time/sleep.go:Timer）

```go
type Timer struct {
    C <-chan Time
    r runtimeTimer
}
```

Timer对外只暴露一个channel，指定的时间来到是时候就往该channel写入系统时间，也就是一个事件。

Timer的使用场景：

1. 设置超时时间：

   goroutine从channel读数据的时候，如果channel中没有数据的话，那么goroutine将会阻塞，一直等待channel有新的数据写入，有时候不希望gouroutine被永久的阻塞，如果当一段时间以后channel还没有数据写入，就可以判定为超时，简单示例代码：

   ```go
   func WaitChannel(conn <-chan string) bool {
       timer := time.NewTimer(1 * time.Second)
       
       select {
           case <- conn:
           	timer.Stop
           	return true
           case <- timer.C: //此时超时
           	return false
       }
   }
   ```

2. 延迟执行方法：

   应用示例代码：

   ```go
   func DelayFunction() {
       timer := time.NewTimer(time.Second * 2)
       select {
           //计时结束就去做事
       case <- timer.C:
           DoSomething()
       }
   }
   ```

Timer对外的主要几个方法有;

```go
time.NewTimer(d) :创建一个Timer
time.Stop(): 停止当前Timer
time.Reset(d):重置当前Timer
```

#### Timer的原理

Timer的本质是把一个定时任务交给专门的协程进行监控，这个任务的载体就是runtimeTimer，也就是说每一次创建一个Timer就意味着创建一个runtimeTimer变量，然后将它交给系统进行监控。

runtimeTimer在源码包的src/time/sleep.go:runtimeTimer中定义了数据结构：

```go
type runtimeTimer struct {
    tb uintptr   // 存储当前定时器的数组地址
    i int		// 存储当前定时器的数组下标
    when int64 		//当前定时器触发时间
    period int64	// 当前定时器周期性触发间隔
    f func(interface{}, uintptr) //定时器触发时的回调函数
    arg interface{}  // 定时器触发时执行回调函数传递的参数一
    seq uintptr   //定时器触发时执行回调函数传递的参数二（这个参数只在网络收发场景下使用）：timer本身不实用该参数
}
```

创建Timer的源码：

```go
// NewTimer creates a new Timer that will send
// the current time on its channel after at least duration d.
func NewTimer(d Duration) *Timer {
	c := make(chan Time, 1)  //创建channel
	t := &Timer{   //   构建struct
		C: c,   //	赋值
		r: runtimeTimer{
			when: when(d), //  触发时间
			f:    sendTime,  // 触发后执行sendTime函数
			arg:  c,     //触发后执行sendTime函数附带的参数
		},
	}
	startTimer(&t.r)   // 启动定时器，只是把runtimeTimer放到系统协程的堆中，由系统协程维护
	return t
}
func sendTime(c interface{}, seq uintptr) {
	// Non-blocking send of time on c.
	// Used in NewTimer, it cannot block anyway (buffer).
	// Used in NewTicker, dropping sends on the floor is
	// the desired behavior when the reader gets behind,
	// because the sends are periodic.
	select {
	case c.(chan Time) <- Now():
	default:
	}
}
```

