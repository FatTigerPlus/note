mmap()系统调用函数会直接把内核缓冲区里的数据**映射**到用户空间，这样，操作系统内核与用户空间就不需要再进行任何的数据拷贝操作。

它的主要代码是：

```c
#include <sys/mman.h>
void *mmap(void *addr, size_t length, int prot, int flags, int fd, off_t offset);
int munmap(void *addr, size_t length);//解除映射；取消参数start所指向的映射内存，参数length表示欲取消的内存大小
```

mmap系统调用的几个参数：

>    1）addr： 指定映射的起始地址, 通常设为NULL, 由系统指定。
>    2）length: 映射到内存的文件长度。
>    3） prot：  映射区的保护方式, 可以是:
>        PROT_EXEC: 映射区可被执行
>        PROT_READ: 映射区可被读取
>        PROT_WRITE: 映射区可被写入
>
>    4）flags: 映射区的特性, 可以是:
>       MAP_SHARED:写入映射区的数据会复制回文件, 且允许其他映射该文件的进程共享。
>       MAP_PRIVATE:对映射区的写入操作会产生一个映射区的复制(copy-on-write), 对此区域所做的修改不会写回原文件。
>
>    5）fd: 由open返回的文件描述符, 代表要映射的文件。
>    6）offset: 以文件开始处的偏移量, 必须是分页大小的整数倍, 通常为0, 表示从文件头开始映射。





sendfile()系统调用；该系统调用，可以直接把内核缓冲区里的数据拷贝到 socket 缓冲区里。

主要代码：

```c
#include <sys/sendfile.h>
ssize_t sendfile(int out_fd, int in_fd, off_t *offset, size_t count);
```

它的前两个参数分别是目的端和源端的文件描述符，后面两个参数是源端的偏移量和复制数据的长度，返回值是实际复制数据的长度。