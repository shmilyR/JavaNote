# equals() 
+ 自反性 x.equal(x);//true
+ 对称性 x.equal(y) == y.equal(x);true
+ 传递性 if(x.equal(y) && y.equal(z)) x.equal(z);//true
+ 一致性 多次调用equals()方法 结果不变 x.equals(y) == x.equals(y);//true
+ 与null的比较 对任何不是null的对象x调用x.equals(null) 结果都为false x.equals(null) //false
### equals()和==
+ ==比较的是变量(栈)内存中存放的对象的(堆)内存地址，用来判断两个对象的地址是否相同，即是否是指相同一个对象。比较的是真正意义上的指针操作。
    + 比较的是操作符两端的操作数是否是同一个对象；
    + 两边的操作数必须是同一类型的(可以是父子类之间))才能编译通过；
    + 比较的是地址，如果是具体的阿拉伯数字的比较，值相等则为true.
```java
int i = 10;
long j = 10L;
float k = 10;

System.out.println(i==j);
System.out.println(i==k);
```
输出结果：
```java
true
true
```
+ equals():equals()用来比较的是两个对象的内容是否相等，由于所有的类都是继承自java.lang.Object类的，所以适用于所有对象，如果没有对该方法进行覆盖的话，调用的仍然是Object类中的方法，而Object中的equals()方法返回的是==的判断。  
Object中的equals()源码：
```java
public static boolean equals(Object a, Object b) {
    return (a == b) || (a != null && a.equals(b));
}
```
String是Java中唯一不需要new就可以产生对象的途径。
```java
String string = "kakka";//先在内存中找是否有"kakka"这个对象，如果有，就让string指向"kakka",如果没有，就创建一个新的对象保存"kakka".
String str = new String("kakka");//不管内存里是不是已经有"kakka"这个对象，都新建一个对象来保存。
```
对于String的测试：
```java
String string = "kakak";
String string3 = "kakak";
String string1 = new String("kakak");
String string2 = new String("kakak");
System.out.println(string==string1);//使用==时，判断的是类型和内容
System.out.println(string.equals(string1));//使用equals()时，判断的是内容
System.out.println(string==string3);
System.out.println(string1.equals(string2));
```
输出结果：
```java
false
true
true
true
```
<font color="red">"=="和equals()的区别就在于对象是否对equals()方法进行了重写，若没有进行重写，则"=="和equals()一样，由源码可知。</font>  
## instanceof运算符
instanceof操作符用于判断一个引用类型所引用的对象是否是一个类的实例。
+ 对于引用类型变量，Java编译器只根据变量被先声明的类去编译。
+ instanceof左边操作元被显式声明的类型与与右边操作元必须是同种类或者有继承的关系，即位于继承树的同一个继承分支上，否则编译出错。  

在比较时，一般先比较HashCode，如果HashCode相等，再比较equals()方法，否则，不相等。