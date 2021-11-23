---
title: ConcurrentHashMap
date: 2021-11-22 19:29:02
updated: 2021-11-22 19:29:02
categories: [Professional,Java,Java集合]
tags: [Java集合,线程安全,CAS,Volatile]
---
> ConcurrentHashMap 的基本实现逻辑与 {% post_link HashMap %}  相似，数组 + 链表 + {% post_link 红黑树  %}。

1. ## 历史版本

    注：ConcurrentHashMap ==在 jdk1.5 上引入的，在 jdk1.8 中出现源码变动。==
    在 jdk1.5 到 jdk1.7，ConcurrentHashMap 底层数据结构是 Segment 数组组成，每个 Segment 类似于一个 HashTable。
    ```Java
    /**
         * Stripped-down version of helper class used in previous version,
         * declared for the sake of serialization compatibility
         */
        static class Segment<K,V> extends ReentrantLock implements Serializable {
            private static final long serialVersionUID = 2249069246763182397L;
            final float loadFactor;
            Segment(float lf) { this.loadFactor = lf; }
        }
    ```

    Segment 继承于 {% post_link ReentrantLock %} ，在 put 操作时，先根据 hash 计算索引定位到具体的 Segment，然后在操作时只需要锁住相应 Segment 就可以了。

    <!-- more -->

    jdk1.8 中还保留了 segment 主要是兼容低版本 jdk。
    Segment 的个数最多只有 16（默认 16）,默认负载因子 0.75，==扩容只针对某个 Segment 的内部，Segment 初始化后不会变动==。
    有一点需要注意，ConcurrentHashMap ==先判断是否需要扩容，等扩容完成后插入值。==
    put 操作不允许 `key = null` 和 `value = null`。
    ConcurrentHashMap 的 get 操作是没有加锁的，原因在于变量都使用 {% post_link Volatile %} 修饰。

    ```Java
    /**
         * The array of bins. Lazily initialized upon first insertion.
         * Size is always a power of two. Accessed directly by iterators.
         */
    transient volatile Node<K,V>[] table;
    
        /**
         * The next table to use; non-null only while resizing.
         */
    private transient volatile Node<K,V>[] nextTable;
    ```

    ```Java
    public native Object getObjectVolatile(Object var1, long var2);
    ```

2. ## J**dk1.8 的改动**

    jdk1.8 的主要改动了底层的数据结构，采用和 {% post_link HashMap %} 一样的 数组 + 链表 + {% post_link 红黑树 %} 的形式。加锁方式采用 {% post_link CAS %} + {% post_link Synchronized %}。
    相关操作会直接重置所有节点 hash 的为相应的值，在其他操作是会对应判断。
    ```Java
    static final int MOVED     = -1; // hash for forwarding nodes （数组迁移到新数组时会使用）
    static final int TREEBIN   = -2; // hash for roots of trees
    static final int RESERVED  = -3; // hash for transient reservations
    ```

    ```Java
    //迁移数组时的过程中会预先将 Node 设置为 ForwardingNode
    static final class ForwardingNode<K,V> extends Node<K,V> {
            final Node<K,V>[] nextTable;
            ForwardingNode(Node<K,V>[] tab) {
                super(MOVED, null, null, null);
                this.nextTable = tab;
            }
    }
    ```

    比如，在 get 是如果发现 `hash < 0`，会重复查询操作。
    ```Java
    Node<K,V> find(int h, Object k) {
                // loop to avoid arbitrarily deep recursion on forwarding nodes
                outer: for (Node<K,V>[] tab = nextTable;;) {
                    Node<K,V> e; int n;
                    if (k == null || tab == null || (n = tab.length) == 0 ||
                        (e = tabAt(tab, (n - 1) & h)) == null)
                        return null;
                    for (;;) {
                        int eh; K ek;
                        if ((eh = e.hash) == h &&
                            ((ek = e.key) == k || (ek != null && k.equals(ek))))
                            return e;
                        if (eh < 0) {
                            if (e instanceof ForwardingNode) {
                                tab = ((ForwardingNode<K,V>)e).nextTable;
    			    //发现查找的还是在移动的	
                                continue outer;
                            }
                            else
                                return e.find(h, k);
                        }
                        if ((e = e.next) == null)
                            return null;
                    }
                }
            }
    }
    ```

3. ## 总结

    1. 整体来看，ConcurrentHashMap 采用了与 HashMap 的数据结构和计算逻辑。不同之处在于 ConcurrentHashMap 用 {% post_link Volatile %} 修饰数组。
    2. ConcurrentHashMap 采用 {% post_link CAS %} + {% post_link Synchronized %} + {% post_link Volatile %} 保证的数据的线程安全。
    3. ConcurrentHashMap 的查询操作没有加锁，遍历速度更快（遇到正在数据正在修改，会重复查询操作）。

　　