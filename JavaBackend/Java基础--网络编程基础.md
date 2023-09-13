### 基础概念
端口号：用于标识计算机上某个特定的**网络程序**
范围 0--65535  （2个字节标识端口 2^16）
0--1024端口已经被占用，比如ssh 22，ftp 21， smtp 25， http 80
常见的网络程序端口号：
tomcat：8080
mysql：3306

TCP协议：传输控制协议
1.使用TCP协议前，须先建立TCP连接，形成传输数据通道
2.传输前采用三次握手方式，是可靠的
3.连接中可进行大数据量的传输
4.传输完毕，需释放已经建立的连接，效率低

UDP协议：用户数据协议
1.将数据，source，des封装成数据包，不需要建立连接
2.每个数据包的大小限制在64k以内，不适合传输大量数据
3.无需连接，不可靠
4.发送数据结束后无需释放资源，速度快

InetAddress类
```java
//1.获取本机的InetAddress对象 
InetAddress localHost = InetAddress.getLocalHost();
//2.根据指定主机名，获取InetAddress对象 
InetAddress host1 = InetAddress.getByName("...");
//3.根据域名返回InetAddress对象
InetAddress host2 = InetAddress.getByName("www.baidu.com");
//4.通过InetAddress对象获取相应ip地址 
String hostAddress = host.getHostAddress();
//5.获取对应的主机名/域名
String hostName = host2.getHostName();
```

Socket
1.套接字Socket 开发网络应用程序被广泛采用，以至于成为事实的标准
2.通信的两端都要有Socket，是两台机器间通信的端点
3.网络通信其实就是Socket之间的通信
4.Socket允许程序把网络连接当成一个流，数据在两个Socket之间通过IO传输
![[socket读写数据.png]]

![[Server_client_socket.png]]