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

https://www.javanav.com/interview/98abdda5f6fb4841ada42d45177e5c5a.html

#### HashMap

HashMap是一个用于存储Key-Value键值对的集合，每一个键值对也叫做Node。这些个键值对（Node）分散存储在一个数组当中，这个数组就是HashMap的主干,HashMap数组每一个元素的初始值都是Null。

内部使用 **数组+链表/红黑树**

当链表长度>=8 & 数组长度的大小>=64 时，链表会转换为红黑树

HashMap的数据插入原理：

![HashMap初始化](HashMap初始化.png)

1. 判断数组是否为空，为空进行初始化;
2. 不为空，计算 key 的 hash 值，通过(n - 1) & hash计算应当存放在数组中的下标 index;
3. 查看 table[index] 是否存在数据，没有数据就构造一个Node节点存放在 table[index] 中；
4. 存在数据，说明发生了hash冲突(存在二个节点key的hash值一样), 继续判断key是否相等，相等，用新的value替换原数据(onlyIfAbsent为false)；
5. 如果不相等，判断当前节点类型是不是树型节点，如果是树型节点，创造树型节点插入红黑树中；(如果当前节点是树型节点证明当前已经是红黑树了)
6. 如果不是树型节点，创建普通Node加入链表中；判断链表长度是否大于 8并且数组长度大于64， 大于的话链表转换为红黑树
7. 插入完成之后判断当前节点数是否大于阈值，如果大于开始扩容为原数组的二倍

##### 查找

+ 计算hash值:对hashCode的值重新计算，增加hash的复杂度，因为下面是通过hash&(length-1)来计算位置的，如果length过小，那么只有hash的低几位参与&运算，而高位数据在没有任何用处；因此让hashCode高16位与低16位进行异或，以此增大低位信息的随机性，使得hash尽可能的分布开来，降低hash的冲突

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
    //HashMap在结构上被修改的次数加1
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
            //static final int MAXIMUM_CAPACITY = 1 << 30;
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

