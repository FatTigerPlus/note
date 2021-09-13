src/runtime/chan.go

### Channel

通道本身是并发安全的，也是golang自带的唯一一个满足并发安全系的类型。

Go 语言提供了一种不同的并发模型，也就是通信顺序进程（Communicating sequential processes，CSP）[1](https://draveness.me/golang/docs/part3-runtime/ch06-concurrency/golang-channel/#fn:1)。Goroutine 和 Channel 分别对应 CSP 中的实体和传递信息的媒介，Go 语言中的 Goroutine 会通过 Channel 传递数据。

![image-20200822192432364](http://akatsuke.com/image-20200822192432364.png)

###### channel的本质先进先出（FIFO：first in, first out）的队列：

​	他的具体规则如下：

1. 先从 Channel 读取数据的 Goroutine 会先接收到数据
2. 先向 Channel 发送数据的 Goroutine 会得到先发送数据的权利

- 发送方会向缓冲区中写入数据，然后唤醒接收方，多个接收方会尝试从缓冲区中读取数据，如果没有读取到就会重新陷入休眠；
- 接收方会从缓冲区中读取数据，然后唤醒发送方，发送方会尝试向缓冲区写入数据，如果缓冲区已满就会重新陷入休眠

  channel中的各个元素值严格按照发送的顺序排列的，先被发送的数据一定会先被接收。元素的发送和接收都需要使用到操作符<-，一个尖括号加一个减号形象的代表元素值的传输方向，创建channel的示例代码：

```go
package main
func main() {
    chann := make(chan int, 3)//创建一个缓冲空间为3的，存储int类型值的channel
    chann <- 1 //向channel中发送数据
    chann <- 2
    chann <- 3
    ele := <- chann
    fmt.Println(ele) //输出1
}
```

写已经关闭的管道会触发panic，读已经关闭的管道不会触发panic（已经关闭的管道仍然可以读取）：从代码可以看出来，向已经关闭的channel写入数据会触发panic

![image-20210302212724731](C:\Users\X1 Carbon\AppData\Roaming\Typora\typora-user-images\image-20210302212724731.png)

```go
func main() {
    c := make(chan int, 3)
    close(c)
    c <- 3  //触发panic
    fmt.Println(<-c)//输出0，不会触发panic
}
```

声明管道的两种方式：

```go
1. 变量声明：
var c chan int//这种方式声明的管道值为nil
2.使用内建函数make()，可以创建无缓冲管道和带缓冲管道 
ch1 := make(chan int)
ch2 := make(chan int, 2)
```

管道的操作：

1. 操作符：管道的操作符“<-”表示数据的流向，管道在左表示向管道写入数据，管道在右表示从管道读取数据

   ```go
   ch := make(chan int, 3)
   ch <-1 //向管道写入数据1
   fmt.Println(<-ch) //从管道种读取数据
   ```

   

channel的创建过程：

```go
//1：这一阶段会对传入 make 关键字的缓冲区大小进行检查，如果我们不向 make 传递表示缓冲区大小的参数，那么就会设置一个默认值 0，也就是当前的 Channel 不存在缓冲区：（cmd\compile\internal\gc>）
func typecheck1(n *Node, top int) (res *Node) {
	switch n.Op {
	case OMAKE:
		...
		switch t.Etype {
		case TCHAN:
			l = nil
			if i < len(args) { // 带缓冲区的异步 Channel
				...
				n.Left = l
			} else { // 不带缓冲区的同步 Channel
				n.Left = nodintconst(0)
			}
			n.Op = OMAKECHAN
		}
	}
}
//2：OMAKECHAN 类型的节点最终都会在 SSA 中间代码生成阶段之前被转换成调用 runtime.makechan 或者 runtime.makechan64 的函数
func walkexpr(n *Node, init *Nodes) *Node {
	switch n.Op {
	case OMAKECHAN:
		size := n.Left
		fnname := "makechan64"
		argtype := types.Types[TINT64]

		if size.Type.IsKind(TIDEAL) || maxintval[size.Type.Etype].Cmp(maxintval[TUINT]) <= 0 {
			fnname = "makechan"
			argtype = types.Types[TINT]
		}
		n = mkcall1(chanfn(fnname, 1, n.Type), n.Type, init, typename(n.Type), conv(size, argtype))
	}
}
//3：runtime.makechan 和 runtime.makechan64 会根据传入的参数类型和缓冲区大小创建一个新的 Channel 结构，其中后者用于处理缓冲区大小大于 2 的 32 次方的情况，因为这在 Channel 中并不常见，所以我们重点关注 runtime.makechan：
func makechan(t *chantype, size int) *hchan {
	elem := t.elem
	mem, _ := math.MulUintptr(elem.size, uintptr(size))
	var c *hchan
	switch {
	case mem == 0:
		c = (*hchan)(mallocgc(hchanSize, nil, true))
		c.buf = c.raceaddr()
	case elem.kind&kindNoPointers != 0:
		c = (*hchan)(mallocgc(hchanSize+mem, nil, true))
		c.buf = add(unsafe.Pointer(c), hchanSize)
	default:
		c = new(hchan)
		c.buf = mallocgc(mem, elem, true)
	}
	c.elemsize = uint16(elem.size)
	c.elemtype = elem
	c.dataqsiz = uint(size)
	return c
}
```

