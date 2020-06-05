# 3月份的面试经验

### Java基础





### 多线程

#### 什么是乐观锁，什么是悲观锁？

**乐观锁**：总是往最好的方向想，每次去拿数据都不加锁，只有当去修改数据时才会检查数据是否被修改。

乐观锁适用于读多写少的情况，因为每次读是不加锁的，所以读的效率相当高，但是当写冲突比较多的情况，乐观锁会不断尝试修改而陷入自旋，这种情况反而比悲观锁需要更多的时间。

乐观锁还有一个ABA问题，有t1和t2两个线程，t1获取v的值是A，尝试修改v的值时还是A，但是是在这个过程中，v的值可能被t2改成了B，然后又改回了A，t1对于这个过程是不知情的。怎么解决这个问题呢？可以通过加版本号的方式，实现的思路就是添加一个version字段，每一次对v的修改 version的值都加一，尝试去修改v的值时，对比version字段是否一致，如果一致才修改，否则重新读取version。

**悲观锁**：总是往最坏的方向想，每次去拿数据都加锁；适用于写多读少的情况。



#### Java中的乐观锁有哪些实现？悲观锁有哪些实现？

**乐观锁实现**: juc 下面的 atomic 包下的 AtomicXXX ，实现原理是通过CAS算法实现的。

**悲观锁实现**: synchronized 关键字 和  Lock



#### CAS 是什么？

CAS 是 compare and swap 的简写，字面意思是 ”比较并替换“，一个cas操作涉及到

> 我们假设内存中的原数据 V，旧的预期值A，需要修改的值B
>
> 1. 比较A 与V是否相等 （比较）
> 2. 如果比较相等，将B写入V （替换）
> 3. 返回操作是否成功

当多个线程同时对某个资源进行CAS操作，只能有一个线程操作成功，但是并不会阻塞其他线程,其他线程只会收到操作失败的信号



#### synchronized 实现原理？

**synchronized代码块**

通过反编译得知，synchronized 在代码块开始的地方 插入了一个 monitor-enter的指令, 当前线程尝试去获对象的锁也就是monitor，当锁的计数器为0时可以获取成功，获取成功时将锁的计数器的值加1，没有获取到锁的线程则进入阻塞状态。

在代码块结束的地方插入了一个 monitor-exit 指令，获取到锁的线程会将锁的计数器置为0，表明锁被释放。

**synchronized方法**

会将方法打上 ACC_SYNCHRONIZED 的标记， JVM在调用方式时会检查是否有此标记，如果有，执行线程先持有monitor，在方式结束时或者抛出异常时释放monitor



#### JVM在jdk1.6后对synchronized做了哪些优化？

**偏向锁**：它是一种针对加锁操作的优化手段，大多数情况，锁不仅不存在多线程竞争，而且总是由同一个线程获得，因此为了减少同一个线程获取锁而产生的消耗而引入了偏向锁，偏向锁的核心思想是 **如果一个线程获得了锁，那么锁就进入了偏向模式，当这个线程再次请求锁时，无需再做任何同步操作就可获得锁** ，这样就省去了大量关于锁申请的操作，对于没有大量锁竞争的情况，偏向锁有很好的优化效果，但是对于锁竞争激烈的情况，偏向锁失效了，此时会升级成轻量级锁。

**轻量级锁**：轻量级锁的本意是在没有多线程竞争的情况下，较少重量级锁使用操作系统互斥量产生的性能消耗，因为使用轻量级锁是不需要申请互斥量；轻量级锁实现加锁的原理是将 Mark Word 中的部分字节CAS更新指向线程栈中的 Lock Record，如果更新成功，则获取轻量级锁成功，否则说明已经有线程获取了轻量级锁，发生了锁竞争，接下来膨胀为自旋锁。

**自旋锁**：轻量级锁失效后，虚拟机为了避免线程真实的在操作系统层面挂起，会使用自旋锁进行优化，这是基于大多数情况锁的持有时间不会太长，如果直接挂起就得不偿失，因为操作系统线程之间切换需要从用户态转换到内核态，这个状态之间的转换需要相对较长的时间，因此自旋锁假设在不久的将来，当前线程可以获取到锁，因此虚拟机会让当前线程进入自旋一定时间，如果获得锁，就顺利进入临界区，如果还不能获得锁，那就将线程在操作系统层面挂起。最后没办法也就只能升级到重量级锁了。

**适应性自旋锁**：适应性自旋锁带来的改进是：自旋的时间不再固定了，而是由前一次在同一个锁上的自旋时间以及锁拥有者的状态来决定。

**锁消除**：指的是在虚拟机即时编译器在运行时，对一些代码上要求同步，但是被检测是不可能存在共享数据竞争的锁进行消除。如果StringBuffer的append是被synchronized修饰的，但是我们在方法中运行是没有同步问题的。这个时候就会进行锁消除。





#### ReentrantLock 实现原理？



### synchronized 和 ReentrantLock 的区别？

synchonized 和 ReentrantLock 都是悲观锁的实现，并且都是可重入锁。

主要的区别是:

**公平锁和非公平锁**

**公平锁**: 在多个线程抢同一把锁时，没有抢到的线程会进入等待队列，公平锁会按照队列排队的顺序来获取锁

**非公平锁**：不关心队列中的线程，谁抢到锁就是谁的

ReentrantLock可以选择使用公平锁或者非公平锁，但是 synchronized 只能是非公平锁

```java
public ReentrantLock(boolean fair) {
  sync = fair ? new FairSync() : new NonfairSync();
}
```

默认非公平锁

```java
public ReentrantLock() {
    sync = new NonfairSync();
}
```

非公平锁
