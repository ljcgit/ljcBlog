## 1.Collection 和 Collections 有什么区别？
Java.util.Collection是一个集合接口（集合类的一个顶级接口）。它提供了对集合对象进行基本操作的通用接口方法。 Collection接口在Java类库中有很多具体的实现。Collection接口的意义是为各种具体的集合提供最大化的统一操作方式，其直接继承接口有List与Set。

Collections则是集合类的一个工具类/帮助类，其中提供了一系列的静态方法，用于对集合中元素进行排序，搜索以及线程安全等各种操作。

## 2.List、Set、Map 之间的区别是什么？
+ **List:**
1. 可以允许重复的对象；
2. 可以插入多个null元素；
3. 是一个有序容器，保持了每个元素的插入顺序，输出的顺序就是插入的顺序‘
4. 常用的实现类有ArrayList，LinkedList和Vector。ArrayList最为流行，它提供了使用索引的随意访问，而LinkedList则对于经常需要从List中添加或删除元素的场合更为合适。
> + ArrayList:长度可变的数组，可以对元素进行随机的访问，向ArrayList中插入元素与删除元素的速度慢。JDK8 中ArrayList扩容的实现是通过grow()方法里使用语句newCapacity = oldCapacity + (oldCapacity >> 1)（即1.5倍扩容）计算容量，然后调用Arrays.copyof()方法进行对原数组进行复制。
> + LinkedList:采用链表数据结构，插入和删除速度块，但访问速度慢。

+ **Set:**
1. 不允许重复对象；
2. 无序容器，你无法保证每个元素的存储顺序，TreeSet通过Comparator或者Comparable维护了一个排序顺序；
3. 只允许一个null元素；
4. Set接口最流行的几个实现类是HashSet，LinkedHashSet以及TreeSet。最流行的是基于HashMap实现的HashSet；TreeSet还实现了SortedSet接口，因此TreeSet是一个根据其compare()和compareTo()的定义进行排序的有序容器。
> + HashSet： HashSet按照哈希算法来存取集合中的对象，存取速度比较快。当HashSet中的元素个数超过数组大小*loadFactor（默认值为0.75）时，就会进行近似两倍扩容（newCapacity = (oldCapacity << 1) + 1）。
> + TreeSet ：TreeSet实现了SortedSet接口，能够对集合中的对象进行排序。


+ **Map:**
1. Map不是collection的子接口或者实现类。Map是一个接口；
2. Map的每个Entry都持有两个对象，也就是一个键一个值，Map可能会持有相同的值对象但键对象必须是唯一的；
3. TreeMap也通过Comparator或者Comparable维护了一个排序顺序；
4. Map里你可以拥有随意个null值但最多只能有一个null键；
5. Map接口最流行的几个实现类是HashMap，LinkedHashMap，Hashtable和TreeMap。（HashMap和TreeMap最常用）。
> + HashMap：HashMap基于散列表实现，其插入和查询<K,V>的开销是固定的，可以通过构造器设置容量和负载因子来调整容器的性能。
> + LinkedHashMap：类似于HashMap，但是迭代遍历它时，取得<K,V>的顺序是其插入次序，或者是最近最少使用(LRU)的次序。
> + TreeMap：TreeMap基于红黑树实现。查看<K,V>时，它们会被排序。TreeMap是唯一的带有subMap()方法的Map，subMap()可以返回一个子树。


> **什么场景下使用list，set，map呢？**
> + 如果你经常会使用索引来对容器中的元素进行访问，那么 List 是你的正确的选择。如果你已经知道索引了的话，那么 List 的实现类比如 ArrayList 可以提供更快速的访问,如果经常添加删除元素的，那么肯定要选择LinkedList。
> + 如果你想容器中的元素能够按照它们插入的次序进行有序存储，那么还是 List，因为 List 是一个有序容器，它按照插入顺序进行存储。
> + 如果你想保证插入元素的唯一性，也就是你不想有重复值的出现，那么可以选择一个 Set 的实现类，比如 HashSet、LinkedHashSet 或者 TreeSet。所有 Set 的实现类都遵循了统一约束比如唯一性，而且还提供了额外的特性比如 TreeSet 还是一个 SortedSet，所有存储于 TreeSet 中的元素可以使用 Java 里的 Comparator 或者 Comparable 进行排序。LinkedHashSet 也按照元素的插入顺序对它们进行存储。
> + 如果你以键和值的形式进行数据存储那么 Map 是你正确的选择。你可以根据你的后续需要从 Hashtable、HashMap、TreeMap 中进行选择。

## 3.HashMap 和 Hashtable 有什么区别？
1.HashMap几乎可以等价于Hashtable，除了HashMap是非synchronized的，并可以接受null(HashMap可以接受为null的键值(key)和值(value)，而Hashtable则不行)。
2.HashMap是非synchronized，而Hashtable是synchronized，这意味着Hashtable是线程安全的，多个线程可以共享一个Hashtable；而如果没有正确的同步的话，多个线程是不能共享HashMap的。Java 5提供了ConcurrentHashMap，它是HashTable的替代，比HashTable的扩展性更好。
3.另一个区别是HashMap的迭代器(Iterator)是fail-fast迭代器，而Hashtable的enumerator迭代器不是fail-fast的。所以当有其它线程改变了HashMap的结构（增加或者移除元素），将会抛出ConcurrentModificationException，但迭代器本身的remove()方法移除元素则不会抛出ConcurrentModificationException异常。但这并不是一个一定发生的行为，要看JVM。这条同样也是Enumeration和Iterator的区别。
4.由于Hashtable是线程安全的也是synchronized，所以在单线程环境下它比HashMap要慢。如果你不需要同步，只需要单一线程，那么使用HashMap性能要好过Hashtable。
5.HashMap不能保证随着时间的推移Map中的元素次序是不变的。
6.HashMap的key-value支持key-value，null-null，key-null，null-value四种。而Hashtable只支持key-value一种（即key和value都不为null这种形式）。既然HashMap支持带有null的形式，那么在HashMap中不能由get()方法来判断HashMap中是否存在某个键， 而应该用containsKey()方法来判断，因为使用get的时候，当返回null时，你无法判断到底是不存在这个key，还是这个key就是null，还是key存在但value是null。


## 4.哪些集合类是线程安全的？
**线程安全的集合对象：**
+ Vector
+ Hashtable
+ StringBuffer

**非线程安全的集合对象：**
+ ArrayList
+ LinkedList
+ HashMap
+ HashSet
+ TreeMap
+ TreeSet
+ StringBuilder

## 5.如何实现数组和 List 之间的转换？
**List转数组**
+ for循环
+ 利用list对象的toArray(T[] a)方法
> 一定要加上T[] a参数，不然会导致无法转换类型成功。

**数组转List**
+ for循环
+ 直接使用Arrays.asList()方法：
List<String> list = Arrays.asList(arrays);
+ 将asList()方法返回的值作为new的参数：
ArrayList<String> arrayList = new ArrayList<String>(Arrays.asList(arrays));
> 注意：虽然asList()方法返回的是一个ArrayList的对象，不过该对象是数组java.util.Arrays类中的，而不是java.util.ArrayList。所以建议用第三种方法转换，不然在调用方法上可能会出现错误（add）。
+ Collections.addAll()

## 6.在 Queue 中 poll()和 remove()有什么区别？
在对空集合进行操作时，poll()会返回一个null，而remove()直接报异常。
