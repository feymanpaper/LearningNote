### sync.Locker
sync.Locker 是 go 标准库 sync 下定义的锁接口：
```go
// A Locker represents an object that can be locked and unlocked.
type Locker interface {
    Lock()
    Unlock()
}
```

任何实现了 Lock 和 Unlock 两个方法的类，都可以作为一种锁的实现，最常见的为 go 标准库实现的 sync.Mutex.
在 ants 中，作者不希望使用 Mutex 这种重锁(拿不到锁会gopark使自己进入阻塞，上下文开销相对较大)，而是自定义实现了一种轻量级的自旋锁：
![](Pasted%20image%2020231221112556.png)
```go
type spinLock uint32
const maxBackoff = 16
func (sl *spinLock) Lock() {
    backoff := 1
    //尝试执行CAS操作将锁的状态从0改成1
    for !atomic.CompareAndSwapUint32((*uint32)(sl), 0, 1) {
        for i := 0; i < backoff; i++ {
	        //主动让渡gosched
            runtime.Gosched()
        }
        //能感知到系统繁忙，越忙就不断提高backoff
        if backoff < maxBackoff {
            backoff <<= 1
        }
    }
}
func (sl *spinLock) Unlock() {
	//解锁不需要作任何校验
    atomic.StoreUint32((*uint32)(sl), 0)
```

• 通过一个整型状态值标识锁的状态：0-未加锁；1-加锁；
• 加锁成功时，即把 0 改为 1；解锁时则把 1 改为 0；改写过程均通过 atomic 包保证并发安全；
• 加锁通过 for 循环 + cas 操作实现自旋，无需操作系统介入执行 park 操作；
• 通过变量 backoff 反映抢锁激烈度，每次抢锁失败，执行 backoff 次让 cpu 时间片动作；backoff 随失败次数逐渐升级，封顶 16.
### sync.Cond
