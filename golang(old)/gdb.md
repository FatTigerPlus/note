### GDB调试：

先编写test.go

```go
package main

import (
    `fmt`
    `reflect`
)

type person struct {
    name string
    age int
}
func Inc() (v int) {
    defer func() {v ++}()
    return 100
}
func main() {
    p := person{"12312",12313}
    t:= reflect.TypeOf(p.name)
    println(len(t.String()))
    fmt.Println(t)
    fmt.Println(reflect.ValueOf(p.age))
}
```

编译go文件;

**sudo go build  -gcflags "-N -l" test.go**

然后使用gdb启动调试

sudo gdb test

执行run命令：

![image-20200825114008079](http://akatsuke.com/image-20200825114008079.png)

使用b（breakpoint）命令添加断点

![image-20200825114247508](http://akatsuke.com/image-20200825114247508.png)

使用run命令执行代码：

![image-20200825114338733](http://akatsuke.com/image-20200825114338733.png)

继续执行下一个断点：

![image-20200825134353258](http://akatsuke.com/image-20200825134353258.png)

查看所有断点：

![image-20200825134435844](http://akatsuke.com/image-20200825134435844.png)

在gdb环境下，运行info goroutines命令，报错如下：

![image-20200825135631085](http://akatsuke.com/image-20200825135631085.png)

出现这种情况的实现，先去到go的安装路径的bin目录，运行以下命令：

![image-20200825135926632](http://akatsuke.com/image-20200825135926632.png)

踩坑（服务器本地的gdb版本太低）：

![image-20200825142847793](http://akatsuke.com/image-20200825142847793.png)





入口：

![image-20201216224309236](http://akatsuke.com/image-20201216224309236.png)

![image-20201216224336152](http://akatsuke.com/image-20201216224336152.png)

![image-20201216224659129](http://akatsuke.com/image-20201216224659129.png)

![image-20201216224720901](http://akatsuke.com/image-20201216224720901.png)

