https://mp.weixin.qq.com/s/QgNndPgN1kqxWh-ijSofkw
一道考题(TODO)
要求实现一个map：
(1）面向高并发；
(2） 只存在插入和查询操作 0(1)；
(3） 查询时，若 key 存在，直接返回 val;若 key 不存在，阻塞直到 key val 对被放入后，获取 val 返回;
等待指定时长仍未放入，返回超时错误；
（4）写出真实代码，不能有死锁或者 panic 风险

```go
type MyConcurrentMap struct{
// ...
}

func (m *MyConcurrentMap) Put (k, v int) {

}

func (m *MyConcurrentMap)Get(k int, maxWaitingDuration time.Duration) (int,error) {

}
```

### channel数据结构
![](Pasted%20image%2020231219130119.png)
hchan数据结构
```go
type hchan struct {
    qcount   uint           // total data in the queue
    dataqsiz uint           // size of the circular queue
    buf      unsafe.Pointer // points to an array of dataqsiz elements
    elemsize uint16
    closed   uint32
    elemtype *_type // element type
    sendx    uint   // send index
    recvx    uint   // receive index
    recvq    waitq  // list of recv waiters //因接收而陷入阻塞的协程队列；
    sendq    waitq  // list of send waiters //因发送而陷入阻塞的协程队列；
    lock mutex
}
```

sudog：用于包装协程的节点
```go
type sudog struct {
    g *g //goroutine
    next *sudog
    prev *sudog
    elem unsafe.Pointer // data element (may point to stack)
    isSelect bool //标识当前协程是否处在 select 多路复用的流程中
    c        *hchan  //标识与当前 sudog 交互的 chan
}
```

waitq：阻塞的协程队列
```go
type waitq struct {
    first *sudog
    last  *sudog
}
```

### 构造器函数
![](Pasted%20image%2020231219152759.png)

```go
func makechan(t *chantype, size int) *hchan {
    elem := t.elem
    // ...
    mem, overflow := math.MulUintptr(elem.size, uintptr(size))
    if overflow || mem > maxAlloc-hchanSize || size < 0 {
        panic(plainError("makechan: size out of range"))
    }
    var c *hchan
    switch {
    case mem == 0:
        // Queue or element size is zero.
        c = (*hchan)(mallocgc(hchanSize, nil, true))
        // Race detector uses this location for synchronization.
        c.buf = c.raceaddr()
    case elem.ptrdata == 0:
        // Elements do not contain pointers.
        // Allocate hchan and buf in one call.
        c = (*hchan)(mallocgc(hchanSize+mem, nil, true))
        c.buf = add(unsafe.Pointer(c), hchanSize)
    default:
        // Elements contain pointers.
        c = new(hchan)
        c.buf = mallocgc(mem, elem, true)
    }
    c.elemsize = uint16(elem.size)
    c.elemtype = elem
    c.dataqsiz = uint(size)
    lockInit(&c.lock, lockRankHchan)
    return
}
```

• 判断申请内存空间大小是否越界，mem 大小为 element 类型大小与 element 个数相乘后得到，仅当无缓冲型 channel 时，因个数为 0 导致大小为 0；
• 根据类型，初始 channel，分为 无缓冲型、有缓冲元素为 struct 型、有缓冲元素为 pointer 型 channel;
• 倘若为无缓冲型，则仅申请一个大小为默认值 96 的空间；
• 如若有缓冲的 struct 型，则一次性分配好 96 + mem 大小的空间，并且调整 chan 的 buf 指向 mem 的起始位置；
• 倘若为有缓冲的 pointer 型，则分别申请 chan 和 buf 的空间，两者无需连续；
• 对 channel 的其余字段进行初始化，包括元素类型大小、元素类型、容量以及锁的初始化.

### 写流程

case1: 对于未初始化的 chan，写入操作会引发死锁；
case2:对于已关闭的 chan，写入操作会引发 panic.
```go
func chansend1(c *hchan, elem unsafe.Pointer) {
    chansend(c, elem, true, getcallerpc())
}

func chansend(c *hchan, ep unsafe.Pointer, block bool, callerpc uintptr) bool {
    if c == nil {
        gopark(nil, nil, waitReasonChanSendNilChan, traceEvGoStop, 2)
        throw("unreachable")
    }
    lock(&c.lock)
    if c.closed != 0 {
        unlock(&c.lock)
        panic(plainError("send on closed channel"))
    }
    // ...
```
case3:写时存在阻塞读协程
![](Pasted%20image%2020231219153002.png)
```go
func chansend(c *hchan, ep unsafe.Pointer, block bool, callerpc uintptr) bool {
    // ...
    lock(&c.lock)
    // ...
    if sg := c.recvq.dequeue(); sg != nil {
        // Found a waiting receiver. We pass the value we want to send
        // directly to the receiver, bypassing the channel buffer (if any).
        send(c, sg, ep, func() { unlock(&c.lock) }, 3)
        return true
    }
    // ...
```
• 加锁；
• 从阻塞度协程队列中取出一个 goroutine 的封装对象 sudog；
• 在 send 方法中，会基于 memmove 方法，直接将元素拷贝交给 sudog 对应的 goroutine；
• 在 send 方法中会完成解锁动作.

case4:写时无阻塞读协程但环形缓冲区仍有空间
![](Pasted%20image%2020231219153209.png)
```go
func chansend(c *hchan, ep unsafe.Pointer, block bool, callerpc uintptr) bool {
    // ...
    lock(&c.lock)
    // ...
    if c.qcount < c.dataqsiz {
        // Space is available in the channel buffer. Enqueue the element to send.
        qp := chanbuf(c, c.sendx)
        typedmemmove(c.elemtype, qp, ep)
        c.sendx++
        if c.sendx == c.dataqsiz {
            c.sendx = 0
        }
        c.qcount++
        unlock(&c.lock)
        return true
    }
    // ...
}
```
• 加锁；
• 将当前元素添加到环形缓冲区 sendx 对应的位置；
• sendx++;
• qcount++;
• 解锁，返回.

case5:写时无阻塞读协程且环形缓冲区无空间
![](Pasted%20image%2020231219153302.png)

```go
func chansend(c *hchan, ep unsafe.Pointer, block bool, callerpc uintptr) bool {
    // ...
    lock(&c.lock)
    // ...
    gp := getg()
    mysg := acquireSudog()
    mysg.elem = ep
    mysg.g = gp
    mysg.c = c
    gp.waiting = mysg
    c.sendq.enqueue(mysg)
    atomic.Store8(&gp.parkingOnChan, 1)
    gopark(chanparkcommit, unsafe.Pointer(&c.lock), waitReasonChanSend, traceEvGoBlockSend, 2)
    gp.waiting = nil
    closed := !mysg.success
    gp.param = nil
    mysg.c = nil
    releaseSudog(mysg)
    return true
}
```
• 加锁；
• 构造封装当前 goroutine 的 sudog 对象；
• 完成指针指向，建立 sudog、goroutine、channel 之间的指向关系；
• 把 sudog 添加到当前 channel 的阻塞写协程队列中；
• park 当前协程；
• 倘若协程从 park 中被唤醒，则回收 sudog（sudog能被唤醒，其对应的元素必然已经被读协程取走）；
• 解锁，返回

写流程总体串联
![](Pasted%20image%2020231219153434.png)
### 读流程
case1：读空 channel, park 挂起，引起死锁
case2：channel 已关闭且内部无元素, 返回对应类型的0值
```go
func chanrecv(c *hchan, ep unsafe.Pointer, block bool) (selected, received bool) {
    lock(&c.lock)
    if c.closed != 0 {
        if c.qcount == 0 {
            unlock(&c.lock)
            if ep != nil {
                typedmemclr(c.elemtype, ep)
            }
            return true, false
        }
        // The channel has been closed, but the channel's buffer have data.
    } 
    // ...
```
case3：读时有阻塞的写协程
```go
func chanrecv(c *hchan, ep unsafe.Pointer, block bool) (selected, received bool) {
    lock(&c.lock)
    if sg := c.sendq.dequeue(); sg != nil {
        recv(c, sg, ep, func() { unlock(&c.lock) }, 3)
        return true, true
     }
     // ...
}
```
![](Pasted%20image%2020231219154350.png)
• 加锁；
• 从阻塞写协程队列中获取到一个写协程；
• 倘若 channel 无缓冲区，则直接读取写协程元素，并唤醒写协程；
• 倘若 channel 有缓冲区，则读取缓冲区头部元素，并将写协程元素写入缓冲区尾部后唤醒写协程；
• 解锁，返回.

case4：读时无阻塞写协程且缓冲区有元素
• 加锁；
• 获取到 recvx 对应位置的元素；
• recvx++
• qcount--
• 解锁，返回

case5：读时无阻塞写协程且缓冲区无元素
• 加锁；
• 构造封装当前 goroutine 的 sudog 对象；
• 完成指针指向，建立 sudog、goroutine、channel 之间的指向关系；
• 把 sudog 添加到当前 channel 的阻塞读协程队列中；
• park 当前协程；
• 倘若协程从 park 中被唤醒，则回收 sudog（sudog能被唤醒，其对应的元素必然已经被写入）；
• 解锁，返回
![](Pasted%20image%2020231219154743.png)
### 阻塞与非阻塞模式
在上述源码分析流程中，均是以阻塞模式为主线进行讲述，忽略非阻塞模式的有关处理逻辑. 此处阐明两个问题：
• 非阻塞模式下，流程逻辑有何区别？
• 何时会进入非阻塞模式？

非阻塞模式逻辑区别
非阻塞模式下，读/写 channel 方法通过一个 bool 型的响应参数，用以标识是否读取/写入成功.
• 所有需要使得当前 goroutine 被挂起的操作，在非阻塞模式下都会返回 false；
• 所有是的当前 goroutine 会进入死锁的操作，在非阻塞模式下都会返回 false；
• 所有能立即完成读取/写入操作的条件下，非阻塞模式下会返回 true.

何时进入非阻塞模式?
默认情况下，读/写 channel 都是阻塞模式，只有在 select 语句组成的多路复用分支中，与 channel 的交互会变成非阻塞模式：

```go
ch := make(chan int)
select{
  case <- ch:
  default:
}
```

```go
func selectnbsend(c *hchan, elem unsafe.Pointer) (selected bool) {
    return chansend(c, elem, false, getcallerpc())
}
func selectnbrecv(elem unsafe.Pointer, c *hchan) (selected, received bool) {
    return chanrecv(c, elem, false)
}
```
在 select 语句包裹的多路复用分支中，读和写 channel 操作会被汇编为 selectnbrecv 和 selectnbsend 方法，底层同样复用 chanrecv 和 chansend 方法，但此时由于第三个入参 block 被设置为 false，导致后续会走进非阻塞的处理分支.

### 关闭
![](Pasted%20image%2020231219160402.png)

```go
func closechan(c *hchan) {
    if c == nil {
        panic(plainError("close of nil channel"))
    }
    lock(&c.lock)
    if c.closed != 0 {
        unlock(&c.lock)
        panic(plainError("close of closed channel"))
    }
    c.closed = 1
    var glist gList
    // release all readers
    for {
        sg := c.recvq.dequeue()
        if sg == nil {
            break
        }
        if sg.elem != nil {
            typedmemclr(c.elemtype, sg.elem)
            sg.elem = nil
        }
        gp := sg.g
        gp.param = unsafe.Pointer(sg)
        sg.success = false
        glist.push(gp)
    }
    // release all writers (they will panic)
    for {
        sg := c.sendq.dequeue()
        if sg == nil {
            break
        }
        sg.elem = nil
        gp := sg.g
        gp.param = unsafe.Pointer(sg)
        sg.success = false
        glist.push(gp)
    }
    unlock(&c.lock)
    // Ready all Gs now that we've dropped the channel lock.
    for !glist.empty() {
        gp := glist.pop()
        gp.schedlink = 0
        goready(gp, 3)
```

• 关闭未初始化过的 channel 会 panic；
• 加锁；
• 重复关闭 channel 会 panic；
• 将阻塞读协程队列中的协程节点统一添加到 glist；
• 将阻塞写协程队列中的协程节点统一添加到 glist；(注意这些写协程后续都会panic, 在源码chansend方法gopark之后可以找到)
• 唤醒 glist 当中的所有协程.

