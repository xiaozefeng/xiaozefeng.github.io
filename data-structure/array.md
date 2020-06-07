# 如何实现属于自己的动态数组类？

首先对于这个数组类我们应该有哪些要求呢？

1. 必须要能存储任意类型的元素

   可以使用 `Object[]`

2. 可以存放重复元素

3. 存放的元素是按照放入时的顺序排列的

   数组是一块连续的内存，按照索引顺序遍历，起顺序就和放入时一样

4. 支持动态容量变化，但是用户是无感知的

   Java提供的 数组是静态的，需要写逻辑处理这一块



### 定义类和成员变量

```java
public class Array<E> {
    private Object[] data;
    private int size;

    public Array() {
        this(10);
    }

    public Array(int capacity) {
        data = new Object[capacity];
        size = 0;
    }

    public int getSize() {
        return size;
    }

    public int getCapacity() {
        return data.length;
    }

    public boolean isEmpty() {
        return size == 0;
    }
}
```

`data` 用来存放具体的数组，`size` 用来维护当前 `Array` 有多少数组。



### 向数组中指定位置添加元素  add(int index, E e )

```java
public void add(int index, E e) {
    if (index < 0 || index > size)
        throw new IllegalArgumentException("Add failed. index required >=0 and <size.");

    if (size >= data.length)
        resize(data.length * 2);


    // for (int i = size - 1; i >= index; i--)
    //     data[i + 1] = data[i];

    if (size - index >= 0)
        System.arraycopy(data, index, data, index + 1, size - index);

    data[index] = e;
    size++;
}

// resize
private void resize(int newCapacity) {
   if (size >= 0)
     data = Arrays.copyOf(data, newCapacity);
}
```

向数组中添加元素，需要检查 `index` 不能小于0 ，不能` > size` , 不能小于0 很好理解，不能 `>size`怎么说呢？ 

当我们初始一个数组时， 如果没有传入 `capacity` 时， 默认给10 个 `capacity` ， 这时会开辟一个 10 个空间的数组

此时的 `size ==0` ，如果向 index 为 1的位置添加元素， 那么 `index 为0` 的位置就被跳过了，相当于浪费了这个空间。



我们再来看另一种特殊的情况，第一次我们开辟了 10 个大小的数组空间，不断添加元素后，此时已经添加了10个元素了，这时要插入第 11个元素， 此时的数组已经容纳不了这么多元素了，而且Java自带的数组是不能自动扩容的，此时就需要触发  `resize(data.length *2)`操作。

`resize` 操作就是将 原来数组中的数组，拷贝到新的数组中 ，并将 `data 指向这个新的数组`, 这个新的数组的容量是原来的 两倍。

#### 那么为了是原来的两倍呢? 为什么不是 `+ 10` 或者 `+1000`

因为取一个常数是不合理的， 比如此时的容量是 10000 ，如果你下次扩容， 只 +10 那么很快，又会继续扩容，这样会频繁出发扩容

那 `+1000` 呢，如果次数只需要存储 20 个元素， 你扩容，一下子从 10 扩容到 10+1000 ，会导致大量空间被浪费。

最后说道 `data.length *2 ` ，比如10 ，扩容后就是 20, 比如 10000 扩容后就是 20000, 两倍的好处， 扩容后的数量跟原来的数量是在一个`级别上`的。像Java 自带的 `ArrayList` 其扩容机制是 `1.5 ` 倍, 其实是一样的，都是让扩容后的容量和之前的容量在一个数量级别上。



#### add和 resize中 可以看到我没有用 for 循环，而是使用了Java 自带的函数

其实用for 循环也是可以的， 不过 for 循环会对每一个元素都访问一遍。

而使用 `System.arraycopy` 或者 `Arrays.copyOf` 相当于对内存做移动，而且是 `native`方法，效率是比 `for` 循环是更好的。

















