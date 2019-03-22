#Map
## **类型声明**

public interface Map<K,V>

## 接口说明

key-value 数据结构，python中对应结构为dict（字典）。key,value一一对应。注释中提到:  
` This interface takes the place of the <tt>Dictionary</tt> class, which was a totally abstract class rather than an interface. `  
查看了一下Dictionary类：  
```
public abstract
class Dictionary<K,V> {
    public Dictionary() {
    }
    abstract public int size();
    abstract public boolean isEmpty();
    abstract public Enumeration<K> keys();
    abstract public Enumeration<V> elements();
    abstract public V get(Object key);
    abstract public V put(K key, V value);
    abstract public V remove(Object key);
}

```
基本和map类似，但map将k-v结构提到了接口高度，高于抽象类。估计某个远古版本之前k-v结构是用Dictionary，后来替换为map。

map接口有3个视图，包括key的集合、value的集合、k-v映射集合。map的顺序定义为其迭代器返回的顺序。**当key是可变对象时需要特别谨慎，因为key作为可变对象在变化时有可能影响到equal方法的比较，导致错误**。例如map是无法将自己作为key，因为这时会影响到hashcode取值。在此我们做个简单测试：
```
public static void main(String[] args) {
  HashMap m = new HashMap<>();
  HashMap m2 = new HashMap<>();
  System.out.println("m初始化hashcode:"+m.hashCode());
  m.put(m2,"sdf");
  System.out.println("m加入m2后的hahscode:"+m.hashCode());
  System.out.println("m中m2对应从value:"+m.get(m2));
  m2.put("ss", "sd");
  System.out.println("m2改变后m的hashcode:"+m.hashCode());
  System.out.println("m2改变后m中m2对应从value:"+m.get(m2));
}
```
运行结果如下：
```
m初始化hashcode:0
m加入m2后的hahscode:113717
m中m2对应从value:sdf
m2改变后m的hashcode:113668
m2改变后m中m2对应从value:null
```
可以看到，每次修改Map数据时其hashcode都会变化，且当key为map时，当m2添加了数据，m中却找不到m2对应的value值。后面我们会知道，hashmap是以equal方法和hashcode作为判断是否相等的条件。由于修改动态对象导致数据的hashcode变化，最终会使我们无法在map中找到对应value。**建议设计map时，key填入string或数字类型等静态数据，尽量不要填动态对象**。

通常而言，大部分map的实现类会有两个构造方法，一个是无参构造方法，另一个参数是map，利用传入的map生成新的map，通常用于复制一个map。  

当操作不支持时会抛出UnsupportedOperationException。  

当key和value有严格要求时，对于错误的key,value通常会抛出NullPointerException或ClassCastException

map的equal方法通常调用Object的hashcode方法来比较Key是否相同。

当map做一些递归操作时通常会引起自我递归的异常。通常包括equals，hashCode，toString，clone方法。但是大部分实现类没有处理这个异常，所以需要注意。测试代码：
```
public static void main(String[] args) {
  HashMap m = new HashMap<>();
  System.out.println("m2改变后m中m2对应从value:"+m.get(m2));
  m.put(m,1);
  System.out.println(m.hashCode());
}
```
运行时则会因为内存溢出而崩溃，归根结底是因为调用hashcode时自身递归引用了。

## 实现类参考

[HashMap](HashMap.md)

[TreeMap](TreeMap.md)

[Hashtable](Hashtable.md)

[SortedMap](SortedMap.md)

## 代码解析

```
    //返回存有的key-value映射数量，当数量超过Integer.MAX_VALUE，返回Integer.MAX_VALUE。
    int size();
    boolean isEmpty();
    boolean containsKey(Object key);
    boolean containsValue(Object value);
    V get(Object key);
    V put(K key, V value);
    V remove(Object key);

    //将所有数据一次性加入map中，类似循环使用put方法，在操作中如果参数发生了变化，则其结果是未定义的
    void putAll(Map<? extends K, ? extends V> m);
    void clear();

    //返回key的集合，注意返回结果并不是key的copy，这意味着对返回set的操作会直接反应到map上，反之亦然。
    Set<K> keySet();

    //返回value的集合，注意返回结果并不是value的copy，这意味着对返回set的操作会直接反应到map上，反之亦然。
    Collection<V> values();

    //返回k-v的集合，注意返回结果并不是k-v的copy，这意味着对返回set的操作会直接反应到map上，反之亦然。
    Set<Map.Entry<K, V>> entrySet();

    /**
     * 用作迭代器遍历时返回键值对，其他方式无法获取entry。当遍历时，map发生修改操作，则返回
     * 的entry是未定义的（setValue方法除外）
     */
    interface Entry<K,V> {
        K getKey();
        V getValue();

        /**
         * 对entry设值。通常都不建议一边遍历map，一边修改值。但需要时可以通过这个方法
         * 来设置value，对value的改变会直接反应到map上
         */
        V setValue(V value);
        boolean equals(Object o);
        int hashCode();

        /**
         * 返回按key自然排序的entry比较器
         * @since 1.8
         */
        public static <K extends Comparable<? super K>, V> Comparator<Map.Entry<K,V>> comparingByKey() {
            return (Comparator<Map.Entry<K, V>> & Serializable)
                (c1, c2) -> c1.getKey().compareTo(c2.getKey());
        }

        /**
         * 返回按value自然排序的entry比较器
         * @since 1.8
         */
        public static <K, V extends Comparable<? super V>> Comparator<Map.Entry<K,V>> comparingByValue() {
            return (Comparator<Map.Entry<K, V>> & Serializable)
                (c1, c2) -> c1.getValue().compareTo(c2.getValue());
        }

        /**
         * 按指定key比较器返回entry比较器
         * @since 1.8
         */
        public static <K, V> Comparator<Map.Entry<K, V>> comparingByKey(Comparator<? super K> cmp) {
            Objects.requireNonNull(cmp);
            return (Comparator<Map.Entry<K, V>> & Serializable)
                (c1, c2) -> cmp.compare(c1.getKey(), c2.getKey());
        }

        /**
         * 按指定value比较器返回entry比较器
         * @since 1.8
         */
        public static <K, V> Comparator<Map.Entry<K, V>> comparingByValue(Comparator<? super V> cmp) {
            Objects.requireNonNull(cmp);
            return (Comparator<Map.Entry<K, V>> & Serializable)
                (c1, c2) -> cmp.compare(c1.getValue(), c2.getValue());
        }
    }

    boolean equals(Object o);

    //定义为entryset的hash总值，所以map内容改变了，hashcode也会改变
    int hashCode();

    // Defaultable methods

    /**
     * @since 1.8
     */
    default V getOrDefault(Object key, V defaultValue) {
        V v;
        return (((v = get(key)) != null) || containsKey(key))
            ? v
            : defaultValue;
    }

    /**
     * @since 1.8
     */
    default void forEach(BiConsumer<? super K, ? super V> action) {
        Objects.requireNonNull(action);
        for (Map.Entry<K, V> entry : entrySet()) {
            K k;
            V v;
            try {
                k = entry.getKey();
                v = entry.getValue();
            } catch(IllegalStateException ise) {
                // this usually means the entry is no longer in the map.
                throw new ConcurrentModificationException(ise);
            }
            action.accept(k, v);
        }
    }

    /**
     * @since 1.8
     */
    default void replaceAll(BiFunction<? super K, ? super V, ? extends V> function) {
        Objects.requireNonNull(function);
        for (Map.Entry<K, V> entry : entrySet()) {
            K k;
            V v;
            try {
                k = entry.getKey();
                v = entry.getValue();
            } catch(IllegalStateException ise) {
                // this usually means the entry is no longer in the map.
                throw new ConcurrentModificationException(ise);
            }

            // ise thrown from function is not a cme.
            v = function.apply(k, v);

            try {
                entry.setValue(v);
            } catch(IllegalStateException ise) {
                // this usually means the entry is no longer in the map.
                throw new ConcurrentModificationException(ise);
            }
        }
    }

    /**
     * @since 1.8
     */
    default V putIfAbsent(K key, V value) {
        V v = get(key);
        if (v == null) {
            v = put(key, value);
        }

        return v;
    }

    /**
     * @since 1.8
     */
    default boolean remove(Object key, Object value) {
        Object curValue = get(key);
        if (!Objects.equals(curValue, value) ||
            (curValue == null && !containsKey(key))) {
            return false;
        }
        remove(key);
        return true;
    }

    /**
     * @since 1.8
     */
    default boolean replace(K key, V oldValue, V newValue) {
        Object curValue = get(key);
        if (!Objects.equals(curValue, oldValue) ||
            (curValue == null && !containsKey(key))) {
            return false;
        }
        put(key, newValue);
        return true;
    }

    /**
     * @since 1.8
     */
    default V replace(K key, V value) {
        V curValue;
        if (((curValue = get(key)) != null) || containsKey(key)) {
            curValue = put(key, value);
        }
        return curValue;
    }

    /**
     * @since 1.8
     */
    default V computeIfAbsent(K key,
            Function<? super K, ? extends V> mappingFunction) {
        Objects.requireNonNull(mappingFunction);
        V v;
        if ((v = get(key)) == null) {
            V newValue;
            if ((newValue = mappingFunction.apply(key)) != null) {
                put(key, newValue);
                return newValue;
            }
        }

        return v;
    }

    /**
     * @since 1.8
     */
    default V computeIfPresent(K key,
            BiFunction<? super K, ? super V, ? extends V> remappingFunction) {
        Objects.requireNonNull(remappingFunction);
        V oldValue;
        if ((oldValue = get(key)) != null) {
            V newValue = remappingFunction.apply(key, oldValue);
            if (newValue != null) {
                put(key, newValue);
                return newValue;
            } else {
                remove(key);
                return null;
            }
        } else {
            return null;
        }
    }

    /**
     * @since 1.8
     */
    default V compute(K key,
            BiFunction<? super K, ? super V, ? extends V> remappingFunction) {
        Objects.requireNonNull(remappingFunction);
        V oldValue = get(key);

        V newValue = remappingFunction.apply(key, oldValue);
        if (newValue == null) {
            // delete mapping
            if (oldValue != null || containsKey(key)) {
                // something to remove
                remove(key);
                return null;
            } else {
                // nothing to do. Leave things as they were.
                return null;
            }
        } else {
            // add or replace old mapping
            put(key, newValue);
            return newValue;
        }
    }

    /**
     * @since 1.8
     */
    default V merge(K key, V value,
            BiFunction<? super V, ? super V, ? extends V> remappingFunction) {
        Objects.requireNonNull(remappingFunction);
        Objects.requireNonNull(value);
        V oldValue = get(key);
        V newValue = (oldValue == null) ? value :
                   remappingFunction.apply(oldValue, value);
        if(newValue == null) {
            remove(key);
        } else {
            put(key, newValue);
        }
        return newValue;
    }

```

注意default 方法和Map.Entry中的static方法。JAVA 1.8以前interface中只能定义而不能实现方法。但JDK1.8中为了加强接口的能力，使得接口可以存在具体的方法，前提是方法需要被default或static关键字所修饰[[1]](https://blog.csdn.net/SnailMann/article/details/80231593)。主要是为了支持lambda表达式。

由于entry添加了一例如comparingByKey的static方法，对map的排序可采用如下简单的形式。主要是将entry转化为stream形式，再调用sorted方法排序。[参考原文](https://blog.csdn.net/kaka0930/article/details/52996486###)。
```
    public static void sortByKey(Map map)
		{
			List<Map.Entry<String, Pet>> compareByValue = (List<Entry<String, Pet>>) map
					.entrySet()
					.stream()
					.sorted(Map.Entry.comparingByValue((Pet p1,Pet p2)->{
					       return p1.name.length()-p2.name.length();
					}))
					.collect(Collectors.toList());

			compareByValue.forEach(System.out::println);
    }

```
