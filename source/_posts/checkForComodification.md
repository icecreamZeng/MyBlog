---
title: checkForComodification
date: 2021-11-22 19:16:35
update: 2021-11-22 19:16:35
categories: [Professional,Java,Java集合]
tags: [Java,Java集合,迭代器,常见异常]
---

```Java
//执行这段代码是会抛出异常 ConcurrentModificationException
for (String str : list) {   
    if ("remove".equals(str)) {
        list.remove(str);
    }
}

//具体位置
final void checkForComodification() {  
 	if (modCount != expectedModCount)  
 		throw new ConcurrentModificationException();  
}
```

<!-- more -->　　

具体原因是，在调用 `list.remove()` 方法时方法内部会调用 `fastRemove()` 方法。会修改 `modCount`。
而 `foreach` 实际是通过 `Iterator` 实现的，遍历时会调用 `checkForComodification` 检查集合。

```Java
private void fastRemove(int index) {  
 modCount++;  
 int numMoved = size - index - 1;  
 if (numMoved > 0)  
 	System.arraycopy(elementData, index+1, elementData, index, numMoved);  
 elementData[--size] = null; // clear to let GC do its work  
}
```