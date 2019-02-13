Q:Java集合
=== 
![Image text](https://github.com/IceDarron/Note/blob/master/Image/jdk_collection.png)
![Image text](https://github.com/IceDarron/Note/blob/master/Image/jdk_LinkedHash.png)
+ Iterator
>java.util.Iterator接口主要定义了遍历集合对象的方法，通过迭代器模式，创建迭代器来遍历各种集合，每个集合中都有具体的实现。通过迭代器遍历时，在多线程下存在遍历过程中，另一个线程修改集合，这时候通过fast-fail机制可以抛出异常。

+ Collection
>The root interface in the <i>collection hierarchy</i>.  A collection represents a group of objects, known as its <i>elements</i>.  Some
 collections allow duplicate elements and others do not.  Some are ordered and others unordered.  The JDK does not provide any  <i>direct</i> implementations of this interface: it provides implementations of more specific subinterfaces like <tt>Set</tt> and  <tt>List</tt>.  This interface is typically used to pass collections around and manipulate them where maximum generality is desired.

+ List
>具体实现类：ArrayList（数组），Vector（数组），LinkedList（双向链表），Stack（数组）

+ Set
>具体实现类：HashSet（散列表HashMap），TreeSet（二叉树），LinkedHashSet（LinkedHashMap通过继承HashSet并使用它的构造函数实现）
TreeSet 是一个有序的集合，它的作用是提供有序的Set集合。它继承于AbstractSet抽象类，实现了NavigableSet<E>, Cloneable, java.io.Serializable接口。
TreeSet 继承于AbstractSet，所以它是一个Set集合，具有Set的属性和方法。
TreeSet 实现了NavigableSet接口，意味着它支持一系列的导航方法。比如查找与指定目标最匹配项。
TreeSet 实现了Cloneable接口，意味着它能被克隆。
TreeSet 实现了java.io.Serializable接口，意味着它支持序列化。
TreeSet是基于TreeMap实现的。TreeSet中的元素支持2种排序方式：自然排序 或者 根据创建TreeSet 时提供的 Comparator 进行排序。这取决于使用的构造方法。

+ Queue
>具体实现类：LinkedBlockingQueue（单向链表实现的阻塞队列），PriorityQueue（数组），ArrayDeque（数组，可以实现队列或栈）
Queue本身是一种先入先出的模型(FIFO)。
Deque是Queue的子接口，是一种增强，代表一个双端队列。同时Deque不仅可以作为双端队列使用，而且可以被当成栈来使用，所以可以使用出栈，入栈的方法。

+ Map
>具体实现类：HashMap（散列表），TreeMap（红黑树数据结构），LinkedHashMap（散列表+双向链表）
>TreeMap 是一个有序的key-value集合，它是通过红黑树实现的。
TreeMap 继承于AbstractMap，所以它是一个Map，即一个key-value集合。
TreeMap 实现了NavigableMap接口，意味着它支持一系列的导航方法。比如返回有序的key集合。
TreeMap 实现了Cloneable接口，意味着它能被克隆。
TreeMap 实现了java.io.Serializable接口，意味着它支持序列化。
TreeMap基于红黑树（Red-Black tree）实现。该映射根据其键的自然顺序进行排序，或者根据创建映射时提供的 Comparator 进行排序，具体取决于使用的构造方法。
TreeMap的基本操作 containsKey、get、put 和 remove 的时间复杂度是 log(n) 。
另外，TreeMap是非同步的。 它的iterator 方法返回的迭代器是fail-fastl的。
TreeMap的本质是R-B Tree(红黑树)，它包含几个重要的成员变量： root, size, comparator。
root是红黑数的根节点。它是Entry类型，Entry是红黑数的节点，它包含了红黑数的6个基本组成成分：key(键)、value(值)、left(左孩子)、right(右孩子)、parent(父节点)、color(颜色)。Entry节点根据key进行排序，Entry节点包含的内容为value。红黑数排序时，根据Entry中的key进行排序；Entry中的key比较大小是根据比较器comparator来进行判断的。size是红黑数中节点的个数。

+ Collections&Arrays
>提供集合和数组之间的转换，排序，最大值等方法。