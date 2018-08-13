 
 https://www.cnblogs.com/LittleHann/p/3690187.html
Collection     
├List   
│├LinkedList   
│├ArrayList   
│└Vector   
│　└Stack   
└Set   
├HashSet   
└TreesSet   
Map   
├Hashtable   
├HashMap   
└WeakHashMap   
# Java集合类
+ Collection 一组"对立"的元素，通常这些元素都服从某种规则；
    + List必须保证元素特定的顺序
        + LinkedList
        + ArrayList
        + Vector
            + Stack
    + Set不能有重复元素
        + HashSet
        + TreeSet
    + Queue保持一个队列(先进先出)的顺序
+ Map 一组"键值对"对象 
    + Hashtable
    + HashMap
    + WeakHashMap
  
Collection和Map的区别在于容器中每个位置保存的元素个数：
+ Collection 每个位置只能保存一个元素
+ Map保存的是"键值对"，可以通过键找到对应的值。
# Java集合类架构
## Interface Iterable
迭代器接口，是Collection的父接口。实现这个接口的对象允许使用foreach进行遍历，也就是说，所有的Collection集合对象都具有“foreach可遍历性”。这个Iterable接口只有一个方法：iterator().它返回一个代表当前集合对象的泛型$<T>$迭代器。用于之后的遍历操作。
### Collection
Collection是最基本的集合接口，一个Collection代表一组Object的集合，这些Object被称作Collection的元素。Collection是一个接口，用来提供规范定义，不能被实例化使用。  
#### 1.Set
Set集合里的多个对象之间没有明显的顺序。Set继承自Collection接口，不能包含有重复元素。  
Set判断两个对象相同不是使用"=="运算符，而是根据equals方法。也就是说，我们在加入一个新元素的时候，如果这个新元素对象和Set中已有对象进行equals比较，都返回false,则Set会接收新元素对象，否则拒绝。  
在Set中，应注意两点：
+ 为Set集合里的元素的实现类实现一个有效的equals(Object)方法；
+ 对Set构造函数，传入的Collection参数不能包含重复的元素。
> HashSet  

HashSet是Set接口的典型实现，HashSet使用Hash算法来存储集合中的元素，因此具有良好的存取和查找性能。当向HashSet集合中存入一个元素时，HashSet会调用该对象的hashCode()方法来得到该对象的hashCode值，然后根据该hashCode值决定该对象在HashSeT中的存储位置。  
<font color="red">HashSet集合判断两个相等的标准是两个对象通过equals()方法比较相等，并且两个对象的hashCode()方法的返回时相等。</font>  
+ LinkedHashSet
> SortedSet

此接口主要用于排序操作，即实现此接口的子类都属于排序的子类。
+ TreeSet是Sorted接口的实现类，TreeSeT可以确保集合元素处于排序状态。
> EnumSet

是一个专门为枚举类设计的集合类，EnumSet中所有元素都必须是指定枚举类型的枚举值，该枚举类型在创建EnumSet时显式、或隐式的指定，EnumSet的集合元素也是有序的，他们以枚举值在Enum类中的定义顺序来决定集合元素的顺序。
 #### 2.List
 > ArrayList 

 是基于数组实现的List类，它封装了一个动态的增长的、允许再分配的Object[]数组。
 > Vector

 Vector和ArrayList在用法上基本没有区别。JDK1.2以前使用，之后出来了集合框架，就将Vector改为实现List接口。
 + Stack:是Vector提供的一个子类，用于模拟"栈"
> LinkedList

继承AbstractSequentialList<E>，实现List<E>,能对它进行队列操作，即可以根据索引来随机访问集合中的元素。实现Deque<E>,能将LinkedList当作双端队列使用，也可以作为“栈来使用”， 实现Cloneable和 java.io.Serializable
 #### 3.Queue
 用来模拟“队列”。队列的头部放着队列中存放时间最长的元素，队列的尾部存放着队列中存放时间最短的元素，新元素插入到队列的尾部，访问元素poll操作会返回队列头部的元素，队列不允许随机访问队列中的元素。
 > PriorityQueue

 并不是一个标准的队列实现，PriorityQueue保存队列元素的顺序并不是按照加入队列的顺序，而是按照队列元素的大小重新排序。
 > Deque

 代表一个"双端队列"，可以从两端来添加、删除元素，可当作队列，也可当作“栈”。
 + ArrayDeque 是一个基于数组的双端队列,和ArrayList类似，他们的底层都采用一个动态的、可重分配的Object[]数组来存储集合元素，当集合元素超出该数组的容量时，系统会在底层重新分配一个Object[]数组类存储集合元素。
 + LinkedList
 ### Map
 Map用于保存具有"映射关系"的数据，因此Map集合里保存着两组值，一组用于保存Map里的key,另外一组值用于保存Map里的value.key和value都可以是任何引用类型的数据。Map的key不允许重复，即同一个Map对象的任何两个Key通过equals()方法比较结果总是返回false.  
 Java是先实现了Map，然后通过包装了一个所有value都为null的Map就实现了Set集合。  
 Map的这些实现类和子接口中key集的存储形式和Set集合完全相同（即key不能重复）  
Map的这些实现类和子接口中value集的存储形式和List非常类似(即value值可以重复、根据索引来查找)
 #### 1.HashMap
 和HashSeT集合不能保证元素的顺序一样，HashMap也不能保证key-value对的顺序。并且类似于HashSet判断两个key是否相的标准也是：两个key通过equals()方法比较返回true，同时两个key的hashCode值也必须相等。
 + LinkedHashMap 也使用双向链表来维护key-value对的次序，该链表负责维护Map的迭代顺序，与key-value对的插入顺序一致。
 #### 2.HashTable
 是一个古老的Map实现类。
 > Properties 

 Properties类可以把Map对向和属性文件关联起来，从而可以把Map对象中的key-value对写如到属性文件中，也可以把属性文件中的"属性名-属性值"加载到Map对象中。
 #### 3.SortedMap
 > TreeMap

 TreeMap就是一个红黑树结构，每个key-value对即作为红黑树的一个节点。TreeMap存储key-value对(节点)时，需要根据key对节点进行排序。TreeMap可以保证所有的key-value处于有序状态。同样，TreeMap也有两种排序方式：自然排序、定制排序。
 #### 4.WeakHashMap
 与HashMap用法基本相似。区别在于，HashMap的key保留了对实际对象的"强引用"，这意味着只要该HashMap对象不被销毁，该HashMap所引用的对象就不会被垃圾回收。  
 但WeakHashMap的key只保留了对实际对象的弱引用，这意味着如果WeakHashMap对象Key所引用的对象没有被其它强引用变量所引用，则这些key所引用的的对象可能被垃圾回收，当垃圾回收了该key所对应的实际对象之后，WeakHashMap也可能自动删除这些key所对应的key-value对。
 #### 5.IdentifyHashMap
 与HashMap的实现机制基本相似，当且仅当两个key严格相等时(ket1==key2),才会认为两个key相等。
 #### 6.EnumMap