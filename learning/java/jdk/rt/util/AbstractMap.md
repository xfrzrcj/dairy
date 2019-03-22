# AbstractMap
## **类型声明**
public abstract class AbstractMap<K,V> implements Map<K,V>
## 类说明
是map的实现类，实现了一部分map的功能。大部分map，如[HashMap](HashMap.md),都会继承本类。本类的containsKey,containsValue,get,remove方式都是通过遍历entryset的方式实现，且不支持put操作（会抛出UnsupportedOperationException）

## 代码解析

```
transient volatile Set<K>        keySet;
transient volatile Collection<V> values;
```
注意keyset和values的两个修饰词transient、volatile。transient阻止对象序列化(大概是隐私性要求)。[volatile](https://www.cnblogs.com/chengxiao/p/6528109.html)确保共享变量对所有线程的可见性。

```
public int hashCode() {
        int h = 0;
        Iterator<Entry<K,V>> i = entrySet().iterator();
        while (i.hasNext())
            h += i.next().hashCode();
        return h;
    }
```
这边hashcode即为map中所定义的将所有entry的hashcode相加。
```
public int hashCode() {
            return (key   == null ? 0 :   key.hashCode()) ^
                   (value == null ? 0 : value.hashCode());
        }
```
这段是entry的hashCode定义，采用key异或value的方式获取值。
```
public String toString() {
        Iterator<Entry<K,V>> i = entrySet().iterator();
        if (! i.hasNext())
            return "{}";

        StringBuilder sb = new StringBuilder();
        sb.append('{');
        for (;;) {
            Entry<K,V> e = i.next();
            K key = e.getKey();
            V value = e.getValue();
            sb.append(key   == this ? "(this Map)" : key);
            sb.append('=');
            sb.append(value == this ? "(this Map)" : value);
            if (! i.hasNext())
                return sb.append('}').toString();
            sb.append(',').append(' ');
        }
    }
```
大部分map的默认tostring都是用的这个。可以看到这里对做了`key   == this ? "(this Map)" : key`的判断，避免自引用导致的递归死循环。
