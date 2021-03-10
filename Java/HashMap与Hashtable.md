#### HashMap与Hashtable

1. 继承父类不同

```java
public class HashMap<K,V> 
    extends AbstractMap<K,V>
    implements Map<K,V>, Cloneable, Serializable 
```

```java
public class Hashtable<K,V>
    extends Dictionary<K,V>
    implements Map<K,V>, Cloneable, Serializable 
```

2. 对NULL的支持不同

+ Hashtable既不支持Null key也不支持Null value

  ```java
  public synchronized V put(K key, V value) {
          // Make sure the value is not null
          //如果值为空那么会报空指针异常
          if (value == null) {
              throw new NullPointerException();
          }
  
          // Makes sure the key is not already in the hashtable.
          Entry<?,?> tab[] = table;
          //如果key为空，调用hashCode()方法会报空指针异常
          int hash = key.hashCode();
  ```

+ HashMap中，key可以为null（但这样的键只有一个）可以有一个或多个键所对应的值为null。当get()方法返回null值时，可能是 HashMap中没有该键，也可能使该键所对应的值为null。因此，在HashMap中不能由get()方法来判断HashMap中是否存在某个键， 而应该用containsKey()方法来判断。

3. 线程安全性不同

   Hashtable是线程安全的，它的每个方法中都加入了Synchronize方法。在多线程并发的环境下，可以直接使用Hashtable，不需要自己为它的方法实现同步

   HashMap不是线程安全的，在多线程并发的环境下，可能会产生死锁等问题。具体的原因在下一篇文章中会详细进行分析。使用HashMap时就必须要自己增加同步处理，虽然HashMap不是线程安全的，但是它的效率会比Hashtable要好很多


4. 初始容量大小和每次扩充容量大小的不同

   Hashtable默认的初始大小为11，之后每次扩充，容量变为原来的2n+1。HashMap默认的初始化大小为16。之后每次扩充，容量变为原来的2倍。

   创建时，如果给定了容量初始值，那么Hashtable会直接使用你给定的大小，而HashMap会将其扩充为2的幂次方大小。也就是说Hashtable会尽量使用素数、奇数。而HashMap则总是使用2的幂作为哈希表的大小。

https://zhuanlan.zhihu.com/p/21673805

#### HashMap

HashMap是一个用于存储Key-Value键值对的集合，每一个键值对也叫做Node。这些键值对（Node）分散存储在一个数组当中，这个数组就是HashMap的主干,HashMap数组每一个元素的初始值都是Null。

内部使用 **数组+链表/红黑树**，数组是用来确定位置，利用元素的key的hash值对数组长度取模得到，显然数组的查找效率比较高；而链表是用来解决hash冲突问题，当出现hash值一样的情形，就在数组上的对应位置形成一条链表。当链表长度>=8 & 数组长度的大小>=64 时，链表会转换为红黑树。键值对数量超过阈值(加载因子*当前数组的大小)时，则进行扩容，每次扩容2倍，保证数组的长度是2的次幂。为了存取高效，尽量减少碰撞，数据分配均匀，那么通过数组长度取模是一个非常不错的选择，但计算机中直接求余效率不如位移运算，数组长度为2次幂时，hash&(length-1)是等价于hash%length，这样可以提高位置的计算效率。

```java
/**实际存储的key-value键值对的个数*/
transient int size;

/**阈值，当table == {}时，该值为初始容量（初始容量默认为16）；当table被填充了，也就是为table分配内存空间后，
threshold一般为 capacity*loadFactory。HashMap在进行扩容时需要参考threshold，后面会详细谈到*/
int threshold;

/**负载因子，代表了table的填充度有多少，默认是0.75
加载因子存在的原因，还是因为减缓哈希冲突，如果初始桶为16，等到满16个元素才扩容，某些桶里可能就有不止一个元素了。
所以加载因子默认为0.75，也就是说大小为16的HashMap，到了第13个元素，就会扩容成32。
*/
final float loadFactor;

/**HashMap被改变的次数，由于HashMap非线程安全，在对HashMap进行迭代时，
如果期间其他线程的参与导致HashMap的结构发生变化了（比如put，remove等操作），
需要抛出异常ConcurrentModificationException*/
transient int modCount;
```

保证数组长度为2次幂，让初始二进制右移1，2，4，8，16位，分别与自己位或，把高位第一个为1的数通过不断右移，把高位为1的后面全变为1，最后再进行+1操作，就可得到>=给定容量的2次幂。

```java
static final int tableSizeFor(int cap) {
        int n = cap - 1;
        n |= n >>> 1;
        n |= n >>> 2;
        n |= n >>> 4;
        n |= n >>> 8;
        n |= n >>> 16;
        return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
    }
```

HashMap的数据插入原理：

![HashMap初始化](/image/HashMap初始化.png)

1. 判断数组是否为空，为空进行初始化;
2. 不为空，计算 key 的 hash 值，通过(n - 1) & hash计算应当存放在数组中的下标 index;
3. 查看 table[index] 是否存在数据，没有数据就构造一个Node节点存放在 table[index] 中；
4. 存在数据，说明发生了hash冲突(存在二个节点key的hash值一样), 继续判断key是否相等，相等，用新的value替换原数据(onlyIfAbsent为false)；
5. 如果不相等，判断当前节点类型是不是树型节点，如果是树型节点，创造树型节点插入红黑树中；(如果当前节点是树型节点证明当前已经是红黑树了)
6. 如果不是树型节点，创建普通Node加入链表中；判断链表长度是否大于 8并且数组长度大于64， 大于的话链表转换为红黑树
7. 插入完成之后判断当前节点数是否大于阈值，如果大于开始扩容为原数组的二倍

##### 查找

+ 计算hash值:对hashCode的值重新计算，增加hash的复杂度，降低hash冲突的几率。因为下面是通过hash&(length-1)来计算位置的，如果length过小，那么只有hash的低几位参与&运算，而高位数据在没有任何用处；因此让hashCode高16位与低16位进行异或，以此增大低位信息的随机性，使得hash尽可能的分布开来，降低hash的冲突

```java
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

+ 定位

```java
    final Node<K,V> getNode(int hash, Object key) {
        Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
        // 1. 定位键值对所在桶的位置
        if ((tab = table) != null && (n = tab.length) > 0 &&
            (first = tab[(n - 1) & hash]) != null) {
            if (first.hash == hash && // always check first node
                ((k = first.key) == key || (key != null && key.equals(k))))
                return first;
            if ((e = first.next) != null) {
                 // 2. 如果 first 是 TreeNode 类型，则调用黑红树查找方法
                if (first instanceof TreeNode)
                    return ((TreeNode<K,V>)first).getTreeNode(hash, key);
                // 2. 对链表进行查找
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

只有当数组的长度length是2次幂时，hash&(length-1)才等价与hash%length

```java
//确定桶位置
first = tab[(n-1)&hash];
```

##### 遍历

```java
for(Object key : map.keySet()) {
    // do something
}
```

```java
for(HashMap.Entry entry : map.entrySet()) {
    // do something
}
```

```java
Set keys = map.keySet();
Iterator ite = keys.iterator();
while (ite.hasNext()) {
    Object key = ite.next();
    // do something
}
```

##### 插入

​		首先肯定是先定位要插入的键值对属于哪个桶，定位到桶后，再判断桶是否为空。如果为空，则将键值对存入即可。如果不为空，则需将键值对接在链表最后一个位置，或者更新键值对。

1. 当桶数组 table 为空时，通过扩容的方式初始化 table
2. 查找要插入的键值对是否已经存在，存在的话根据条件判断是否用新值替换旧值
3. 如果不存在，则将键值对链入链表中，并根据链表长度决定是否将链表转为红黑树
4. 判断键值对数量是否大于阈值，大于的话则进行扩容操作

```java
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}
```

```java
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
    Node<K,V>[] tab; 
    Node<K,V> p;  
    int n, i;
    // 初始化数组table大小，table初始化被延迟到插入新数据
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
     // 定位所在的位置，如果桶中不包含键值对，则直接将新键值对节插入即可
        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
        else {
            Node<K,V> e;  //指向节点应该插入的位置（如果不为空，则修改原节点值）
            K k;
             // 如果key值与hash值与链表中的第一个节点相等时，则将e指向该节点
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
            // 如果第一个节点类型是TreeNode，则调用红黑树的插入方法
            else if (p instanceof TreeNode)
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            else {
                //对链表进行遍历，并统计链表长度
                for (int binCount = 0; ; ++binCount) {
                    //链表中没相同的节点时，则将该节点插入链表的最后
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                        // 如果链表长度大于或等于树化阈值，则进行树化操作
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                    }
                    // 当在链表找到相同节点时，终止遍历
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
            }
            //判断要插入的键值对是否存在 HashMap 中
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                // onlyIfAbsent 表示是否仅在 oldValue 为 null 的情况下更新键值对的值
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
    //HashMap在结构上被修改的次数加1,若HashMap内部结构发生变化，快速响应失败
        ++modCount;
    // 键值对数量超过阈值时，则进行扩容
        if (++size > threshold)
            resize();
        afterNodeInsertion(evict);
        return null;
    }
```

##### 扩容

```java
final Node<K,V>[] resize() {
        Node<K,V>[] oldTab = table;
        int oldCap = (oldTab == null) ? 0 : oldTab.length;
        int oldThr = threshold;
        int newCap, newThr = 0;
    // 如果 table 不为空，表明已经初始化过了
        if (oldCap > 0) {
            // 当 table 容量超过容量最大值，则不再扩容
            //static final int MAXIMUM_CAPACITY = 1 << 30; 2的30次方
            if (oldCap >= MAXIMUM_CAPACITY) {
                threshold = Integer.MAX_VALUE;
                return oldTab;
            }
            // 按旧容量和阈值的2倍计算新容量和阈值的大小
            else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                     oldCap >= DEFAULT_INITIAL_CAPACITY)
                newThr = oldThr << 1; // double threshold
        }
       // 如果 table 为空，没有初始化过
        else if (oldThr > 0) 		// oldThr > 0
            /*threshold > 0，且桶数组未被初始化
            通过HashMap(int)和HashMap(int, float)创建HashMap时会产生这种情况
         */ 
            newCap = oldThr;
        else {               // oldCap == 0 && oldThr == 0
          /*桶数组未被初始化，且 threshold 为 0
          通过HashMap()创建HashMap时会产生这种情况
         */
            newCap = DEFAULT_INITIAL_CAPACITY;
            newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
        }
        if (newThr == 0) {
            float ft = (float)newCap * loadFactor;
            newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                      (int)ft : Integer.MAX_VALUE);
        }
        threshold = newThr;
    // 创建新的桶数组，桶数组的初始化也是在这里完成的
        @SuppressWarnings({"rawtypes","unchecked"})
        Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
        table = newTab;
        if (oldTab != null) {
            // 如果旧的桶数组不为空，则遍历桶数组，并将键值对映射到新的桶数组中
            for (int j = 0; j < oldCap; ++j) {
                Node<K,V> e;
                if ((e = oldTab[j]) != null) {
                    oldTab[j] = null;
                    if (e.next == null)
                        newTab[e.hash & (newCap - 1)] = e;
                    else if (e instanceof TreeNode)
                        // 重新映射时，需要对红黑树进行拆分
                        ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                    else { // preserve order
                        Node<K,V> loHead = null, loTail = null;
                        Node<K,V> hiHead = null, hiTail = null;
                        Node<K,V> next;
                         // 遍历链表，并将链表节点按原顺序进行分组
                        do {
                            next = e.next;
                            if ((e.hash & oldCap) == 0) {
                                if (loTail == null)
                                    loHead = e;
                                else
                                    loTail.next = e;
                                loTail = e;
                            }
                            else {
                                if (hiTail == null)
                                    hiHead = e;
                                else
                                    hiTail.next = e;
                                hiTail = e;
                            }
                        } while ((e = next) != null);
                        // 将分组后的链表映射到新桶中
                        if (loTail != null) {
                            loTail.next = null;
                            newTab[j] = loHead;
                        }
                        if (hiTail != null) {
                            hiTail.next = null;
                            newTab[j + oldCap] = hiHead;
                        }
                    }
                }
            }
        }
        return newTab;
    }
```

##### hash算法

​		Hash函数是指把一个大范围映射到一个小范围。把大范围映射到一个小范围的目的往往是为了节省空间，使得数据容易保存。 比较出名的有MurmurHash、MD4、MD5等等

1. 开发定址法：当冲突发生时，使用某种探测技术在散列表中形成一个探测序列。沿此序列逐个单元地查找，直到找到给定的关键字，或者碰到一个开放的地址（即该地址单元为空）为止（若要插入，在探查到开放的地址，则可将待插入的新结点存人该地址单元）
2. 再哈希法：有多个不同的Hash函数，当发生冲突时，使用第二个，第三个，….，等哈希函数，计算地址，直到无冲突。虽然不易发生聚集，但是增加了计算时间
3. 链地址法：每个哈希表节点都有一个next指针，多个哈希表节点可以用next指针构成一个单向链表，被分配到同一个索引上的多个结点用单向链表连接起来
4. 建立公共溢出区：将哈希表分为基本表和溢出表两部分，凡是和基本表发生冲突的元素，一律填入溢出表

+ String中hashcode的实现：以31为权，每一位为字符的ASCII值进行运算，用自然溢出来等效取模。哈希计算公式可以计为`s[0]*31^(n-1) + s[1]*31^(n-2) + ... + s[n-1]`。主要是因为31是一个奇质数，所以`31*i=32*i-i=(i<<5)-i`，这种位移与减法结合的计算相比一般的运算快很多

```java
public int hashCode() {
        int h = hash;
        if (h == 0 && value.length > 0) {
            char val[] = value;

            for (int i = 0; i < value.length; i++) {
                h = 31 * h + val[i];
            }
            hash = h;
        }
        return h;
    }
```

##### *jdk1.8中hashmap的改变*

+ 数组+链表→数组+链表+红黑树

当链表长度小于8时，查询速度还行，但当元素个数大于8时，性能就会比较差，而转为红黑树，使得查询效率就可提高，虽然新增节点的效率会变慢。如果使用的是二叉查找树，可能会存在单链的情况，那么和原来的链表就没啥区别了，使得遍历查找的速度变慢。而红黑树是二叉平衡树，保证树的高度尽可能的低。至于为什么选择8，可能是经过计算吧，可能是在长度为8时，与其保证链表结构的查找开销，不如转换为红黑树，改为维持其平衡开销。当红黑树的节点数量<=6时就会退化链表，这与链表转为红黑树的阈值8相差了个7，7用作缓冲，这样可以防止链表和数之间频发的转换。假设一下，如果设计成链表个数超过8则链表转换成树结构，链表个数小于8则树结构转换成链表，如果一个HashMap不停的插入、删除元素，链表个数在8左右徘徊，就会频繁的发生树转链表、链表转树，效率会很低。将时间复杂度由O(n)降为O(logn)

​	理想情况下随机hashCode算法下所有链表中节点的分布频率会遵循**泊松分布**，我们可以看到，一个链表中链表长度达到8个元素的概率为0.00000006，几乎是不可能事件。这种不可能事件都发生了，说明链表中的节点数很多，查找起来效率不高，链表中节点的分布频率会遵循泊松分布，达到8的概率很小。TreeNodes占用空间是普通Nodes的两倍（相较于链表结构，链表只有指向下一个节点的指针，二叉树则需要左右指针，分别指向左节点和右节点），所以只有当链表包含足够多的节点时才会转成TreeNodes（考虑到时间和空间的权衡），而是否足够多就是由TREEIFY_THRESHOLD的值决定的。当红黑树中节点数变少时，又会转成普通的链表。并且我们查看源码的时候发现，链表长度达到8就转成红黑树，当长度降到6就转成普通链表。

+ 优化了高位运算的hash算法：h^(h>>16)

通过hashCode高16位与低16位进行异或，以此增大低位信息的随机性，增加hash的复杂度，降低hash冲突的几率。因为计算某个key的位置时是通过hash&(length-1)，如果length过小，那么只有hash的低几位参与&运算，而高位数据在没有任何用处。加上高16位异或低16位的“扰动函数”后，可以减少了碰撞的几率。

+ rehash：遍历每一个元素，每一个节点，重新计算他们的索引值，存入新的数组中→扩容后，元素要么是在原位置，要么是在原位置在移动2次幂的位置（解决死循环）

由于数组的长度是2次幂，而每次扩容是扩大为原数组大小的2倍，则扩容后n-1比扩容前的n-1仅仅是高位多了一个1而已（16-1:1111；32-1:11111）；而计算元素位置时通过h=(n-1)&hash；任何数与1的与运算就是它本身，则元素的新位置，就看hashcode对应高位上是0还是1，如果是0的话，则重新计算出的h不变；如果是1的话，则h会比原来的大2次幂(即扩容前的大小)。

+ 头插法改成了尾插法，在插入时，1.7先判断是否需要扩容，再插入，1.8先进行插入，插入完成再判断是否需要扩容；1.7头插法扩容时，头插法会使链表发生反转，多线程环境下会产生环

##### *HashMap在并发编程环境下存在的问题*

+ 多线程扩容，引起的死循环问题（死循环）
+ 多线程put的时候可能导致元素丢失（数据丢失）
+ put非null元素后get出来的却是null（数据覆盖）

在jdk1.8中，死循环问题已经解决。其他两个问题还是存在，可以使用ConcurrentHashmap或HashTable、Collections.synchronizedMap实现线程安全的Map。HashTable是直接在操作方法上加synchronized关键字，锁住整个数组，粒度比较大，Collections.synchronizedMap是使用Collections集合工具的内部类，通过传入Map封装出一个SynchronizedMap对象，内部定义了一个对象锁，方法内通过对象锁实现；ConcurrentHashMap使用分段锁，降低了锁粒度，让并发度大大提高。ConcurrentHashMap成员变量使用volatile 修饰，免除了指令重排序，同时保证内存可见性，另外使用CAS操作和synchronized结合实现赋值操作，多线程操作只会锁住当前操作索引的节点。

##### HashMap的key

+ key可以为null，但只有一个，因为key为null时，会以0作为hash，即存在数组中第一个位置
+ 一般用Integer、String这种不可变类当HashMap当key，而且String最为常用。String是不可变的，在String类中有个私有实例字段hash表示该串的哈希值，在第一次调用hashCode方法时，字符串的哈希值被计算并且赋值给hash字段，之后再调用hashCode方法便可以直接取hash字段返回，不需要重新计算。这就使得字符串很适合作为Map中的键，字符串的处理速度要快过其它的键对象。这就是HashMap中的键往往都使用字符串。在HashMap中对象的查找、插入等操作都需要用到equals()和hashCode()方法，那么作为key的对象对这两个方法的正确重写是非常重要的，而这些类已经很规范的覆写了hashCode()以及equals()方法。如果使用可变类当做key的话，那么key的hashcode可能会发生改变，导致之前put进map的值，无法get出来；例如用list作为key，先往map中put值，而后来作为key的list又添加新元素，这可能会导致list的hashConde发生改变，而无法找到原先put进去的值。
+ 设计key的重要考虑：重写hashcode和equals、设计一个不可变类
  + 重写hashcode和equals：两个对象相等，那么hashcode一定相等；两个对象不相等，hashcode不一定不相等；hashcode相等，两个对象不一定相等；hashcode不相等，两个对象一定不相等。
  + 设计不可变类：类使用final修饰，保证类不可被继承；所有成员变量私有化并加上final修饰；不提供修改成员变量的方法；通过构造器初始化所有成员，如果是成员引用使用深拷贝，防止外部修改传来的引用而改变内部成员；在getter方法中，不要直接返回对象本身，而是克隆对象，并返回对象的拷贝 这种做法也是防止对象外泄，防止通过getter获得内部可变成员对象后对成员变量直接操作，导致成员变量发生改变。

##### 有序map

HashMap内部节点是是无序的，而LinkedHashMap、TreeMap是有序的

+ LinkedHashMap

LinkedHashMap内部维护了一个单链表，有头尾节点，同时LinkedHashMap节点Entry内部除了继承HashMap的Node属性，还有before 和 after用于标识前置节点和后置节点。可以实现按插入的顺序或访问顺序排序。

+ TreeMap

TreeMap是按照Key的自然顺序或者Comprator的顺序进行排序，内部是通过红黑树来实现。所以要么key所属的类实现Comparable接口，或者自定义一个实现了Comparator接口的比较器，传给TreeMap用于key的比较。

#### ConcurrentHashmap

https://cloud.tencent.com/developer/article/1448721

HashTable，它是线程安全的，它在所有涉及到多线程操作的都加上了synchronized关键字来锁住整个table，这就意味着所有的线程都在竞争一把锁，在多线程的环境下，它是安全的，但是无疑是效率低下的。其实HashTable有很多的优化空间，锁住整个table这么粗暴的方法可以变相的柔和点，比如在多线程的环境下，对不同的数据集进行操作时其实根本就不需要去竞争一个锁，因为他们不同hash值，不会因为rehash造成线程不安全，所以互不影响，这就是锁分离技术，将锁的粒度降低，利用多个锁来控制多个小的table

+ JDK1.7（ReentrantLock+Segment+HashEntry）

ConcurrentHashMap的数据结构是由一个Segment数组和多个HashEntry组成，Segment数组的意义就是将一个大的table分割成多个小的table来进行加锁，也就是上面的提到的锁分离技术，而每一个Segment元素存储的是HashEntry数组+链表，这个和HashMap的数据存储结构一样。

ConcurrentHashMap定位一个元素的过程需要进行两次Hash操作，第一次Hash定位到Segment，第二次Hash定位到元素所在的链表的头部，因此，这一种结构的带来的副作用是Hash的过程要比普通的HashMap要长，但是带来的好处是写操作的时候可以只对元素所在的Segment进行加锁即可，不会影响到其他的Segment，这样，在最理想的情况下，ConcurrentHashMap可以最高同时支持Segment数量大小的写操作（刚好这些写操作都非常平均地分布在所有的Segment上），所以，通过这一种结构，ConcurrentHashMap的并发能力可以大大的提高

![jdk7CHM](/image/jdk7CHM.png)

**初始化**: Segment的大小取值都是以2的N次方，无关concurrencyLevel的取值，当然concurrencyLevel最大只能用16位的二进制来表示，即65536，换句话说，Segment的大小最多65536个，没有指定concurrencyLevel元素初始化，Segment的大小size默认为16。

```java
int size =1;
while(size < concurrencyLevel) {
++a;
size <<=1;
}      
```

每一个Segment元素下的HashEntry的初始化也是按照位于运算来计算,HashEntry大小的计算也是2的N次方（cap <<=1）， cap的初始值为1，所以HashEntry最小的容量为2

```java
int cap =1;
while(cap < c)
cap <<=1;
```

+ JDK1.8（synchronized+CAS+HashEntry+红黑树）

JDK1.8的实现已经摒弃了Segment的概念，而是直接用Node数组+链表+红黑树的数据结构来实现，并发控制使用Synchronized和CAS来操作，整个看起来就像是优化过且线程安全的HashMap，虽然在JDK1.8中还能看到Segment的数据结构，但是已经简化了属性，只是为了兼容旧版本

```java
public class ConcurrentHashMap<K,V> extends AbstractMap<K,V>
    implements ConcurrentMap<K,V>, Serializable {
    
    //node数组最大容量
    private static final int MAXIMUM_CAPACITY = 1 << 30;

    /**
     * The default initial table capacity.  Must be a power of 2
     * 默认table数组初始容量，必须2的幂数
     */
    private static final int DEFAULT_CAPACITY = 16;

    /**
     * The largest possible (non-power of two) array size.
     * Needed by toArray and related methods.
     数组可能最大值，需要与toArray（）相关方法关联 
     */
    static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

    /**
     * The default concurrency level for this table. Unused but
     * defined for compatibility with previous versions of this class.
     并发级别，遗留下来的，为兼容以前的版本
     */
    private static final int DEFAULT_CONCURRENCY_LEVEL = 16;

    //负载因子
    private static final float LOAD_FACTOR = 0.75f;

    //链表转红黑树阈值>=8
    static final int TREEIFY_THRESHOLD = 8;

    //树转链表阀值，<=6
    static final int UNTREEIFY_THRESHOLD = 6;

    /**
     * The smallest table capacity for which bins may be treeified.
     * (Otherwise the table is resized if too many nodes in a bin.)
     * The value should be at least 4 * TREEIFY_THRESHOLD to avoid
     * conflicts between resizing and treeification thresholds.
     */
    static final int MIN_TREEIFY_CAPACITY = 64;

    /**
     * Minimum number of rebinnings per transfer step. Ranges are
     * subdivided to allow multiple resizer threads.  This value
     * serves as a lower bound to avoid resizers encountering
     * excessive memory contention.  The value should be at least
     * DEFAULT_CAPACITY.
     */
    private static final int MIN_TRANSFER_STRIDE = 16;

    /**
     * The number of bits used for generation stamp in sizeCtl.
     * Must be at least 6 for 32bit arrays.
     */
    private static int RESIZE_STAMP_BITS = 16;


    // 2^16-1=65535，help resize的最大线程数  
    private static final int MAX_RESIZERS = (1 << (32 - RESIZE_STAMP_BITS)) - 1;

    //32-16=16，sizeCtl中记录size大小的偏移量 
    private static final int RESIZE_STAMP_SHIFT = 32 - RESIZE_STAMP_BITS;

    /*
     * Encodings for Node hash fields. See above for explanation.
     */
    
    //forwarding nodes的hash值
    static final int MOVED     = -1;
    //树根节点的hash值 
    static final int TREEBIN   = -2;
    
    //ReservationNode的hash值
    static final int RESERVED  = -3;
   
    static final int HASH_BITS = 0x7fffffff; // usable bits of normal node hash

    // 可用处理器数量
    static final int NCPU = Runtime.getRuntime().availableProcessors();
    
    //存放node的数组  
    transient  volatile  Node<K,V>[] table;  
    
    /*多个线程的共享变量，是操作的控制标识符
    ，]用来控制table的初始化和扩容的操作，不同的值有不同的含义
    当为负数时：-1代表正在初始化，-N代表有N-1个线程正在 进行扩容
    当为0时：代表当时的table还没有被初始化
    当为正数时：表示初始化或者下一次进行扩容的大小
    */
    private  transient  volatile  int  sizeCtl;     
    
    //哈希表扩容时生成的数据，数组为扩容前的2倍
    private transient volatile Node<K,V>[] nextTable;
    
    //一个特殊的Node节点，Hash地址为-1，存储着nextTable的引用，只有table发生扩用的时候，ForwardingNode才会发挥作用，作为一个占位符放在table中表示当前节点为null或者已被移动
    static final class ForwardingNode<K,V> extends Node<K,V> {
        final Node<K,V>[] nextTable;
        ForwardingNode(Node<K,V>[] tab) {
            // Node(int hash, K key, V val, Node<K,V> next)  MOVE=-1
            super(MOVED, null, null, null);
            this.nextTable = tab;
        }
}
```

插入

> 1. 如果没有初始化就先调用initTable（）方法来进行初始化过程
> 2. 如果没有hash冲突就直接CAS插入
> 3. 如果还在进行扩容操作就先进行扩容
> 4. 如果存在hash冲突，就加锁来保证线程安全，这里有两种情况，一种是链表形式就直接遍历到尾端插入，一种是红黑树就按照红黑树结构插入，
> 5. 最后一个如果Hash冲突时会形成Node链表，在链表长度超过8，Node数组超过64时会将链表结构转换为红黑树的结构，break再一次进入循环
> 6. 如果添加成功就调用addCount（）方法统计size，并且检查是否需要扩容

```java
final V putVal(K key, V value, boolean onlyIfAbsent) {
    	//不允许键值为null，这点与线程安全的Hashtable保持一致，和HashMap不同
        if (key == null || value == null) throw new NullPointerException();
    	//计算hash：(h ^ (h >>> 16)) & HASH_BITS   HASH_BITS = 0x7fffffff,
    	//spread()加多了 &0x7fffffff，让结果为正数
        int hash = spread(key.hashCode());
    	// 用于记录相应链表的长度
        int binCount = 0;
        for (Node<K,V>[] tab = table;;) {
            Node<K,V> f; int n, i, fh;
            if (tab == null || (n = tab.length) == 0)
                //数组为空，先进行初始化
                tab = initTable();
            
             // 找该 hash 值对应的数组下标，得到第一个节点 f
            else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
                // 如果数组该位置为空，用一次CAS操作将这个新值放入其中即可，如果 CAS 失败，那就是有并发操作，进到下一个循环就好了
                /*casTable()方法的第一个参数是Node数组的引用，第二个参数是待操作的下标，第三个参数是期望值，第四个参数是待操作的Node节点，实现的是将Node数组下标为参数二的节点替换成参数四的节点，如果期望值和实际值不符返回false，否则参数四的节点成功替换上去，返回ture，即插入成功。注意这里：如果插入成功了则跳出for循环，插入失败的话(其他线程抢先插入了)，那么会执行到下面的代码。
                */
                if (casTabAt(tab, i, null,
                             new Node<K,V>(hash, key, value, null)))
                    break;                  
            }
             //如果该下标上的节点的哈希地址为MOVE(-1)(即链表的头节点为ForwardingNode节点)，则表示table需要扩容，值得注意的是ConcurrentHashMap初始化和扩容不是用同一个方法
            else if ((fh = f.hash) == MOVED)
                tab = helpTransfer(tab, f);
            
            //如果该下标上的节点既不是空也不是需要扩容，则表示这个链表可以插入值，将进入到链表中，将新节点插入或者覆盖旧值。
            else {
                V oldVal = null;
               //通过关键字synchroized对当前index位置的头结点加锁(相当于锁住锁住该下标上的链表)，其他位置的节点并没有加锁，所以其他线程可以安全的获得其他下标上的链表进行操作，也正是因为这个所以提高了ConcurrentHashMap的效率，提高了并发度。
                synchronized (f) {
                    if (tabAt(tab, i) == f) {
                        //头结点的 hash 值大于 0，说明是链表
                        if (fh >= 0) {
                            //用于累加，记录链表的长度
                            binCount = 1;
                            // 遍历链表
                            for (Node<K,V> e = f;; ++binCount) {
                                K ek;
                                // 如果发现了"相等"的 key，判断是否要进行值覆盖，然后也就可以 break 了
                                if (e.hash == hash &&
                                    ((ek = e.key) == key ||
                                     (ek != null && key.equals(ek)))) {
                                    //相同的key进行put就会覆盖原先的value 
                                    oldVal = e.val;
                                    if (!onlyIfAbsent)
                                        e.val = value;
                                    break;
                                }
                                Node<K,V> pred = e;
                                // 到了链表的最末端，将这个新值放到链表的最后面
                                if ((e = e.next) == null) {
                                    //插入链表尾部  
                                    pred.next = new Node<K,V>(hash, key,
                                                              value, null);
                                    break;
                                }
                            }
                        }
                        //如果该下标上的节点的哈希地址小于0 且为树节点表示该节点是红黑树结构  
                        else if (f instanceof TreeBin) {
                            Node<K,V> p;
                            binCount = 2;
                            //调用红黑树的插值方法插入新节点
                            if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,
                                                           value)) != null) {
                                 //如果插入的结果不为null，则表示为替换
                                oldVal = p.val;
                                if (!onlyIfAbsent)
                                    p.val = value;
                            }
                        }
                    }
                }//synchronized到这里结束
                // binCount != 0 说明上面在做链表操作
                if (binCount != 0) {
                    //如果链表的长度>=8时就会进行红黑树的转换
                    if (binCount >= TREEIFY_THRESHOLD)
                        treeifyBin(tab, i);
                    if (oldVal != null)
                        return oldVal;
                    break;
                }
            }
        }
    	//统计size，并且检查是否需要扩容 
        addCount(1L, binCount);
        return null;
    }
```

initTable()：初始化操作，初始化方法中的并发问题是通过对 sizeCtl 进行一个 CAS 操作来控制的

```java
private final Node<K,V>[] initTable() {
        Node<K,V>[] tab; int sc;
    	//空的table才能进入初始化操作
        while ((tab = table) == null || tab.length == 0) {
            //sizeCtl<0表示其他线程已经在初始化了或者扩容了，挂起当前线程
            if ((sc = sizeCtl) < 0)
                Thread.yield(); // lost initialization race; just spin
             //CAS操作SIZECTL设为-1，代表抢到了锁
            else if (U.compareAndSwapInt(this, SIZECTL, sc, -1)) {
                try {
                    if ((tab = table) == null || tab.length == 0) {
                        // DEFAULT_CAPACITY 默认初始容量是 16
                        int n = (sc > 0) ? sc : DEFAULT_CAPACITY;
                        @SuppressWarnings("unchecked")
                        // 初始化数组，长度为 16 或初始化时提供的长度
                        Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n];
                        // 将这个数组赋值给 table，table 是 volatile 的
                        table = tab = nt;
                        // 如果n为16的话，那么这里 sc = 12，其实就是 0.75 * n
                        sc = n - (n >>> 2);
                    }
                } finally {
                    // 设置 sizeCtl 为 sc，我们就当是 12 吧
                    sizeCtl = sc;
                }
                break;
            }
        }
        return tab;
    }
```

helpTransfer()：调用多个工作线程一起帮助进行扩容，这样的效率就会更高，而不是只有检查到要扩容的那个线程进行扩容操作，其他线程就要等待扩容操作完成才能工作

```java
final Node<K,V>[] helpTransfer(Node<K,V>[] tab, Node<K,V> f) {
        Node<K,V>[] nextTab; int sc;
        if (tab != null && (f instanceof ForwardingNode) &&
            (nextTab = ((ForwardingNode<K,V>)f).nextTable) != null) {
            int rs = resizeStamp(tab.length);
            //新的nextTba已经存在前提下才能帮助扩容
            while (nextTab == nextTable && table == tab &&
                   (sc = sizeCtl) < 0) {
                if ((sc >>> RESIZE_STAMP_SHIFT) != rs || sc == rs + 1 ||
                    sc == rs + MAX_RESIZERS || transferIndex <= 0)
                    break;
                if (U.compareAndSwapInt(this, SIZECTL, sc, sc + 1)) {
                    //调用扩容方法
                    transfer(tab, nextTab);
                    break;
                }
            }
            return nextTab;
        }
        return table;
    }
```

transfer()：扩容，数组长度为 n，所以我们有 n 个迁移任务，让每个线程每次负责一个小任务是最简单的，每做完一个任务再检测是否有其他没做完的任务，帮助迁移就可以了，而 Doug Lea 使用了一个 stride，简单理解就是步长，每个线程每次负责迁移其中的一部分，如每次迁移 16 个小任务。所以，我们就需要一个全局的调度者来安排哪个线程执行哪几个任务，这个就是属性 transferIndex 的作用。第一个发起数据迁移的线程会将 transferIndex 指向原数组最后的位置，然后从后往前的 stride 个任务属于第一个线程，然后将 transferIndex 指向新的位置，再往前的 stride 个任务属于第二个线程，依此类推。当然，这里说的第二个线程不是真的一定指代了第二个线程，也可以是同一个线程，其实就是将一个大的迁移任务分为了一个个任务包。

```java
private final void transfer(Node<K,V>[] tab, Node<K,V>[] nextTab) {
        int n = tab.length, stride;
    	// stride 在单核下直接等于 n，多核模式下为 (n>>>3)/NCPU，最小值是 16
    	// stride 可以理解为”步长“，有 n 个位置是需要进行迁移的，
    	//将这 n 个任务分为多个任务包，每个任务包有 stride 个任务
        if ((stride = (NCPU > 1) ? (n >>> 3) / NCPU : n) < MIN_TRANSFER_STRIDE)
            stride = MIN_TRANSFER_STRIDE;
    	// 如果 nextTab 为 null，先进行一次初始化
    //前面我们说了，外围会保证第一个发起迁移的线程调用此方法时，参数 nextTab 为 null
    //       之后参与迁移的线程调用此方法时，nextTab 不会为 null
        if (nextTab == null) {            
            try {
                @SuppressWarnings("unchecked")
                //构建一个nextTable对象，其容量为原来容量的两倍
                Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n << 1];
                nextTab = nt;
            } catch (Throwable ex) { 
                sizeCtl = Integer.MAX_VALUE;
                return;
            }
            // nextTable 是 ConcurrentHashMap 中的属性
            nextTable = nextTab;
            // transferIndex 也是 ConcurrentHashMap 的属性，用于控制迁移的位置
            transferIndex = n;
        }
        int nextn = nextTab.length;
    	// 连接点指针，用于标志位（fwd的hash值为-1，fwd.nextTable=nextTab）
    // ForwardingNode 翻译过来就是正在被迁移的 Node
    // 这个构造方法会生成一个Node，key、value 和 next 都为 null，关键是 hash 为 MOVED
    // 后面我们会看到，原数组中位置 i 处的节点完成迁移工作后，
    //    就会将位置 i 处设置为这个 ForwardingNode，用来告诉其他线程该位置已经处理过了
    //    所以它其实相当于是一个标志
        ForwardingNode<K,V> fwd = new ForwardingNode<K,V>(nextTab);
    	// 当advance == true时，表明该节点已经处理过了,即做完了一个位置的迁移工作，可以准备做下一个位置的了
        boolean advance = true;
        boolean finishing = false; 
    	// i 是位置索引，bound 是边界，注意是从后往前
        for (int i = 0, bound = 0;;) {
            Node<K,V> f; int fh;
            //// advance 为 true 表示可以进行下一个位置的迁移了
        //   简单理解结局：i 指向了 transferIndex，bound 指向了 transferIndex-stride
            while (advance) {
                int nextIndex, nextBound;
                if (--i >= bound || finishing)
                    advance = false;
                // 将 transferIndex 值赋给 nextIndex
            // 这里 transferIndex 一旦小于等于 0，说明原数组的所有位置都有相应的线程去处理了
                else if ((nextIndex = transferIndex) <= 0) {
                    i = -1;
                    advance = false;
                }
                // 用CAS计算得到的transferIndex
                else if (U.compareAndSwapInt
                         (this, TRANSFERINDEX, nextIndex,
                          nextBound = (nextIndex > stride ?
                                       nextIndex - stride : 0))) {
                    //nextBound 是这次迁移任务的边界，注意，是从后往前
                    bound = nextBound;
                    i = nextIndex - 1;
                    advance = false;
                }
            }
            if (i < 0 || i >= n || i + n >= nextn) {
                int sc;
                // 已经完成所有节点复制了
                if (finishing) {
                    nextTable = null;
                    // table 指向nextTable 
                    table = nextTab;
                    // sizeCtl阈值为原来的1.5倍，n 是原数组长度
                    sizeCtl = (n << 1) - (n >>> 1);
                    // 跳出死循环
                    return;
                }
               // sizeCtl 在迁移前会设置为 (rs << RESIZE_STAMP_SHIFT) + 2
            // 然后，每有一个线程参与迁移就会将 sizeCtl 加 1，
            // 这里使用 CAS 操作对 sizeCtl 进行减 1，代表做完了属于自己的任务
                if (U.compareAndSwapInt(this, SIZECTL, sc = sizeCtl, sc - 1)) {
                    // 任务结束，方法退出
                    if ((sc - 2) != resizeStamp(n) << RESIZE_STAMP_SHIFT)
                        return;
           //也就是说，所有的迁移任务都做完了，也就会进入到上面的 if(finishing){} 分支了
                    finishing = advance = true;
                    i = n; // recheck before commit
                }
            }
            // 遍历的节点为null，则放入到ForwardingNode 指针节点
            else if ((f = tabAt(tab, i)) == null)
                advance = casTabAt(tab, i, null, fwd);
            //f.hash == -1 表示遍历到了ForwardingNode节点，意味着该节点已经处理过了
            else if ((fh = f.hash) == MOVED)
                advance = true; // already processed
            else {
                // 节点加锁,开始处理数组该位置处的迁移工作
                synchronized (f) {
                    if (tabAt(tab, i) == f) {
                        Node<K,V> ln, hn;
                        // 头结点的 hash 大于 0，说明是链表的 Node 节点
                        if (fh >= 0) {
                             //构造两个链表：一个是原链表，另一个是原链表的反序排列
                            int runBit = fh & n;
                            Node<K,V> lastRun = f;
                            for (Node<K,V> p = f.next; p != null; p = p.next) {
                                int b = p.hash & n;
                                if (b != runBit) {
                                    runBit = b;
                                    lastRun = p;
                                }
                            }
                            if (runBit == 0) {
                                ln = lastRun;
                                hn = null;
                            }
                            else {
                                hn = lastRun;
                                ln = null;
                            }
                            for (Node<K,V> p = f; p != lastRun; p = p.next) {
                                int ph = p.hash; K pk = p.key; V pv = p.val;
                                if ((ph & n) == 0)
                                    ln = new Node<K,V>(ph, pk, pv, ln);
                                else
                                    hn = new Node<K,V>(ph, pk, pv, hn);
                            }
                            //在nextTable i 位置处插上链表
                            setTabAt(nextTab, i, ln);
                            // 在nextTable i + n 位置处插上链表
                            setTabAt(nextTab, i + n, hn);
                            //在table[i]位置处插上ForwardingNode表示该节点已经处理过
                            //将原数组该位置处设置为 fwd，代表该位置已经处理完毕，
                        //    其他线程一旦看到该位置的 hash 值为 MOVED，就不会进行迁移了
                            setTabAt(tab, i, fwd);
                            // advance 设置为 true，代表该位置已经迁移完毕 
                            advance = true;
                        }
                        //如果是TreeBin，则按照红黑树进行处理，处理逻辑与上面一致
                        else if (f instanceof TreeBin) {
                            TreeBin<K,V> t = (TreeBin<K,V>)f;
                            TreeNode<K,V> lo = null, loTail = null;
                            TreeNode<K,V> hi = null, hiTail = null;
                            int lc = 0, hc = 0;
                            for (Node<K,V> e = t.first; e != null; e = e.next) {
                                int h = e.hash;
                                TreeNode<K,V> p = new TreeNode<K,V>
                                    (h, e.key, e.val, null, null);
                                if ((h & n) == 0) {
                                    if ((p.prev = loTail) == null)
                                        lo = p;
                                    else
                                        loTail.next = p;
                                    loTail = p;
                                    ++lc;
                                }
                                else {
                                    if ((p.prev = hiTail) == null)
                                        hi = p;
                                    else
                                        hiTail.next = p;
                                    hiTail = p;
                                    ++hc;
                                }
                            }
                            // 如果一分为二后，节点数<=6，那么将红黑树转换回链表
                            ln = (lc <= UNTREEIFY_THRESHOLD) ? untreeify(lo) :
                                (hc != 0) ? new TreeBin<K,V>(lo) : t;
                            hn = (hc <= UNTREEIFY_THRESHOLD) ? untreeify(hi) :
                                (lc != 0) ? new TreeBin<K,V>(hi) : t;
                            // 将 ln 放置在新数组的位置 i
                            setTabAt(nextTab, i, ln);
                            // 将 hn 放置在新数组的位置 i+n
                            setTabAt(nextTab, i + n, hn);
                            // 将原数组该位置处设置为 fwd，代表该位置已经处理完毕，
                        //    其他线程一旦看到该位置的 hash 值为 MOVED，就不会进行迁移了
                            setTabAt(tab, i, fwd);
                            // advance 设置为 true，代表该位置已经迁移完毕
                            advance = true;
                        }
                    }
                }
            }
        }
    }
```

addCount()：计算ConcurrentHashMap的size

```java
private final void addCount(long x, int check) {
        CounterCell[] as; long b, s;
        if ((as = counterCells) != null ||
            !U.compareAndSwapLong(this, BASECOUNT, b = baseCount, s = b + x)) {
            CounterCell a; long v; int m;
            boolean uncontended = true;
            //如果多个线程都在执行，则CAS失败，执行fullAddCount，全部加入count
            if (as == null || (m = as.length - 1) < 0 ||
                (a = as[ThreadLocalRandom.getProbe() & m]) == null ||
                !(uncontended =
                  U.compareAndSwapLong(a, CELLVALUE, v = a.value, v + x))) {
                fullAddCount(x, uncontended);
                return;
            }
            if (check <= 1)
                return;
            s = sumCount();
        }
    	//check>=0表示需要进行扩容操作
        if (check >= 0) {
            Node<K,V>[] tab, nt; int n, sc;
            while (s >= (long)(sc = sizeCtl) && (tab = table) != null &&
                   (n = tab.length) < MAXIMUM_CAPACITY) {
                int rs = resizeStamp(n);
                if (sc < 0) {
                    if ((sc >>> RESIZE_STAMP_SHIFT) != rs || sc == rs + 1 ||
                        sc == rs + MAX_RESIZERS || (nt = nextTable) == null ||
                        transferIndex <= 0)
                        break;
                    if (U.compareAndSwapInt(this, SIZECTL, sc, sc + 1))
                        transfer(tab, nt);
                }
                else if (U.compareAndSwapInt(this, SIZECTL, sc,
                                             (rs << RESIZE_STAMP_SHIFT) + 2))
                    transfer(tab, null);
                s = sumCount();
            }
        }
    }
```

查找

> 1. 计算hash值，定位到该table索引位置，如果是首节点符合就返回
> 2. 如果遇到扩容的时候，会调用标志正在扩容节点ForwardingNode的find方法，查找该节点，匹配就返回
> 3. 以上都不符合的话，就往下遍历节点，匹配就返回，否则最后就返回null

```java
public V get(Object key) {
        Node<K,V>[] tab; Node<K,V> e, p; int n, eh; K ek;
    	//计算hash
        int h = spread(key.hashCode());
    //如果table不为空 且 table长度大于0 且 计算出的下标上bucket不为空，则代表这个bucket存在，进入到bucket中查找，其中(n - 1) & h为计算出键key相对应的数组下标的算法。
        if ((tab = table) != null && (n = tab.length) > 0 &&
            (e = tabAt(tab, (n - 1) & h)) != null) {
            if ((eh = e.hash) == h) {
                //如果首节点的Node元素就是要查找的元素 return 
                if ((ek = e.key) == key || (ek != null && key.equals(ek)))
                    return e.val;
            }
            //如果头节点的哈希地址小于0，则代表该位置存的是红黑树，在红黑树中查找
            else if (eh < 0)
                return (p = e.find(h, key)) != null ? p.val : null;
            //如果头节点的哈希地址不小于0，则代表当前位置为链表，遍历链表，在链表中查找
            while ((e = e.next) != null) {
                if (e.hash == h &&
                    ((ek = e.key) == key || (ek != null && key.equals(ek))))
                    return e.val;
            }
        }
        return null;
    }
```

#### HashMap、Hashtable、ConcurrentHashMap

|              | HashMap                                     | ConcurrentHashMap                                            | Hashtable                                         |
| ------------ | ------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------- |
| extends      | abstract AbstractMap                        | abstract AbstractMap                                         | abstract Dictionary                               |
| implements   | Map                                         | ConcurrentMap                                                | Map                                               |
| 数据结构     | 数组+链表+红黑树                            | 数组+链表+红黑树                                             | 数组+链表                                         |
| 默认容量大小 | 16                                          | 16                                                           | 11                                                |
| 默认负载因子 | 0.75                                        | 0.75                                                         | 0.75                                              |
| 扩容         | 当键值对数量大于阈值，则容量扩容到原来的2倍 | 当键值对数量大于等于sizeCtl，单线程创建新哈希表，多线程复制bucket到新哈希表，容量扩容到原来的2倍 | 当键值对数量大于等于阈值，则容量扩容到原来的2倍+1 |
| 线程安全     | 否                                          | 是CAS+synchronized锁某个数组某个下标位置，效率高             | 是(synchronized锁整个数组，效率较低)              |
| hash地址     | hahsCode^(hashCode>>>16)                    | (hahsCode^ (hahsCode>>> 16)) & 0x7fffffff                    | hashCode                                          |
| 定位         | hash&(n-1)                                  | hash&(n-1)                                                   | (hash & 0x7FFFFFFF) % n                           |
| key为null值  | 是                                          | 否                                                           | 否                                                |
| 链表插入     | 尾插法                                      | 尾插法                                                       | 头插法                                            |
| 统计元素数量 | 成员变量size                                | 遍历CounterCell数组的值进行累加，最后加上baseCount的值即为size | 成员变量count                                     |

#### 红黑树



