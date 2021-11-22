---
title: Java集合
thumbnail: https://cdn.jsdelivr.net/gh/ZengHGitHub/imgs/img/202111101642332.png
date: 2021-11-22 19:16:35
categories: [Professional,Java,Java集合]
tags: [Java,Java集合,数据结构,线程安全]
---

1. ## **Java 集合关系**

　　![](https://cdn.jsdelivr.net/gh/ZengHGitHub/imgs/img/202111101642332.png)
整体上集合分为 `Collection` 和 `Map`, 所有集合的实现类都要实现其中一个。

<!-- more -->

2. ## Collection

   1. ### `Iterable` 接口

   意为可迭代的。主要提供集合的迭代功能。
   Iterable 源码逻辑很简单，仅包含三个方法：
   创建迭代器 (Iterator)、实现 `foreach` 的方法（注意使用时容易出现 {% post_link checkForComodification %}）以及在 jdk8 引入的供 `Stream` 使用的并行迭代器 (`Spliterator`)。每个实现了这个接口的实现类都必须实现自己的迭代器。
   ![](https://cdn.jsdelivr.net/gh/ZengHGitHub/imgs/img/202111101704427.png)

   2. ### `Collection` 接口

   ![](https://cdn.jsdelivr.net/gh/ZengHGitHub/imgs/img/202111101815140.png)
   提供集合的基本功能方法，包括 `size` 、 `contains` 、 `add` 、 `remove` 、 `empty` 等。

   3. ### `List` 接口和 `AbstractList` 抽象类

   ![](https://cdn.jsdelivr.net/gh/ZengHGitHub/imgs/img/202111101820693.png)   ![](https://cdn.jsdelivr.net/gh/ZengHGitHub/imgs/img/202111101821048.png)
   `List` 直接继承自 `Collection`，所以包含 `Collection` 的全部接口，但同时 `List` 也是有序容器，所以有自己特有的一些方法，主要是对容器中指定位置元素的操作。
   `AbsrtactList` 实现了 `List` 的大部分接口以及来自 `Iterable` 的 Iterator () 方法。

   4. ### `Vector` 、 `Stack` 和 `ArrayList` 类

   ![](https://cdn.jsdelivr.net/gh/ZengHGitHub/imgs/img/202111101829343.png)
   ![](https://cdn.jsdelivr.net/gh/ZengHGitHub/imgs/img/202111101846281.png)
   ![](https://cdn.jsdelivr.net/gh/ZengHGitHub/imgs/img/202111101847412.png)
   `Vector` 、 `Stack` 和 `ArrayList` 底层都是可增长的对象数组。
   `Vector` 初始化时可以指定初始容量（默认 10）和扩容数量（默认当前容量），数组在对象初始化时初始化。每次增加元素时，检查当前数组已满，如果满则按旧的数组长度扩容，如果扩容后的数组长度还是不足以装下所有元素，则新数组长度按更大的值算。每次删除指定元素时，都会生成新的数组。
   `ArrayList` 的逻辑大部分与 `Vector` 相同。不同点在于，`ArrayList` 初始化时不会初始化数组，并且 `ArrayList` 的扩容因子固定在 1.5 不能变动。
   最重要的一点是 `Vector` 的方法都被  {% post_link Synchronized %} 修饰，这意味着 `Vector` 相比 `ArrayList` 是线程安全的，但是访问速度也更慢。    `Stack` 是直接继承自 `Vector` 的类，公有方法也都被{% post_link Synchronized %} 修饰。
   **考点：**
   1）为什么推荐使用 `ArrayList` 而非 `Vector`？
   在不需要考虑线程安全的情况下，由于 `Vector` 方法都被 {% post_link Synchronized %} 修饰 `ArrayList` 拥有更好的效率。并且 `Vector` 默认扩容一倍，比 `ArrayList` 占有更多的空间。
   从历史原因来看，Vector 是 jdk1.0 的产物，后续的 jdk 需要兼容 Vector。
   如果需要保证线程安全，`Collections. SynchronizedList ()` 可以让 ArrayList 变得线程安全。
   那么问题来了，{% post_link Collections. SynchronizedList ()、CopyOnWriteArrayList 与 Vector 有什么区别?  %}
   2） `ArrayList` 有哪些优缺点？
    由于 ArrayList 底层是数组，它的优点在于随机访问和顺序添加元素。
    缺点也很明显，指定位置删除或添加元素时，需要重新生成数组，耗费性能。
   3)为什么推荐使用 `Deque` 而非 `Stack`?
    Deque 作为双端队列，可以实现 Stack 的先进后出的功能，同时 Stack 继承自 Vector 效率较低。

   5. ### `Set` 、 `HashSet` 、 `TreeSet` 、 `LinkedHashSet` 、 `EnumSet` 、 `NavigableSet`

   HashSet 的继承关系如下：
   ![](https://cdn.jsdelivr.net/gh/ZengHGitHub/imgs/img/202111120056130.png)
   需要额外说明的一点是 AbstractSet 重写了三个方法：
   `equal ()` ：依次比较地址、类型、元素个数、每个元素（重写自 Object）。
   `hashCode ()` ：将所有元素的 hashcode 都加上（重写自 Object）。
   `removeAll ()` ：增加了一个判断，如果要删除的集合元素个数比原集合少，则改为遍历要删除的集合，尝试在在原集合中删除（遍历查找）。（重写自 `AbstractCollection`）
   `HashSet` 内部使用 `HashMap` 保存数据。相关操作参考 `HashMap`。
   `TreeSet` 的情况也是如此，内部使用 `TreeMap` 保存数据。
   ![](https://cdn.jsdelivr.net/gh/ZengHGitHub/imgs/img/202111120324902.png)
   不过 LinkedHashSet 的情况比较特殊。
   ![](https://cdn.jsdelivr.net/gh/ZengHGitHub/imgs/img/202111120147008.png)

   ```Java
   public class LinkedHashSet<E> extends HashSet<E> implements Set<E>, Cloneable, java.io.Serializable {  
     
    private static final long serialVersionUID = -2851667679971038690L;  
     
    public LinkedHashSet(int initialCapacity, float loadFactor) {  
    	super(initialCapacity, loadFactor, true);  
    }  
     
    public LinkedHashSet(int initialCapacity) {  
    	super(initialCapacity, .75f, true);  
    }  
     
    public LinkedHashSet() {  
    	super(16, .75f, true);  
    }  
    
    public LinkedHashSet(Collection<? extends E> c) {  
    	super(Math.max(2*c.size(), 11), .75f, true);  
    	addAll(c);  
    }  
     
    @Override  
    public Spliterator<E> spliterator() {  
    	return Spliterators.spliterator(this, Spliterator.DISTINCT | Spliterator.ORDERED);  
    }  
   }
   ```

   表面上 `LinkedHashSet` 只继承自 `HashSet`。代码也非常简单。
   但构造方法调用了 `HashSet` 专门准备的构造方法：

   ```Java
    HashSet(int initialCapacity, float loadFactor, boolean dummy) { 
    	map = new LinkedHashMap<>(initialCapacity, loadFactor);  
    }
   ```

   所以在创建的时候实际创建的时 LinkedHashMap。
   `EnumSet` 是抽象类，应用场景为集合参数是枚举值。

   6. ### `Queue` 、 `Deque` 、 `ArrayDeque` 、 `PriorityQueue` 、 `LinkedList`

   `Queue` （队列） 需要满足可以从队尾添加元素并从头部删除。
   `DeQeque` （双端队列） 需要满足从头尾都可以添加和删除元素, 但不允许从中间添加元素。
   ![](https://cdn.jsdelivr.net/gh/ZengHGitHub/imgs/img/202111120214663.png)
   `ArrayDeque` 是常见的双端队列，底层数据结构是数组，通过分别指向首尾的两个索引控制队列。
   ![](https://cdn.jsdelivr.net/gh/ZengHGitHub/imgs/img/202111120312032.png)
   `LinkedList` 同时拥有 `List` 和 `Deque` 的属性。底层数据结构是链表，两个指针分别指向链表的首部和尾部。保证可以 `LinkedList` 从首部和尾部均可操作元素。
   {% post_link PriorityQueue %}（优先队列)实现了{% post_link 小根堆 堆%}。

3. ## Map

   1. ### `Map`、`HashMap`、`HashTable`、`LinkedHashMap`、`SortedMap`、`TreeMap`、`WeakHashMap`

   与 `Collection` 不同，`Map` 是由多个键值映射组成的，Map 映射中不能存在重复的键。
   提供返回键集（即 Set）、值集（即 Collection）、键值映射集（即键值 Set 集合）。
   其中比较重要的实现类是 {% post_link HashMap %}、{% post_link LinkedHashMap %}、{% post_link HashTable %}、{% post_link TreeMap %}、{% post_link ConcurrentHashMap %}、{% post_link WeakHashMap%}。

