## 什么是 CAS
CAS（Compare and Swap），即比较并替换，实现并发算法时常用到的一种技术。

CAS 的思想很简单：三个参数，一个当前内存值V、旧的预期值 A、即将更新的值 B，当且仅当预期值 A 和内存值 V 相同时，将内存值修改为 B 并返回 true，否则什么都不做，并返回 false。
## 问题
一个 n++ 的问题。
````java
public class Case {

    public volatile int n;

    public void add() {
        n++;
    }
}
````
add() 的字节码指令为：
````java
public void add();
    flags: ACC_PUBLIC
    Code:
      stack=3, locals=1, args_size=1
         0: aload_0       
         1: dup           
         2: getfield      #2                  // Field n:I
         5: iconst_1      
         6: iadd          
         7: putfield      #2                  // Field n:I
        10: return
````
n++ 被拆分为了几个指令：
1. 执行 getfield 拿到原始 n
2. 执行 iadd 进行加 1 操作
3. 执行 putfield 写把累加后的值写回 n

通过 volatile 修饰的变量可以保证线程之间的可见性，但是并不能保证这 3 个指令的原子执行，在多线程并发执行下，无法做到线程安全，得到正确的结果。
## 如何解决
在 add 方法加上 synchronized 修饰解决。
````java
public class Case {

    public volatile int n;

    public synchronized void add() {
        n++;
    }
}
````
这个方案性能上差了一点。

除了低性能的加锁方案，我们还可以使用 JDK 自带的 CAS 方案，在 CAS 中，比较和替换是一组原子操作，不会被外部打断，而且在性能上更占优势。
## AtomicInteger
````java
public class AtomicInteger extends Number implements java.io.Serializable {
    private static final jdk.internal.misc.Unsafe U = jdk.internal.misc.Unsafe.getUnsafe();
    private static final long VALUE = U.objectFieldOffset(AtomicInteger.class, "value");

    private volatile int value;
````
1. Unsafe，是 CAS 的核心类，由于 Java 方法无法直接访问底层系统，需要通过 native 方法来访问，Unsafe 相当于一个后门，基于该类可以直接操作特定内存的数据。
2. 变量 valueOffset，表示该变量值在内存中的偏移地址，因为 Unsafe 就是根据内存偏移地址获取数据的。
3. 变量 value 使用 volatile 修饰，保证了多线程之间内存的可见性。

看看 AtomicInteger 如何实现并发下的累加操作：
````java
public final int getAndAdd(int delta) {
    return U.getAndAddInt(this, VALUE, delta);
}

@HotSpotIntrinsicCandidate
public final int getAndAddInt(Object o, long offset, int delta) {
    int v;
    do {
        v = getIntVolatile(o, offset);
    } while (!weakCompareAndSetInt(o, offset, v, v + delta));
    return v;
}
````
假设有两个线程同时执行 getAndAdd 操作：
1. AtomicInteger 中的 value 原始值为 3，即主内存中 AtomicInteger 中的 value 为 3，根据 Java 内存模型，线程 A 和 线程 B 各自持有一份 value 的副本，值为 3。
2. 线程 A 通过 getIntVolatile(o, offset) 拿到 value 值 3，此时线程 A 被挂起。
3. 线程 B 也通过 getIntVolatile(0, offset) 拿到 value 值 3，但是没有被挂起，执行 weakCompareAndSetInt 方法比较内存值也为 3，成功修改内存值为 2。
4. 此时线程 A 恢复，执行 weakCompareAndSetInt 方法比较，发现自己手中的值 3 和内存中的值 2 不一致，说明该值已经被其他线程提前修改过了，则重新来一遍。
5. 重新获取 value 值，因为变量 value 被 volatile 修饰，所以其他线程对它的修改，线程 A 总是能看到，得到 2，线程 A 继续执行 weakCompareAndSetInt 进行比较替换，直到成功。 

方法的作用是原子的更新变量。跟进 weakCompareAndSetInt
````java
@HotSpotIntrinsicCandidate
public final boolean weakCompareAndSetInt(Object o, long offset,int expected,int x) {
    return compareAndSetInt(o, offset, expected, x);
}
````
其中 compareAndSetInt 是 native 方法。

在 OpenJDK 中拿到了 compareAndSetInt 的 cpp 代码
````c++
UNSAFE_ENTRY(jboolean, Unsafe_CompareAndSetInt(JNIEnv *env, jobject unsafe, jobject obj, jlong offset, jint e, jint x)) {
  oop p = JNIHandles::resolve(obj);
  if (oopDesc::is_null(p)) {
    // 这句首先拿到 value 在内存中的地址，通过 VALUE 即内存偏移量
    volatile jint* addr = (volatile jint*)index_oop_from_field_offset_long(p, offset);
    // 这里直接进行原子操作，保证原子性
    return RawAccess<>::atomic_cmpxchg(x, addr, e) == e;
  } else {
    assert_field_offset_sane(p, offset);
    return HeapAccess<>::atomic_cmpxchg_at(x, p, (ptrdiff_t)offset, e) == e;
  }
} UNSAFE_END
````
CAS 优点：
不会阻塞线程，不会进行线程上下文切换
CAS 缺点：
1. 线程自旋，cpu 空转
2. 不能保证多个变量的原子性
3. 不能感知到 ABA 问题，可以考虑使用版本号解决
   > ABA 问题：如果变量 V 初次读取的时候是 A，并且在准备赋值的时候检查到它仍然是 A，那能说明它的值没有被其他线程修改过了吗？  
   > 如果在这段期间曾经被改成 B，然后又改回 A，那 CAS 操作就会误认为它从来没有被修改过。针对这种情况，Java 并发包中提供了一个带有标记的原子引用类 AtomicStampedReference，它可以通过控制变量值的版本来保证 CAS 的正确性。