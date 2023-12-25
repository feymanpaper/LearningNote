https://mp.weixin.qq.com/s/jIWe3nMP6yiuXeBQgmePDg
## Introduction
线程
通常语义中的线程，指的是内核级线程，核心点如下：
（1）是操作系统最小调度单元；
（2）创建、销毁、调度交由内核完成，cpu 需完成用户态与内核态间的切换；
（3）可充分利用多核，实现并行.

协程(coroutine)，又称为用户级线程，核心点如下：
（1）与线程存在映射关系，为 M：1；
（2）创建、销毁、调度在用户态完成，对内核透明，所以更轻；
（3）从属同一个内核级线程，无法并行；一个协程阻塞会导致从属同一线程的所有协程无法执行.

Goroutine，经 Golang 优化后的特殊“协程”，核心点如下：
相对于coroutine, 中间加了一层调度器实现协程和线程的动态绑定
（1）与线程存在映射关系，为 M：N；
（2）创建、销毁、调度在用户态完成，对内核透明，足够轻便；
（3）可利用多个线程，实现并行；
（4）通过调度器的斡旋，实现和线程间的**动态绑定**和灵活调度；
（5）栈空间大小可动态扩缩，因地制宜.(coroutine是固定的)
![[Pasted image 20231219081641.png]]
![[Pasted image 20231219082043.png]]

Golang 在调度 goroutine 时，针对“如何减少加锁行为”，“如何避免资源不均”等问题都给出了精彩的解决方案，这一切都得益于经典的 “gmp” 模型.

## GMP
GMP = goroutine + machine + processor

goroutine:
（1）g 即goroutine，是 golang 中对协程的抽象；
（2）g 有自己的运行栈、状态、以及执行的任务函数（用户通过 go func 指定）；
（3）g 需要绑定到 p 才能执行，在 g 的视角中，p 就是它的 cpu.

processor:
（1）p 即 processor，是 golang 中的调度器；
（2）p 是 gmp 的中枢，借由 p 承上启下，实现 g 和 m 之间的动态有机结合；
（3）对 g 而言，p 是其 cpu，g 只有被 p 调度，才得以执行；
（4）对 m 而言，p 是其执行代理，为其提供必要信息的同时（可执行的 g、内存分配情况等），并隐藏了繁杂的调度细节；
（5）p 的数量决定了 g 最大并行数量，可由用户通过 GOMAXPROCS 进行设定（超过 CPU 核数时无意义）.

machine:
（1）m 即 machine，是 golang 中对线程的抽象；
（2）m 不直接执行 g，而是先和 p 绑定，由其实现代理；
（3）借由 p 的存在，m 无需和 g 绑死，也无需记录 g 的状态信息，因此 g 在全生命周期中可以实现跨 m 执行.
![](Pasted%20image%2020231219083101.png)
GMP 宏观模型如上图所示，下面对其要点和细节进行逐一介绍：
（1）M 是线程的抽象；G 是 goroutine；P 是承上启下的调度器；
（2）M调度G前，需要和P绑定；
（3）全局有多个M和多个P，但同时并行的G的最大数量等于P的数量；
（4）G的存放队列有三类：P的本地队列；全局队列；和wait队列（图中未展示，为io阻塞就绪态goroutine队列）；
（5）M调度G时，优先取P本地队列，其次取全局队列，最后取wait队列；这样的好处是，取本地队列时，可以接近于无锁化，减少全局锁竞争；
（6）为防止不同P的闲忙差异过大，设立work-stealing机制，本地队列为空的P可以尝试从其他P本地队列偷取一半的G补充到自身队列.

## 核心数据结构
g数据结构
```go
type g struct {
    // ...
    m         *m   //m：在 p 的代理，负责执行当前 g 的 m；   
    // ...
    sched     gobuf
    // ...
}
type gobuf struct {
    sp   uintptr //保存 CPU 的 rsp 寄存器的值，指向函数调用栈栈顶；
    pc   uintptr //保存 CPU 的 rip 寄存器的值，指向程序下一条执行指令的地址；
    ret  uintptr //保存系统调用的返回值；
    bp   uintptr // for framepointer-enabled architectures //保存 CPU 的 rbp 寄存器的值，存储函数栈帧的起始位置.
}
```

![](Pasted%20image%2020231219083624.png)
```go
const(
  _Gidle = itoa // 0 //为协程开始创建时的状态，此时尚未初始化完成；
  _Grunnable // 1 //协程在待执行队列中，等待被执行；
  _Grunning // 2 //协程正在执行，同一时刻一个 p 中只有一个 g 处于此状态；
  _Gsyscall // 3 //协程正在执行系统调用
  _Gwaiting // 4 //协程处于挂起态，需要等待被唤醒. gc、channel 通信或者锁操作时经常会进入这种状态；
  _Gdead // 6 It means that currentily this goroutine is unused. 协程刚初始化完成或者已经被销毁，会处于此状态；
  _Gcopystack // 8 //协程正在栈扩容流程中；
  _Gpreempted // 9 //协程被抢占后的状态.
)
```

m数据结构
```go
type m struct {
    g0      *g     // goroutine with scheduling stack //g0：一类特殊的调度协程，不用于执行用户函数，负责执行 g 之间的切换调度. 与 m 的关系为 1:1；
    // ...
    tls           [tlsSlots]uintptr // thread-local storage (for x86 extern register) 
    //tls：thread-local storage，线程本地存储，存储内容只对当前线程可见. 线程本地存储的是 m.tls 的地址，m.tls[0] 存储的是当前运行的 g，因此线程可以通过 g 找到当前的 m、p、g0 等信息.
    // ...
    p puintptr // attached p for executing go code (nil if not executing go code)
    curg *g // current running goroutine
}
```

p数据结构
```go
type p struct {
    // ...
    runqhead uint32 //队列头部；
    runqtail uint32 //队列尾部
    runq     [256]guintptr //runq：本地 goroutine 队列，最大长度为 256.
    
    runnext guintptr //runnext：下一个可执行的 goroutine.
    // ...
    m muintptr // back-link to associated m (nil if idle)
}
```

全局队列
```go
type schedt struct {
    // ...
    lock mutex //lock：一把操作全局队列时使用的互斥锁；
    // ... 
    runq     gQueue //runq：全局 goroutine 队列；
    runqsize int32 //runqsize：全局 goroutine 队列的容量.
    // ...
}
```
## 调度流程
goroutine 的类型可分为两类：
I 负责调度普通 g 的 g0，执行固定的调度流程，与 m 的关系为一对一；
II 负责执行用户函数的普通 g.

m 通过 p 调度执行的 goroutine 永远在普通 g 和 g0 之间进行切换，当 g0 找到可执行的 g 时，会调用 gogo 方法，调度 g 执行用户定义的任务；当 g 需要主动让渡或被动调度时，会触发 mcall 方法，将执行权重新交还给 g0.
```go
func gogo(buf *gobuf)
// ...
func mcall(fn func(*g))
```
![](Pasted%20image%2020231219101538.png)

（1）主动调度
一种用户主动执行让渡的方式，主要方式是，用户在执行代码中调用了 runtime.Gosched 方法，此时当前 g 会当让出执行权，主动进行队列等待下次被调度执行.
```go
func Gosched() {    checkTimeouts()    mcall(gosched_m)}
```
（2）被动调度
因当前不满足某种执行条件，g 可能会陷入阻塞态无法被调度，直到关注的条件达成后，g才从阻塞中被唤醒，重新进入可执行队列等待被调度.
常见的被动调度触发方式为因 channel 操作或互斥锁操作陷入阻塞等操作，底层会走进 gopark 方法.
```go
func gopark(unlockf func(*g, unsafe.Pointer) bool, lock unsafe.Pointer, reason waitReason, traceEv byte, traceskip int) {
    // ...
    mcall(park_m)
}
```
goready 方法通常与 gopark 方法成对出现，能够将 g 从阻塞态中恢复，重新进入等待执行的状态.
```go
func goready(gp *g, traceskip int) {
    systemstack(func() {
        ready(gp, traceskip, true)
    })
}
```
（3）正常调度：
g 中的执行任务已完成，g0 会将当前 g 置为死亡状态，发起新一轮调度.
（4）抢占调度：
倘若 g 执行系统调用超过指定的时长，且全局的 p 资源比较紧缺，此时将 p 和 g 解绑，抢占出来用于其他 g 的调度. 等 g 完成系统调用后，会重新进入可执行队列中等待被调度.
值得一提的是，前 3 种调度方式都由 m 下的 g0 完成，唯独抢占调度不同.
因为发起系统调用时需要打破用户态的边界进入内核态，此时 m 也会因系统调用而陷入僵直，无法主动完成抢占调度的行为.
因此，在 Golang 进程会有一个全局监控协程 monitor g 的存在，这个 g 会越过 p 直接与一个 m 进行绑定，不断轮询对所有 p 的执行状况进行监控. 倘若发现满足抢占调度的条件，则会从第三方的角度出手干预，主动发起该动作.

![](Pasted%20image%2020231219101757.png)

（1）以 g0 -> g -> g0 的一轮循环为例进行串联；
（2）g0 执行 schedule() 函数，寻找到用于执行的 g；
（3）g0 执行 execute() 方法，更新当前 g、p 的状态信息，并调用 gogo() 方法，将执行权交给 g；
（4）g 因主动让渡( gosche_m() )、被动调度( park_m() )、正常结束( goexit0() )等原因，调用 m_call 函数，执行权重新回到 g0 手中；
（5）g0 执行 schedule() 函数，开启新一轮循环.

```go
func schedule() {
    // ...
    gp, inheritTime, tryWakeP := findRunnable() // blocks until work is available
    // ...
    execute(gp, inheritTime)
}
```

如何寻找到用于执行的 g, 对应findRunnable
![](Pasted%20image%2020231219102425.png)
（1）p 每执行 61 次调度，会从全局队列中获取一个 goroutine 进行执行，并将一个全局队列中的 goroutine 填充到当前 p 的本地队列中.
除了获取一个 g 用于执行外，还会额外将一个 g 从全局队列转移到 p 的本地队列，让全局队列中的 g 也得到更充分的执行机会.
I 取得 p 本地队列队首的索引，同时对本地队列加锁：
II 倘若 p 的局部队列未满，则成功转移 g，将 p 的对尾索引 runqtail 值加 1 并解锁队列.
III 倘若发现本地队列 runq 已经满了，则会返回来将本地队列中一半的 g 放回全局队列中，帮助当前 p 缓解执行压力，这部分内容位于 runqputslow 方法中.
（2）尝试从 p 本地队列中获取一个可执行的 goroutine
I 倘若当前 p 的 runnext 非空，直接获取即可：
II 加锁从 p 的本地队列中获取 g.
需要注意，虽然本地队列是属于 p 独有的，但是由于 work-stealing 机制的存在，其他 p 可能会前来执行窃取动作，因此操作仍需加锁.
但是，由于窃取动作发生的频率不会太高，因此当前 p 取得锁的成功率是很高的，因此可以说p 的本地队列是接近于无锁化，但没有达到真正意义的无锁.
III 倘若本地队列为空，直接终止并返回；
IV 倘若本地队列存在 g，则取得队首的 g，解锁并返回.
（3）倘若本地队列没有可执行的 g，会从全局队列中获取：加锁，尝试并从全局队列中取队首的元素.
（4）倘若本地队列和全局队列都没有 g，则会获取准备就绪的网络协程：需要注意的是，刚获取网络协程时，g 的状态是处于 waiting 的，因此需要先更新为 runnable 状态.
（5）work-stealing: 从其他 p 中偷取 g.偷取操作至多会遍历全局的 p 队列 4 次，过程中只要找到可窃取的 p 则会立即返回.为保证窃取行为的公平性，遍历的起点是随机的. 窃取动作的核心逻辑位于 runqgrab 方法当中：
I 每次对一个 p 尝试窃取前，会对其局部队列加锁；
II 尝试偷取其现有的一半 g，并且返回实际偷取的数量.

开始执行g, 对应execute
![](Pasted%20image%2020231219103248.png)
当 g0 为 m 寻找到可执行的 g 之后，接下来就开始执行 g. 
（1）更新 g 的状态信息，建立 g 与 m 之间的绑定关系；
（2）更新 p 的总调度次数；
（3）调用 gogo 方法，执行 goroutine 中的任务.

主动让渡gosched_m
g 执行主动让渡时，会调用 mcall 方法将执行权归还给 g0，并由 g0 调用 gosched_m 方法，
（1）将当前 g 的状态由执行中切换为待执行 _Grunnable：
（2）调用 dropg() 方法，将当前的 m 和 g 解绑；
（3）将 g 添加到全局队列当中：
（4）开启新一轮的调度：

被动让渡park_m 与 ready
g 需要被动调度时，会调用 mcall 方法切换至 g0，并调用 park_m 方法将 g 置为阻塞态，执行流程位于 runtime/proc.go 的 gopark 方法当中：
（1）将当前 g 的状态由 running 改为 waiting；
（2）将 g 与 m 解绑；
（3）执行新一轮的调度 schedule.
当因被动调度陷入阻塞态的 g 需要被唤醒时，会由其他协程执行 goready 方法将 g 重新置为可执行的状态
被动调度如果需要唤醒，则会其他 g 负责将 g 的状态由 waiting 改为 runnable，然后会将其添加到唤醒者的 p 的本地队列中：
（1）先将 g 的状态从阻塞态改为可执行的状态；
（2）调用 runqput 将当前 g 添加到唤醒者 p 的本地队列中，如果队列满了，会连带 g 一起将一半的元素转移到全局队列.

正常调度goexit0
当 g 执行完成时，会先执行 mcall 方法切换至 g0，然后调用 goexit0 方法，内容为 
（1）将 g 状态置为 dead；
（2）解绑 g 和 m；
（3）开启新一轮的调度.

抢占调度retake
![](Pasted%20image%2020231219110040.png)
抢占调度的执行者不是 g0，而是一个全局的 monitor g
（1）加锁后，遍历全局的 p 队列，寻找需要被抢占的目标：
（2）倘若某个 p 同时满足下述条件，则会进行抢占调度：
I 执行系统调用超过 10 ms；
II p 本地队列有等待执行的 g；
III 或者当前没有空闲的 p 和 m.
（3）抢占调度的步骤是，先将当前 p 的状态更新为 idle，然后步入 handoffp 方法中，判断是否需要为 p 寻找接管的 m（因为其原本绑定的 m 正在执行系统调用）：
（4）当以下四个条件满足其一时，则需要为 p 获取新的 m：
I 当前 p 本地队列还有待执行的 g；
II 全局繁忙（没有空闲的 p 和 m，全局 g 队列为空）
III 需要处理网络 socket 读写请求
（5）获取 m 时，会先尝试获取已有的空闲的 m，若不存在，则会创建一个新的 m.

系统调用前reentersyscall
同样与 g 的系统调用有关，但是视角切换回发生系统调用前，与 g 绑定的原 m 当中.
在 m 需要执行系统调用前，会先执行位于 runtime/proc.go 的 reentersyscall 的方法：
（1）此时执行权同样位于 m 的 g0 手中；
（2）保存当前 g 的执行环境；
（3）将 g 和 p 的状态更新为 syscall；
（4）解除 p 和 当前 m 之间的绑定，因为 m 即将进入系统调用而导致短暂不可用；
（5）将 p 添加到 当前 m 的 oldP 容器当中，后续 m 恢复后，会优先寻找旧的 p 重新建立绑定关系.

系统调用后exitsyscall
当 m 完成了内核态的系统调用之后，此时会步入位于 runtime/proc.go 的 exitsyscall 函数中，尝试寻找 p 重新开始运作：
（1）方法执行之初，此时的执行权是普通 g.倘若此前设置的 oldp 仍然可用，则重新和 oldP 绑定，将当前 g 重新置为 running 状态，然后开始执行后续的用户函数；
（2）old 绑定失败，则调用 mcall 方法切换到 m 的 g0，并执行 exitsyscall0 方法：
（3）将 g 由系统调用状态切换为可运行态，并解绑 g 和 m 的关系：
（4）从全局 p 队列获取可用的 p，如果获取到了，则执行 g：
（5）如若无 p 可用，则将 g 添加到全局队列，当前 m 陷入沉睡. 直到被唤醒后才会继续发起调度.
