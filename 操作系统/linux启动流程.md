linux的模式:

1. 实模式

   > 只能选址1M，每个段最大寻址64kb

2. 保护模式

> 在32位操作系统下面，最大能够寻址4g

计算机的内存分类;

1. ROM(read only memory)；只读存储器
2. RAM(read access memory)：随机存储器

ROM是只读的，上面固化了一些初始化的程序，也就是BIOS（basic input and output system）：基本输入输出系统。

BIOS的大概界面；

![img](http://akatsuke.com/13187b1ffe878bc406da53967e8cddb7.png)

在开机开始的时候，也就是实模式的时候，这个时候只有1MB的内存空间，

1mb内存空间的地址就是0x00000到0xFFFFF，对于这个地址空间的分配（x86的系统）：

<img src="http://akatsuke.com/5f364ef5c9d1a3b1d9bb7153bd166bfc.jpeg" alt="img" style="zoom: 33%;" />

从图中可以看出： 0xF0000到0xFFFFF这个地址空间内部分配给了ROM，也就是说访问到这一部分的地址空间的时候，就会访问到ROM的地址空间。