![](Golang/img/1697776119445.png)
### 单例模式
这篇博客讲了sync.Once的底层实现
https://mp.weixin.qq.com/s/KRgNwJt1C7q2ckeqCu9pCQ

饿汉式
在包被加载时创建
实例是在包被导入时初始化的，所以如果初始化耗时，会导致程序加载时间比较长。
```go
var ins *Singleton = &Singleton{}  
  
type Singleton struct {  
}  
  
func GetInsOr() *Singleton {  
    return ins  
}
```

懒汉式(使用最多的)
在第一次被使用时创建
缺点时非并发安全的, 因此在使用的过程中要加锁
```go
var ins *Singleton  
var mu sync.Mutex  
  
type Singleton struct {  
}  
  
func GetInsOr() *Singleton {  
    if ins == nil {  
       mu.Lock()  
       if ins == nil {  
          ins = &Singleton{}  
       }  
       mu.Unlock()  
    }  
    return ins  
}
```

go语言中用`sync.Once`来实现
使用`once.Do`可以确保ins实例全局只被创建一次，once.Do函数还可以确保当同时有多个创建动作时，只有一个创建动作在被执行。
```go
var ins *Singleton  
var once sync.Once  
  
type Singleton struct {  
}  
  
func GetInsOr() *Singleton {  
    once.Do(func() {  
       ins = &Singleton{}  
    })  
    return ins  
}
```

### 简单工厂模式
接受一些参数，然后返回对象实例
```go
type Person struct {  
    name string  
    age  int  
}  
  
func NewPerson(_name string, _age int) *Person {  
    return &Person{  
       name: _name,  
       age:  _age,  
    }  
}
```

### 抽象工厂模式
它和简单工厂模式的唯一区别，就是它返回的是接口而不是结构体。
通过返回接口，可以**在你不公开内部实现的情况下，让调用者使用你提供的各种功能**
```go
type Person interface {  
    Greet()  
}  
  
type person struct {  
    name string  
    age  int  
}  
  
func (p person) Greet() {  
    fmt.Println("I am fool")  
}  
  
func NewPerson(_name string, _age int) Person {  
    return person{  
       name: _name,  
       age:  _age,  
    }  
}
```

### 工厂方法模式
在**工厂方法模式**中，依赖工厂函数，我们可以通过实现工厂函数来创建多种工厂，将对象创建从由一个对象负责所有具体类的实例化，变成由一群子类来负责对具体类的实例化，从而将过程解耦。
```go
type Person struct {  
    name string  
    age  int  
}  
  
func NewPersonFactory(_age int) func(_name string) Person {  
    return func(name string) Person {  
       return Person{  
          age:  _age,  
          name: name}  
    }  
}

//使用
func main() {  
    newBaby := NewPersonFactory(1)  
    baby := newBaby("john")  
  
    newTeenager := NewPersonFactory(16)  
    teen := newTeenager("jill")  
}
```

### 策略模式
策略模式（Strategy Pattern）定义一组算法，将每个算法都封装起来，并且使它们之间可以互换。
```go
type IStrategy interface {  
    do(int, int) int  
}  
  
type add struct {  
}  
  
func (*add) do(a, b int) int {  
    return a + b  
}  
  
type sub struct {  
}  
  
func (*sub) do(a, b int) int {  
    return a - b  
}  
  
type Operator struct {  
    IStrategy  
}  
  
func (op *Operator) setStrategy(strategy IStrategy) {  
    op.IStrategy = strategy  
}  
func (op *Operator) Calculate(a, b int) int {  
    return op.do(a, b)  
}
```
### 模板模式
模板模式 (Template Pattern)定义一个操作中算法的骨架，而将一些步骤延迟到子类中
模板模式就是将一个类中能够公共使用的方法放置在抽象类中实现，将不能公共使用的方法作为抽象方法，强制子类去实现，这样就做到了将一个类作为一个模板，让开发者去填充需要填充的地方。

```go
type Cooker interface {
	fire()
	cooke()
	outfire()
}

// 类似于一个抽象类
type CookMenu struct {
}

func (CookMenu) fire() {
	fmt.Println("开火")
}

// 做菜，交给具体的子类实现
func (CookMenu) cooke() {
}

func (CookMenu) outfire() {
	fmt.Println("关火")
}

// 封装具体步骤
func doCook(cook Cooker) {
	cook.fire()
	cook.cooke()
	cook.outfire()
}

type XiHongShi struct {
	CookMenu
}

func (*XiHongShi) cooke() {
	fmt.Println("做西红柿")
}

type ChaoJiDan struct {
	CookMenu
}

func (ChaoJiDan) cooke() {
	fmt.Println("做炒鸡蛋")
}
```

### 代理模式
代理模式 (Proxy Pattern)，可以为另一个对象提供一个替身或者占位符，以控制对这个对象的访问。

```go
type Seller interface {
	sell(name string)
}

// 火车站
type Station struct {
	stock int //库存
}

func (station *Station) sell(name string) {
	if station.stock > 0 {
		station.stock--
		fmt.Printf("代理点中：%s买了一张票,剩余：%d \n", name, station.stock)
	} else {
		fmt.Println("票已售空")
	}

}

// 火车代理点
type StationProxy struct {
	station *Station // 持有一个火车站对象
}

func (proxy *StationProxy) sell(name string) {
	if proxy.station.stock > 0 {
		proxy.station.stock--
		fmt.Printf("代理点中：%s买了一张票,剩余：%d \n", name, proxy.station.stock)
	} else {
		fmt.Println("票已售空")
	}
}
```

### 选项模式
选项模式有很多优点，例如：支持传递多个参数，并且在参数发生变化时保持兼容性；支持任意顺序传递参数；支持默认值；方便扩展；通过WithXXX的函数命名，可以使参数意义更加明确，等等。
不过，为了实现选项模式，我们增加了很多代码，所以在开发中，要根据实际场景选择是否使用选项模式。选项模式通常适用于以下场景：
- 结构体参数很多，创建结构体时，我们期望创建一个携带默认值的结构体变量，并选择性修改其中一些参数的值。
- 结构体参数经常变动，变动时我们又不想修改创建实例的函数。例如：结构体新增一个retry参数，但是又不想在NewConnect入参列表中添加`retry int`这样的参数声明。
如果结构体参数比较少，可以慎重考虑要不要采用选项模式。
```go
type Connection struct {
	addr    string
	cache   bool
	timeout time.Duration
}

const (
	defaultTimeout = 10
	defaultCaching = false
)

type options struct {
	timeout time.Duration
	caching bool
}

// Option overrides behavior of Connect.
type Option interface {
	apply(*options)
}

type optionFunc func(*options)

func (f optionFunc) apply(o *options) {
	f(o)
}

func WithTimeout(t time.Duration) Option {
	return optionFunc(func(o *options) {
		o.timeout = t
	})
}

func WithCaching(cache bool) Option {
	return optionFunc(func(o *options) {
		o.caching = cache
	})
}

// Connect creates a connection.
func NewConnect(addr string, opts ...Option) (*Connection, error) {
	options := options{
		timeout: defaultTimeout,
		caching: defaultCaching,
	}

	for _, o := range opts {
		o.apply(&options)
	}

	return &Connection{
		addr:    addr,
		cache:   options.caching,
		timeout: options.timeout,
	}, nil
}

func main() {
	connect, err := NewConnect("abc", WithCaching(false), WithTimeout(time.Second))
	if err != nil {
		return
	}
	fmt.Println(connect.timeout)
}
```