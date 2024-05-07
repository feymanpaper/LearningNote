## 课程内容
![[LInux高并发服务器开发引言.png]]
![[Linux高并发服务器开发目录.png]]

## Linux系统编程入门
### GCC
-GCC 原名为 GNU C语言编译器 (GNU C compiler)
-GCC (GNU compiler collection, GNU编译器套件）是由 GNU 开发的编程语言译器。GNU 编译器套件包括objective-C、 Java、Ada 和 Go 语言前端，也包括了这些语言的库（如 1ibstactt, 1ibgcj等）

-GCC 不仅支持C 的许多“方言”，也可以区别不同的C 语言标准;可以使用命令行选项来控制编译器在翻译源代码时应该遵循哪个 。 标准。例如，当使用命令行参数 ﻿std-cgg、启动 GCC 时，编译器支持 C99标准。

-安装命令 sudo apt install gcc g++（版本＞4.8.5）
-查看版本 gcclgt+ -v/--version

#### GCC工作流程 
![[GCC工作流程.png]]
预处理：头文件展开，宏替换，删掉注释
编译：转成汇编代码
汇编，链接
```
// mac os下
gcc test.c -E -o test.i  //生成预处理代码
gcc test.i -S -o test.s.  
gcc test.s -c -o test.o
gcc test.o -o test.out
./test.out
```

![[GCC常用参数选项.png]]

![[gcc常用选项2.png]]

#### gcc和g++的区别
gcc 和g++都是GNU(组织）的一个编译器。
```
误区一：gcc 只能编译c代码，g++只能编译c++ 代码。两者都可以，请注意：
后缀为 .c的，gcc把它当作是c程序，而g++当作是c++程序

后缀为 .cpp 的，两者都会认为是 C++程序，C++的语法规则更加严谨一些

编译阶段，g++ 会调用gcc，对于 c++ 代码，两者是等价的，但是因为 gcc命令不能自动和 C+＋ 程序使用的库联接，所以通常用 g++来完成链接，为了统一起见，干脆编译/链接统统用 g++了，这就给人一种错觉，好像cpp程序只能用 g++ 似的
```

```
误区二：gcc 不会定义_cplusplus宏。而g++会
实际上，这个宏只是标志着编译器将会把代码按C 还是C++语法来解释

如上所述，如果后缀为.c，并且采用gcc编译器，则该宏就是未定义的，否则，就是已定义
```

```
误区三：编译只能用 gcc，链接只能用 g++

严格来说，这句话不算错误，但是它混淆了概念，应该这样说：编译可以用gcc/g++，而链接可以用g++ 或者 gcc -Lstdc++。

gcc命令不能自动和C++程序使用的库联接，所以通常使用g++ 来完成联接。但在编译阶段，g++ 会自动调用gcc，二者等价
```

### 什么是库
库文件是计算机上的一类文件，可以简单的把库文件看成一种代码仓库，它提供给使用者一些可以直接拿来用的变量、函数或类。

库是特殊的一种程序，编写库的程序和编写一般的程序区别不大，只是库不能单独运行。

库文件有两种，静态库和动态库（共享库），区别是：静态库在程序的链接阶段被复制到了程序中；动态库在链接阶段没有被复制到程序中，而是程序在运行时由系统动态加载到内存中供程序调用。

库的好处：1. 代码保密 2.方便部署和分发

#### 静态库的制作与使用 
命名规则：xxx库的名字自己写
Linux：libxxx.a 
windows: libxxx.lib

制作：
gcc获得.o文件，将.o文件打包，使用ar工具(archive)
```
ar rcs libxxx.a xxx.o xxx.o
//r -将文件插入备存文件
//c -建立备存文件
//s -建立索引
```
注意，需要提供给库文件及头文件给对方使用 
![[静态库示例目录.png]]
假设源文件都放在src目录下
先执行
```
gcc -c add.c sub.c mult.c div.c -I ../include/
```
再打包库文件
```
ar rcs libsuanshu.a add.o div.o mult.o sub.o
```
移动到lib目录 
```
mv libsuanshu.a ../lib/
```
编译main.c并运行 ，注意是使用库的名字而不是整个库文件的名字
```
gcc main.c -o app -I ./include/ -L ./lib/ -l suanshu
./app
```

#### 动态库的制作与使用 
命名规则：xxx库的名字自己写
Linux：libxxx.so，在Linux下是一个可执行文件
windows: libxxx.dll

制作：
gcc得到.o文件，得到和位置无关的代码
```
gcc -c -fpic/-fPIC a.c b.c
```
gcc得到动态库
```
gcc -shared a.o b.o -o libcalc.so
```

以上面为例子:
```
gcc -c -fpic add.c div.c mul.c sub.c
gcc -shared add.o sub.o mul.o div.o -o libcalc.so
gcc main.c -o main -I include/ -L lib/ -l cal
```
此时会出现错误：
error while loading shared libraries: cannot open shared object file: No such file or directory


静态库：GCC 进行链接时，会把静态库中代码打包到可执行程序中
动态库：GCC 进行链接时，动态库的代码不会被打包到可执行程序中

程序启动之后，动态库会被动态加载到内存中，通过 ldd (list dynamic dependencies) 命令检查动态库依赖关系
```
ldd main //main为可执行文件 
```
如何定位共享库文件呢？

当系统加载可执行代码时候，能够知道其所依赖的库的名宇，但是还需要知道绝对路径。此时就需要系统的动态载入器来获取该绝对路径。对于elf格式的可执行程序，是由ld-linux .so来完成的，它先后搜索elf文件的
DT_RPATH段一＞环境变量LD_LIBRARY_PATH-＞/etc/ld.so.cache文件列表—＞ /lib/, /usr/lib
目录找到库文件后将其载入内存。

解决方法1:修改环境变量 
第一种方式：
这种方式是临时的，关闭再开一个终端就又失效了
```
export LD_LIBRARY_PATH = $LD_LIBRARY_PATH:/home/nowcoder/Linux/lesson06/library/lib //so文件的绝对路径
```
第二种方式：
用户级别，永久配置
```
//linux下
vim .bashrc
//加上上面的路径
source .bashrc
```
第三种方式：
系统级别的配置，永久
```
sudo vim /etc/profile
//加上上面的路径
sudo source /etc/profile
```

解决方法2:配置 /etc/ld.so.cache
....

解决方法3:将so放到/lib或者/usr/lib（不建议，会和系统的冲突 ）

#### 静态库和动态库对比 
静态库，动态库区别来自链接阶段如何处理，链接成可执行程序。分别称为静态链接方式和动态链接方式
静态库制作过程
![[静态库制作过程.png]]
动态库制作过程
![[动态库制作过程.png]]
静态库
优点：静态库被打包到应用程序中加载速度快，发布程序无需提供静态库，移植方便
缺点：消耗系统资源，浪费内存；更新、部署、发布麻烦
![[静态库加载内存.png]]
动态库
优点：可以实现进程间资源共享（共享库）;更新、部署、发布简单;可以控制何时加载动态库
缺点：加载速度比静态库慢，发布程序时需要提供依赖的动态库zzzxxxx
![[动态库加载内存.png]]

### Makefile
一个工程中的源文件不计其数，其按类型、功能、模块分别放在若干个目录中，Makefile 文件定义了一系列的规则来指定哪些文件需要先编译，哪些文件需要后编译，哪些文件需要重新编译，甚至于进行更复杂的功能操作，因为 Makefile 文件就像一个 She11 脚本一样，也可以执行操作系统的命令。

Makefile 带来的好处就是“自动化编译”，一旦写好，只需要一个make 命令，整个工程完全自动编译，极大的提高了软件开发的效率。make 是一个命令工具，是一个解释 Makefi1e 文件中指令的命令工具，一般来说，大多数的 IDE 都有这个命令，比如 Delphi的make, Visual Ct+ 的nmake, Linux下GNU 的make。  
![[Makefile命名和规则.png]]
```make
app:sub.c add.c mult.c div.c main.c
	gcc sub.c add.c mult.c div.c main.c -o app
```
![[makefile工作原理.png]]
Makefile中的其他规则一般是为第一条规则服务的
如果其他的规则跟第一个规则没关系，就不会执行
如果只改了main.c则只会执行两条指令而不用全部执行，提高效率
```make
app:sub.o add.o mult.o div.o main.o
	gcc sub.o add.o mult.o div.o main.o -o app
sub.o:sub.c
	gcc -c sub.c -o sub.o
add.o:add.c
	gcc -c add.c -o add.o
mult.o:mult.c
	gcc -c mult.c -o mult.o
div.o:div.c
	gcc -c div.c -o div.o
main.o:main.c
	gcc -c main.c -o main.o
```
![[makefile变量.png]]

```
src = sub.o add.o mult.o div.o main.o
target = app
$(target):$(src)
	$(CC) $(src) -o $(target)
sub.o:sub.c
	gcc -c sub.c -o sub.o
add.o:add.c
	gcc -c add.c -o add.o
mult.o:mult.c
	gcc -c mult.c -o mult.o
div.o:div.c
	gcc -c div.c -o div.o
main.o:main.c
	gcc -c main.c -o main.o

```

![[makefile模式匹配.png]]
```
src = sub.o add.o mult.o div.o main.o
target = app
$(target):$(src)
	$(CC) $(src) -o $(target)
%.o:%.c
	$(CC) -c $< -o $@
```

![[makefile_wildcard.png]]
![[makefile_patsubst.png]]
如果touch了一个clean文件，再执行，此时由于clean文件的更新时间比makefile中的clean更新时间晚，所以不会执行；为了解决，需要把clean定义成伪目标
.PHONY表示后面的是一个伪目标，所以不会和更新机制对比
```
src = $(wildcard ./*.c)
objs = $(patsubst) %.c, %.o, $(src)
target = app
$(target):$(objs)
	$(CC) $(objs) -o $(target)
%.o:%.c
	$(CC) -c $< -o $@
.PHONY:clean
clean:
	rm $(objs) -f
```

### GDB
![[什么是GDB.png]]
通常，在为调试而编译时，我们会（） 关掉编译器的优化选项（-o）, 并打开调试选项（-g）。另外，、-Wall 在尽量不影响程序行为的情况下选项打开所有warning， 也可以发现许多问题，避免一些不必要的 BUG。

```
gcc -g -Wall program.c -o program
```

-g、选项的作用是在可执行文件中加入源代码的信息，比如可执行文件中第几条机器指令对应源代码的第几行，但并不是把整个源文件嵌入到可执行文件中，所以在调试时必须保证 gdb 能找到源文件
![[gdb命令.png]]
![[gdb断点.png]]
![[gdb调试命令.png]]

### 文件IO
标准c库IO函数是跨平台的，在window下调用window的api，在linux下调用linux下的api
![[标准c库IO函数.png]]
文件描述符，定位文件
文件读写指针，读数据写数据
IO缓冲区，提高执行效率，默认8192byte，8k
![[标准c库IO和LInux系统IO关系.png]]
虚拟地址空间
程序不占用内存空间，只占用磁盘空间，是文件
进程：运行起来的程序，占用内存
![[虚拟地址空间.png]]
File指针里面封装了一个文件描述符file_no
因此我们调用c标准库的fopen可以打开一个文件

一个进程可以打开多个文件，所以是用数组(文件描述符表)来存储文件描述符，默认大小是1024，最多同时打开1024个文件
前3个文件描述符默认打开，标准输入，标准输出，标准错误，指向当前终端(万物皆文件) 

一个文件可以被打开多次，占用的文件描述符不一样
使用close方法关闭文件描述符
![[文件描述符.png]]
```
//打开linux开发手册第二章的open函数 
man 2 open
```

```
#include <fcntl.h>

int open(const char *pathname, int flags);
参数:
	- pathname:要打开的文件路径
	- 对文件的操作权限设置还有其他设置
	  O_RDONLY...
返回值:返回一个新的文件描述符，如果调用失败返回-1

errno:属于Linux系统函数库里面的一个全局变量，记录的是最近的错误号

#include<stdio.h>
void perror(const char *s); //作用打印error对应的错误描述

#include<unistd.h>
int close(int fd; //关闭文件描述符
```
![[linux系统IO函数.png]]
open打开文件
```c
#include<fcntl.h>
#include<stdio.h>
#include<unistd.h>
int main(){
    int fd = open("a.txt", O_RDONLY);
    if(fd == -1){
        perror("open");
    }
    close(fd);
    return 0;
}
```
open创建一个文件
```c
#include<fcntl.h>
#include<stdio.h>
#include<unistd.h>
int main(){
    int fd = open("a.txt", O_RDWR|O_CREAT, 0777);
    if(fd == -1){
        perror("open");
    }
    close(fd);
    return 0;
}
```
read&write
```c
#include<unistd.h>
ssize_t read(int fd, void *buf, size_t count);
//参数:
	- fd: 文件描述符，open得到的
	- buf:需要读取数据存放的地方，数组的地址（传出参数）
	- count:指定的数组大小
//返回值:
	- 成功:>0:返回实际的读取到的字节数; =0文件已经读取完了
	- 失败:-1,失败并且设置error

ssize_t write(int fd, const void *buf, size_t count);
//参数
	- fd: 文件描述符，open得到的
	- buf:要往磁盘写入的数据
	- count:要写的数据的实际大小
//返回值:
	- 成功:>0:返回实际写入的字节数; =0文件已经读取完了
	- 失败:-1,失败并且设置error
```

```c
char buf[1024] = {0};
int len = 0;
while((len = read(srcfd, buf, sizeof(buf))) > 0){
	write(destfd, buf, len);
}
close(destfd);
close(srcfd);
```
lseek
```c
off_t lseek(int fd, off_t offset, int whence);
//参数:
	- fd
	- offset文件偏移量
	- whence:SEEK_SET设置文件指针的偏移量;SEEK_CUR设置偏移量:当前位置+第二个参数offset的值;SEEK_END:设置偏移量:文件大小+第二个参数offset值 
//返回值:返回文件指针的位置

 //作用
	 1.移动文件指针到文件头
	 lseek(fd, 0, SEEK_SET);
	 2.获取当前文件指针的位置
	 lseek(fd, 0, SEEK_CUR);
	 3.获取文件长度
	 lseek(fd, 0, SEEK_END);
	 4.扩展文件的长度,当前文件10b，增加100字节
	 //注意需要写一次数据
	 lseek(fd, 100, SEEK_END);
```

```c
int fd = open("hello.txt", O_RDWR);
if(fd == -1){
	perror("open");
	return -1;
}
//扩展文件长度
int ret = lseek(fd, 100, SEEK_END);
if(ret == -1){
	perror("lseek");
	return -1;
}
//写入空数据
write(fd, " ", 1);
close(fd);
return 0;
```
stat获取一个文件相关的信息
```c
int stat(const char *pathname, struct stat *statbuf);
//参数
	- pathname:操作的文件路径
	- statbuf:结构体变量，传出参数，用于保存获取到的文件信息
```
![[stat结构体.png]]
文件类型用4个位表示
![[st_mode变量.png]]
```c
struct stat statbuf;
int ret = stat("a.txt", &statbuf);
if(ret == -1){
	perror("stat");
	return -1;
}
printf(statbuf.st_size);
return 0;
```

#### 模拟实现ls -l指令
```c
#include<stdio.h>
#include<sys/types.h>
#include<sys/stat.h>
#include<unistd.h>
#include<pwd.h>
#include<grp.h>
#include<time.h>
#include<string.h>
//模拟实现ls -l命令
// -rw-rw-r-- 1 nowcoder nowcoder 12 12月 3 15:48 a.txt
int main(int argc, char * argv[]){
    if(argc < 2){
        printf("%s filename\n", argv[0]);
        return -1;
    }
    //通过stat函数获取用户传入文件的信息
    struct stat st;
    int ret = stat(argv[1], &st);
    if(ret == -1){
        perror("stat");
        return -1;
    }

    //获取文件类型和文件权限
    char perms[11] = {0}; 
    switch(st.st_mode & S_IFMT){
        case S_IFLNK:
            perms[0] = 'l';
            break;
        case S_IFDIR:
            perms[0]= 'd';
            break;
        case S_IFREG:
            perms[0] = '-';
            break;
        case S_IFBLK:
            perms[0] = 'b';
            break;
        case S_IFCHR:
            perms[0] = 'c';
            break;
        case S_IFSOCK:
            perms[0] = 's';
            break;
        case S_IFIFO:
            perms[0] = 'p';
            break;
        default:
            perms[0] = '?';
            break;
    }
    //判断文件访问权限
    //文件所有者 
    perms[1] = (st.st_mode & S_IRUSR) ? 'r':'-';
    perms[2] = (st.st_mode & S_IWUSR) ? 'w':'-';
    perms[3] = (st.st_mode & S_IXUSR) ? 'x':'-';
    //文件所在组
    perms[4] = (st.st_mode & S_IRGRP) ? 'r':'-';
    perms[5] = (st.st_mode & S_IWGRP) ? 'w':'-';
    perms[6] = (st.st_mode & S_IXGRP) ? 'x':'-';
    //其他人
    perms[7] = (st.st_mode & S_IROTH) ? 'r':'-';
    perms[8] = (st.st_mode & S_IROTH) ? 'w':'-';
    perms[9] = (st.st_mode & S_IROTH) ? 'x':'-';
    //硬连接数
    int linkNum = st.st_nlink;

    //文件所有者
    char *fileUser = getpwuid(st.st_uid)->pw_name;
    //文件所在组 
    char *fileGrp = getgrgid(st.st_gid)->gr_name;
    //文件大小
    long int fileSize = st.st_size;
    //获取修改时间
    char *time = ctime(&st.st_ctimespec);
    char mtime[512] = {0};
    strncpy(mtime, time, strlen(time) - 1);
    char buf[1024];
    sprintf(buf, "%s %d %s %s %d %s %s", perms, linkNum, fileUser, fileGrp, fileSize, mtime, argv[1]);
    printf("%s\n", buf);
    return 0;
}
```


![[文件属性操作函数.png]]

![[目录操作函数.png]]
Dir* 目录流
用来遍历目录，比如统计目录下的文件
![[目录关闭函数.png]]
![[dirent结构体信息.png]]
dup，多个文件描述符可以指向同一个文件，dup会从空闲的文件描述符选一个最小没被用的作为新的拷贝文件描述符，指向同一个文件
dup2，重定向，比如oldfd指向a.txt，newfd指向b.txt；调用函数之后newfd也指向a.txt
oldfd必须是一个有效的文件描述符
![[dup函数.png]]
fcntl函数
cmd表示对文件描述符如何操作
F_DUPFD:复制文件描述符
F_GETFL:获取指定文件描述符文件状态flag，获取的flag和我们通过open函数传递的flag是一个东西
F_SETFL:设置文件描述符文件状态flag；必须项(O_RDONLY, O_WRONLY, ORDWR)不可以被修改; 可选项:(O_APPEND, O_NONBLOCK)
阻塞与非阻塞，描述的是函数调用的行为.比如终端是阻塞的，等待输入
![[fcntl函数.png]]

## Linux多进程开发
### 进程概述
#### 程序和进程
程序：是包含了一系列信息的文件，这些信息包含了如何在运行时创建一个进程

二进制格式标识：每个程序文件都包含用于描达可执行文件格式的元信息。内核利用此信息来解释文件中的其他信息。（ELF可执行连接格式）

机器语言指令：对程序算法进行编码。

程序入口地址：标识程序开始执行时的起始指令位置。

数据：程序文件包含的变量初始值和程序使用的宇面量值（比如字符串）。

符号表及重定位表：描述程序中西数和变量的位置及名称。这些表格有多重用途，其中包括调试和运行时的符号解析（动态链接）。  

共享库和动态链接信息：程序文件所包含的一些字段，列出了程序运行时需要使用的共享库，以及加载共享库的动态连接器的路径名。 ﻿

其他信息：程序文件还包含许多其他信息，用以描述如何创建进程。

进程：是正在运行的程序的实例。是一个具有一定独立功能的程序关于某个数据集合的一次运行活动。它是操作系统动态执行的基本单元，在传统的操作系统中，进程既是基本的分配单元，也是基本的执行单元。

可以用一个程序来创建多个进程，进程是由内核定义的抽象实体，并为该实体分配用以执行程序的各项系统资源。从内核的角度看，进程由用户内存空间和一系列内核数据结构组成，其中用户内存空间包含了程序代码及代码所使用的变量，而内核数据结构则用于维护进程状态信息。记录在内核数据结构中的信息包括许多与进程相关的标识号(ID:）、虛拟内存表、打开文件的描述符表、信号传递及处理的有关信息、进程资源使用及限制、当前工作目录和大量的其他信息。

#### 单道/多道程序设计
单道程序，即在计算机内存中只允许一个的程序运行。

多道程序设计技术是在计算机内存中同时存放几道相互独立的程序，使它们在管理程序控制下，相互穿插运行，两个或两个以上程序在计算机系统中同处于开始到结束之问的状态，这些程序共享计算机系统资源。引入多道程序设计技术的根本目的是为了提高 CPU 的利用率。

对于一个单CPU 系统来说，程序同时处于运行状态只是一种宏观上的概念，他们虽然都已经开始运行，但就微观而言，任意时刻，CPU 上运行的程序只有一个。

在多道程序设计模型中，多个进程轮流使用 CPU。而当下常见 CPU 为纳秒级，1秒可以执行大约 10 亿条指令。由于人眼的反应速度是毫秒级，所以看似同时在运行。

#### 时间片
时间片 （timeslice）又称为“量子 （quantum）“或“处理器片 (proces sor slice)

是操作系统分配给每个正在运行的进程微观上的一段CPU 时间。事实上，虽然一合计算机通常可能有多个 CPU，但是同一个 CPU 永远不可能真正地同时运行多个任务。在

只考虑一个 CPU 的情况下，这些进程'看起来像"同时运行的，实则是轮番穿插地运行，由于时间片通常很短（在Iinux 上为5ms800ms） ，用户不会感觉到。

时间片由操作系统内核的调度程序分配给每个进程。首先，内核会给每个进程分配相等的初始时间片，然后每个进程轮番地执行相应的时间，当所有进程都处于时问片耗尽的

状态时，内核会重新为每个进程计算并分配时间片，如此往复。

#### 并行和并发
并发是两个队列交替使用一个咖啡机
并行是两个队列同时使用一个咖啡机
![[并行和并发.png]]

#### PCB进程控制块
为了管理进程，内核必须对每个进程所做的事情进行清楚的描述。内核为每个进程分配一个 PCB(Processing control Block)进程控制块，维护进程相关的信息，Linux 内核的进程控制块是task_struct结构体。

在 /usr/src/linux-headers-xxx/include /linux/sched.h 文件中可以查看 struct task_struct 结构体定义。其内部成员有很多，我们只需要掌握以下

部分即可：
-   ﻿进程id：系统中每个进程有唯一的 id，用pid_t类型表示，其实就是一个非负整数
-   ﻿进程的状态：有就绪、运行、挂起、停止等状态
-   ﻿进程切换时需要保存和恢复的一些CPU寄存器
-   ﻿描述虛拟地址空间的信息
-   ﻿描述控制终端的信息
- 当前工作目录 (Current working Directory)
- umask 掩码
- 文件描述符表，包含很多指向 fi1e 结构体的指针和信号相关的信息
- 用户 id和组id
- 会话 (Session） 和进程组
- 进程可以使用的资源上限 (Resource Limit)


### 进程状态
![[进程三态.png]]
![[进程五态.png]]
### 进程命令
![[查看进程.png]]
![[进程相关命令.png]]
![[实时显示进程动态.png]]

./a.out &让进程在后台运行
![[杀死进程.png]]

### 进程函数
![[进程号和相关函数.png]]
进程创建
![[进程创建.png]]
当前终端运行了一个程序，终端是一个进程，然后创建了一个子进程来运行程序
```c
pid_t fork(void);
//返回值会返回两次，一次在父进程中，一次在子进程中
//在父进程中返回创建的子进程ID
//在子进程中返回0
//在父进程中返回-1表示创建子进程失败，并且设置errno
pid_t pid = fork();
```
上述代码，子进程不会执行fork了，注意，子进程只会执行fork之后的代码
```c
#include <unistd.h>
#include<stdio.h>
 int main(){
    printf("The child process will not start here\n");
    pid_t pid = fork();
    if(pid == 0){
        printf("I am child process %d; My parent's process %d\n", getpid(), getppid());
    }else if(pid > 0){
        printf("I am parent process %d; My parent process %d\n", getpid(), getppid());
    }
    for(int i = 0;i < 3;i++){
        printf("%d\n", i);
        sleep(3);
    }
    return 0;
 }
```
fork以后，子进程用户区数据和父进程一样，内核区也会拷贝，但是pid不一样
上述代码父进程栈空间里面pid_t pid = fork()的返回值为子进程的pid，子进程栈空间里面为0
两个进程之间的数据互不影响
The child process and the parent process run tn separate memory spaces.
At the time of fork() both memory spaces have the same content. Memory writes, file mappings (mmap (2)), and unmappings (munmap (2)) performed by one of the processes do not affect the other.

实际上，更准确来说，Linux 的 fork0 使用是通过写时搭贝 (copy-on-write) 实现。写时拷贝是一种可以推迟甚至避免拷贝数据的技术。内核此时并不复制整个进程的地址空间，而是让父子进程共享同一个地址空间。只用在需要写入的时候才会复制地址空间，从而使各个进行拥有各自的地址空间。也就是说，资源的复制是在需要写入的时候才会进行，在此之前，只有以只读方式共享。

注意：fork之后父子进程共享文件，fork产生的子进程与父进程相同的文件文件猫达符指向相同的文件表，引1用计数增加，共享文件偏移指針。
![[父子进程虚拟地址空间.png]]

父子进程之间的关系：
区别：
1.fork函数返回值不同
2.pid和ppid
3.信号集

共同点：
某些状态下，子进程刚被创建出来，还没有执行任何的写数据操作（用户区的数据，文件描述符表）

父子进程对变量是不是共享的？
刚开始的时候，是共享的，篡改了数据就不共享了


#### GDB调试多进程调试
![[gdb多进程调试.png]]
视频说gdb 8.0版本以上的gdb在调试多进程时会有bug

#### exec函数族
exec 函数族的作用是根据指定的文件名找到可执行文件，并用它来取代调用进程的内容，换句话说，就是在调用进程内部执行一个可执行文件。

exec 函数族的函数执行成功后不会返回，因为调用进程的实体，包括代码段，数据段和堆栈等都已经被新的内容取代，只留下进程 ID 等一些表面上的信息仍保持原样，颇有些神似〝三十六计“中的〝金蝉脱壳〞。看上去还是1旧的躯壳，却已经注入了新的灵魂。只有调用失败了，它们才会返回 一1，从原程序的调用点接着往下执行。

执行exec("a.out")之后，原先用户区的内容会被指定的a.out的内容所替换
![[exec函数族.png]]

exec函数，前六个都是标准库里面的，最后一个execve是linux的，常用的是execl和execlp
![[exec函数.png]]

The **exec** family of functions replaces the current process image with a new process image.  
The initial argument for these functions is the pathname of a file which is to be executed.

 The const char *arg0 and subsequent ellipses in the **execl**(), **execlp**(), and **execle**() functions can be thought of as arg0, arg1, ..., argn.  Together they describe a list of one or more pointers to null-terminated strings that represent the argument list available to the executed program.  The first argument, by convention, should point to the file name associated with the file being executed.  The list of arguments must be terminated by a NULL pointer.
 
 If any of the **exec**() functions returns, an error will have occurred.  The return value is -1, and the global variable errno will be set to indicate the error.

#### 进程控制
exit是标准库的，多做了一些事情，调用退出处理函数，刷新IO缓冲关闭文件描述符，最后调用_exit
![[进程退出exit.png]]
```c
//只会输出hello,因为\n会刷新缓冲区，但是_exit(0)不会
printf("hello\n");
printf("world");
_exit(0);
return 0;

//会输出
//hello
//world
printf("hello\n");
printf("world");
exit(0);
return 0;
```

##### 孤儿进程
父进程运行结束，但子进程还在运行（未运行结束）．这样的子进程就称为孤儿进程(Orphan Process) •

每当出现一个孤儿进程的时候，内核就把孤儿进程的父进程设置为 init（pid=1）．而 init进程会循环地 wait(）它的已经退出的子进程。这样，当一个孤儿进程凄凉地结束了其生命周期的时候．init 进程就会代表党和政府出面处理它的一切善后工作。

因此孤儿进程并不会有什么危害。
```c
#include <unistd.h>
#include<stdio.h>
 int main(){
    printf("fuck you\n");
    pid_t pid = fork();
    if(pid == 0){
        sleep(3);
        printf("I am child process %d; My parent's process %d\n", getpid(), getppid());
    }else if(pid > 0){
        printf("I am parent process %d; My parent process %d\n", getpid(), getppid());
    }
    for(int i = 0;i < 3;i++){
        printf("%d\n", i);
    }
    return 0;
 }
```
可以看到子进程的ppid为1，子进程和父进程的内核区是一样的，因此也在该终端输出（文件描述符012，标准输入标准输出标准错误）
![[孤儿进程演示.png]]

##### 僵尸进程
每个进程结束之后，都会释放自己地址空间中的用户区数据，内核区的 PCB 没有办法自己释放掉，需要父进程去释放。

进程终止时，父进程尚末回收，子进程残留资源（PCB） 存放于内核中，变成僵尸(zombie）进程。

僵尸进程不能被 kill-9杀死。

这样就会导致一个问题，如果父进程不调用 wait(）或waitpid（）的话，那么保留的那段信息就不会释放，其进程号就会一直被占用，但是系统所能使用的进程号是有限的，如果大量的产生僵尸进程，将因为没有可用的进程号而导致系统不能产生新的进程，此即为僵尸进程的危害，应当避免。

#### 进程回收
在每个进程退出的时候，内核释放该进程所有的资源、包括打开的文件、占用的内存等。但是仍然为其保留一定的信息，这些信息主要主要指进程控制块PCB的信息（包括进程号、退出状态、运行时间等）。

父进程可以通过调用wait或waitpid得到它的退出状态同时彻底清除掉这个进程。

wait(）和waitpid(）函数的功能一样，区别在于，wait(）函数会阻塞，waitpid(）可以设置不阻塞，waitpid（）还可以指定等待哪个子进程结束。

注意：一次wait或waitpid调用只能清理一个子进程，清理多个子进程应使用循环。

```c
//等待任意一个进程结束，如果任意一个子进程结束了，此函数会回收子进程

//参数，进程退出时的状态信息，传出参数
//返回值，成功返回被回收的进程pid，失败-1（所有子进程结束或者调用函数失败）

//调用wait的进程会被挂起（阻塞），直到它的子进程退出或者收到一个不能忽略的信号才被唤醒
//如果没有子进程了或者子进程都结束了，函数立即返回，返回-1
pid_t wait(int *stat_loc);
```
return 0相当于exit(0)
![[退出信息相关宏函数.png]]
```c
#include<sys/wait.h>
#include <unistd.h>
#include<stdio.h>
#include <stdlib.h>
int main(){
    pid_t pid;
    //创建5个子进程
    for(int i = 0; i < 5; i++){
        pid = fork();
        if(pid == 0){
            break;
        }
    }
    if(pid > 0){
        //父进程
        while(1){
            printf("parent %d\n", getpid());
            // int ret = wait(NULL);
            int st;
            int ret = wait(&st);
            if(ret == -1){
                break;
            }
            if(WIFEXITED(st)){
                //是不是正常退出 
                printf("退出的状态码: %d\n", WEXITSTATUS(st));
            }
            if(WIFSIGNALED(st)){
                //是不是异常终止
                printf("是被哪个信号干掉了: %d\n", WTERMSIG(st));
            }

            printf("child die %d\n", ret);
            sleep(1);
        }
    }else if(pid == 0){
        // while(1){
            printf("child %d\n", getpid());
            sleep(2);
            exit(1);
        // }
    }
    return 0;
}
```
waitpid
```c
//pid = 0:回收当前进程组的子进程
//pid > 0:某个子进程的pid
//pid = -1:回收所有子进程，相当于wait
//pid < -1:某个进程组的组id的绝对值，回收指定进程组的子进程
//options:0-阻塞，WNOHANG-非阻塞
//返回值: >0返回子进程pid;=0表示还有子进程活着(options = WNOHANG);-1错误/没有子进程
pid_t waitpid(pid_t pid, int *stat_loc, int options);
```
waitpid使用
```c
#include<sys/wait.h>
#include <unistd.h>
#include<stdio.h>
#include <stdlib.h>
int main(){
    pid_t pid;
    //创建5个子进程
    for(int i = 0; i < 5; i++){
        pid = fork();
        if(pid == 0){
            break;
        }
    }

    if(pid > 0){
        //父进程
        while(1){
            printf("parent %d\n", getpid());
            sleep(1);
            // int ret = wait(NULL);
            int st;
            int ret = waitpid(-1, &st, WNOHANG);
            
            if(ret == -1){
                break;
            }else if(ret == 0){
                //说明还有子进程存在
                continue;
            }else if(ret > 0){
                if(WIFEXITED(st)){
                    //是不是正常退出 
                    printf("退出的状态码: %d\n", WEXITSTATUS(st));
                }
                if(WIFSIGNALED(st)){
                    //是不是异常终止
                    printf("是被哪个信号干掉了: %d\n", WTERMSIG(st));
                }
                printf("child die %d\n", ret);
            }
        }
    }else if(pid == 0){
        while(1){
            printf("child %d\n", getpid());
            sleep(2);
            // exit(1);
        }
    }
    return 0;
}
```

### 进程间通信
进程是一个独立的资源分配单元，不同进程（这里所说的进程通常指的是用户进程）之间的资源是独立的，没有关联，不能在一个进程中直接访问另一个进程的资源。

但是，进程不是孤立的，不同的进程需要进行信息的交互和状态的传递等，因此需要进程间通信( IPC: Inter Processes Communication )。

进程间通信的目的：
-   ﻿数据传输：一个进程需要将它的数据发送给另一个进程。
-   ﻿通知事件：一个进程需要向另一个或一组进程发送消息，通知它（它们）发生了某种
-   事件（如进程终止时要通知父进程）。
-   ﻿资源共享：多个进程之间共享同样的资源。为了做到这一点，需要内核提供互斥和同步机制。
- 进程控制：有些进程希望完全控制另一个进程的执行（如 Debug进程），此时控制进程希望能够拦截另一个进程的所有陷入和异常，并能够及时知道它的状态改变。
![[进程间通信方式.png]]
#### 匿名管道
进程间通信，ls和wc -l两个进程，中间是一个管道
![[匿名管道.png]]
管道特点：
管道其实是一个在内核内存中维护的缓冲区，这个缓冲器的存储能力是有限的，不同的操作系统大小不一定相同。

管道拥有文件的特质（管道的两端分别是文件描述符）：读操作、写操作，匿名管道没有文件实体，有名管道有文件实体，但不存储数据公 可以按照操作文件的方式对管道进行操作。

一个管道是一个字节流，使用管道时不存在消息或者消息边界的概念，从管道读取数据的进程可以读取任意大小的数据块，而不管写入进程写入管道的数据块的大小是多少。

通过管道传递的数据是顺序的，从管道中读取出来的宇节的顺序和它们被写入管道的顺序是完全一样的。
![[管道特点.png]]
fork子进程之后，是共享文件描述符的，因此可以父进程使用5，子进程使用6来通信
![[为什么可以用管道进程间通信.png]]
环形队列
![[管道数据结构.png]]
![[匿名管道使用.png]]
管道默认是阻塞的，如果管道中没有数据，read阻塞，如果管道满了，write阻塞
```c
//The pipe() function creates a pipe (an object that allows unidirectional data flow) and allocates a pair of file descriptors.  The first descriptor connects to the read end of the pipe; the second connects to the write end.
int pipe(int fildes[2]);
//注意匿名管道只能用于具有关系的进程间通信，父子兄弟
```
半双工，双方互发，注意文件描述符，读和写要用同一个。注意，下列代码如果不sleep会产生问题，比如子进程读了自己写的，或者父进程读了自己写的。所以正确应该一个进程读，close关闭写进程，另外一个进程写，close关闭读进程
```c
#include <unistd.h>
#include<stdio.h>
#include <sys/types.h>
#include <sys/uio.h>
#include <string.h>
#include <stdlib.h>
int main(){
    //在fork之前创建管道
    int pipefd[2];
    int ret = pipe(pipefd);
    if(ret == -1){
        perror("pipe");
        exit(0);
    }
    pid_t pid = fork();
    if(pid > 0){
        //parent
        char buf[1024]= {0};
        while(1){
            int len = read(pipefd[0], buf, sizeof(buf));
            printf("parent recv %s, pid: %d\n", buf, getpid());
            char *str = "hello, I am parent";
            write(pipefd[1], str, strlen(str));
            sleep(2);
        }
    }else if(pid == 0){
        //child
        char buf[1024] = {0};
        while(1){
            char *str = "hello, I am child";
            write(pipefd[1], str, strlen(str));
            int len = read(pipefd[0], buf, sizeof(buf));
            printf("child recv %s, pid: %d\n", buf, getpid());
            sleep(2);
        }

    }
    return 0;
}
```
![[为什么不sleep会产生问题.png]]

实现IPC（ps aux | grep xxx）父子进程通信
子进程ps aux，子进程结束后，将数据发给父进程
父进程获取到数据，过滤
pipe()
execlp()默认发送到当前终端
dup子进程将标准输出stdout_fileno重定向到管道的写端


管道的读写特点：

使用管道时，需要注意以下几种特殊的情况（假设都是阻塞I/0操作）
1.所有的指向管道写端的文件描述符都关闭了（管道写端引用计数为0），有进程从管道的读端读数据，那么管道中剩余的数据被读取以后，再次read会返回0，就像读到文件末尾一样。

2.如果有指向管道写端的文件描述符没有关闭（管道的写端引用计数大于0），而持有管道写也没有往管道中写数据，这个时候有进程从管道中读取数据，那么管道中剩余的数据被读取后
再次read会阻塞，直到管道中有数据可以读了才读取数据并返回。

3.如果所有指向管道读端的文件描述符都关闭了（管道读端引用计数等于0）这个时候有进程向管道写数据，那么该进程会收到一个信号 SIGPIPE，通常会导致进程终止

4.如果指向管道读端的文件描述符没有关闭（管道读端引用计数大于0），而持有管道读端也没有从管道中读数据，这时有进程向管道中写数据，那么在管道被写满的时候再次write会阻塞，直到管道中有空位置才能再次写入数据并返回。

总结：
读管道：管道中有数据，read返回实际读到的字节数；管道中无数据：写端被全部关闭，read返回0（相当于读到文件的末尾)；写端没有完全关闭，read阻塞等待

写管道：管道读端全部被关闭，进程异常终止（进程收到SIGPIPE信号）；管道读端没有全部关闭：管道已满，write阻塞；管道没有满：write将数据写入，并返回实际写入的字节数

设置管道非阻塞就是设置文件描述符非阻塞
```c
int flags = fcntl(fd, F_GETFL);
flags |= O_NONBLOCK;
fcntl(fd, F_SETFL, flags);
```

#### 有名管道
匿名管道，由于没有名字，只能用于亲缘关系的进程问通信。为了克服这个缺点，提出了有名管道（FIFO） ，也叫命名管道、FIFO文件。

有名管道(FIFO) 不同于匿名管道之处在于它提供了一个路径名与之关联，以 FIFO的文件形式存在于文件系统中，并且其打开方式与打开一个普通文件是一样的，这样即使与 FIFO 的创建进程不存在亲缘关系的进程，只要可以访问该路径，就能够彼此通过 FIFO 相互通信，因此，通过 FIFO 不相关的进程也能交换数据。一旦打开了 FIFO，就能在它上面使用与操作屬名管道和其他文件的系统调用一样的

I/0系统调用了（如read ()、write（）和close（)）。与管道一样，FIFO也有一个写入端和读取端，并且从管道中读取数据的顺序与写入的顺序是一样的。FIFO 的名称也由此而来：先入先出。

有名管道(FIFO)和匿名管道(pipe）有一些特点是相同的，不一样的地方在于：
1.FIFO 在文件系统中作为一个特殊文件存在，但FIFO 中的内容却存放在内存（内核内存中维护的一块缓冲区）中。2.﻿﻿当使用 FIFO 的进程退出后，FIFO文件将继续保存在文件系统中以便以后使用。3.FIFO 有名字，不相关的进程可以通过打开有名管道进行通信。
![[有名管道的使用.png]]
```c
//用法和open一样
int mkfifo(const char *path, mode_t mode);
```
有名管道的注意事项
1.一个为只读而打开一个管道的进程会阻塞，直到另外一个进程为只写打开管道
2.一个为只写而打开一个管道的进程会阻塞，直到另外一个进程为只读打开管道

读管道：管道中有数据，read返回实际读到的字节数；管道中无数据：写端被全部关闭，read返回0（相当于读到文件的末尾)；写端没有完全关闭，read阻塞等待

写管道：管道读端全部被关闭，进程异常终止（进程收到SIGPIPE信号）；管道读端没有全部关闭：管道已满，write阻塞；管道没有满：write将数据写入，并返回实际写入的字节数

读端
```c
#include <stdio.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
 #include <stdlib.h>
int main(){
    //1.打开管道文件
    int fd = open("fifo1", O_RDONLY);
    if(fd == -1){
        perror("open");
        exit(0);
    }
    while(1){
        char buf[1024] = {0};
        int len = read(fd, buf, sizeof(buf));
        if(len == 0){
            printf("写端断开连接了...\n");
            break;
        }
        printf("recv buf: %s\n, buf", buf);
    }
    close(fd);
    return 0;
}
```
写端
```c
#include <stdio.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <string.h>
#include <stdlib.h>
int main(){
    int ret = access("fifo1", F_OK);
    if(ret == -1){
        printf("管道不存在,创建管道\n");
        //创建管道文件
        ret = mkfifo("fifo1", 0664);
        if(ret == -1){
            perror("mkfifo");
            exit(0);
        }
    }
    //以只写的方式打开管道
    int fd = open("fifo1", O_WRONLY);
    if(fd == -1){
        perror("open");
        exit(0);
    }
    for(int i = 0; i < 100; i++){
        char buf[1024];
        sprintf(buf, "hello, %d\n", i);
        printf("write data : %s\n", buf);
        write(fd, buf, strlen(buf));
        sleep(1);
    }
    close(fd);
    return 0;
}
```
案例，使用有名管道和匿名管道（父子）完成聊天功能(匿名管道父子进程来实现两边都可以同时读同时写，有名管道来实现进程间通信)
![[有名管道聊天.png]]
chatA
```c
#include <stdio.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <string.h>
#include <stdlib.h>
int main(){
    int ret = access("fifochat_1", F_OK);
    if(ret == -1){
        printf("管道不存在,创建对应的有名管道\n");
        ret = mkfifo("fifochat_1", 0664);
        if(ret == -1){
            perror("mkfifo");
            exit(0);
        }
    }

    ret = access("fifochat_2", F_OK);
    if(ret == -1){
        printf("管道不存在,创建对应的有名管道\n");
        ret = mkfifo("fifochat_2", 0664);
        if(ret == -1){
            perror("mkfifo");
            exit(0);
        }
    }
    //以只写的方式打开管道1
    int fdw = open("fifochat_1", O_WRONLY);
    if(fdw == -1){
        perror("open");
        exit(0);
    }
    printf("打开管道1成功，等待写入...\n");
    //以只读方式打开管道2
    int fdr = open("fifochat_2", O_RDONLY);
    if(fdr == -1){
        perror("open");
        exit(0);
    }
    printf("打开管道2成功，等待读取...\n");

    pid_t pid = fork();
    if(pid > 0){
        //parent
        char buf[128];
        while(1){
            memset(buf, 0, 128);
            fgets(buf, 128, stdin);
            //写数据
            ret = write(fdw, buf, strlen(buf));
            if(ret == -1){
                perror("write");
                break;
            }
        }
        close(fdw);
        wait(NULL);
    }else if(pid == 0){
        //child
        char buf[128];
        while(1){
            memset(buf, 0, 128);
            ret = read(fdr, buf, 128);
            if(ret <= 0){
                perror("read");
                break;
            }
            printf("buf : %s\n", buf);
        }
        close(fdr);
    }else{
        perror("fork");
        exit(0);
    }
    
    return 0;
}
```
chatB
```c
#include <stdio.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <string.h>
#include <stdlib.h>
int main(){
    int ret = access("fifochat_1", F_OK);
    if(ret == -1){
        printf("管道不存在,创建对应的有名管道\n");
        ret = mkfifo("fifochat_1", 0664);
        if(ret == -1){
            perror("mkfifo");
            exit(0);
        }
    }

    ret = access("fifochat_2", F_OK);
    if(ret == -1){
        printf("管道不存在,创建对应的有名管道\n");
        ret = mkfifo("fifochat_2", 0664);
        if(ret == -1){
            perror("mkfifo");
            exit(0);
        }
    }
    //以只读的方式打开管道1
    int fdr = open("fifochat_1", O_RDONLY);
    if(fdr == -1){
        perror("open");
        exit(0);
    }
    printf("打开管道1成功，等待读取...\n");
    //以只写方式打开管道2
    int fdw = open("fifochat_2", O_WRONLY);
    if(fdw == -1){
        perror("open");
        exit(0);
    }
    printf("打开管道2成功，等待写入..\n");

    pid_t pid = fork();
    if(pid > 0){
        //parent
        char buf[128];
        while(1){
            memset(buf, 0, 128);
            fgets(buf, 128, stdin);
            //写数据
            ret = write(fdw, buf, strlen(buf));
            if(ret == -1){
                perror("write");
                break;
            }
        }
        close(fdw);
        wait(NULL);
    }else if(pid == 0){
        //child
        char buf[128];
        while(1){
            memset(buf, 0, 128);
            ret = read(fdr, buf, 128);
            if(ret <= 0){
                perror("read");
                break;
            }
            printf("buf : %s\n", buf);
        }
        close(fdr);
    }else{
        perror("fork");
        exit(0);
    }

    return 0;
}
```

#### 内存映射
![[内存映射.png]]
```c
//**mmap** – allocate memory, or map files or devices into memory
#include <sys/mman.h>
void mmap(void *addr, size_t len, int prot, int flags, int fd, off_t offset);
//参数
	-addr:传NULL，由内核指定
	-要映射的文件长度，不能为0，建议使用文件的长度，stat lseek
	-对申请的内存映射区的操作权限(PROT_EXEC, PROT_READ,...)
	-flags(MAP_SHARED映射区的数据会自动和磁盘同步，进程间通信必须设置;MAP_PRIVATE不同步，对原来的文件不会修改，copy on write)
	-文件描述符，通过open一个磁盘文件得到，文件大小不能为0，文件权限不能和prot冲突
	-偏移量，一般不用，必须是4k整数倍，
//返回值:成功返回创建内存的首地址，失败返回MAP_FAILED

//释放内存映射
int munmap(void *addr, size_t len);
//参数:addr要释放的内存首地址; length:要释放的内存大小
```
使用内存映射实现进程通信
1.父子进程：还没有子进程的时候，通过唯一的父进程，先创建内存映射区，然后创建子进程，父子进程共享创建的内存映射区
2.没有关系的进程间通信：准备一个大小不是0的磁盘文件，进程1通过磁盘文件创建内存映射区，得到操作这块内存的指针；进程2通过磁盘文件创建内存映射区，得到操作这块内存的指针；使用内存映射区通信

注意：内存映射区通信是非阻塞的
父子进程间示例
```c
    int fd = open("test.txt", O_RDWR);
    //获取文件大小
    int size = lseek(fd, 0, SEEK_END);
    //创建内存映射区
    void *ptr = mmap(NULL, size, PROT_READ | PROT_WRITE, MAP_SHARED, fd, 0);
    if(ptr == MAP_FAILED){
        perror("mmap");
        exit(0);
    }
    pid_t pid = fork();
    if(pid == 0){
        //
        strcpy((char *)ptr, "nihao a, son!!!");
    }else{
        wait(NULL);
        char buf[64];
        strcpy(buf, (char *)ptr);
        printf("read data %s\n", buf);
    }
    munmap(ptr, size);
    return 0;
```

![[mmap思考问题.png]]
1.可以进行++，但是释放的时候会出错；可以++之前保存一下地址，释放的时候用开始的那个地址
2.错误，返回 MAP_FAILED; 建议 open函数中的权限建议和port权限参数保持一致
3.偏移量必须是4k的整数倍，返回MAP_FAILED
4.第二个参数length = 0，第三个参数port只指定了写权限（必须指定读权限）
5.可以的，但是创建的文件大小为0的话，不行
可以对新的文件进行扩展（lseek，truncate ）
6.
int fd = open(...)
mmap(..)
close(fd)
映射区还存在，没有任何影响
7.操作非法内存，段错误 

使用内存映射实现文件拷贝的功能
1.对原始文件进行内存映射
2.创建新文件并扩展
3.把新文件的数据映射到内存中
4.通过内存拷贝将第一个文件的内存数据拷贝到新的文件内存中
5.释放资源
```c
    int fd = open("english.txt", O_RDWR);//虽然此时读权限已经够了
    if(fd == -1) {
        perror("open");
        exit(0);
    }
    // 获取原始文件的大小
    int len = lseek(fd, 0, SEEK_END);
    // 2.创建一个新文件（拓展该文件）
    int fd1 = open("cpy.txt", O_RDWR | O_CREAT, 0664);
    if(fd1 == -1) {
        perror("open");
        exit(0);
    }
    // 对新创建的文件进行拓展
    truncate("cpy.txt", len);//len要和原始文件大小一样
    write(fd1, " ", 1);
    // 3.分别做内存映射
    void * ptr = mmap(NULL, len, PROT_READ | PROT_WRITE, MAP_SHARED, fd, 0);
    void * ptr1 = mmap(NULL, len, PROT_READ | PROT_WRITE, MAP_SHARED, fd1, 0);
    if(ptr == MAP_FAILED) {
        perror("mmap");
        exit(0);
    }
    if(ptr1 == MAP_FAILED) {
        perror("mmap");
        exit(0);
    }
    // 内存拷贝
    memcpy(ptr1, ptr, len);
    // 释放资源  先打开的后释放，后打开的先释放
    munmap(ptr1, len);
    munmap(ptr, len);
```

匿名映射：不需要文件实体进行内存映射

#### 信号
![[信号.png]]
![[信号2.png]]
![[linux信号一览表.png]]
![[linux信号一览表2.png]]
![[linux信号一览3.png]]
![[信号5种默认处理作用.png]]
GDB可以通过调试Core文件得到错误信息
![[信号相关函数.png]]
真实时间 =  user-cpu time + system-cpu time +消耗的时间(IO等等)
alarm和setitimer的区别是alarm只能完成一次，而setitimer能完成周期性的事情
默认发送SIGALRM信号，默认是把当前进程终止

signal信号捕捉函数
![[signal捕捉信号.png]]

![[两个信号捕捉函数.png]]

许多信号相关的系统调用都需要能表示一组不同的信号，多个信号可使用一个称之为信号集的数据结构来表示，其系统数据类型为 sigset_t。

在 **PCB 中有两个非常重要的信号集**。一个称之为“阻塞信号集”，另一个称之为“未决信号集”。这两个信号集都是内核使用位图机制来实现的。但操作系统不允许我们直接对这两个信号集进行位操作。而需自定义另外一个集合，借助信号集操作函数来对 PCB 中的这两个信号集进行修改。

信号的“未决” 是一种状态，指的是从信号的产生到信号被处理前的这一段时问。

信号的〝阻塞”是一个开关动作，指的是阻止信号被处理，但不是阻止信号产生。

信号的阻塞就是让系统暂时保留信号留待以后发送。由于另外有办法让系统忽略信号，所以一般情况下信号的阻塞只是暂时的，只是为了防止信号打断敏感的操作。

信号集
![[阻塞信号集和未决信号集.png]]
1.  ﻿﻿用户通过键盘ctrl +C，产生2号信号SIGINT（信号被创建）
2.  ﻿﻿信号产生但是没有被处理（未决）
	在内核中将所有的没有被处理的信号存储在一个集合中（未决信号集）
	﻿﻿SIGINT信号状态被存储在第二个标志位上
		﻿﻿这个标志位的值为0， 说明信号不是未决状态
		﻿﻿﻿这个标志位的值为1，说明信号处于未决状态
3. 这个未决状态的信号，需要被处理，处理之前要和另一个信号集(阻塞信号集)比较
	阻塞信号集默认不阻塞任何信号
	如果想要阻塞某些信号，需要用户调用系统API
4. 处理的时候和阻塞信号集中的标志位进行查询，看是不是对该信号设置阻塞了
	如果没有阻塞，信号就被处理
	如果阻塞了，信号就处于未决状态，直到阻塞解除，这个信号就被处理
![[信号集相关的函数.png]]
sigemptyset:清空信号集的数据，将信号集所有的标识位置为0；参数:set传出参数，需要操作的信号集;返回0成功失败-1
sigfillset:将信号集所有的标识位置为1....
sigaddset:设置信号集中的某一个信号对应的标识位为1，表示阻塞这个信号
sigdelset: 设置信号集中的某一个信号对应的标识位为0，表示不阻塞这个信号
sigismember:判断某个信号是否阻塞
(上面几种都是对自定义信号集进行修改)
sigprocmask:操作都内核当中信号集
sigpending:获取内核中的未决信号集，传出参数
![[两个信号捕捉函数.png]]
sigaction:检查或者改变信号的处理，信号捕捉
参数:signum-需要捕捉的信号的编号或者宏值(SIGKILL, SIGSTOP不能被捕捉，忽略，阻塞)
act-捕捉信号之后的处理动作
oldact-上一次对信号捕捉相关的设置，一般不使用,传递NULL

**最好使用sigaction而不是signal**

内核实现信号捕捉的过程
![[内核实现信号捕捉的过程.png]]
![[Pasted image 20230331113339.png]]
未决信号集中的每个信号只有1个标识位，只能标识1次的状态
1.内核中有一个阻塞信号集，信号捕捉的过程中会使用临时的阻塞信号集，信号处理完后会恢复内核中的阻塞信号集
2.捕捉一个信号执行回调函数期间，信号会被默认屏蔽掉，别人再发这个信号，不会执行这个回调函数，只有等上一个回调函数执行完后才会再执行
3.阻塞常规信号不支持排队，同时发送了很多个信号只能记录1个，剩下的会被丢弃

![[sigchild信号.png]]
 SIGCHILD能解决僵尸进程（wait会阻塞）
 ```c
  #include <unistd.h>
 #include <stdio.h>
 #include <sys/types.h>
 #include <sys/stat.h>
 #include <signal.h>
 #include <sys/wait.h>

void myFun(int num){
    printf("捕捉到的信号: %d\n", num);
    // while(1){
    //     wait(NULL);
    // }
    while(1){
        int ret = waitpid(-1, NULL, WNOHANG);
        if(ret > 0){
            printf("child die, pid = %d\n", ret);
        }else if(ret == 0){
            //还有子进程活着
            break;
        }else if(ret == -1){
            //没有子进程活着
            break;
        }
    }
}

int main(){
    //提前设置好阻塞信号集，阻塞SIGCHLD，因为有可能子进程很快结束，父进程还没注册完信号捕捉
    sigset_t set;
    sigemptyset(&set);
    sigaddset(&set, SIGCHLD);
    sigprocmask(SIG_BLOCK, &set, NULL);

    pid_t pid;
    for(int i = 0; i < 20; i++){
        pid = fork();
        if(pid == 0) break;
    }
    if(pid > 0){
        //捕捉子进程死亡时发送的SIGCHLD信号
        struct sigaction act;
        act.sa_flags = 0;
        act.sa_handler = myFun;
        sigemptyset(&act.sa_mask);
        sigaction(SIGCHLD, &act, NULL);
        //注册完信号捕捉以后，解除阻塞
        sigprocmask(SIG_UNBLOCK, &set, NULL);

        while(1){
            printf("parent process %d\n", getpid());
            sleep(2);
        }
    }else if(pid == 0){
        printf("child process %d\n", getpid());  
    }
}
```

#### 共享内存（效率最高）
共享内存允许两个或者多个进程共享物理内存的同一块区域(通常被称为段）。由于一个共享内存段会称为一个进程用户空间的一部分，因此这种 IPC 机制无需内核介入。所有需要做的就是让一个进程将数据复制进共享内存中，并且这部分数据会对其他所有共享同一个段的进程可用。

与管道等要求发送进程将数据从用户空间的缓冲区复制进内核内存（管道是内核中的一块缓存空间）和接收进程将数据从内核内存复制进用户空间的缓冲区的做法相比，这种 IPC 技术的速度更快。

使用步骤
1.调用 shmget() 创建一个新共享内存段或取得一个既有共享内存段的标识符（即由其他进程创建的共享内存段）。这个调用将返回后续调用中需要用到的共享内存标识符。

2.关联进程:使用 shmat() 来附上共享内存段，即使该段成为调用进程的虚拟内存的一部分。

3.此刻在程序中可以像对待其他可用内存那样对待这个共享内存段。为引用这块共享内存，程序需要使用由 shmat() 调用返回的 addr 值，它是一个指向进程的虚拟地址空间中该共享内存段的起点的指针。

4.调用 shmdt()来分离共享内存段。在这个调用之后，进程就无法再引用这块共享内存了。这一步是可选的，并且在进程终止时会自动完成这一步。

5.调用 shmctl() 来州除共享内存段。只有当当前所有附加内存段的进程都与之分离之后内存段才会销毁。只有一个进程需要执行这一步。

共享内存操作函数
![[shmget说明.png]]
![[shmat说明.png]]
![[shmdt说明.png]]
删除共享内存，共享内存要删除才会消失，创建共享内存的进程被销毁了对共享内存是没有任何影响的
![[shmctl说明.png]]
共享内存操作命令
![[共享内存操作命令.png]]
![[共享内存说明.png]]
共享内存和内存映射区别
1.共享内存可以直接创建，内存映射需要磁盘映射(匿名映射除外)
2.共享内存效率更高，
3.共享内存:所有进程操作的是同一块共享内存，内存映射:每个进程在自己的虚拟空间有一块独立的内存
4.数据安全，进程突然退出:共享内存还存在，但是内存映射区消失了
运行进程的电脑死机，数据存在共享内存中没有了，内存映射区的数据由于磁盘文件的数据还在所以内存映射区的数据还在
5.生命周期
-内存映射区，进程退出，内存映射区销毁
-共享内存，进程退出，共享内存还在，手动删除(所有关联的进程数为0)或者关机，如果一个进程退出，会自动和共享内存取消关联

write_shm.c
```c
#include <stdio.h>
#include <sys/ipc.h>
#include <sys/shm.h>
#include <stdlib.h>
#include <string.h>
int main(){
    //1.创建1个共享内存
    int shmid = shmget(100, 4096, IPC_CREAT|0664);
    //2.关联进程
    void *ptr = shmat(shmid, NULL, 0);
    //3.写数据
    char *str = "helloworld";
    memcpy(ptr, str, strlen(str) + 1);

    printf("按任意键继续\n");
    getchar();
    //4.解除关联
    shmdt(ptr);
    //5.删除共享内存
    shmctl(shmid, IPC_RMID, NULL);
}
```
read_shm.c
```c
#include <stdio.h>
#include <sys/ipc.h>
#include <sys/shm.h>

int main(){
    //1.获取1个共享内存
    int shmid = shmget(100, 0, IPC_CREAT);
    //2.关联进程
    void *ptr = shmat(shmid, NULL, 0);
    //3.读数据
    printf("%s \n", (char *)ptr);
    printf("按任意键继续\n");
    getchar();
    //4.解除关联
    shmdt(ptr);
    //5.删除共享内存
    shmctl(shmid, IPC_RMID, NULL);
}
```

#### 守护进程
终端
在 UNIX 系统中，用户通过终端登录系统后得到一个shell 进程，这个终端成为 shell 进程的控制终端 (controlling Terminal），进程中，控制终端是保存在 PCB 中的信息，而fork（） 会复制PCB 中的信息，因此由 shel1进程启动的其它进程的控制终端也是这个终端。

默认情况下（没有重定向），每个进程的标准输入、标准输出;和标准错误输出都指向控制终端，进程从标准输入读也就是读用户的键盘输入，进程往标准输出或标准错误输出写也就是输出到显示器上。

在控制终端输入一些特殊的控制键可以给前台进程(后台进程不会)发信号，例如Ctrl+C 会产生 SIGINT信号，Ctrl+\ 会产生 SIGQUIT信号。

进程组
进程组和会话在进程之问形成了一种两级层次关系：进程组是一组相关进程的集合，会话是一组相关进程组的集合。进程组合会话是为支持 shell 作业控制而定义的抽象概念，用户通过 shell 能够交互式地在前台或后台运行命令。

进行组由一个或多个共享同一进程组标识符(PGID） 的进程组成。一个进程组拥有一个进程组首进程，该进程是创建该组的进程，其进程 ID 为该进程组的工D，新进程会继承其父进程所属的进程组 ID。

进程组拥有一个生命周期，其开始时间为首进程创建组的时刻，结束时间为最后一个成员进程退出组的时刻。一个进程可能会因为终止而退出进程组，也可能会因为加入了另外一个进程组而退出进程组。进程组首进程无需是最后一个离开进程组的成员。

会话
会话是一组进程组的集合。会话首进程是创建该新会话的进程，其进程 ID 会成为会话 ID。新进程会继承其父进程的会话 ID。

一个会话中的所有进程共享单个控制终端。控制终端会在会话首进程首次打开一个终端设备时被建立。一个终端最多可能会成为一个会话的控制终端。

在任一时刻，会话中的其中一个进程组会成为终端的前合进程组，其他进程组会成为后合进程组。只有前台进程组中的进程才能从控制终端中读取输入。当用户在控制终端中输入终端宇符生成信号后，该信号会被发送到前台进程组中的所有成员。

当控制终端的连接建立起来之后，会话首进程会成为该终端的控制进程。

进程组会话控制终端的关系
![[进程组会话控制终端的关系.png]]
进程组，会话操作函数
![[进程组，会话操作函数.png]]

守护进程
守护进程 (Daemon Process)，也就是通常说的 Daemon 进程（精灵进程），是Linux 中的后台服务进程。它是一个生存期较长的进程，通常独立于控制终端并且周期性地执行某种任务或等待处理某些发生的事件。一般采用以 d 结尾的名字。

守护进程具备下列特征：
-生命周期很长，守护进程会在系统启动的时候被创建并一直运行直至系统被关闭。
-它在后台运行并且不拥有控制终端。没有控制终端确保了内核永远不会为守护进程自动生成任何控制信号以及终端相关的信号（如 SIGINT、SIGQUIT)。

Linux 的大多数服务器就是用守护进程实现的。比如，Internet 服务器 inetd,Web服务器httpd等。

![[守护进程创建步骤.png]]
1，2步和核心业务逻辑是必须的。
为什么要fork？为什么了让子进程不成为进程组首进程，让子进程后续能执行setsid。
为什么父进程退出？为了避免终端产生shell提示符。
为什么子进程要setsid？为了摆脱控制终端，也为了避免进程组id和会话id冲突

daemon.c
```c
/*
    写一个守护进程，每隔2s获取系统时间并且写入到磁盘文件中
*/
#include <stdio.h>
#include <sys/stat.h>
#include <sys/types.h>
#include <unistd.h>
#include <fcntl.h>
#include <sys/time.h>
#include <time.h>
#include <stdlib.h>
#include <string.h>
#include <signal.h>
void work(int num){
    //获取系统时间，写入磁盘文件
    time_t tm = time(NULL);
    struct tm *loc = localtime(&tm);
    // char buf[1024];
    // sprintf(buf, "%d-%d-%d-%d-%d-%d",loc->tm_year, loc->tm_mon, loc->tm_mday
    // ,loc->tm_hour, loc->tm_min, loc->tm_sec);
    
    // printf("%s", buf);
    char *str = asctime(loc);
    int fd = open("time.txt", O_RDWR|O_CREAT|O_APPEND, 0664);
    write(fd, str, strlen(str));
    close(fd);
    
}

int main(){
    //1.创建子进程,退出父进程
    pid_t pid = fork();
    if(pid > 0){
        exit(0);
    }
    //2.将子进程重新创建一个会话
    setsid();
    //3.设置掩码
    umask(022);
    //4.更改工作目录
    chdir("/Users/feymanpaper/codeSpace/cppWorkSpace/lesson_linux");
    //5.关闭，重定向文件描述符 
    int fd = open("/dev/null", O_RDWR);
    dup2(fd, STDIN_FILENO);
    dup2(fd, STDOUT_FILENO);
    dup2(fd, STDERR_FILENO);
    //6.业务逻辑
    //捕捉定时信号
    struct sigaction act;
    act.sa_flags = 0;
    act.sa_handler = work;
    sigemptyset(&act.sa_mask);
    sigaction(SIGALRM, &act, NULL);
    //创建定时器
    struct itimerval val;
    val.it_value.tv_sec = 2;
    val.it_value.tv_usec = 0;
    val.it_interval.tv_sec = 2;
    val.it_interval.tv_usec = 0;
    setitimer(ITIMER_REAL, &val, NULL);
    //不让进程结束
    while(1){
        sleep(10);
    }
    
    return 0;
}
```
## Linux多线程开发
### 线程概述 
与进程 (process）类似，线程（thread）是允许应用程序并发执行多个任务的一种机制。一个进程可以包舍多个线程。同一个程序中的所有线程均会独立执行相同程序，且共享同一份全局内存区域，其中包括初始化数据段、未初始化数据段，以及堆内存段。（传统意义上的 UNIX 进程只是多线程程序的一个特例，该进程只包含一个线程）

进程是 CPU 分配资源的最小单位，线程是操作系统调度执行的最小单位。

线程是轻量级的进程 (LWP： Light weight Process)，在Linux环境下线程的本质仍是进程。

查看指定进程的 IWB 号：ps -lf pid

### 线程与进程区别
进程间的信息难以共享。由于除去只读代码段外，父子进程并末共享内存，因此必须采用一些进程间通信方式，在进程间进行信息交换。

调用 fork(）来创建进程的代价相对较高，即便利用写时复制技术，仍热需要复制诸如内存页表和文件描述符表之类的多种进程属性，这意味着fork(）调用在时间上的开销依然不菲。

线程之间能够方便、快速地共享信息。只需将数据复制到共享（全局或堆）变量中即可。

创建线程比创建进程通常要快 10 倍甚至更多。线程间是共享虛拟地址空间的，无需采用写时复制来复制内存，也无需复制页表。
![[线程和进程虚拟地址空间.png]]
线程之间共享资源和非共享资源
![[线程之间共享资源和非共享.png]]
NPTL
当 Linux 最初开发时，在内核中并不能真正支持线程。但是它的确可以通过clone(）系统调用将进程作为可调度的实体。这个调用创建了调用进程 (calling process) 的一个拷贝，这个拷贝与调用进程共享相同的地址空间。LinuxThreads 项目使用这个调用来完全在用户空间模拟对线程的支持。不幸的是，这种方法有一些缺点，尤其是在信号处理、调度和进程间同步等方面都存在问题。另外，这个线程模型也不符合 POSIX 的要求。

要改进 Linuxthreads，需要内核的支持，并且重写线程库。有两个相互竞争的项目开始来满足这些要求。一个包括 IBM 的开发人员的团队开展了 NGPT (Next-GenerationPOSIX Threads）项目。同时，Red Hat 的一些开发人员开展了 NPTL项目。NGPT在2003年中期被放弃了，把这个领域完全留给了 NPTL。

NPTL， 或称为 Native PoSIX Thread Library，是Linux线程的一个新实现，它克服了 IinuxThreads 的缺点，同时也符合 POSIX 的需求。与LinuxThreads 相比，它在性能和稳定性方面都提供了重大的改进。

查看当前 pthread版本：getconf GNU_LIBPTHREAD_VERSION

### 线程操作
![[线程操作.png]]
一般情况下，main函数所在的线程我们称之为主线程，其余创建的线程为子线程
程序中默认只有一个进程，fork函数---2
程序中默认只有一个线程，pthread_create函数----2

```c
int pthread_create(pthread_t *thread, const pthread_attr_t *attr, void *(*start_routine)(void *), void *arg);
//创建一个子线程
//参数:thread传出参数，线程创建成功后，子线程的ID被写入到该变量中
//attr:设置线程的属性，一般NULL
//start_routine:函数指针，子线程需要处理的逻辑代码
//arg:给第三个参数使用，传参
//返回值:成功0 失败错误号(获取错误号信息:char *strerror(int errnum));

//
//编译的时候要Compile and link with -pthread
//gcc pthread_create.c -o create -pthread
```
主线程退出，对其他线程没有影响。但是到return 0 进程退出了就有影响
```c
int pthread_join(pthread_t thread, void **value_ptr);
//wait for thread termination
//和一个已经终止的线程进行连接，回收子线程的资源
//这个函数是阻塞函数，调用一次只能回收一个子线程，一般在主线程中使用

//参数:thread: 需要回收的子线程的ID
//retval:接收子线程退出时的返回值

//返回值：0成功 非0失败的错误号
```
有僵尸进程也有僵尸线程
```c
int pthread_detach(pthread_t thread);
//功能:分离1个线程，被分离的线程在终止的时候，会自动释放资源返回给系统
//1.不能多次分离，会产生不可预料的行为
//2.不能去连接一个已经分离的线程，会报错

//参数:需要分离的线程id
//返回值:成功0 失败错误号
```
示例代码pthread_detach
```c
#include <stdio.h>
#include <string.h>
#include <pthread.h>
#include <unistd.h>
void *callback(void *arg){
    printf("child thread id :%d\n", pthread_self());
    return NULL;
}
int main(){
    pthread_t tid;
    int ret = pthread_create(&tid, NULL, callback, NULL);
    if(ret != 0){
        char *errstr = strerror(ret);
        printf("error : %s\n", errstr);
    }
    //输出主线程和子线程的id
    printf("tid: %ld, main thread id : %ld", tid, pthread_self());
    //设置子线程分离,子线程分离后，子线程结束时对应的资源就不需要主线程释放
    pthread_detach(tid);

    pthread_exit(NULL);
}
```

```c
int pthread_cancel(pthread_t thread); 
//功能:取消线程(让线程终止)
//取消某个线程，可以终止某个线程的运行
//但是并不是立马终止，而是当子线程执行到一个取消点，线程才会终止。
//取消点：系统规定好的一些系统调用，我们可以粗略的理解为从用户区到内核区的切换，这个位置称之为取消点
```
pthread_cancel示例 
```c
#include <stdio.h>
#include <string.h>
#include <pthread.h>
#include <unistd.h>

void *callback(void *arg){
    printf("child thread id :%d\n", pthread_self());
    for(int i = 0; i < 5; i++){
        printf("child : %d\n", i);
        sleep(3);
    }
    return NULL;
}
int main(){
    pthread_t tid;
    int ret = pthread_create(&tid, NULL, callback, NULL);
    if(ret != 0){
        char *errstr = strerror(ret);
        printf("error : %s\n", errstr);
    }
    //输出主线程和子线程的id
    printf("tid: %ld, main thread id : %ld\n", tid, pthread_self());
    //取消线程
    pthread_cancel(tid);

    for(int i = 0; i < 5; i++){
        printf("%d\n", i);
    }

    pthread_exit(NULL);
}
```

### 线程属性
![[线程属性.png]]
初始化线程属性变量，释放线程属性的资源，获取线程分离的状态属性，设置线程分离的状态属性

### 线程同步
线程的主要优势在于，能够通过全局变量来共享信息。不过，这种便捷的共享是有代价的：必须确保多个线程不会同时修改同一变量，或者某一线程不会读取正在由其他线程修改的变量。

临界区是指访问某一共享资源的代码片段，并且这段代码的执行应为原子操作，也就是同时访问同一共享资源的其他线程不应终端该片段的执行。

线程同步：即当有一个线程在对内存进行操作时，其他线程都不可以对这个内存地址进行操作，直到该线程完成操作，其他线程才能对该内存地址进行操作，而其他线程则处于等待状态。

#### 互斥量
为避免线程更新共享变量时出现问题，可以使用互斥量（mutex是mutual exclusion
的缩写）来确保同时仅有一个线程可以访问某项共享资源。可以使用互斥量来保证对任意共享资源的原子访问。

互斥量有两种状态：已锁定 (locked） 和未锁定（unlocked）。任何时候．至多只有一个线程可以锁定该互斥量。试图对已经锁定的某一互斥量再次加锁，将可能阻塞线程或者报
错失败，具体取决于加锁时使用的方法。

一旦线程锁定互斥量，随即成为该互斥量的所有者，只有所有者才能给互斥量解锁。一般情况下，对每一共享资源（可能由多个相关变量组成）会使用不同的互斥量，每一线程在访问
同一资源时将采用如下协议：
•针对共享资源锁定互斥量
•访问共享资源
•对互斥量解锁
![[互斥量.png]]
互斥量相关操作函数
![[互斥量相关操作函数.png]]
```c
int pthread_mutex_init**(pthread_mutex_t *mutex,
						 const pthread_mutexattr_t *attr);
//初始化互斥量
//参数:mutex需要初始化的互斥量变量
//attr:互斥量相关的属性,NULL
```
释放互斥量资源；加锁(阻塞的，如果有一个线程加锁了，那么其他的线程只能阻塞等待)；尝试加锁，如果加锁失败，不会阻塞会直接返回；解锁
卖票示例
```c
// 使用多线程买票示例
//3个窗口一共100张票

#include <stdio.h>
#include <pthread.h>
#include <unistd.h>
int tickets = 1000;

//创建一个互斥量
pthread_mutex_t mutex;

void *sellTicket(void *arg){

    //卖票
    while(1){
        //加锁
        pthread_mutex_lock(&mutex);
        if(tickets > 0){
            usleep(6000);
            printf("%ld 正在卖第 %d张门票\n", pthread_self, tickets);
            tickets--;
        }else{
            pthread_mutex_unlock(&mutex);
            break;
        }
        //解锁 
        pthread_mutex_unlock(&mutex);
    }


    return NULL;
}
int main(){
    //初始化互斥量
    pthread_mutex_init(&mutex, NULL);

    //创建3个子线程
    pthread_t tid1, tid2, tid3;
    pthread_create(&tid1, NULL, sellTicket, NULL);
    pthread_create(&tid2, NULL, sellTicket, NULL);
    pthread_create(&tid3, NULL, sellTicket, NULL);
    //回收子线程的资源
    pthread_join(tid1, NULL);
    pthread_join(tid2, NULL);
    pthread_join(tid3, NULL);

    // //设置线程分离
    // pthread_detach(tid1);
    // pthread_detach(tid2);
    // pthread_detach(tid3);

    pthread_exit(NULL); //退出主线程

    //释放互斥量资源
    pthread_mutex_destroy(&mutex);
}
```
#### 死锁
![[死锁.png]]

#### 读写锁
当有一个线程已经持有互斥锁时，互斥锁将所有试图进入临界区的线程都阻塞住。但是考虑一种情形，当前持有互斥锁的线程只是要读访问共享资源，而同时有其它几个线程也想读取这个共享资源，但是由于互斥锁的排它性，所有其它线程都无法获取锁，也就无法读访问共享资源了，但是实际上多个线程同时读访问共享资源并不会导致问题。

在对数据的读写操作中，更多的是读操作，写操作较少，例如对数据库数据的读写应用。
为了满足当前能够允许多个读出，但只允许一个写入的需求，线程提供了读写锁来实现。

读写锁的特点：
口 如果有其它线程读数据，则允许其它线程执行读操作，但不允许写操作。
口 如果有其它线程写数据，则其它线程都不允许读、写操作。
口写是独占的，写的优先级高。

读写锁相关操作函数
![[读写锁相关操作函数.png]]
```c
//案例:8个线程操作同一个全局变量
//3个线程不定时写这个全局变量,5个线程不定时读这个全局变量 

#include <stdio.h>
#include <pthread.h>
#include <unistd.h>
int num = 1;

pthread_mutex_t mutex;
void *writeNum(void *arg){
    while(1){
        pthread_mutex_lock(&mutex);
        num++;
        printf("++write, tid: %ld, num: %d\n", pthread_self(), num);
        pthread_mutex_unlock(&mutex);
        usleep(100);
    }
    return NULL;
}
void *readNum(void *arg){
    while(1){
        pthread_mutex_lock(&mutex);
        printf("==read, tid: %ld, num: %d\n", pthread_self(), num);
        pthread_mutex_unlock(&mutex);
        usleep(100);
    }
    return NULL;
}
int main(){
    pthread_mutex_init(&mutex, NULL);

    pthread_t wtids[3], rtids[5];
    for(int i = 0; i < 3; i++){
        pthread_create(&wtids[i], NULL, writeNum, NULL);
    }
    for(int i = 0; i < 5; i++){
        pthread_create(&rtids[i], NULL, readNum, NULL);
    }
    //设置线程分离
    for(int i = 0; i < 3; i++){
        pthread_detach(wtids[i]);
    }
    for(int i = 0; i < 5; i++){
        pthread_detach(rtids[i]);
    }
    pthread_exit(NULL);

    pthread_mutex_destroy(&mutex);
    return 0;
}
```
用互斥量也行，但是read也会阻塞，为了提高效率，让读能并发，需要使用读写锁
```c
//案例:8个线程操作同一个全局变量
//3个线程不定时写这个全局变量,5个线程不定时读这个全局变量 

#include <stdio.h>
#include <pthread.h>
#include <unistd.h>
int num = 1;
// pthread_mutex_t mutex;
pthread_rwlock_t rwlock;
void *writeNum(void *arg){
    while(1){
        // pthread_mutex_lock(&mutex);
        pthread_rwlock_wrlock(&rwlock);
        num++;
        printf("++write, tid: %ld, num: %d\n", pthread_self(), num);
        // pthread_mutex_unlock(&mutex);
        pthread_rwlock_unlock(&rwlock);
        usleep(100);
    }
    return NULL;
}
void *readNum(void *arg){
    while(1){
        // pthread_mutex_lock(&mutex);
        pthread_rwlock_rdlock(&rwlock);
        printf("==read, tid: %ld, num: %d\n", pthread_self(), num);
        // pthread_mutex_unlock(&mutex);
        pthread_rwlock_unlock(&rwlock);
        usleep(100);
    }
    return NULL;
}
int main(){
    // pthread_mutex_init(&mutex, NULL);
    pthread_rwlock_init(&rwlock, NULL);
    pthread_t wtids[3], rtids[5];
    for(int i = 0; i < 3; i++){
        pthread_create(&wtids[i], NULL, writeNum, NULL);
    }
    for(int i = 0; i < 5; i++){
        pthread_create(&rtids[i], NULL, readNum, NULL);
    }
    //设置线程分离
    for(int i = 0; i < 3; i++){
        pthread_detach(wtids[i]);
    }
    for(int i = 0; i < 5; i++){
        pthread_detach(rtids[i]);
    }
    pthread_exit(NULL);

    // pthread_mutex_destroy(&mutex);
    pthread_rwlock_destroy(&rwlock);
    return 0;
}
```

生产者-消费者模型
![[生产者消费者模型.png]]
最开始的代码是判断如果head为空，就不停执行while循环，但这会导致，消费者没数据了仍然不断循环检查。更好的方式是消费者发现没数据了，让生产者去生产(条件变量解决)
```c
#include <stdio.h>
#include <pthread.h>
#include <unistd.h>
#include <stdlib.h>

//创建1个互斥量解决线程同步的问题
pthread_mutex_t mutex;
struct Node{
    int num;
    struct Node *next;
};
//头节点
struct Node *head = NULL;
void *producer(void *arg){
    //不断创建新的节点添加到链表中
    while(1){
       pthread_mutex_lock(&mutex);
       struct Node *newNode = (struct Node *) malloc(sizeof(struct Node));
       newNode->next = head;
       head = newNode;
       newNode->num = rand() % 1000;
       printf("add node, num :%d, tid: %ld\n", newNode->num, pthread_self());
       pthread_mutex_unlock(&mutex);
       usleep(100);
    }
    return NULL;
}
void *customer(void *arg){
    while(1){
        pthread_mutex_lock(&mutex);
        //保存头节点的指针
        struct Node *tmp = head;
        //判断是否有数据
        if(head!=NULL){
            head = head->next;
            printf("del node, num: %d, tid : %ld\n", tmp->num, pthread_self());
            free(tmp);
            pthread_mutex_unlock(&mutex);
            usleep(100);
        }else{
            //没有数据
            pthread_mutex_unlock(&mutex);
        }

    }
    return NULL;
}

int main(){
    pthread_mutex_init(&mutex, NULL);

    //创建5个生产者线程和5个消费者线程
    pthread_t ptids[5], ctids[5];
    for(int i = 0; i < 5; i++){
        pthread_create(&ptids[i], NULL, producer, NULL);
        pthread_create(&ctids[i], NULL, customer, NULL);
    }
    for(int i = 0; i < 5; i++){
        pthread_detach(ptids[i]);
        pthread_detach(ctids[i]);
    }

    //防止主线程把互斥量销毁了,如果是连接的话就不需要写这个,因为连接会阻塞直到子线程销毁
    while(1){
        sleep(10);
    }
    pthread_mutex_destroy(&mutex);
    pthread_exit(NULL);


    return 0;
}
```
#### 条件变量
![[条件变量.png]]
```c
int pthread_cond_wait(pthread_cond_t *cond, pthread_mutex_t *mutex);
//等待，调用了该函数，线程会阻塞
pthread_cond_timewait(..)
//等待多长时间，调用了这个函数线程会阻塞，直到指定的时间结束
pthread_cond_signal(...)
//唤醒一个或者多个等待的线程
pthread_cond_broadcast(...)
//唤醒所有等待的线程 
```
用条件变量改进生产者-消费者模型
```c
#include <stdio.h>
#include <pthread.h>
#include <unistd.h>
#include <stdlib.h>

//创建1个互斥量解决线程同步的问题
pthread_mutex_t mutex;
//创建条件变量
pthread_cond_t cond;

struct Node{
    int num;
    struct Node *next;
};
//头节点
struct Node *head = NULL;
void *producer(void *arg){
    //不断创建新的节点添加到链表中
    while(1){
       pthread_mutex_lock(&mutex);
       struct Node *newNode = (struct Node *) malloc(sizeof(struct Node));
       new ->next = head;
       head = newNode;
       newNode->num = rand() % 1000;
       printf("add node, num :%d, tid: %ld\n", newNode->num, pthread_self());
       //只要生产了一个，就通知消费者消费
       pthread_cond_signal(&cond);
       pthread_mutex_unlock(&mutex);
       usleep(100);
    }
    return NULL;
}
void *customer(void *arg){
    while(1){
        pthread_mutex_lock(&mutex);
        //保存头节点的指针
        struct Node *tmp = head;
        //判断是否有数据
        if(head!=NULL){
            head = head->next;
            printf("del node, num: %d, tid : %ld\n", tmp->num, pthread_self());
            free(tmp);
            pthread_mutex_unlock(&mutex);
            usleep(100);
        }else{
            //没有数据,需要等待
            //cond_wait在阻塞的时候会解锁，阻塞结束的时候上锁
            pthread_cond_wait(&cond, &mutex);
            pthread_mutex_unlock(&mutex);
        }

    }
    return NULL;
}

int main(){
    pthread_mutex_init(&mutex, NULL);
    pthread_cond_init(&cond, NULL);
    //创建5个生产者线程和5个消费者线程
    pthread_t ptids[5], ctids[5];
    for(int i = 0; i < 5; i++){
        pthread_create(&ptids[i], NULL, producer, NULL);
        pthread_create(&ctids[i], NULL, customer, NULL);
    }
    for(int i = 0; i < 5; i++){
        pthread_detach(ptids[i]);
        pthread_detach(ctids[i]);
    }

    //防止主线程把互斥量销毁了,如果是连接的话就不需要写这个,因为连接会阻塞直到子线程销毁
    while(1){
        sleep(10);
    }
    pthread_mutex_destroy(&mutex);
    pthread_cond_destroy(&cond);
    pthread_exit(NULL);
    return 0;
}
```
#### 信号量 
必须要和互斥锁共同使用才能保证线程安全
![[信号量操作.png]]
```c
sem_init(...)
//初始化信号量
//参数:sem信号量变量的地址, pshared 0用在线程间,非0用在进程间
//value:信号量的值
sem_destory(...)
//释放资源
sem_wait(...)
//对信号量加锁,值-1,如果值为0就阻塞 
sem_post(...)
//对信号量解锁,对信号量的值+1
```
信号量改进生产者消费者模型
```
sem_t psem;
sem_t csem;
init(psem, 0, 8);
init(csem, 0, 0);
producer(){
	sem_wait(&psem);
	sem_post(&csem);
}
customer(){
	sem_wait(&csem);
	sem_post(&psem);
}
```
代码示例
```c
#include <stdio.h>
#include <pthread.h>
#include <unistd.h>
#include <stdlib.h>
#include <semaphore.h>
//创建1个互斥量解决线程同步的问题
pthread_mutex_t mutex;
//创建2个信号量
sem_t psem;
sem_t csem;

struct Node{
    int num;
    struct Node *next;
};
//头节点
struct Node *head = NULL;
void *producer(void *arg){
    //不断创建新的节点添加到链表中
    while(1){
        sem_wait(&psem);
       pthread_mutex_lock(&mutex);
       struct Node *newNode = (struct Node *) malloc(sizeof(struct Node));
       newNode->next = head;
       head = newNode;
       newNode->num = rand() % 1000;
       printf("add node, num :%d, tid: %ld\n", newNode->num, pthread_self());
       //只要生产了一个，就通知消费者消费
       pthread_mutex_unlock(&mutex);
       sem_post(&csem);
    //    usleep(100);
    }
    return NULL;
}

void *customer(void *arg){
    while(1){
        sem_wait(&csem);
        pthread_mutex_lock(&mutex);
        //保存头节点的指针
        struct Node *tmp = head;
        //判断是否有数据
        printf("1\n");
        head = head->next;
        printf("del node, num: %d, tid : %ld\n", tmp->num, pthread_self());
        free(tmp);
        pthread_mutex_unlock(&mutex);
        sem_post(&psem);
        // usleep(100);
    }
    return NULL;
}

int main(){
    pthread_mutex_init(&mutex, NULL);
    sem_init(&psem, 0, 8);
    sem_init(&csem, 0, 0);
    //创建5个生产者线程和5个消费者线程
    pthread_t ptids[5], ctids[5];
    for(int i = 0; i < 5; i++){
        pthread_create(&ptids[i], NULL, producer, NULL);
        pthread_create(&ctids[i], NULL, customer, NULL);
    }
    for(int i = 0; i < 5; i++){
        pthread_detach(ptids[i]);
        pthread_detach(ctids[i]);
    }

    //防止主线程把互斥量销毁了,如果是连接的话就不需要写这个,因为连接会阻塞直到子线程销毁
    while(1){
        sleep(10);
    }
    pthread_mutex_destroy(&mutex);

    pthread_exit(NULL);
    return 0;
}
```

## Linux网络编程
TCP三次握手，三次握手，服务端需要确定自己能发能收，也需要确定客户端能发能收；客户端需要确定自己能发能收，也需要确定服务端能发能收，需要至少4次，但是ack和syn合并在一起，所以三次握手

```c
listenfd = socket();   // 打开一个网络通信端口
bind(listenfd);        // 绑定
listen(listenfd);      // 监听
while(1) {
  connfd = accept(listenfd);  // 阻塞建立连接
  pid_t pid = fork(); //创建子进程
  if(pid == 0){
	  //子进程
	  int n = recv(connfd, buf);  // 阻塞读数据
	  doSomeThing(buf);  // 利用读到的数据做些什么	
	  close(connfd);     // 关闭连接，循环等待下一个连接  
  }else{
	  //父进程
	  doOtherThing();
  }
}
```

```c
listenfd = socket();   // 打开一个网络通信端口
bind(listenfd);        // 绑定
listen(listenfd);      // 监听
while(1) {
  connfd = accept(listenfd);  // 阻塞建立连接
  pthread_create（doWork, connfd);  // 创建一个子线程
}

void doWork(connfd) {
  int n = recv(connfd, buf);  // 阻塞读数据
  doSomeThing(buf);  // 利用读到的数据做些什么
  close(connfd);     // 关闭连接，循环等待下一个连接
}
```

```c
listenfd = socket();   // 打开一个网络通信端口
listenfd.setblocking(False); //设为非阻塞
bind(listenfd);        // 绑定
listen(listenfd);      // 监听
conn_list = []; //存储建立连接的conn_socket
while(1) {
	try:
		connfd = accept(listenfd);  // 阻塞建立连接
	    connfd.setblocking(False); //设为非阻塞
	    conn_list.add(connfd); //加入到conn_list
	except:
	    pass
	    
	// 遍历处理建立连接的conn_socket
	for conn_fd in conn_list:
		try:
			data = recv(conn_fd, buf);
			doSomething(buf);
			close(conn_fd);
			remove(conn_list, conn_fd);
		except:
			pass  
}
```

```c
listenfd = socket();   // 打开一个网络通信端口
bind(listenfd);        // 绑定
listen(listenfd);      // 监听
inputs = [listenfd];  //要监听读事件的文件描述符数组
outpus = []; //要监听写事件的文件描述符数组
exceptions = []; //要监听异常事件的文件描述符数组
while(1) {
	r, w, e = select(inputs, outputs, exceptions);
	// 遍历可读的fd
	for fd in r:
		// 有客户端连接到来
		if fd == listenfd:
			connfd = accept(listenfd);
			inputs.append(connfd);
		// 有fd可以进行读操作
		else:
			data = recv(fd, buf);
			doSomething(buf);
			outputs.append(fd);
			
	//遍历可写的fd
	for fd in w:
		...
	//遍历有异常的fd
	for fd in e:
		...
	
}
```




