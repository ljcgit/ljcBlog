## 1.构造函数
```java
    public HashMap(int initialCapacity, float loadFactor) {
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal initial capacity: " +
                                               initialCapacity);
        if (initialCapacity > MAXIMUM_CAPACITY)
            initialCapacity = MAXIMUM_CAPACITY;
        if (loadFactor <= 0 || Float.isNaN(loadFactor))
            throw new IllegalArgumentException("Illegal load factor: " +
                                               loadFactor);
        this.loadFactor = loadFactor;
        this.threshold = tableSizeFor(initialCapacity);
    }
```

#### 1.1tableSizeFor()
```java
    static final int tableSizeFor(int cap) {
        int n = cap - 1;
        n |= n >>> 1;  //将最高位及第二位都变1
        n |= n >>> 2;   //前四位都变成1
        n |= n >>> 4;   
        n |= n >>> 8;
        n |= n >>> 16;
        return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
    }
```
首先会对cap进行减一操作。这是为了避免cap已经是2的幂，如果不执行减1操作，则进行完后面的无符号右移操作后，threshold的大小将会是cap的2倍减1。



## 2.put()方法
```java
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        //如果table为空
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
        //利用(n-1)&hash来实现取模效果，得到数组下标
        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
        else {
            Node<K,V> e; K k;
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
            else if (p instanceof TreeNode)
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            else {
                for (int binCount = 0; ; ++binCount) {
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                    }
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
            }
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;
        if (++size > threshold)
            resize();
        afterNodeInsertion(evict);
        return null;
    }
```

#### 2.1 参数意义
     * @param hash hash for key
     * @param key the key
     * @param value the value to put
     * @param onlyIfAbsent if true, don't change existing value
     * @param evict if false, the table is in creation mode.
     * @return previous value, or null if none
+ onlyIfAbsent：true表示对相同的key，新值不会替换掉原来不为null的值。

#### 2.2 put流程
+ 首先判断原数组是否为null或者数组长度为0，若是则进行初始化大小；
+ 根据(n - 1) & hash得到数组的下标，如果当前下标位置处没有元素，则直接将该位置指向新添加的节点；
+ 如果存在元素，如果头节点的hash和key都相等，则直接找到相应节点；
+ 如果是以红黑树的形式存在，通过遍历红黑树来找到相应节点，如果找到相应节点，就将节点返回，否则将新节点添加到树中；
+ 如果以链表形式存在，则会进行遍历链表，如果未找到相同节点，就会创建一个新节点添加到链表中；
+ 当链表中添加新节点时，如果节点个数超过8，就会将链表转换为红黑树；
+ 如果找到了相应节点，会根据onlyIfAbsent状态值和原对象值是否为空，进行值的替换；
+ 如果是添加了新的元素，并且节点个数超过阈值就会对数组进行扩容。


## 3.resize()
```java
    final Node<K,V>[] resize() {
        Node<K,V>[] oldTab = table;
        int oldCap = (oldTab == null) ? 0 : oldTab.length;  //原容器大小
        int oldThr = threshold;
        int newCap, newThr = 0;
        if (oldCap > 0) {
            //如果容量超过了最大容量值，就将阈值设置为最大整数
            if (oldCap >= MAXIMUM_CAPACITY) {
                threshold = Integer.MAX_VALUE;
                return oldTab;
            }
            //将容器容量扩大一倍
            else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                     oldCap >= DEFAULT_INITIAL_CAPACITY)
                newThr = oldThr << 1; // 阈值也进行扩大一倍
        }
        else if (oldThr > 0) // 初始容量设置为阈值
            newCap = oldThr;
        else {               // 初始阈值为零表示使用默认值
            newCap = DEFAULT_INITIAL_CAPACITY;
            newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
        }
        //如果原来容量为并且原阈值大于0
        if (newThr == 0) {
            float ft = (float)newCap * loadFactor;
            newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                      (int)ft : Integer.MAX_VALUE);
        }
        threshold = newThr;
        @SuppressWarnings({"rawtypes","unchecked"})
            Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
        table = newTab;
        if (oldTab != null) {
            for (int j = 0; j < oldCap; ++j) {
                Node<K,V> e;
                if ((e = oldTab[j]) != null) {
                    oldTab[j] = null;
                   //如果原数组下标中只有一个元素
                    if (e.next == null)
                        newTab[e.hash & (newCap - 1)] = e;
                    else if (e instanceof TreeNode)
                        ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                    else { // preserve order
                        Node<K,V> loHead = null, loTail = null;
                        Node<K,V> hiHead = null, hiTail = null;
                        Node<K,V> next;
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
#### 3.1扩容操作
+ 将原数组长度和阈值通过移位操作扩大1倍；
+ 遍历每个下标位置处的值；
+ 如果只有一个节点，通过e.hash & (newCap - 1）计算扩容后的数组下标，将元素放入新下标处。
+ 如果是以红黑树的形式存在，通过(e.hash & bit) == 0判断最高为0，如果为0表示该节点下标值不变将其放去l树中，否则放入h树中；
+ 当分类完成后，判断l树元素个数是否小于等于6，如果小于等于6就将树转换为链表的形式，放入原下标位置处；
+ 同样判断h判断l树元素个数是否小于等于6，如果小于等于6就将树转换为链表的形式，放入原下标+原数组长度的位置处；
+ 如果原本以链表的形式存在的话，同样按照红黑树的分类方法，分为l树和h树，并放入相应的下标位置处。



##  4.split()
```java
        final void split(HashMap<K,V> map, Node<K,V>[] tab, int index, int bit) {
            TreeNode<K,V> b = this;
            // Relink into lo and hi lists, preserving order
            TreeNode<K,V> loHead = null, loTail = null;
            TreeNode<K,V> hiHead = null, hiTail = null;
            int lc = 0, hc = 0;
            for (TreeNode<K,V> e = b, next; e != null; e = next) {
                next = (TreeNode<K,V>)e.next;
                e.next = null;
                //如果e.hash的最高位为0，说明数组下标未变
                if ((e.hash & bit) == 0) {
                    if ((e.prev = loTail) == null)
                        loHead = e;
                    else
                        loTail.next = e;
                    loTail = e;
                    ++lc;
                }
                else {
                    if ((e.prev = hiTail) == null)
                        hiHead = e;
                    else
                        hiTail.next = e;
                    hiTail = e;
                    ++hc;
                }
            }
      
            if (loHead != null) {
                //如果l树数量小于等于6，就将l树变成链表，index位置指向该链表
                if (lc <= UNTREEIFY_THRESHOLD)
                    tab[index] = loHead.untreeify(map);
                else {
                    tab[index] = loHead;
                    if (hiHead != null) // (else is already treeified)
                        loHead.treeify(tab);
                }
            }
            if (hiHead != null) {
                if (hc <= UNTREEIFY_THRESHOLD)
                    tab[index + bit] = hiHead.untreeify(map);
                else {
                    tab[index + bit] = hiHead;
                    if (loHead != null)
                        hiHead.treeify(tab);
                }
            }
        }
```

## 5.为什么说HashMap是线程不安全的？
假如有两个线程要插入数据，线程A执行了put()方法，发现要插入的下标位置处数据为空，同时，线程B也执行了put()方法，得到了和A相同的数组下标，也发现该下标位置处数据为空，A就直接将数据放入该下标处，B也将数据放入该下标处，就会导致A的数据被B覆盖。



## 6. Java1.7和1.8hashmap的区别？
| | 1.7 | 1.8 |
| :-: | :-: | :-: |
| 数据结构 | 数组+链表 | 数组+链表+红黑树 |
| 插入数据方式 | 头插法 | 尾插法 | 
| 扩容后的存储位置的计算方式 | 重新进行&操作 | 原位置或原位置+原数组长度 | 
| key为null的处理 |  通过单独的方法执行 | 直接获取hash值时返回0 |


## 7.为什么负载因子的值为0.75？大一点会怎么样？小一点会怎么样？
如果负载因子过大，会导致每个数组位置存储节点过多，加大查询时间；如果过小，就会导致频繁的进行扩容操作，占用较多的内存空间。

## 8.为什么数组长度总是2的幂？
+ 不同的hash值发生的碰撞概率比较小，会使得数据在数组中分布均匀，空间利用率较高，查询速度较快。
+ (n - 1) & hash 可以实现取模效果，速度和效率比直接取模操作要好。

## 9. 红黑树的特性？
1.每个节点要么是红色，要么是黑色；
2.根节点永远是黑色的；
3.所有的叶节点都是是黑色的（注意这里说叶子节点其实是上图中的 NIL 节点）；
4.每个红色节点的两个子节点一定都是黑色；
5.从任一节点到其子树中每个叶子节点的路径都包含相同数量的黑色节点；

## 10. 红黑树比avl好的地方？
红黑树牺牲了严格的高度平衡的优越条件，来使得搜索、插入、删除的时间复杂度都是O(log2n)。

## 11.链表长度减小会从红黑树变为entry吗？
```java
            if (root == null || root.right == null ||
                (rl = root.left) == null || rl.left == null) {
                tab[index] = first.untreeify(map);  // too small
                return;
            }
```

## 12.ConcurrentHashMap如何实现线程安全？1.7和1.8？
#### 12.1 JDK1.7
JDK1.7中采用了分段锁技术，主要有Segment和HashEntry，其中Segment继承于ReentrantLock，HashEntry和HashMap相似，只是value和next都是volatile修饰的。

> 扩容：concurrencyLevel（并发度）一经指定，不可改变，只会增加Segment中链表数组的容量大小，这样可以保证不需要对整个ConcurrentHashMap做rehash操作。


#### 12.2 JDK1.8
JDK1.8通过Node[] + 链表/红黑树实现。

**put：**
先根据key的hash值，找到相应位置。
+ 如果相应位置的Node还未初始化，即为null，通过cas操作插入数据；
+ 如果相应位置不为空，且当前节点不处于MOVED状态，则对该节点加synchronized锁，如果该节点的hash不小于0，则遍历链表更新节点或插入新节点。
+ 如果该节点是红黑树类型，则通过putTreeVal方法往红黑中插入节点；
+ 如果binCount不为0，同时大于等于8，则将链表转换为红黑树，如果oldVal不为空，说明是更新操作，直接返回旧值；
+ 如果插入了一个新节点，就执行addCount()方法尝试更新元素个数baseCount。
