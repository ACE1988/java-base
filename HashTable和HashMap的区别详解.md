## 一、HashMap简介
* HashMap是基于哈希表实现的，每一个元素是一个key-value对，其内部通过单链表解决冲突问题，容量不足（超过了阀值）时，同样会自动增长。
* HashMap是非线程安全的，只是用于单线程环境下，多线程环境下可以采用concurrent并发包下的concurrentHashMap。
* HashMap 实现了Serializable接口，因此它支持序列化，实现了Cloneable接口，能被克隆。
* HashMap存数据的过程是：
  *  HashMap内部维护了一个存储数据的Entry数组，HashMap采用链表解决冲突，每一个Entry本质上是一个单向链表。当准备添加一个key-value对时，首先通过hash(key)方法计算hash值，然后通过indexFor(hash,length)求该key-value对的存储位置，计算方法是先用hash&0x7FFFFFFF后，再对length取模，这就保证每一个key-value对都能存入HashMap中，当计算出的位置相同时，由于存入位置是一个链表，则把这个key-value对插入链表头。
  * HashMap中key和value都允许为null。key为null的键值对永远都放在以table[0]为头结点的链表中。



      图中，紫色部分即代表哈希表，也称为哈希数组，数组的每个元素都是一个单链表的头节点，链表是用来解决冲突的，如果不同的key映射到了数组的同一位置处，就将其放入单链表中。

 * HashMap内存储数据的Entry数组默认是16，如果没有对Entry扩容机制的话，当存储的数据一多，Entry内部的链表会很长，这就失去了HashMap的存储意义了。所以HasnMap内部有自己的扩容机制。HashMap内部有：
 * 变量size，它记录HashMap的底层数组中已用槽的数量；
 * 变量threshold，它是HashMap的阈值，用于判断是否需要调整HashMap的容量（threshold = 容量*加载因子）    
 * 变量DEFAULT_LOAD_FACTOR = 0.75f，默认加载因子为0.75
 * HashMap扩容的条件是：当size大于threshold时，对HashMap进行扩容  
 * 扩容是是新建了一个HashMap的底层数组，而后调用transfer方法，将就HashMap的全部元素添加到新的HashMap中（要重新计算元素在新的数组中的索引位置）。 很明显，扩容是一个相当耗时的操作，因为它需要重新计算这些元素在新的数组中的位置并进行复制处理。因此，我们在用HashMap的时，最好能提前预估下HashMap中元素的个数，这样有助于提高HashMap的性能。
 * HashMap共有四个构造方法。构造方法中提到了两个很重要的参数：初始容量和加载因子。这两个参数是影响HashMap性能的重要参数，其中容量表示哈希表中槽的数量（即哈希数组的长度），初始容量是创建哈希表时的容量（从构造函数中可以看出，如果不指明，则默认为16），加载因子是哈希表在其容量自动增加之前可以达到多满的一种尺度，当哈希表中的条目数超出了加载因子与当前容量的乘积时，则要对该哈希表进行 resize 操作（即扩容）。
 * 下面说下加载因子，如果加载因子越大，对空间的利用更充分，但是查找效率会降低（链表长度会越来越长）；如果加载因子太小，那么表中的数据将过于稀疏（很多空间还没用，就开始扩容了），对空间造成严重浪费。如果我们在构造方法中不指定，则系统默认加载因子为0.75，这是一个比较理想的值，一般情况下我们是无需修改的。
 * 另外，无论我们指定的容量为多少，构造方法都会将实际容量设为不小于指定容量的2的次方的一个数，且最大值不能超过2的30次方


## 二、Hashtable简介
 * Hashtable同样是基于哈希表实现的，同样每个元素是一个key-value对，其内部也是通过单链表解决冲突问题，容量不足（超过了阀值）时，同样会自动增长。
 * Hashtable也是JDK1.0引入的类，是线程安全的，能用于多线程环境中。
 * Hashtable同样实现了Serializable接口，它支持序列化，实现了Cloneable接口，能被克隆

## 三、HashTable和HashMap区别
1. 继承的父类不同
      Hashtable继承自Dictionary类，而HashMap继承自AbstractMap类。但二者都实现了Map接口。
2.线程安全性不同
      * javadoc中关于hashmap的一段描述如下：此实现不是同步的。如果多个线程同时访问一个哈希映射，而其中至少一个线程从结构上修改了该映射，则它必须保持外部同步。
      * Hashtable 中的方法是Synchronize的，而HashMap中的方法在缺省情况下是非Synchronize的。在多线程并发的环境下，可以直接使用Hashtable，不需要自己为它的方法实现同步，但使用HashMap时就必须要自己增加同步处理。（结构上的修改是指添加或删除一个或多个映射关系的任何操作；仅改变与实例已经包含的键关联的值不是结构上的修改。）这一般通过对自然封装该映射的对象进行同步操作来完成。如果不存在这样的对象，则应该使用 Collections.synchronizedMap 方法来“包装”该映射。最好在创建时完成这一操作，以防止对映射进行意外的非同步访问，如下所示：
      * Map m = Collections.synchronizedMap(new HashMap(...));
      * Hashtable 线程安全很好理解，因为它每个方法中都加入了Synchronize。这里我们分析一下HashMap为什么是线程不安全的：
      * HashMap底层是一个Entry数组，当发生hash冲突的时候，hashmap是采用链表的方式来解决的，在对应的数组位置存放链表的头结点。对链表而言，新加入的节点会从头结点加入。


（1）在hashmap做put操作的时候会调用下面方法：

[java] view plain copy
// 新增Entry。将“key-value”插入指定位置，bucketIndex是位置索引。      
    void addEntry(int hash, K key, V value, int bucketIndex) {      
        // 保存“bucketIndex”位置的值到“e”中      
        Entry<K,V> e = table[bucketIndex];      
        // 设置“bucketIndex”位置的元素为“新Entry”，      
        // 设置“e”为“新Entry的下一个节点”      
        table[bucketIndex] = new Entry<K,V>(hash, key, value, e);      
        // 若HashMap的实际大小 不小于 “阈值”，则调整HashMap的大小      
        if (size++ >= threshold)      
            resize(2 * table.length);      
    }  
      在hashmap做put操作的时候会调用到以上的方法。现在假如A线程和B线程同时对同一个数组位置调用addEntry，两个线程会同时得到现在的头结点，然后A写入新的头结点之后，B也写入新的头结点，那B的写入操作就会覆盖A的写入操作造成A的写入操作丢失


(3）addEntry中当加入新的键值对后键值对总数量超过门限值的时候会调用一个resize操作，代码如下：

[java] view plain copy
// 重新调整HashMap的大小，newCapacity是调整后的容量      
    void resize(int newCapacity) {      
        Entry[] oldTable = table;      
        int oldCapacity = oldTable.length;     
        //如果就容量已经达到了最大值，则不能再扩容，直接返回    
        if (oldCapacity == MAXIMUM_CAPACITY) {      
            threshold = Integer.MAX_VALUE;      
            return;      
        }      
     
        // 新建一个HashMap，将“旧HashMap”的全部元素添加到“新HashMap”中，      
        // 然后，将“新HashMap”赋值给“旧HashMap”。      
        Entry[] newTable = new Entry[newCapacity];      
        transfer(newTable);      
        table = newTable;      
        threshold = (int)(newCapacity * loadFactor);      
    }  
      这个操作会新生成一个新的容量的数组，然后对原数组的所有键值对重新进行计算和写入新的数组，之后指向新生成的数组。

      当多个线程同时检测到总数量超过门限值的时候就会同时调用resize操作，各自生成新的数组并rehash后赋给该map底层的数组table，结果最终只有最后一个线程生成的新数组被赋给table变量，其他线程的均会丢失。而且当某些线程已经完成赋值而其他线程刚开始的时候，就会用已经被赋值的table作为原始数组，这样也会有问题。

3. 是否提供contains方法
      HashMap把Hashtable的contains方法去掉了，改成containsValue和containsKey，因为contains方法容易让人引起误解。
      Hashtable则保留了contains，containsValue和containsKey三个方法，其中contains和containsValue功能相同
4. key和value是否允许null值
      其中key和value都是对象，并且不能包含重复key，但可以包含重复的value。
      通过上面的ContainsKey方法和ContainsValue的源码我们可以很明显的看出：
      Hashtable中，key和value都不允许出现null值。但是如果在Hashtable中有类似put(null,null)的操作，编译同样可以通过，因为key和value都是Object类型，但运行时会抛出NullPointerException异常，这是JDK的规范规定的。
      HashMap中，null可以作为键，这样的键只有一个；可以有一个或多个键所对应的值为null。当get()方法返回null值时，可能是 HashMap中没有该键，也可能使该键所对应的值为null。因此，在HashMap中不能由get()方法来判断HashMap中是否存在某个键， 而应该用containsKey()方法来判断。
5. 两个遍历方式的内部实现上不同
      Hashtable、HashMap都使用了 Iterator。而由于历史原因，Hashtable还使用了Enumeration的方式 。
6. hash值不同
      哈希值的使用不同，HashTable直接使用对象的hashCode。而HashMap重新计算hash值。
      hashCode是jdk根据对象的地址或者字符串或者数字算出来的int类型的数值。
      Hashtable计算hash值，直接用key的hashCode()，而HashMap重新计算了key的hash值，Hashtable在求hash值对应的位置索引时，用取模运算，而HashMap在求位置索引时，则用与运算，且这里一般先用hash&0x7FFFFFFF后，再对length取模，&0x7FFFFFFF的目的是为了将负的hash值转化为正值，因为hash值有可能为负数，而&0x7FFFFFFF后，只有符号外改变，而后面的位都不变。
7. 内部实现使用的数组初始化和扩容方式不同
      HashTable在不指定容量的情况下的默认容量为11，而HashMap为16，Hashtable不要求底层数组的容量一定要为2的整数次幂，而HashMap则要求一定为2的整数次幂。
      Hashtable扩容时，将容量变为原来的2倍加1，而HashMap扩容时，将容量变为原来的2倍。
      Hashtable和HashMap它们两个内部实现方式的数组的初始大小和扩容的方式。HashTable中hash数组默认大小是11，增加的方式是 old*2+1。
