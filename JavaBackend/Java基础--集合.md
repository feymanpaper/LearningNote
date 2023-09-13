
![[java集合框架.png]]

## 基础概念 
数组有不足的地方
1.长度开始时必定指定，一旦指定，不能更改
2.保存的必须为同一类型的元素
3.使用数组进行增加/删除元素，麻烦

集合的好处
1.可以动态保存任意多个对象，使用方便
2.提供了一系列方便的操作对象方法，add，remove，set，get等
3.使用集合添加，删除新元素的示意代码--简洁
![[jdk_collection.png]]

![[jdk_map.png]]
## Collection 
### Collection接口和常用方法
Collection接口实现类的特点
1.Collection实现子类可以存放多个元素，每个元素可以是Object
2.有些Collection的实现类，有些可以放重复元素，有些不可以
3.Collection接口没有直接的实现子类，是通过子接口Set和List来实现的

Iterator迭代器
1.Iterator对象称为迭代器，主要用于遍历Collection集合中的元素
2.所有实现了Collection接口的集合类都有一个iterator()方法，返回一个实现了Iterator接口的对象，返回一个迭代器
3.Iterator结构
4.Iterator仅用于遍历集合，本身并不存放对象
```java
Collection col = new ArrayList();
Iterator iterator = col.iterator();
while(iterator.hasNext()){
	//返回下一个元素，类型是Object
	//编译类型是Object，运行类型是取决于真正存放的
	Object next = iterator.next();
}
//退出循环后，此时iterator迭代器指向最后一个元素
//iterator.next()会导致NoSuchElementException
//若需要再次遍历，需要重置迭代器
//iterator = col.iterator();
```
增强for循环,既可以在Collection也可以在数组用
其底层仍然是Iterator迭代器
```java
for(Object book:col){
	...
}
```

## List接口方法
...
add的时候会发生装箱，比如Integer.valueOf(int i)

### ArrayList底层结构和源码分析

1.permits all elements, including null。ArrayList可以加入null
2.是由数组实现数据存储的
3.基本等同于Vector，除了ArrayList是线程不安全（执行效率高），多线程的情况下，不建议使用ArrayList

类定义
```java
public class ArrayList<E> extends AbstractList<E>  
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
```
1)ArrayList中维护了一个Object类型的数组elementData. 
transient Obiect[] elementData; //不会被序列化
2）当创建ArrayList对象时，如果使用的是无参构道器，则初始elementData容量为0，第1次添加，则扩容elementData为10，如需要再次扩容，则扩容elementData为1.5倍。
3)如果使用的是指定大小的构造器，则初始elementData容量为指定大小，如果需要扩容，则直接扩容elementData为1.5倍。

mCount++记录集合被修改的次数
无参构造
```java
public ArrayList() {  
    this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;  
}
```
有参构造
```java
public ArrayList(int initialCapacity) {  
    if (initialCapacity > 0) {  
        this.elementData = new Object[initialCapacity];  
    } else if (initialCapacity == 0) {  
        this.elementData = EMPTY_ELEMENTDATA;  
    } else {  
        throw new IllegalArgumentException("Illegal Capacity: "+  
                                           initialCapacity);  
    }  
}
```
扩容使用的是Arrays.copyOf()
```java
private Object[] grow(int minCapacity) {  
    int oldCapacity = elementData.length;  
    if (oldCapacity > 0 || elementData != DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {  
        int newCapacity = ArraysSupport.newLength(oldCapacity,  
                minCapacity - oldCapacity, /* minimum growth */  
                oldCapacity >> 1           /* preferred growth */);  
        return elementData = Arrays.copyOf(elementData, newCapacity);  
    } else {  
        return elementData = new Object[Math.max(DEFAULT_CAPACITY, minCapacity)];  
    }  
}
```

### Vector底层结构和源码分析
类定义
```java
public class Vector<E>  
    extends AbstractList<E>  
    implements List<E>, RandomAccess, Cloneable, java.io.Serializable
```
Vector底层也是一个对象数组，protected Objectll elementData;
Vector是线程同步的，即线程安全，操作方法带有synchronized

如果没指定capacityIncrement，就是按下列表说的
![[vector_arraylist_对比.png]]

### LinkedList底层结构和源码分析
LinkedList底层实现了双向链表和双端队列特点
1.可以添加任意元素（元素可以重复），包括null
2 ﻿﻿线程不安全，没有实现同步

1.LinkedList底层维护了一个双向链表。
2.LinkedList中维护了两个属性first和last分别指向 首节点和尾节点
每个节点 (Node对象），里面又维护了prev、next、item三个厲性，其中通过  
prev指向前一个，通过next指向后一个节点。最终实现双向链表。所以LinkedList的元素的添加和删除，不是通过数组完成的，相对来说效率较高。

add方法是调用linkLast
```java
void linkLast(E e) {  
    final Node<E> l = last;  
    final Node<E> newNode = new Node<>(l, e, null);  
    last = newNode;  
    if (l == null)  
        first = newNode;  
    else        l.next = newNode;  
    size++;  
    modCount++;  
}
```
remove默认是removeFirst()
```java
private E unlinkFirst(Node<E> f) {  
    // assert f == first && f != null;  
    final E element = f.item;  
    final Node<E> next = f.next;  
    f.item = null;  
    f.next = null; // help GC  
    first = next;  
    if (next == null)  
        last = null;  
    else        next.prev = null;  
    size--;  
    modCount++;  
    return element;  
}
```

![[ArrayList_LinkedList比较.png]]

## Set接口方法
无序（添加和取出顺序不一致），没有索引
不允许重复元素，最多包含一个null

Set接口也是Collection的子接口，因此常用方法和Collecton一样，遍历方式也一样
可以使用迭代器，增强for循环，但是不能用索引

### HashSet底层结构和源码分析
HashSet实现了Set接口
HashSet实际上是HashMap
```java
public HashSet() {  
    map = new HashMap<>();  
}
```
不能加入重复对象的真正含义？
```java
set.add("lucy"); //返回true
set.add("lucy"); //返回false
set.add(new Dog("tom")); // 返回true 
set.add(new Dog("tom")); //返回true

//经典的面试题
//看源码做分析，add发生了什么？==>底层机制
set.add(new String("hsp")); //返回true
set.add(new String("hsp")); //返回false
```

HashSet底层是HashMap，HashMap底层是（数组+链表+红黑树）
1.HashSet底层是HashMap
2.添加一个元素时，会先得到hash值-会转成索引值
3.找到存储数据表table，看这个索引位置是否已经存放有元素
4.如果没有，直接加入
5.如果有，调用equals比较，如果相同就放弃添加，如果不相同就添加到最后
6.Java8中，如果一条链表的元素个数到达TREEIFY_THRESHOLD（默认8），并且table大小 >= MIN_TREEIFY_CAPACITY(默认64)，就会转化成红黑树

初始化无参数构造
```java
public HashSet() {  
    map = new HashMap<>();  
}
```
执行add方法
```java
public boolean add(E e) {  
    return map.put(e, PRESENT)==null;  
}
```
其中的PRESENT只是占位的, 共享
```java
private static final Object PRESENT = new Object();
```
执行put方法，该方法会执行hash(key)得到key对应的hash值，不等价于hashcode

hash
```java
static final int hash(Object key) {  
    int h;  
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);  
}
```

1.Hashset底层是HashMap，第一次添加时，table 数组扩容到 16，临界值  (threshold)是 16\*加载因子  (loadFactor)是0.75= 12
2.如果table数组使用到了临界值 12(size到达了12),就会扩容到 16 \* 2= 32，新的临界值就是 32 \* 0.75=24依次类推  (每加入一个节点Node，size就会++)
3.在Java8中，如果一条链表的元素个数到达 TREEIFY THRESHOLD（默认是8）。  并且table的大小＞=  MIN_ TREEIFY_ CAPACITY（默认64）,就会进行树化（红黑树)，否则仍然采用数组扩容机制

### LinkedHashSet底层结构和源码分析
说明
1）在LinkedHastset 中维护了一个hash表和双向链表
(LinkedHashSet 有 head 和 tail )
2） 每一个节点有before 和after 属性，这样可以形成双向链表
3） 在添加一个元素时，先求hash值，在求索引，确定该元素布
table的位置，然后将添加的元素加入到双向链表(如果已经存在，不添加(原则和hashset一样）
tail.next = new Element // 示意代码
newElement.pre = tail
tail = newEelment;
4）这样的话，我们遍历LinkedHashset 也能确保插入顺序和遍历顺序一致

1.LinkedHashset 加入顺序和取出元素/数据的顺序一致
2.LinkedHashset 底层维护的是一个LinkedHashMap（是HashMap的子类）
3.LinkedHashset 底层结构〔数组table+双向链表）
4.添加第一次时，直接将 数组table 扩容到 16，存放的结点类型是 LinkedHashMap$Entry
5.数组是 HashMap\$Node[] 存放的元素/数据是 LinkedHashMap\$Entry类型

### Map接口和方法
注意：这里讲的是JDK8的Map接口特点 Map_.java

1)Map与Collection并列存在。用于保存具有映射关系的数据：Key-Value
2)Map 中的 key 和 value 可以是任何引用类型的数据，会封装到HashMap$Node
对象中
3）Map 中的key 不允许重复，原因和HashSet 一样，前面分析过源码。
4 ﻿﻿Map 中的value 可以重复
5.Map 的key 可以为null, value 也可以为null，注意 key 为null，只能有一个，value 为null,可以多个。
6.常用String类作为Map的 key
7.key 和 value 之间存在单向一对一关系，即通过指定的 key 总能找到对应的 value
![[Pasted image 20230523172648.png]]

1.k-v till HashMap$Node node = newNode (hash, key, value, null)
2.k-v 为了方便程序员的遍历，还会 创建 Entryset 集合，该集合存放的元素的类型 Entry，而一个Entry。对象就有k,v Entryset<Entry<k, V>>即：
transient Set<Map.Entry<K, V>> entrySet;
3.entryset 中，定义的类型是Map.Entry ，但是实际上存放的还是 HashMap$Node
这是因为static class Node<K, V> implements Map. Entry<K, V>
4．当把 HashMap$Node 对象 存放到entryset 就方便我们的遍历，因为 Map。Entry 提供了重要方法
K.getKey(); V.getValue();

//（1）增强for
1/第一组：先取出 所有的Key ，通过Key 取出对应的Value
```java
Set keyset = map.keySet();
for (Object key : keyset) {
System.out.println(key + "-" + map.get (key));
}
```
//（2）迭代器
```java
Iterator iterator = keyset.iterator ();
while (iterator.hasNext()) {
	Object key = iterator.next ()
	System.out.println(key + "-" + map.get(key));
}
```

//第二组：把所有的values取出
//(1〕 增强for
```java
Collection values = map. values ();
for (Object value : values) {
	System.out.println(value);
}
```
//(2） 选代器
```java
Iterator iterator2 = values. iterator();
while (iterator2. hasNext()) {
	Object value = iterator2.next();
	System.out.println(value);
}
```
//第三组：通过Entryset 来获取 k-v
```java
Set entrySet = map.entrySet ();// EntrySet<Map. Entry<K, V>>
for (Object entry : entrySet) {
//将entry 转成 Map.Entry
	Map.Entry m = (Map.Entry) entry;
	System.out.println(m.getKey () + "-" + m.getValue ());
}
```

1)Map接口的常用实现类：HashMap、Hashtable和Properties。
2)HashMap是 Map 接口使用频率最高的实现类。
3)HashMap 是以 key-val 对的方式来存储数据(HlashMap$Node类型）
4)key 不能重复，但是值可以重复，允许使用nul建和nul值。
5)如果添加相同的key，则会覆盖原来的key-val,等同于修改.（key不会替换，val会替换）
6)与Hashset一样，不保证映射的顺序，因为底层是以hash表的方式来存储的。
7)HashMap没有实现同步，因此是线程不安全的

![[HashMap.png]]

### HashTable
线程安全，JDK不维护了

### 集合选择
![[Pasted image 20230530171220.png]]

试分析HashSet和Treeset分别如何实现去重的

(1)HashSet的去重机制：hashCode0 + equals0，底层先通过存入对象,进行运算得到一个
hash值，通过hash值得到对应的索引，如果发现table素引1所在的位置，没有数据，就直接存放如果有数据，就进行equals比较這历比较，如果比较后，不相同，就加入，香则就不加入，（2) Treeset的去重机制：如果你传入了一个Comparator匿名对象，就使用实现的compare去重，如果方法返口0,就认为是相同的元素/数据，就不添加，如果你没有传入一个Comparator
匿名对象，则以你添加的对象实现的Compareable接口的compareTo去重.