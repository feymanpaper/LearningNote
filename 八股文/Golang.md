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
如果申请的size为0，则直接return一个固定地址zerobase。在Golang中如[0]int、 struct{}所需要大小均是0，这也是为什么很多开发者在通过Channel做同步时，发送一个struct{}数据，因为不会申请任何内存，能够适当节省一部分内存空间
