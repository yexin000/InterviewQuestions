
## Java基础

#### 1.HashMap源码，实现原理，JDK8中对HashMap做了怎样的优化
##### 什么是哈希表

哈希表（HashTable，散列表）是根据key-value进行访问的数据结构，他是通过把key映射到表中的一个位置来访问记录，加快查找的速度，其中映射的函数叫做散列函数，存放记录的数组叫做散列表，哈希表的主干是数组。

![avatar](https://img-blog.csdn.net/20180325131913594?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxNzg2Njky/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

上图就是一个值插入哈希表中的过程，存在的问题是不同的值在经过hash函数之后可能会映射到相同的位置上，当插入一个元素时，如果该位置已经被占用，就会产生冲突，也就是所谓的哈希冲突，解决哈希冲突的方式：

    （1）开放定址法：当插入一个元素时，发生冲突，继续检查散列表的其他项，直到找到一个位置来放置这个元素，至于检查的顺序可以自定义；

    （2）再散列法：使用多个hash函数，如果一个发生冲突，使用下一个hash函数，直到找到一个位置，这种方法增加了计算的时间；

    （3）链地址法：在数组的位置使用链表，将同一个hashCode的元素放在链表中，HashMap就是使用的这种方法，数组+链表的结构。
    
##### 什么是HashMap
HashMap是基于哈希表的Map接口的实现，提供所有可选的映射操作，允许使用null值和null键，存储的对象时一个键值对对象Entry<K,V>;

HashMap是基于**数组+链表**的结构实现，在内部维护这一个数组table，数组的每个位置保存着每个链表的表头结点，查找元素时，先通过hash函数得到key值对应的hash值，再根据hash值得到在数组中的索引位置，拿到对应的链表的表头，最后去遍历这个链表，得到对应的value值。

![avatar](https://img-blog.csdn.net/20180325134901559?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxNzg2Njky/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

HashMap存储的对象是一个**Entry**实体，Entry是HashMap中的一个静态内部类。

HashMap的初始化：

    在指定了初始化容量和负载因子时，首先会对参数进行检查，符合条件后，会根据初始化容量进行调整，使其必须是2的次幂，因此即使用户在使用时指定了容量，实际初始化的容量不一定等于指定的容量，在设置好容量之后，根据当前容量和负载因子计算出下一次扩容时的阈值threshold，也就是map中的元素个数达到这个值之后就要进行扩容。
    
HashMap的put操作：

    由于HashMap中允许存在null的key，因此在插入元素的时候，需要对key值进行判断，其主要步骤为：

    （1）如果key为null值，单独进行处理，putForNullKey（value）；

    （2）如果key不是null值，根据key的hashCode再次计算hash值 hash(key.hashCode());

    （3）根据hash值和table数组的长度得到该元素在数组中的索引位置i=indexFor(hash,table.length);

    （4）取得该位置的链表，如果链表不为空，对链表进行遍历，判断key值在链表中是否存在，已存在用新的value值替代旧值，同时返回旧值；

    （5）否则，向该位置的链表的表头插入新的元素addEntry(hash,key,value,i)；

    在put操作中，最终决定元素存放的位置时需要对key的hashCode再一次进行异或、移位等操作，目的是为了使最终在数组中存储的位置尽可能的分布均匀。
    
HashMap的get操作：

    HashMap的get操作相比put操作就简单很多，首先是判断key值是否为null，如果是null，则直接调用getForNummKey()方法去table[0]的位置查找元素，拿到table[0]处的链表进行遍历，这里只需要判断链表中的key值是否是null，返回对应的value值。

    如果key值不是null，则需要对key.hashCode再次计算hash值，调用indexFor方法拿到在table中的索引位置，获取到对应的链表。拿到链表后，需要对链表进行遍历，判断hashCode值是否相等以及key值是否相等，共同判断key值是否在HashMap中存在，从而拿到对应的value值。在这里也就说明了为什么把一个对象放入到HashMap的时候，最好是重写hashCode()方法和equals方法，hashCode()可以确定元素在数组中的位置，而equals方法在链表比较的时候会用到。
    
##### JDK8中HashMap优化
由于当链表长度太长会严重影响HashMap的性能，在jdk1.8里加入了红黑树的实现，当链表的长度大于8时，转换为红黑树的结构。

#### 2.HashMap是怎样扩容的，为什么都是2的N次幂大小
创建一个新的数组，容量是当前数组容量的2倍，然后调用transfer方法，将旧数组中的元素复制到新的数组中，返回新的数组，每次的扩容操作都是将数组翻倍，然后对每一个元素重新计算位置。
因为HashMap定位元素位置时算法是：

    计算key的hashcode值，然后跟数组的长度-1做一次“与”运算（&）
2的N次幂长度-1的二进制地位都是1，在进行hash&(length-1)操作的时候，只要hash对应的最左边的差异位为0，能够保证所有的位置都可能被用到，所以设置长度为2的次幂的原因也是为了尽可能的保证所有的位置都可以被存储元素。

#### 3.HashMap、HashTable、ConcurrentHashMap的区别
Hashmap ，ConcurrentHashmap继承AbstractMap类，
而HashTable是继承自Dictionary类。

HashMap线程不安全，HashMap的key可为null，其他两个不行。

Hashtable线程安全，锁住整个对象，数组+链表，在线程竞争激烈的情况下效率非常低。

ConcurrentHashMap线程安全，CAS+同步锁(synchronized)，数组+链表+红黑树，避免对全局加锁，改成了局部加锁操作，极大地提高了并发环境下的操作速度。

#### 4.HashMap在高并发下如果没有处理线程安全会有怎样的安全隐患，具体表现是什么

Hashmap在并发环境下，可能出现的问题：

1、多线程put时可能会导致get无限循环，具体表现为CPU使用率100%；

原因：在向HashMap put元素时，会检查HashMap的容量是否足够，如果不足，则会新建一个比原来容量大两倍的Hash表，然后把数组从老的Hash表中迁移到新的Hash表中，迁移的过程就是一个rehash()的过程，多个线程同时操作就有可能会形成循环链表，所以在使用get()时，就会出现无限循环的情况

2、多线程put时可能导致元素丢失

原因：当多个线程同时执行addEntry(hash,key ,value,i)时，如果产生哈希碰撞，导致两个线程得到同样的bucketIndex去存储，就可能会发生元素覆盖丢失的情况

#### 5.java中四种修饰符的限制范围
|  | 类内部 | 本包 | 子类 | 外部包 |
| ------ | ------ | ------ | ------ | ------ |
| public | √ | √ | √ | √ |
| protected | √ | √ | √ | × |
| default  | √ | √ | × | × |
| private | √ | × | × | × |

区别：

（1）public：可以被所有其他类所访问。

（2）private：只能被自己访问和修改。

（3）protected：自身，子类及同一个包中类可以访问。

（4）default（默认）：同一包中的类可以访问

#### 6.Object类中的方法
 1. registerNatives()   //私有方法
 
 2. getClass()    //返回此 Object 的运行类。
 
 3. hashCode()    //用于获取对象的哈希值。
 
 4. equals(Object obj) //用于确认两个对象是否“相同”。
 
 5. clone()    //创建并返回此对象的一个副本。 
 
 6. toString()   //返回该对象的字符串表示。   
 
 7. notify()    //唤醒在此对象监视器上等待的单个线程
 。   
 8. notifyAll()     //唤醒在此对象监视器上等待的所有线程。  
 
 9. wait(long timeout)    //在其他线程调用此对象的 notify() 方法或 notifyAll() 方法，或        者超过指定的时间量前，导致当前线程等待。 
 
10. wait(long timeout, int nanos)    //在其他线程调用此对象的 notify() 方法或 notifyAll() 方法，或者其他某个线程中断当前线程，或者已超过某个实际时间量前，导致当前线程等待。

11. wait()    //用于让当前线程失去操作权限，当前线程进入等待序列

12. finalize()    //当垃圾回收器确定不存在对该对象的更多引用时，由对象的垃圾回收器调用此方法。

#### 7.接口和抽象类的区别
接口的方法默认是 public，所有方法在接口中不能有实现(Java 8 开始接口方法可以有默认实现），抽象类可以有非抽象的方法

接口中的实例变量默认是 final
类型的，而抽象类中则不一定

一个类可以实现多个接口，但最多只能实现一个抽象类

一个类实现接口的话要实现接口的所有方法，而抽象类不一定

接口不能用 new 实例化，但可以声明，但是必须引用一个实现该接口的对象 从设计层面来说，抽象是对类的抽象，是一种模板设计，接口是行为的抽象，是一种行为的规范。

备注:在JDK8中，接口也可以定义静态方法，可以直接用接口名调用。实现类和实现是不可以调用的。如果同时实现
两个接口，接口中定义了一样的默认方法，必须重写，不然会报错

#### 8.动态代理的两种方式以及区别

**JDK动态代理**：利用反射机制生成一个实现代理接口的匿名类，在调用具体方法前调用InvokeHandler来处理。

**CGlib动态代理**：利用ASM（开源的Java字节码编辑库，操作字节码）开源包，将代理对象类的class文件加载进来，通过修改其字节码生成子类来处理。

**区别**：JDK代理只能对实现接口的类生成代理；CGlib是针对类实现代理，对指定的类生成一个子类，并覆盖其中的方法，这种通过继承类的实现方式，不能代理final修饰的类。

#### 9.java序列化的方式
1、Java原生序列化
Java原生序列化方法即通过Java原生流(InputStream和OutputStream之间的转化)的方式进行转化。需要注意的是JavaBean实体类必须实现Serializable接口，否则无法序列化。

2、Json序列化
Json序列化一般会使用jackson包，通过ObjectMapper类来进行一些操作，比如将对象转化为byte数组或者将json串转化为对象。

3、FastJson序列化
fastjson 是由阿里巴巴开发的一个性能很好的Java 语言实现的 Json解析器和生成器。特点：速度快，测试表明fastjson具有极快的性能，超越任其他的java json parser。功能强大，完全支持java bean、集合、Map、日期、Enum，支持范型和自省。无依赖，能够直接运行在Java SE 5.0以上版本 
支持Android。

#### 10.传值、传地址、传引用的区别
    传值   
    是把实参的值赋值给形参   
    那么对形参的修改，不影响实参的值   
        
    传地址   
    是传值的一种特殊方式，只是他传递的是地址，不是普通的如int   
    那么传地址以后，实参和形参都指向同一个对象   
        
    传引用   
    真正的以地址的方式传递参数   
    传递以后，形参和实参都是同一个对象，只是他们名字不同而已   
    对形参的修改将影响实参的值
    
基本类型（byte,short,int,long,double,float,char,boolean）为传值；对象类型（Object，数组，容器）为传引用；String、Integer、Double等immutable类型因为类的变量设为final属性，无法被修改，只能重新赋值或生成对象。当Integer作为方法参数传递时，对其赋值会导致原有的引用被指向了方法内的栈地址，失去原有的的地址指向，所以对赋值后的Integer做任何操作都不会影响原有值。

#### 11.一个ArrayList在循环过程中删除，会不会出现问题，为什么

ArrayList的remove方法，一种是根据下标删除，一种是根据元素删除。

for循环正向删除，会遗漏连续重复的元素，因为在删除元素后面的所有元素都要向前移动一个位置

for循环反向删除，没有问题（单线程），因为反向删除的时候，循环遍历完了的元素下标才有可能移动（已经遍历的元素，下标变化了也没有影响），所以没有遍历的下标不会移动

用Iterator循环删除的时候，调用ArrayList里面的remove方法，删除元素后modCount会增加，expectedModCount则不变，这样就造成了expectedModCount ！= modCount，会抛出异常

Iterator调用ArrayList的删除方法报错，Iterator调用迭代器自己的删除方法，单线程不会报错，多线程会报错。

forEach循环删除报错，因为编译器会自动将对forEach这个关键字的使用转化为对目标的迭代器的使用

#### 12.@Transactional在什么情况下会失效，为什么

@Transactional注解只能应用到public修饰符上，其它修饰符不起作用，但不报错

默认情况下此注解会对unchecked异常进行回滚，对checked异常不回滚。

    派生于Error或者RuntimeException（比如空指针，1/0）的异常称为unchecked异常。
    继承自Exception的异常统称为checked异常，如 IOException、TimeoutException等。
    如果是checked异常也想回滚的话，注解上写明异常类型即可@Transactional(rollbackFor=Exception.class)
    
#### 13.java集合类框架的基本接口有哪些
集合主要有Collection和Map接口。

List特点：元素有放入顺序，元素可重复。

Map特点：元素按键值对存储，无放入顺序。

Set特点：元素无放入顺序，元素不可重复。

List接口有三个实现类：LinkedList，ArrayList，Vector。

LinkedList：底层基于链表实现，链表内存是散乱的，每一个元素存储本身内存地址的同时还存储下一个元素的地址。链表增删快，查找慢。

ArrayList和Vector的区别：ArrayList是非线程安全的，效率高；Vector是基于线程安全的，效率低；ArrayList线程安全使用CopyOnWriteArrayList。

Set接口有两个实现类：HashSet(底层由HashMap实现)，LinkedHashSet。

SortedSet接口有一个实现类：TreeSet（底层由平衡二叉树实现）。

Map接口有三个实现类：HashMap，HashTable，LinkeHashMap。

HashMap非线程安全，高效，支持null；HashTable线程安全，低效，不支持null。

#### 14.HashSet和TreeSet有什么区别
HashSet：

不能保证元素的排列顺序，顺序有可能发生变化

集合元素可以是null,但只能放入一个null

HashSet底层是采用HashMap实现的

HashSet底层是哈希表实现的

TreeSet：

Treeset中的数据是排好序的，不允许放入null值。

TreeSet是通过TreeMap实现的,只不过Set用的只是Map的key。

TreeSet的底层实现是采用二叉树（红-黑树）的数据结构。

#### 15.LinkeHashMap的实现原理
LinkedHashMap是通过哈希表和链表实现的，它通过维护一个链表来保证对哈希表迭代时的有序性，而这个有序是指键值对插入的顺序。另外，当向哈希表中重复插入某个键的时候，不会影响到原来的有序性。

#### 16.为什么集合类没有实现Cloneable和Seriallizable接口
克隆（cloning）或者序列化（serialization）的语义和含义是跟具体的实现相关的。因此应该由集合类的具体实现类来决定如何被克隆或者序列化

#### 17.数组(Array)和列表(ArrayList)有什么区别，什么时候使用Array而不是ArrayList
Array可以包含基本类型和对象类型，ArrayList只能包含对象类型

数组中是可以间隔存null的，而ArrayList是做不到这一点的。

Array大小是固定的，ArrayList的大小是动态变化的，ArrayList每次存储时会检查空间大小，不够时会扩充为原来的1.5倍

ArrayList是List的实现类，具有更多可操作的方法，ArrayList提供了更多的方法和特性，比如：addAll()，removeAll()，iterator()等等。

对于基本类型数据，集合使用自动装箱来减少编码工作量。但是，当处理固定大小的基本数据类型的时候，这种方式相对比较慢

数组Array它的空间大小是固定的，空间不够时也不能再次申请，所以需要事前确定合适的空间大小。

ArrayList默认长度为10，当你插入值时，会先检查空间，如果不足，则扩充原来的1.5倍。扩充时会创建一个1.5倍的数组，然后复制原来的数据到新数组，摒弃旧数组（比较麻烦的地方）。

当数据长度不变时，尽量使用Array。

当长度不固定时，使用ArrayList，另外，如果需要频繁增加数据时，可预设ArrayList的长度，因为这样减少了复制数组的损耗。

如果我们需要对元素进行频繁的移动或删除，或者是处理的是超大量的数据，那么，使用ArrayList就真的不是一个好的选择，因为它的效率很低，使用数组进行这样的动作就很麻烦，那么，我们可以考虑选择LinkedList。

#### 18.java集合类框架的最佳实践有哪些
 1.根据应用需要正确选择要使用的集合类型对性能非常重要，比如：假如知道元素的大小是固定的，那么选用Array类型而不是ArrayList类型更为合适。

　　2.有些集合类型允许指定初始容量。因此，如果我们能估计出存储的元素的数目，我们可以指定初始容量来避免重新计算hash值或者扩容等。

　　3.为了类型安全、可读性和健壮性等原因总是要使用泛型。同时，使用泛型还可以避免运行时的ClassCastException。

　　4.使用JDK提供的不变类(immutable class)作为Map的键可以避免为我们自己的类实现hashCode()和equals()方法。

　　5.编程的时候接口优于实现

　　6.底层的集合实际上是空的情况下，返回为长度是0的集合或数组而不是null。
　　
#### 19.Set里的元素是不能重复的，区分重复与否的方式，==和equals的区别
元素重复与否是使用equals()方法进行判断的。

equals 方法被用来检测两个对象是否相等，即两个对象的内容是否相等。

==用于比较引用和比较基本数据类型时具有不同的功能： 比较基本数据类型，如果两个值相同，则结果为 true； 而在比较引用时，如果引用指向内存中的同一对象，结果为 true

#### 20.Comparable和Comparator接口是干什么的，列出他们的区别
Comparable是排序接口；若一个类实现了Comparable接口，就意味着“该类支持排序”。

而Comparator是比较器；我们若需要控制某个类的次序，可以建立一个“该类的比较器”来进行排序。

Comparable相当于“内部比较器”，而Comparator相当于“外部比较器”。