# HashMap

#### HashMap 默认的bucket大小是多少？

```java
/**
 * The default initial capacity - MUST be a power of two.
*/
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16
```



#### 如果new HashMap<>(19), bucket数组多大？

HashMap的bucket的长度一定是2的n次幂, 比19的2的n次幂是32, 所以是buckey的数组长度是32

至于为什么bucket的长度是2的n次幂是因为在put方法中要需要key在bucket中的位置时，本应该使用butckt的长度对key的hash取余操作，但是取余操作比较慢，为了提高性能使用了位运算

```java
i = (n - 1) & hash]
```

这里对n是有要求的，必须是2的n次幂, 所以HashMap不会直接用外部传进来的capacity，而是在这个传入的capacity的 基础上进行计算。



#### HashMap什么时候开辟 bucket数组占用内存？

>在put时初始化

```java
// 在putVal方法中会判断bucket是否为空
if ((tab = table) == null || (n = tab.length) == 0)
  n = (tab = resize()).length;
```

##### resize方法

```java
final Node<K,V>[] resize() {
        Node<K,V>[] oldTab = table;
        int oldCap = (oldTab == null) ? 0 : oldTab.length;
        int oldThr = threshold;
        int newCap, newThr = 0;
        if (oldCap > 0) {
            if (oldCap >= MAXIMUM_CAPACITY) {
                threshold = Integer.MAX_VALUE;
                return oldTab;
            }
            else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                     oldCap >= DEFAULT_INITIAL_CAPACITY)
                newThr = oldThr << 1; // double threshold
        }
        else if (oldThr > 0) // initial capacity was placed in threshold
            newCap = oldThr;
        else {     
            // zero initial threshold signifies using defaults 会走到这里
            newCap = DEFAULT_INITIAL_CAPACITY;
            newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
        }
        if (newThr == 0) {
            float ft = (float)newCap * loadFactor;
            newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                      (int)ft : Integer.MAX_VALUE);
        }
        threshold = newThr;
        @SuppressWarnings({"rawtypes","unchecked"})
            Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
        table = newTab;
        // 省略 ...
        return newTab;
    }
```



#### HashMap在什么情况下会扩容？

> 添加元素时 元素个数大于 threshold时，触发扩容

```java
if (++size > threshold)
  resize();
```



那 threshold 是什么呢？怎么计算得来的?

```java
// DEFAULT_LOAD_FACTOR = 0.75f
newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
```





#### HashMap LoadFactor 负载因子

默认的 loadfactor是 0.75 , 也是就是说只有75%的空间被利用，那为什么要设计负载因子呢？ 

##### loadfactor = 1

如果全部空间都给用，如 bucket的长度为 128 , put key value 时， key在bucket中的分布是不均匀的

如果全部空间都给用，那么会造成一个问题：hash冲突严重， 形成链表，查询效率变低

##### loadfactor = 0.5 

这样会造成空间浪费

##### 结论

0.75是一个折中的值

##### HashMap 注释

```
As a general rule, the default load factor (.75) offers a good
 tradeoff between time and space costs.  Higher values decrease the
 space overhead but increase the lookup cost (reflected in most of
 the operations of the HashMap class, including
 The expected number of entries in
the map and its load factor should be taken into account when
 setting its initial capacity, so as to minimize the number of
 rehash operations.  If the initial capacity is greater than the
 maximum number of entries divided by the load factor, no rehash
 operations will ever occur.
```

