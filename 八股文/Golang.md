### channel实现once
利用缓冲区大小为1的channel
```go
type Once struct {
	ch chan struct{}
}

func NewOnce() Once {
	// 无缓冲channel
	once := Once{
		ch: make(chan struct{}, 1),
	}
	// 只注入一个元素
	once.ch <- struct{}{}
	return once
}

func (once Once) Do(f func()) {
	_, ok := <-once.ch
	if !ok {
		return
	}
	f()
	close(once.ch)
	return
}
```
### channel实现mutex
利用缓冲区大小为1的channel
```go
type Semaphore struct {
	ch chan struct{}
}

func NewSemaphore(size int) Semaphore {
	s := Semaphore{
		ch: make(chan struct{}, size),
	}
	return s
}

func (s Semaphore) Lock() {
	s.ch <- struct{}{}
}

func (s Semaphore) UnLock() {
	<-s.ch
}
```

### 描述下什么是GMP？
见mindnode

### 描述下Go内存模型
见mindnode

### 如何申请0空间
```go
func mallocgc(size uintptr, typ *_type, needzero bool) unsafe.Pointer {                        
	// ……（省略部分代码）
	if size == 0 {
	return unsafe.Pointer(&zerobase)
	}
	//……（省略部分代码）
}
```
如果申请的size为0，则直接return一个固定地址zerobase。在Golang中如\[0]int、 struct{}所需要大小均是0，这也是为什么很多开发者在通过Channel做同步时，发送一个struct{}数据，因为不会申请任何内存，能够适当节省一部分内存空间

### 描述一下内存逃逸
即使是用 `new` 申请的内存，如果编译器发现 `new` 出来的内存在函数结束后就没有使用了且申请内存空间不是很大，那么 `new` 申请的内存空间还是会被分配在栈上，毕竟栈访问速度更快且易与管理

第一种情况**变量在函数外部没有引用，优先放到栈中**。最典型的例子就是刚刚说的 `new` 内存分配的问题，当 `new` 出来的内存空间没有被外部引用，且申请的内存不是很大时就会被放在栈上而不是堆上

第二种情况**变量在函数外部存在引用，必定放在堆中**

第三种情况**超过 64k 的内存占用放到堆上**
### 栈为什么比堆快
分配上，栈只需要移动指针，gmp的g的数据结构存储了栈相关寄存器的信息
堆需要加锁
释放，栈需要移动指针
堆需要gc
查找**堆**的链表也会耗费较多时间，所以存储寻址速度慢
CPU硬件操作速度**快**：cpu有专门的寄存器（esp，ebp）来操作**栈**，**堆**是使用间接寻址的
