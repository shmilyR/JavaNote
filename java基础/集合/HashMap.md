# HashMap结构
HashMap是一个散列表，它存储的内容是键值对(key-value)映射。  
底层是数组+链表+红黑树。当链表中的元素超过8个之后，会将链表转换成红黑树。
<img src="image/HashMap结构.PNG"/>  
底层实现：  
```java
//此类是用来实现数组及链表的数据结构
static class Node<K,V> implements Map.Entry<K,V> {
    final int hash;//保存节点的Hash值
    final K key;//保存节点的key值
    V value;//保存节点的value值
    Node<K,V> next;//指向链表结构下的当前节点的next节点。红黑树当中也有使用
    Node(int hash, K key, V value, Node<K,V> next) {
        this.hash = hash;
        this.key = key;
        this.value = value;
        this.next = next;//表示链表的下一个节点
    }

    public final K getKey()        { return key; }
    public final V getValue()      { return value; }
    public final String toString() { return key + "=" + value; }

    //由此可看出，hashCode的计算为key的哈希值^value的哈希值
    public final int hashCode() {
        return Objects.hashCode(key) ^ Objects.hashCode(value);
    }

    public final V setValue(V newValue) {
        V oldValue = value;
        value = newValue;
        return oldValue;
    }
    //重写equals(),若不重写，则于'=='一样
    public final boolean equals(Object o) {
        //==判断的是内容和类型等
        if (o == this)
            return true;
            //判断是否为Map.Entry的实例，若key的值与value的值相等，则说明是相等的。
        if (o instanceof Map.Entry) {
            Map.Entry<?,?> e = (Map.Entry<?,?>)o;
            if (Objects.equals(key, e.getKey()) &&
                Objects.equals(value, e.getValue()))
                return true;
        }
        return false;
    }
}
```
此类是红黑树的数据结构实现：
```java
//红黑树
static final class TreeNode<K,V> extends LinkedHashMap.Entry<K,V> {
    TreeNode<K,V> parent;  //父节点
    TreeNode<K,V> left;//左节点
    TreeNode<K,V> right;//右节点
    TreeNode<K,V> prev;    // needed to unlink next upon deletion
    boolean red;//决定红黑树的颜色节点
    TreeNode(int hash, K key, V val, Node<K,V> next) {
        super(hash, key, val, next);
    }

    /**
        * Returns root of tree containing this node.
        */
    final TreeNode<K,V> root() {
        for (TreeNode<K,V> r = this, p;;) {
            if ((p = r.parent) == null)
                return r;
            r = p;
        }
    }
    //移动红黑树的根节点到前面,以确保根节点是此目录的第一个节点
    static <K,V> void moveRootToFront(Node<K,V>[] tab, TreeNode<K,V> root);
``` 
assert boolean表达式：表达式为true，表示继续执行；如果为false，则程序抛出AssertionError，并终止执行。  
位桶：
```java
transient Node<k,v>[] table;//存储(位桶)的数组
```
# HashMap源码分析
## 类的继承关系
```java
public class HashMap<K,V> extends AbstractMap<K,V>
    implements Map<K,V>, Cloneable, Serializable
```
HashMap继承自父类(AbstractMap),实现了Map、Cloneable、Serializable接口。其中，Map接口定义了一组通用的操作；Cloneable接口则表示可以进行拷贝，在HashMap中，实现的是浅层次拷贝，即对拷贝对象的改变会影响被拷贝的对象；Serializable接口表示HashMap实现了序列化，即可以将HashMap对象保存至本地，之后可以恢复状态。  
HashMap继承了AbstractMap<K,V>,实现了Map<K,V>,Cloneable,Serializable接口。  
HashMap的实现不是同步的，即它不是线程安全的。它的Key、value都可以为null.此外，HashMap中的映射不是有序的。
## 类的属性
```java
public class HashMap<K,V> extends AbstractMap<K,V> implements Map<K,V>, Cloneable, Serializable {
    // 序列号
    private static final long serialVersionUID = 362498820763181265L;    
    // 默认的初始容量是16
    static final int DEFAULT_INITIAL_CAPACITY = 1 << 4;   
    // 最大容量
    static final int MAXIMUM_CAPACITY = 1 << 30; 
    // 默认的填充因子
    static final float DEFAULT_LOAD_FACTOR = 0.75f;
    // 当桶(bucket)上的结点数大于这个值时会转成红黑树
    static final int TREEIFY_THRESHOLD = 8; 
    // 当桶(bucket)上的结点数小于这个值时树转链表
    static final int UNTREEIFY_THRESHOLD = 6;
    // 桶中结构转化为红黑树对应的table的最小大小
    static final int MIN_TREEIFY_CAPACITY = 64;
    // 存储元素的数组，总是2的幂次倍
    transient Node<k,v>[] table; 
    // 存放具体元素的集
    transient Set<map.entry<k,v>> entrySet;
    // 存放元素的个数，注意这个不等于数组的长度。
    transient int size;
    // 每次扩容和更改map结构的计数器
    transient int modCount;   
    // 临界值 当实际大小(容量*填充因子)超过临界值时，会进行扩容
    int threshold;
    // 填充因子
    final float loadFactor;
}
```
```java
 Map<String,Object> map = new HashMap<>(16);
        map.put(null,null);
        map.put("va",null);
        map.put(null,"ajk");
```
```
输出结果：
null===========ajk
va===========null
```
HashMap的两个实例有两个参数影响其性能:"初始容量"和"加载因子"。  
容量是哈希表中桶的数量，初始容量是哈希表在创建时的容量。  
加载因子是哈希表在其容量自动增加之前可以达到多满的一种尺度。当哈希表中的条目数超出了加载因子与当前容量的乘积时，则要对该哈希表进行rehash操作(即重建内部数据结构)，从而哈希表将具有大约两倍的桶数。  
## HashMap的构造函数
```java
//默认的构造方法
 public HashMap() {
        this.loadFactor = DEFAULT_LOAD_FACTOR; //默认的加载因子 为0.75
}
//给定初始容量的构造方法
//关于HashMap的初始容量 
//static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16
public HashMap(int initialCapacity) {
        this(initialCapacity, DEFAULT_LOAD_FACTOR);
}
//设置HashMap的初始容量和加载因子
public HashMap(int initialCapacity, float loadFactor) {
    //对初始容量进行判断 <0 就会抛出异常
    if (initialCapacity < 0)
        throw new IllegalArgumentException("Illegal initial capacity: " + initialCapacity);
    //如果初始容量>最大容量 最大容量为2^30  就把初始容量设置为最大容量
    if (initialCapacity > MAXIMUM_CAPACITY)
        initialCapacity = MAXIMUM_CAPACITY;
        //对加载因子的判断 若加载因子<=0或加载因子为整数时，抛出异常
    if (loadFactor <= 0 || Float.isNaN(loadFactor))
        throw new IllegalArgumentException("Illegal load factor: " +
                                            loadFactor);
    this.loadFactor = loadFactor;
    //threshold是HashMap所能容纳的最大数据量的Node(键值对)个数。
    //threshold = length * Load factor.即数组定义好长度之后，负载因子越大，所能容纳的键值对个数越多。如果超过定义好的threshold，就需要进行扩容。
    this.threshold = tableSizeFor(initialCapacity);
}
public HashMap(Map<? extends K, ? extends V> m) {
    this.loadFactor = DEFAULT_LOAD_FACTOR;
    putMapEntries(m, false);
}
```  
HashMap构造函数中的其他方法：
```java
//由于构造Hash时，需要保证初始容量为2^n，此函数就是为了将初始容量修剪称最接近的2^n数 HashMap中的初始容量为2^n的原因是：减少Hash冲突。
static final int tableSizeFor(int cap) {
    int n = cap - 1;
    n |= n >>> 1;
    n |= n >>> 2;
    n |= n >>> 4;
    n |= n >>> 8;
    n |= n >>> 16;
    return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
}
final void putMapEntries(Map<? extends K, ? extends V> m, boolean evict) {
    //将m的所有元素存入到本地HashMap实例中
    int s = m.size();
    if (s > 0) {
        //判断table是否初始化
        if (table == null) { // pre-size
        //如果table未初始化，则将s作为m的初始容量大小
            float ft = ((float)s / loadFactor) + 1.0F;
            //如果初始容量小于最大容量，就以初始容量为初始容量，否则，以最大容量为初始容量。
            int t = ((ft < (float)MAXIMUM_CAPACITY) ?
                        (int)ft : MAXIMUM_CAPACITY);
            //负载因子*容量 是一个阈值 
            if (t > threshold)
            //tableSizeFor(t)：返回大于t的最小二次幂数值
                threshold = tableSizeFor(t);
        }
        //table已经初始化过了，如果m的长度已经大于扩容的临界值时，需要进行扩容
        else if (s > threshold)
            resize();
        //将m中的所有元素添加到HashMap中
        for (Map.Entry<? extends K, ? extends V> e : m.entrySet()) {
            K key = e.getKey();
            V value = e.getValue();
            putVal(hash(key), key, value, false, evict);
        }
    }
}
```
# hash算法
除留余数法：异或的原因：使得高位也可以参与运算，减少碰撞.  
(n-1)&hash:实际上就是除留。只是&运算的效率高于%
```java
static final int hash(Object key) {
    int h;
    //判断key是否为null，如果不为null,则就将key.hashCode与h进行异或
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```
# HashMap的扩容
+ HashMap中的键值对大于阈值或者初始化时，就调用resize()方法进行扩容；
+ 每次扩展的时候都是扩展2倍；
+ 扩展后的Node对象要么在原位置，要么移动到原偏移量两倍的位置。
## 什么时候进行扩容？
+ 当前容量超过阈值；
+ 当链表中的元素个数超过设定(8)个，当数组的大小还未超过64的时候，此时进行数组的扩容，如果超过则将链表转化为红黑树。
```java
final Node<K,V>[] resize() {
    Node<K,V>[] oldTab = table;//oldTab指向Hash桶数组
    int oldCap = (oldTab == null) ? 0 : oldTab.length;//定义oldTab的容量
    int oldThr = threshold;
    int newCap, newThr = 0;
    //如果oldCap大于0，即table大于0
    if (oldCap > 0) {
        //如果大于最大容量
        if (oldCap >= MAXIMUM_CAPACITY) {
           //临界值变为最大整数值128
            threshold = Integer.MAX_VALUE;//-127~128
            return oldTab;
        }
        //如果oldCap<<1 即oldCap^2（扩容后）小于最大容量并且oldCap>16
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                    oldCap >= DEFAULT_INITIAL_CAPACITY)
                    //此时就执行扩容
            newThr = oldThr << 1; // double threshold
    }
    //如果临界值大于0，就让临界值成为新的容量值
    else if (oldThr > 0) // initial capacity was placed in threshold
        newCap = oldThr;
    else {               // zero initial threshold signifies using defaults
        newCap = DEFAULT_INITIAL_CAPACITY;
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
    }
    if (newThr == 0) {
        float ft = (float)newCap * loadFactor;
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                    (int)ft : Integer.MAX_VALUE);
    }
    threshold = newThr;
    //1、因为初始化散列表时复用了resize()方法，所以前面是对oldTab的判断，是否为0(表示是初始化)，是否已经大于等于了最大容量，判断结束后，newTab会扩大为oldTab的两倍，同时newThr(临界值)也为原来的两倍。
    @SuppressWarnings({"rawtypes","unchecked"})
    //2、确定好newTab的大小后接下来就是初始化newTab散列表结构。
    Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
    //3、如果是初始化，则直接返回新的散列表结构，不是则进行转移。
    table = newTab;
    //-----------------------以上为初始化的扩容-------------------------
    //4、首先是进行遍历
    if (oldTab != null) {
        for (int j = 0; j < oldCap; ++j) {
            Node<K,V> e;
            if ((e = oldTab[j]) != null) {//如果旧的hash桶数组在j结点处不为空，复制给e
                oldTab[j] = null;//将旧的hash桶数组在j结点处设置为空，方便gc
                if (e.next == null)//如果e后面没有Node结点
                    newTab[e.hash & (newCap - 1)] = e;//直接对e的hash值对新的数组长度求模获得存储位置
                else if (e instanceof TreeNode)//如果e是红黑树的类型，那么添加到红黑树中
                    ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                else { // preserve order
                    Node<K,V> loHead = null, loTail = null;
                    Node<K,V> hiHead = null, hiTail = null;
                    Node<K,V> next;
                    do {
                        next = e.next;//将Node结点的next赋值给next
                        if ((e.hash & oldCap) == 0) {//如果结点e的hash值与原hash桶数组的长度作与运算为0
                            if (loTail == null)//如果loTail为null
                                loHead = e;//将e结点赋值给loHead
                            else
                                loTail.next = e;//否则将e赋值给loTail.next
                            loTail = e;//然后将e复制给loTail
                        }
                        else {//如果结点e的hash值与原hash桶数组的长度作与运算不为0
                            if (hiTail == null)//如果hiTail为null
                                hiHead = e;//将e赋值给hiHead
                            else
                                hiTail.next = e;//如果hiTail不为空，将e复制给hiTail.next
                            hiTail = e;//将e复制个hiTail
                        }
                    } while ((e = next) != null);//直到e为空
                    if (loTail != null) {//如果loTail不为空
                        loTail.next = null;//将loTail.next设置为空
                        newTab[j] = loHead;//将loHead赋值给新的hash桶数组[j]处
                    }
                    if (hiTail != null) {//如果hiTail不为空
                        hiTail.next = null;//将hiTail.next赋值为空
                        newTab[j + oldCap] = hiHead;//将hiHead赋值给新的hash桶数组[j+旧hash桶数组长度]
                    }
                }
            }
        }
    }
    return newTab;
}
```
<font color="red">扩容操作需要将旧 table 的所有键值对重新插入到新的 table 中。</font>
# 重要方法
## putVal()
put()方法通过putVal()方法来插入数据。
```java
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    if ((tab = table) == null || (n = tab.length) == 0)
    //判断键值对数组table[i]是否为空或者为null,否则进行resize()扩容.
        n = (tab = resize()).length;
        //(n - 1) & hash 是用来确定元素在桶中的位置的，如果桶为空，就新生成节点放入桶中(此时这个节点是放在桶中的)
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);
    //桶中已经存在元素
    else {
        Node<K,V> e; K k;
         //比较桶中第一个元素(数组中的节点)的hash值相等，key相等。
        if (p.hash == hash &&((k = p.key) == key || (key!= null && key.equals(k))))
            //将第一个元素赋值给e,用e来记录
            e = p;
            //如果p是红黑树
        else if (p instanceof TreeNode)
        //放入树中
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        else {
            //说明此时是在链表中
            for (int binCount = 0; ; ++binCount) {
                //如果e的下一个节点为null，即e为根节点，就新建一个节点插入到尾部
                if ((e = p.next) == null) {
                    p.next = newNode(hash, key, value, null);
                    //节点数目达到转换阈值8，转换为红黑树
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        treeifyBin(tab, hash);
                    break;
                }
                //判断链表中的key值与插入元素的key值是否相等，相等的话，就跳出循环
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                //// 用于遍历桶中的链表，与前面的e = p.next组合，可以遍历链表
                p = e;
            }
        }
        //表示在桶中找到key值，hash值与插入元素相等的节点
        if (e != null) { // existing mapping for key
            //记录e的value值
            V oldValue = e.value;
            //onlyIfAbsent为false或者旧值为null
            if (!onlyIfAbsent || oldValue == null)
              //用新值替换旧值
                e.value = value;
                //返回后回调
            afterNodeAccess(e);
            return oldValue;
        }
    }
    //结构性修改
    ++modCount;
    //实际大小大于阈值则扩容
    if (++size > threshold)
        resize();
        //插入后回调
    afterNodeInsertion(evict);
    return null;
}
```
## getNode()
get()方法就是通过getNode()来实现的。
```java
final Node<K,V> getNode(int hash, Object key) {
    Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
    //table已经初始化，长度大于0，根据hash寻找table中的项也不为空
    if ((tab = table) != null && (n = tab.length) > 0 &&(first = tab[(n - 1) & hash]) != null) {
        // 桶中第一项(数组元素)相等
        if (first.hash == hash && // always check first node
            ((k = first.key) == key || (key != null && key.equals(k))))
            return first;
            //桶中不止有一个节点
        if ((e = first.next) != null) {
            //是红黑树
            if (first instanceof TreeNode)
            //在红黑树中查找
                return ((TreeNode<K,V>)first).getTreeNode(hash, key);
                //否则在链表中查找
            do {
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    return e;
            } while ((e = e.next) != null);
        }
    }
    return null;
}
```
## 为什么初始容量需要为2^n:
举例说明：初始长度为15和16时 Hash值为8和9时 hash&(cap-1)得到Hash位置
+ 初始长度为15，Hash为8,hash&(cap-1) 8&14 1000&1110  1000 
+ 初始长度为15，Hash为9,hash&(cap-1) 9&14 1001&1110  1000  
此时，8和9的位置发生了冲突  
+ 初始长度为16，Hash为8，hash&(cap-1) 8&15 1000&1111 1000
+ 初始长度为16，Hash为9，hash&(cap-1) 9&15 1001&1111 1001  
此时，8和9位置并没有发生冲突。  
这样，（1）在遍历时，初始长度为16的HashMap明显比15的时间复杂度低。也不需要进行冲突解决。（2）在初始容量为15时，最低位永远位0，会浪费掉hash值最低位为1的位置。而初始容量为16时，同时兼顾了最高位和最低位，不存在浪费。  
```java
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    //hashCode()是Object类中的方法 是一个native方法 本地方法 native方法主要用于加载文件和动态链接库 Native意味着与平台有关 可移植性不高。
    //^异或 相同为0 不同为1
}
```
# HashMap的默认长度为16的原因：
如果两个元素不相同,但是hash函数的值相同,这两个元素就是一个碰撞
因为把任意长度的字符串变成固定长度的字符串,所以存在一个hash对应多个字符串的情况,所以碰撞必然存在。  
为了减少hash值的碰撞,需要实现一个尽量均匀分布的hash函数,在HashMap中通过利用key的hashcode值,来进行位运算
公式:index = e.hash & (newCap - 1)  
举个例子:
1.计算"book"的hashcode
    十进制 : 3029737  
    二进制 : 101110001110101110 1001  
2.HashMap长度是默认的16，length - 1的结果  
    十进制 : 15  
    二进制 : 1111  
3.把以上两个结果做与运算  
    101110001110101110 1001 & 1111 = 1001
    1001的十进制 : 9,所以 index=9  
hash算法最终得到的index结果,取决于hashcode值的最后几位  
为了推断HashMap的默认长度为什么是16
现在,我们假设HashMap的长度是10,重复刚才的运算步骤:
hashcode : 101110001110101110 1001
length - 1 :                                     1001
index :                                            1001
再换一个hashcode 101110001110101110 1111 试试:
hashcode : 101110001110101110 1111
length - 1 :                                     1001  
index :                                            1001  
从结果可以看出,虽然hashcode变化了,但是运算的结果都是1001,也就是说,当HashMap长度为10的时候,有些index结果的出现几率
会更大而有些index结果永远不会出现(比如0111),这样就不符合hash均匀分布的原则  
反观长度16或者其他2的幂,length - 1的值是所有二进制位全为1,这种情况下,index的结果等同于hashcode后几位的值  
只要输入的hashcode本身分布均匀,hash算法的结果就是均匀的
所以,HashMap的默认长度为16,是为了降低hash碰撞的几率
# HashMap的遍历
# transient的使用
+ 用途：当一个对象实现了Serilizable接口，这个对象就可以被序列化，我们不关心其内在的原理，只需要了解这个类实现了Serilizable接口，这个类的所有属性和方法都会自动序列化。而在开发过程中，我们可能要求：当对象被序列化时(写入字节序列到目标文件)，有些属性需要序列化，而其它属性不需要被序列化。这个字段的生命周期仅存于调用者的内存中而不会写到磁盘里持久化。  
所以，transient的用途在于：阻止实例中那些用此关键字声明的变量持久化；当对象被反序列化时(从源文件读取字节序列进行重构)，这样的实例变量值不会被持久化和恢复。
+ 使用总结：   
    + 一旦变量被transient修饰，变量将不再是对象持久化的一部分，该变量内容在序列化后无法获得访问；
    + transient关键字只能修饰变量，而不能修饰方法和类。注意，本地变量是不能被transient关键字修饰的。变量如果是用户自定义的类，则该类需要实现Serializable接口；
    + 被transient关键字修饰的变量不再能被序列化，一个静态变量不管是否被transient修饰，均不能被序列化。
```java
private static String name;
```
此时name被static修饰的，但是name的值依然可以被输出，原因是：反序列化后类中static型变量name的值为当前JVM中对应static变量的值，这个值是JVM中的不是反序列化得出的。
# ConcurrentHashMap
