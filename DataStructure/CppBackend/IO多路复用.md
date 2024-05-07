#### 为什么IO多路复用要搭配非阻塞IO
![[为什么IO多路复用要搭配非阻塞IO.png]]

边沿触发
当使用epoll的边沿触发模式时，必须要将描述符设置为非阻塞。

多进程
当多个进程（或线程）在同一个描述符上执行I/O时，从单个进程的视角看，在收到描述符就绪的通知和接下来的I/O调用之间这段时间内，描述符的就绪状态有可能改变。所以，如果描述符是默认的阻塞方式，I/O操作有可能会阻塞，使得进程不能够监测其它描述符。
惊群效应？？

写大量数据
即使是边沿触发模式的API如`select`和`poll`通知字节流套接字（stream socket）可写，如果在一次`write`或`send`调用中写足够多的数据，调用依然有可能阻塞。

内核bug
在极少的情况下，边沿触发模式的API如`select`和`poll`可能返回错误的就绪状态通知。例如Linux手册关于`select`的内容中有如下说明：

> Under Linux, select() may report a socket file descriptor as "ready for reading", while nevertheless a subsequent read blocks. This could for example happen when data has arrived but upon examination has wrong checksum and is discarded. There may be other circumstances in which a file descriptor is spuriously reported as ready. Thus it may be safer to use O_NONBLOCK on sockets that should not block.

正如上面所说的，使用非阻塞I/O，程序会更加健壮。
为了保险起见(Best Practise)，多路复用最好使用非阻塞IO

