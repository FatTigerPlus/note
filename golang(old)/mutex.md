Mutex 是使用最广泛的同步原语（Synchronization primitives），有人也叫做并发原语，同步原语的场景：

> 1. 共享资源。并发地读写共享资源，会出现数据竞争（data race）的问题，所以需要 Mutex、RWMutex 这样的并发原语来保护。
>
> 2. 任务编排。需要 goroutine 按照一定的规律执行，而 goroutine 之间有相互等待或者依赖的顺序关系，我们常常使用 WaitGroup 或者 Channel 来实现。
>
> 3. 消息传递。信息交流以及不同的 goroutine 之间的线程安全的数据交流，常常使用 Channel 来实现。

临界区：

<img src="http://akatsuke.com/image-20201015164743703.png" alt="image-20201015164743703" style="zoom:50%;" />



i ++ 在计算机中的运行的过程：

> 1. 将i的值从内存加载到寄存器
> 2. 在寄存器中进行自增操作
> 3. 将自增获取到的结果回写到内存中

<img src="http://akatsuke.com/image-20201015154034603.png" alt="image-20201015154034603" style="zoom: 67%;" />

因为i++分为上面的三步，所以它并不是原子性的操作，会出现多线程的问题。





