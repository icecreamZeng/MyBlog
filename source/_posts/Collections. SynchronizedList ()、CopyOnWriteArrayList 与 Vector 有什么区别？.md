---
title: Collections. SynchronizedList ()、CopyOnWriteArrayList 与 Vector 有什么区别？
date: 2021-11-22 19:19:33
updated: 2021-11-22 19:19:33
categories: [Professional,Java,Java集合]
toc: true
tags: [Java,Java集合,数据结构,线程安全,CopyOnWriteArrayList,Queue,Set]
---


> `Collections. SynchronizedList ()` 、 `CopyOnWriteArrayList` 和 `Vector` 都是线程安全的集合。
>

1. ## Vector

     `Vector` 的同步做法是给所有公有方法都加上 "Synchronized"[^1] 关键字。
    ```Java
     //Vector
     public synchronized boolean add(E e) {  
     	modCount++;  
     	ensureCapacityHelper(elementCount + 1);  
     	elementData[elementCount++] = e;  
     	return true;  
     }
    
    ```

<!-- more -->

2. ## Collections. SynchronizedList ()

    当使用 `Collections. SynchronizedList ()` 方法创建线程安全的 `List` 时，会根据 List 类型（时候实现了 {% post_link RandomAccessas %} 接口）返回 `SynchronizedRandomAccessList` 和 `SynchronizedList`，前者继承于后者，两者在使用上并无差别。
    ```Java
     // Collections.synchronizedList
     // 根据传入的List类型返回相应的内部类
     public static <T> List<T> synchronizedList(List<T> list) {  
     	return (list instanceof RandomAccess ? new SynchronizedRandomAccessList<>(list) : new SynchronizedList<>(list));  
    }  
      
     static <T> List<T> synchronizedList(List<T> list, Object mutex) { 
     return (list instanceof RandomAccess ? new SynchronizedRandomAccessList<>(list, mutex) : new SynchronizedList<>(list, mutex));  
    }
    ```

    ```Java
     //SynchronizedRandomAccessList 继承自 SynchronizedList
     static class SynchronizedRandomAccessList<E> extends SynchronizedList<E> implements RandomAccess {  
      
     SynchronizedRandomAccessList(List<E> list) {  
     	super(list);  
     }  
      
     SynchronizedRandomAccessList(List<E> list, Object mutex) {  
     	super(list, mutex);  
     }  
      
     public List<E> subList(int fromIndex, int toIndex) {  
     	synchronized (mutex) {  
     	return new SynchronizedRandomAccessList<>(list.subList(fromIndex, toIndex), mutex);  
     	}  
     }  
     private static final long serialVersionUID = 1530674583602358482L;  
      
     /**  
     * Allows instances to be deserialized in pre-1.4 JREs (which do * not have SynchronizedRandomAccessList).  SynchronizedList has * a readResolve method that inverts this transformation upon * deserialization. */ 
     private Object writeReplace() {  
     	return new SynchronizedList<>(list);  
     }  
    }
    ```

    `SynchronizedList` 继承自 `SynchronizedCollection`，一些共有的方法，`SynchronizedCollection` 都做了实现，基本做法都是使用 {% post_link Synchronized %} 关键字修饰代码块。
    ```Java
     //SynchronizedList继承自SynchronizedCollection
     static class SynchronizedList<E> extends SynchronizedCollection<E> implements List<E>{}
     //SynchronizedList类方法
     public E get(int index) {  
     	synchronized (mutex) {return list.get(index);}  
     }  
     public E set(int index, E element) {  
     	synchronized (mutex) {return list.set(index, element);}  
     }  
     public void add(int index, E element) {  
     	synchronized (mutex) {list.add(index, element);}  
     }  
     public E remove(int index) {  
     	synchronized (mutex) {return list.remove(index);}  
     }
    ```

    `SynchronizedList` 中获取 "ListIterator"[^11] 的方法不是同步的，在使用的时候需要自己添加同步代码块。
    ```Java
     //需要额外注意的是遍历遍历方法不是同步的
     public ListIterator<E> listIterator() {  
     	return list.listIterator(); // Must be manually synched by user  
     }  
     public ListIterator<E> listIterator(int index) {  
     	return list.listIterator(index); // Must be manually synched by user  
     }
    ```

    虽然 `SynchronizedList` 看起来是将同步范围缩小了，仅对 `mutex` 对象资源锁定，但是 `mutex` 默认是 `this`，这造就了相比 `Vector`，`SynchronizedList` 的效率并不会变高。
3. ## CopyOnWriteArrayList

　　在实际使用中，如果需要保证 `List` 的线程安全，推荐使用 {% post_link CopyOnWriteArrayList %}。

　　总的来讲，单线程场景下，推荐使用 `ArrayList` 保证更高的效率。
多线程场景下，如果是查询多余修改的场景，推荐使用 `CopyOnWriteArrayList`，如果是修改多余查询的场景，推荐使用 `Vector` 和 `SynchronizedList`。
