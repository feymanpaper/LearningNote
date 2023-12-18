https://mp.weixin.qq.com/s/uNajVcWr4mZpof1eNemfmQ

切片中的元素存放在一块内存地址连续的区域，使用索引可以快速检索到指定位置的元素；切片长度和容量是可变的，在使用过程中可以根据需要进行扩容.
```go
type slice struct {
    // 指向起点的地址
    array unsafe.Pointer
    // 切片长度
    len   int
    // 切片容量
    cap   int
}
```

通过 slice 数据结构定义可以看到，每个 slice header 中存放的是内存空间的地址（array 字段），后续在传递切片的时候，相当于是对 slice header 进行了一次值拷贝，但内部存放的地址是相同的，因此对于 slice 本身属于引用传递操作

### 初始化

切片初始化的源码
```go
func makeslice(et *_type, len, cap int) unsafe.Pointer {
    // 根据 cap 结合每个元素的大小，计算出消耗的总容量
    mem, overflow := math.MulUintptr(et.size, uintptr(cap))
    if overflow || mem > maxAlloc || len < 0 || len > cap {
        // 倘若容量超限，len 取负值或者 len 超过 cap，直接 panic
        mem, overflow := math.MulUintptr(et.size, uintptr(len))
        if overflow || mem > maxAlloc || len < 0 {
            panicmakeslicelen()
        }
        panicmakeslicecap()
    }
    // 走 mallocgc 进行内存分配以及切片初始化
    return mallocgc(mem, et, true)
}
```

### 内容截取
在对切片 slice 执行截取操作时，本质上是一次引用传递操作，因为不论如何截取，底层复用的都是同一块内存空间中的数据，只不过，截取动作会创建出一个新的 slice header 实例.
![[Pasted image 20231218200717.png]]
### 切片扩容
当 slice 当前的长度 len 与容量 cap 相等时，下一次 append 操作就会引发一次切片扩容.

```
切片的扩容流程源码位于 runtime/slice.go 文件的 growslice 方法当中，其中核心步骤如下：
• 倘若扩容后预期的新容量小于原切片的容量，则 panic
• 倘若切片元素大小为 0（元素类型为 struct{}），则直接复用一个全局的 zerobase 实例，直接返回
• 倘若预期的新容量超过老容量的两倍，则直接采用预期的新容量
• 倘若老容量小于 256，则直接采用老容量的2倍作为新容量
• 倘若老容量已经大于等于 256，则在老容量的基础上扩容 1/4 的比例并且累加上 192 的数值，持续这样处理，直到得到的新容量已经大于等于预期的新容量为止
• 结合 mallocgc 流程中，对内存分配单元 mspan 的等级制度，推算得到实际需要申请的内存空间大小
• 调用 mallocgc，对新切片进行内存初始化
• 调用 memmove 方法，将老切片中的内容拷贝到新切片中
• 返回扩容后的新切片
```

```go
func growslice(et *_type, old slice, cap int) slice {
    //... 
    if cap < old.cap {
        panic(errorString("growslice: cap out of range"))
    }


    if et.size == 0 {
        // 倘若元素大小为 0，则无需分配空间直接返回
        return slice{unsafe.Pointer(&zerobase), old.len, cap}
    }
    // 计算扩容后数组的容量
    newcap := old.cap
    // 取原容量两倍的容量数值
    doublecap := newcap + newcap
    // 倘若新的容量大于原容量的两倍，直接取新容量作为数组扩容后的容量
    if cap > doublecap {
        newcap = cap
    } else {
        const threshold = 256
        // 倘若原容量小于 256，则扩容后新容量为原容量的两倍
        if old.cap < threshold {
            newcap = doublecap
        } else {
            // 在原容量的基础上，对原容量 * 5/4 并且加上 192
            // 循环执行上述操作，直到扩容后的容量已经大于等于预期的新容量为止
            for 0 < newcap && newcap < cap {             
                newcap += (newcap + 3*threshold) / 4
            }
            // 倘若数值越界了，则取预期的新容量 cap 封顶
            if newcap <= 0 {
                newcap = cap
            }
        }
    }

    var overflow bool
    var lenmem, newlenmem, capmem uintptr
    // 基于容量，确定新数组容器所需要的内存空间大小 capmem
    switch {
    // 倘若数组元素的大小为 1，则新容量大小为 1 * newcap.
    // 同时会针对 span class 进行取整
    case et.size == 1:
        lenmem = uintptr(old.len)
        newlenmem = uintptr(cap)
        capmem = roundupsize(uintptr(newcap))
        overflow = uintptr(newcap) > maxAlloc
        newcap = int(capmem)
    // 倘若数组元素为指针类型，则根据指针占用空间结合元素个数计算空间大小
    // 并会针对 span class 进行取整
    case et.size == goarch.PtrSize:
        lenmem = uintptr(old.len) * goarch.PtrSize
        newlenmem = uintptr(cap) * goarch.PtrSize
        capmem = roundupsize(uintptr(newcap) * goarch.PtrSize)
        overflow = uintptr(newcap) > maxAlloc/goarch.PtrSize
        newcap = int(capmem / goarch.PtrSize)
    // 倘若元素大小为 2 的指数，则直接通过位运算进行空间大小的计算   
    case isPowerOfTwo(et.size):
        var shift uintptr
        if goarch.PtrSize == 8 {
            // Mask shift for better code generation.
            shift = uintptr(sys.Ctz64(uint64(et.size))) & 63
        } else {
            shift = uintptr(sys.Ctz32(uint32(et.size))) & 31
        }
        lenmem = uintptr(old.len) << shift
        newlenmem = uintptr(cap) << shift
        capmem = roundupsize(uintptr(newcap) << shift)
        overflow = uintptr(newcap) > (maxAlloc >> shift)
        newcap = int(capmem >> shift)
    // 兜底分支：根据元素大小乘以元素个数
    // 再针对 span class 进行取整     
    default:
        lenmem = uintptr(old.len) * et.size
        newlenmem = uintptr(cap) * et.size
        capmem, overflow = math.MulUintptr(et.size, uintptr(newcap))
        capmem = roundupsize(capmem)
        newcap = int(capmem / et.size)
    }

    // 进行实际的切片初始化操作
    var p unsafe.Pointer
    // 非指针类型
    if et.ptrdata == 0 {
        p = mallocgc(capmem, nil, false)
        // ...
    } else {
        // 指针类型
        p = mallocgc(capmem, et, true)
        // ...
    }
    // 将切片的内容拷贝到扩容后的位置 p 
    memmove(p, old.array, lenmem)
    return slice{p, old.len, newcap}
}
```

```go
func Test_slice(t *testing.T){
    s := make([]int,512)  
    s = append(s,1)
    t.Logf("len of s: %d, cap of s: %d",len(s),cap(s))
}
```
答案：
len: 513, cap: 848

首先，如 2.7 小节中谈到的，由于切片 s 原有容量为 512，已经超过了阈值 256，因此对其进行扩容操作会采用的计算共识为 512 * (512 + 3*256)/4 = 832

其次，在真正申请内存空间时，我们会根据切片元素大小乘以容量计算出所需的总空间大小，得出所需的空间为 8byte * 832 = 6656 byte

再进一步，结合分配内存的 mallocgc 流程，为了更好地进行内存空间对其，golang 允许产生一些有限的内部碎片，对拟申请空间的 object 进行大小补齐，最终 6656 byte 会被补齐到 6784 byte 的这一档次. （内存分配时，对象分档以及与 mspan 映射细节可以参考 golang 标准库 runtime/sizeclasses.go 文件，也可以阅读我的文章了解更多细节——golang 内存模型与分配机制）

```
// class  bytes/obj  bytes/span  objects  tail waste  max waste  min align
//     1          8        8192     1024           0     87.50%          8
//     2         16        8192      512           0     43.75%         16
//     3         24        8192      341           8     29.24%          
// ...
//    48       6528       32768        5         128      6.23%        128
//    49       6784       40960        6         256      4.36%        128 
```
再终，在 mallocgc 流程中，我们为扩容后的新切片分配到了 6784 byte 的空间，于是扩容后实际的新容量为 cap = 6784/8 = 848.