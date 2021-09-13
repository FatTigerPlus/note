FASTDFS的角色分类：

1. 跟踪器（tracker）：工作调度，起到负载均衡的效果
2. 存储节点（storage）：完成文件管理的所有功能：存储、同步和提供存取接口，FastDFS同时对文件的meta data进行管理。、

上传的过程：

1. 客服端询问tracker上传到storage，不需要附加参数。
2. tracker返回一台可用的storage。
3. 客服端和storage通讯完成文件上传。

下载的交互流程：

1. client询问tracker下载文件的storage，参数为文件标识（卷名和文件名）。
2.  tracker返回一台可用的storage。
3.  client直接和storage通讯完成文件下载。

