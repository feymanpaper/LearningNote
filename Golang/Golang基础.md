## Go的类型系统type
https://juejin.cn/post/6844903489252900872

这里解释了为什么hp是\[\]int类型却可以用len和append等方法, 而编译器不会直接报错. 因为\[\]是slice切片类型, 是`unnamed type`
```go
type hp []int
func (h hp) Len() int{
	return len(h)
}
```
### type 的分类
在 Go 中所有需要被定义和声明的对象都是 `type，int，string，function，pointer，interface，map，struct `等等。

和大多数计算机语言一样，Go type 默认包含常用的基础数据类型，`boolean，numeric and string`，这些类型称为 `pre-declarered types`，这些基础的数据又可以进一步构成更复杂的类型 `array，struct，map，slice，channel` 等。

Type 可以是带名称和不带名称的，称之为 `named type 和 unnamed type`。

`Named Types` 就是通过 type 关键字为一个已有的 type 起个别名，像这样 `type NewType ExistingType`  NewType 就是名字。

`Pre-declared types` 也是 `Named types`。

`Unamed types `是一个 literal type，也就是没有名字，只有 type 本身，像这样 `[6]int` 没有名字。

每一个类型都有自己的 Underlying type ，如果 T 是 pre-declared type 或者 type literal，它们对应的 underlying type 就是自身 T，否则 T 的 underlying type 是 T 定义时引用的类型的 underlying type。

### underlying type

如果两个 type 都是 named type ，彼此之间不能相互赋值

```go
type NewString string
var my string ="a"
var you NewString = my //cannot use my (type string) as type NewString in assignment
```

如果两个 type 其中一个是 unamed type，彼此之间可以相互赋值

复制代码

```go
package main
type Ptr *int
type Map map[int]string
type MapMap Map
func main() {
    var p *int
    var mm Map
    var mmm MapMap
    var m1 map[int]string = mm
    var m2 map[int]string = mmm
    var ptr Ptr = p
    print(ptr)
    print(m1)
    print(m2)
}
```

为什么有这样的区分？
如果为一个类型起了名字，说明你想要做区分，所以两个 named types 即使 underlying name 相同也是不能相互赋值的。

### Named type 和 Unamed type

当 named types 被作为一个 function 的 receiver 时，它就拥有了自己的方法，unamed types 则不能，这是它们的重要区别。
```go
package main
import (
    "fmt"
)
type NewMap map[int]string
func (nm NewMap) add(key int, value string) {
    nm[key] = value
}
func main() {
    var p NewMap = make(map[int]string)
    p.add(10, "a")
    fmt.Println(p) //map[10:a]
}
```

有个一例外是是 pre-declare types 不能拥有自己的方法。
```go
package main
func (n int) name(){   
    print(n)
}
func main() {
    var n int
    n.name()
}
```

编译器会抛出 cannot define new methods on non-local type int 错误，不能对包之外的 type 定义方法，解决这个问题就是对 pre-declared types 重新定义别名。

### type 的属性继承一：直接继承

Named type 不会从它的 underlying type 或 引用 type 继承 method，但是会继承 field

复制代码

```go
package main
import (
    "fmt"
)
type Person struct {
    name string
}
func (p *Person) Speak() {
    fmt.Println("I am a person")
}
type Student Person
func main() {
    var p Person
    p.Speak()
    var s Student
    s.name = "jone"
    fmt.Println(s.name)
    // s.Speak()
}
```

Named type Student 不会继承来自 Person Speak 的方法，打开注释执行报错 s.Speak undefined (type Student has no field or method Speak)，但是 Person 的 filed name 可以被 Student 继承。

### type 的属性继承二：type embedding

如果一个 type T‘ 被嵌入另一个 type T 作为它的 filed，T’ 的所有 field 和 method 都可以在 T 中使用，这种方法称之为 type embedding。

```go
package main
import (
    "fmt"
)
type I interface {
    Talk()
}
type Person struct {
    name string
}
func (p *Person) Speak() {
    fmt.Println("I am a person")
}
func (p *Person) Talk() {
    fmt.Println("I am talking")
}
type People struct {
    Person
}
func main(){
    var people People
    people.name = "people"
    people.Speak()
    people.Talk()
}
```

### type 转换
Type 之间是可以相互转换的，但要遵循一定的转换规则，详细请看官方规范 https://golang.org/ref/spec#Conversions。

  
