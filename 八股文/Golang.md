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