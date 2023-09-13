## Q0. 注意

据部分同学反馈，主机在做实验时，当客户端开启了5000个线程时会报错can't not start new thread，这可能是主机限制了python程序能开启的最大线程数量，目前还没查到能修改这个值的方法。

**为此我们为大家新提供了一个客户端代码`client_modi.py`，大家在完成实验单线程BIO，单线程NIO，多线程BIO，IO多路复用select，IO多路复用epoll时都以`client_modi.py`来编写服务端**

新代码将不断尝试与服务端建立连接，防止因为服务端的问题导致客户端连接失败使客户端程序崩溃

新代码主要解决了以下问题：
 - 部分主机由于内存限制问题，客户端创建超过5000个线程导致异常(can't not start new thread)
 - 客户端send产生Broken Pipe异常:  此时已经连接成功，但是服务端没有调用accept将connection socket从连接队列中取出来，因此客户端对该connection socket调用send会报Broken Pipe异常
 - 客户端recv产生Connection reset by peer异常: 此时connection socket连接由于未知情况被服务器关闭，因此客户端对该connection socket调用recv会产生Connection reset by peer异常

 
## Q1:too many open files异常

`too many open files`异常表示打开的文件过多，默认情况下，Linux支持打开的最大文件描述符个数是1024，macOS是256，这一点可以在终端里面通过 `ulimit -a`命令查询。
因此在我们的实验中，如果遇到上述情况，需要修改系统配置

如: `ulimit -n 10240`这种方式修改连接符个数为10240（最大值65536），但是这只在当前终端有效(关闭当前终端再起一个终端就失效了)
如果需要永久全局有效，大家可以网上自行查询方法

## Q2 实验要求--IO多路复用代码提交

原实验文档要求里第三题 IO多路复用要求: 
```
利用语言提供的 IO 多路复用接口，实现 IO 多路复用，作业保存为 IOMult_server_BIO.py。
```
**修改为: 
IO复用这题需提交两个文件，即IOMult_server_select.py和IOMult_server_epoll.py。select和epoll在给的模版代码基础上修改就行**

## Q3: 实验要求--多线程服务器

实验文档第一题要求里提到: 
```
`在上一次实验中我们开发的这个 Web 服务器一次只处理一个 HTTP 请求，本小节实验中，我们希望你能实现一个能够同时处理多个请求的高性能多线程服务器。
```
这个是基于上一节课的Web服务器用多线程进行优化，但不要求提交。我们需要大家提交的`high_performance_web_server.py`  是针对我们提供的客户端`client_modi.py`  进行编写的多线程服务端，并且后面在大家进行实验测试的时候，也是用针对`client_modi.py`编写的多线程服务端进行测试。 此外，单线程BIO，单线程NIO，多线程BIO，IO多路复用select，IO多路复用epoll，这5个程序都需要针对提供的`client_modi.py` 进行编写


## Q4: select内核设计问题

关于select方法因为内核设计原因使得最大文件描述符监听数不能超过1024的问题
我们提供了一个简单的解决思路，我们鼓励大家积极思考，用更好的方法解决：

服务端：select多路复用因为操作系统内核实现问题，最大的文件描述符的“监听”上限为1024，因此即使再怎么用ulimit命令设置当前进程的最大文件描述符上限也是无济于事，因此在编写select方法时需要在代码中进行额外处理，防止调用select函数的时候，监听的文件描述符超过1024上限。

- 文件描述符的增加主要是监听socket（listen_socket) 造成的，因此需保证在当文件描述符数组超过1000时不让监听socket(listen_fd) 增加新的文件描述符就可以防止这个错误产生，另外建议为整个处理流程增加异常处理（try-catch），增加程序的健壮性。下面是伪代码：
```python
readable, writeable, exceptional = select.select(inputs,outputs,exceptions)

# handle readable sockets
for s in readable:
	if s is listen_socket:
		if (len(inputs) + len(outputs) + len(exceptions) >= 1000):
			continue
```
至于为什么设1000，因为我们发现如果设置1024仍存在问题。有兴趣的同学可以看源码了解原因。

## Q5: 部分同学编写多线程BIO存在的问题
```python
while True:
	# 等待接受客户端的连接
	conn, addr = serverSocket.accept()
	# print(addr)
	# 创建线程
	try:
		t = threading.Thread(target=thread_func, args=(conn,))
		t.start()
		t.join()
```
部分同学在编写多线程BIO时采用上述写法，其中`join`和一个已经终止的线程进行连接，回收子线程的资源，但需要注意这个函数是阻塞函数，调用一次只能回收一个子线程。

我们希望程序同时接受多个连接，但若调用`join`会使得主线程陷入阻塞，直到回收其子线程资源，因此上述写法的程序其实是串行地创建线程执行业务逻辑。因此需要将`join`这一行去掉。
那么问题来了，主线程不join的话谁负责回收子线程的资源？ 答案: Python是存在GC的。

注意，在C语言中，可以通过`join`或`detach`回收子线程资源。其中detach函数让子线程和父线程分离，且该函数非阻塞，当子线程执行完业务逻辑之后由系统负责回收子线程资源。由于我们是编写Python程序完成实验，没有`detach`方法，因此我们将回收子线程的任务留给GC。

C语言中detach函数
```c
int pthread_detach(pthread_t thread);
//功能:分离1个线程，被分离的线程在终止的时候，会自动释放资源返回给系统
```
C语言中join函数
```c
int pthread_join(pthread_t thread, void **value_ptr);
//wait for thread termination
//和一个已经终止的线程进行连接，回收子线程的资源
//这个函数是阻塞函数，调用一次只能回收一个子线程
```


## Q6: 实验要求--IO多路复用阻塞方式

原实验文档要求里 第三题IO多路复用要求IO方式为阻塞 IO，更改为：不对IO阻塞方式作硬性要求，但是需要注意:

在实际开发中 IO多路复用一般要在如下情况搭配非阻塞IO: 

### select和poll
在极少的情况下，边沿触发模式的API如`select`和`poll`可能返回错误的就绪状态通知。例如Linux手册关于`select`的内容中有如下说明：

> Under Linux, select() may report a socket file descriptor as "ready for reading", while nevertheless a subsequent read blocks. This could for example happen when data has arrived but upon examination has wrong checksum and is discarded. There may be other circumstances in which a file descriptor is spuriously reported as ready. Thus it may be safer to use O_NONBLOCK on sockets that should not block.


### epoll边沿触发模式
Epoll具有两种模式，**边缘触发模式**（**E**dge **T**rigger，ET），默认的模式 **水平触发模式**（**L**evel **T**rigger，LT）。这两种模式的区别在于：
-   对于水平触发模式，一个事件只要有，就会一直触发；
-   对于边缘触发模式，只有一个事件从无到有才会触发。

以 socket 的读事件为例，对于水平模式，只要 socket 上有未读完的数据，就会一直产生 POLLIN 事件；而对于边缘模式，socket 上第一次有数据会触发一次，后续 socket 上存在数据也不会再触发，除非把数据读完后，再次产生数据才会继续触发。对于 socket 写事件，如果 socket 的 TCP 窗口一直不饱和，会一直触发 POLLOUT 事件；而对于边缘模式，只会触发一次，除非 TCP 窗口由不饱和变成饱和再一次变成不饱和，才会再次触发 POLLOUT 事件。

**socket 可读事件水平模式触发条件：**
```
1. socket上无数据 => socket上有数据
2. socket上有数据 => socket上有数据
```

**socket 可读事件边缘模式触发条件：**

```
1. socket上无数据 => socket上有数据
```

**socket 可写事件水平模式触发条件：**

```
1. socket可写   => socket可写
2. socket不可写 => socket可写
```

**socket 可写事件边缘模式触发条件：**

```
1. socket不可写 => socket可写
```

因此epoll在边缘触发模式需要用非阻塞IO，水平触发情况可以用阻塞IO，但建议都用非阻塞IO；以及大家需要根据选取的IO阻塞方式进行IO处理编写，例如epoll边缘触发模式非阻塞IO需要保证一次性读完所有数据，不然可能有数据会被遗漏。

### 多进程/多线程
当多个进程（或线程）在同一个描述符上执行I/O时，从单个进程的视角看，在收到描述符就绪的通知和接下来的I/O调用之间这段时间内，描述符的就绪状态有可能改变。所以，如果描述符是默认的阻塞方式，I/O操作有可能会阻塞，使得进程不能够监测其它描述符。

### 写大量数据
即使是边沿触发模式的API如`select`和`poll`通知字节流套接字（stream socket）可写，如果在一次`write`或`send`调用中写足够多的数据，调用依然有可能阻塞。

正如上面所说的，最好IO多路复用搭配非阻塞I/O，因为在服务器上发生阻塞是非常严重的事故，为了保险起见(Best Practise)，多路复用最好使用非阻塞IO




