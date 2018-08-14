# Java集合类的应用场景 
## Set
equals()决定是否可以加入HashSeT,而HashCode决定存放的位置，他们两者必须同时满足才能允许一个新元素加入。  
如果两个元素的hashCode相同，但是他们的equals()返回值不同，HashSet会在这个位置用链式结构来保存多个对象，而HashSet访问集合元素时也是根据元素的HashCode值来快速定位的，这种链式结构会导致性能下降。  
## TreeSet
采用两种排序方法：自然排序、定制排序。  
为了解决如何比较对象大小的问题，JDK提供了两个接口java.lang.Comparable和java.util.Comparator 
> Collections.sort的用法：

> 自然排序:java.lang.Comparable  

只对实现了Collection接口的集合有效

Comparable中只提供了一个方法:CompareTo(Object object),该方法的返回值时int.如果返回为正数，则表示当前对象(调用该方法的对象)比obj对象"大"；反之"小"；如果为0的话，则表示两对象相等。
```java
//该方法中的泛型T都是Comparable接口的子类，即只有是Comparable接口子类类型的数据，才能进行比较排序。如果其它类型的数据要进行比较排序，必须继承Comparable接口并重写equals()和comparaTo()方法。
 public static <T extends Comparable<? super T>> void sort(List<T> list);
```
> 比较器排序：java.util.Comparator

需要对同一对象进行多种不同方式的排序，这点自然排序不能实现。Comparator接口的一个好处是：将比较排序算法和具体的实现类分离。  
可以在不能够更改具体的实体类时，使用Comparator，通过匿名内部类来实现排序。
```java
 //该方法中指定比较方式Comparator<? super T> c,即c必须实现Comparator<? super T> c这个接口。   
 public static <T> void sort(List<T> list, Comparator<? super T> c)
```
+ 对于Integer或String这些已经实现Comparable接口的类来说，可以直接使用Collection.sort方法传入list参数来实现默认方式(正序)排序；
+ 如果不想使用默认方式(正序)排序，可以通过Collections.sort传入第二个参数类型为Comparator来自定义排序规则；
+ 对于自定义类型，如果想使用Collections.sort的方式进行自定义排序，可以通过实现Comparable接口的compareTo方法来进行；
+ jdk1.8的Comparator接口有很多新增的方法，其中有个reversed()方法，可以切换正序和逆序。  
## Set集合的应用场景
+ HashSet的性能总是比TreeSet好(特别是常用的添加、查询元素等操作),因为TreeSet需要额外的红黑树算法来维护集合元素的次序。只有当需要一个保持排序的Set时，才应该使用TreeSet。否则都应该使用HashSet。
+ 对于普通的插入、删除操作，LinkedHashSet比HashSet要略慢一些，这是由维护链表所带来的开销造成的。不过，因为链表的存在，遍历LinkedHashSet会更快。
+ EnumSet是所有Set实现类中性能最好的，但它只能保存同一个枚举类的枚举值作为集合元素；
+ HashSet、TreeSet、EnumSet都是"线程不安全"的，通常可以通过Collections工具类的synchronizedSortedSet方法来"包装"该Set集合。
```java
SortedSet s = Collections.synchronizedSortedSet(new TreeSet(....))
```
# List
## ArrayList
如果一开始就知道ArrayList集合需要保存多少元素，则可以在创建他们时就指定initialCapacity大小，这样可以减小重新分配的次数，提供性能，ArrayList还提供了如下方法来重新分配Object[]数组：
```java
ensureCapacity(int minCapacity)//将ArrayList集合的Object[]数组长度增加minCapacity;
trimToSize()://调整ArrayList集合的Object[]数组长度为当前元素的个数。程序可以通过此方法来减少ArrayList集合对象占用的内存空间
```
## Stack
后进先出
# ArrayList的应用场景
+ Java提供的List就是一个"线性表接口"，ArrayList（基于数组的线性表）、LinkedList（基于链表的线性表）是线性表的两种典型实现；
+ 因为数组以一块连续内存来保存所有的数组元素，所以数组在随机访问时性能最好，所以内部以数组作为底层实现的集合在随机访问时性能最好；
+ 内部以链表作为底层实现的集合在执行插入、删除操作时有很好的性能；
+ 进行迭代操作时，以链表为底层的集合比以数组为底层实现的集合性能好。
# 遍历
Collection接口继承了Iterable接口，即实现Collection接口的类都具有"可遍历性"。  
Iterable向应用程序提供了遍历Collection集合元素的统一编程接口：
```java
boolean hasNext():是否还有下一个未遍历过的元素
Object next():返回集合里的下一个元素
void remove():删除集合里上一次next方法返回的元素
```
## Iterator实现遍历
```java
//Set是无序的，不能重复的。若要对Set进行排序，则需要TreeSet
Collection collection = new HashSet();
collection.add("张三");
collection.add("李四");
collection.add("王五");

Iterator iterator = collection.iterator();
while (iterator.hasNext()) {
    String name = (String) iterator.next();
    System.out.println(name);
}
```
## foreach实现遍历
```java
for (Object object: collection ) {
    System.out.println(object);
}
```
## ListIterator实现遍历List
List是有序的，可以重复。
```java
boolean hasPrevious();返回该迭代器关联的集合是否还有上一个元素
Object previous();返回该迭代器上一个元素(向前迭代)
void add();在指定位置插入一个元素
```
```java
 List<Student> map =  new ArrayList<>(16);
    map.add(new Student("c","d"));
    map.add(new Student("d","c"));
    map.add(new Student("d","a"));
    map.add(new Student("f","e"));

    ListIterator listIterator = map.listIterator();

    while (listIterator.hasNext()) {
        System.out.println(listIterator.next());
    }
```
# Map
当使用自定义类作为HashMap、HashTable的key时，如果重写该类的equals(Object obj)和hashCode()方法，则应该保证两个方法的判断标准一致--当两个key通过equals()方法比较返回true时，两个key的hashCode()的返回值也应该相同。  
TreeMap中判断两个key相等的标准：
1. 两个key通过comparaTo()方法返回0；
2. equals()返回true.
# Map集合应用场景
1. HashMap和HashTable的效率大致相同，因为他们的实现机制大致相同，但HashMap通常比HashTable要快一点，因为HashTable需要额外的线程同步控制。
2. TreeMap通常比HashMap、HashTable要慢(尤其是在插入、删除key-value对时更慢)，因为TreeMap底层采用红黑树来管理key-value对。
3. 使用TreeMap的一个好处就是：TreeMap中的key-value对总是处于有序状态，无需进行专门的排序操作。