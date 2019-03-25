# HashMap
## **类型声明**
public class HashMap<K,V> extends AbstractMap<K,V>
    implements Map<K,V>, Cloneable, Serializable
由声明可见主要是实现了Map接口，对AbstractMap的利用仅限于toString方法，其他方法基本都重写了。同时标记了Cloneable和Serializable，可序列化，可clone。
## 类说明
与[Hashtable](Hashtable.md)基本一致，但不是线程同步的，且hashmap可以接受null作为key和value.由于采用了算hash分桶的方式，put和get操作基本只需要固定的时间。但遍历的话，其操作时间则与`capacity*size(k-v)`成正比。

在创建hashmap时，capacity和load factor是两个重要指标：capacity默认16，是建立hash table时，桶的数量；而load factor默认0.75，是影响hash table在多满的情况下需要增加capacity的参数。当load factor较大，会减少存储空间，但会增加查询成本，反之亦然。

说人话就是map是一个数组结构，数组中每个元素是一个链表，**默认数组长度为capacity=16**。当执行插入操作时，根据key算一个hash值，由hash值决定数据应该放在数组的哪个位置，对应数组位置上是链表，可以容纳多个相同hash值的value，当map内元素个数达到 **load factor * capacity=12** 时，数组长度变为2*capacity=32，元素会第一次重新分配。注意数组长度是有限制的，当达到entry的最大值时就不会继续增加了。

**注意当有大量元素需要存入hashmap中时，最好在初始化时设置一个合适的值**，因为较大的capacity显然能减少rehash次数，从而更有效地插入数据。

注意hashmap是fail-fast的，意味着当数据被遍历时，如果同时数据被删除了，则会抛出ConcurrentModificationException

**但是注意java8中hashmap的数据结构除了数组和链表外还有[红黑树结构](../../../../algorithm/red_black_tree.md)，当map中有大量数据，链表结构会转化成树结构。何时转换由TREEIFY_THRESHOLD参数决定。**

## 代码解析

* **构造器：**
```JAVA
public HashMap(int initialCapacity, float loadFactor) {
    if (initialCapacity < 0)
        throw new IllegalArgumentException("Illegal initial capacity: " +
                                           initialCapacity);
    if (initialCapacity > MAXIMUM_CAPACITY)
        initialCapacity = MAXIMUM_CAPACITY;
    if (loadFactor <= 0 || Float.isNaN(loadFactor))
        throw new IllegalArgumentException("Illegal load factor: " +
                                           loadFactor);
    this.loadFactor = loadFactor;
    //计算map 的capacity
    this.threshold = tableSizeFor(initialCapacity);
}
public HashMap(int initialCapacity) {
    this(initialCapacity, DEFAULT_LOAD_FACTOR);
}

public HashMap() {
    this.loadFactor = DEFAULT_LOAD_FACTOR; // all other fields defaulted
}

public HashMap(Map<? extends K, ? extends V> m) {
    this.loadFactor = DEFAULT_LOAD_FACTOR;
    putMapEntries(m, false);
}   
```
threshold作为map的变量，表示的是下次rehash时的桶个数阈值（通常而言是等于load factor * capacity）。但是在指定initialCapacity初始化时，threshold其初始值实际上是一个capacity。其中比较有趣的是`public HashMap(int initialCapacity, float loadFactor)`方法中调用的tableSizeFor方法，代码：
```JAVA
static final int tableSizeFor(int cap) {
    int n = cap - 1;
    n |= n >>> 1;
    n |= n >>> 2;
    n |= n >>> 4;
    n |= n >>> 8;
    n |= n >>> 16;
    return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
}
```
由于平时不太使用位操作符，看了半天才看懂其实际功能。其主要功能为，对给定一个正整数a（小于2的30次），计算一个最小值b，使其不小于a,且b=2^x（x为正整数）。首先`>>>`表示忽略符号右移,`|`表示只要两个数有一个1则结果为1。对于一个二进制数`101110`,先右移一位再与原数做或运算，则前两位必定是1，此时将结果右移2位再做或运算则可以将前4位设为1（注意位移数为2^n位），相当于将二进制的所有位，赋值为1，此时再加一就能得到2^x。如下示意图：

![求最小2次方演示](/pic/get_min_2_n.jpg "求最小2次方演示")
