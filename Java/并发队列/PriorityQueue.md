

[TOC]
PriorityQueue
------------
## 1.介绍

PriorityQueue是一种基于Binary Heap（Binary Heap是一种完全二叉树）的无界优先队列，底层使用数组实现。它是用来存放实现了Comparable接口的一些元素。该队列不是线程安全的（线程安全的优先队列参考PriorityBlockingQueue），它不允许存放null和未实现comparable接口的元素。  
队列头默认存放是按照特定顺序存放的元素最小值（小顶堆）。如果存在多个相同最小值，则出队的元素为其中任意一个。也可以自行传入Comparator来实现大顶堆。

## 2.数据结构

PriorityQueue底层是使用数组实现的数据的存储，逻辑结构为一个完全二叉树。默认情况下为小顶堆，子节点总是大于父节点。存储结构和逻辑结构如下图所示：
![20180808230556959](E:\学习文档\学习笔记\image\20180808230556959.png)

## 3.构造函数

PriorityQueue提供了以下构造函数：

```java
public PriorityQueue() {
        this(DEFAULT_INITIAL_CAPACITY, null);
}
public PriorityQueue(int initialCapacity) {
        this(initialCapacity, null);
}
public PriorityQueue(Comparator<? super E> comparator) {
        this(DEFAULT_INITIAL_CAPACITY, comparator);
}
public PriorityQueue(int initialCapacity,Comparator<? super E> comparator) {
        if (initialCapacity < 1)
            throw new IllegalArgumentException();
        this.queue = new Object[initialCapacity];
        this.comparator = comparator;
}
public PriorityQueue(Collection<? extends E> c) {
        if (c instanceof SortedSet<?>) {
            SortedSet<? extends E> ss = (SortedSet<? extends E>) c;
            this.comparator = (Comparator<? super E>) ss.comparator();
            initElementsFromCollection(ss);
        }
        else if (c instanceof PriorityQueue<?>) {
            PriorityQueue<? extends E> pq = (PriorityQueue<? extends E>) c;
            this.comparator = (Comparator<? super E>) pq.comparator();
            initFromPriorityQueue(pq);
        }
        else {
            this.comparator = null;
            initFromCollection(c);
        }
}
```
默认创建的队列大小为11，源码中定义的初始大小为：
```java
//默认初始容量
private static final int DEFAULT_INITIAL_CAPACITY = 11;
```
创建队列可以自定义初始大小，comparator或者集合。对于集合的处理，其实就是将集合转换成数组来初始化Object[] queue，具体源码如下：
```java
private void initFromCollection(Collection<? extends E> c) {
    initElementsFromCollection(c);
    heapify();
}

private void initElementsFromCollection(Collection<? extends E> c) {
    Object[] a = c.toArray();
    // 这里集合c如果不能正确转换为Object[]，则作拷贝。
    if (a.getClass() != Object[].class)
        a = Arrays.copyOf(a, a.length, Object[].class);
    int len = a.length;
    //因为队列中的元素不能有null元素，所以这里做一个判断。
    if (len == 1 || this.comparator != null)
        for (int i = 0; i < len; i++)
            if (a[i] == null)
                throw new NullPointerException();
    this.queue = a;
    this.size = a.length;
}
```
集合处理完毕后，还会调一个`heapify()`，按照逻辑结构进行结构调整，这个后面在添加和删除时再细说：
```java
private void heapify() {
        for (int i = (size >>> 1) - 1; i >= 0; i--)
            siftDown(i, (E) queue[i]);
}
```
## 4.扩容机制

PriorityQueue采用自动扩容机制，当size>=queue.lenth的时候，会调用`grow(int minCapacity)`进行数组扩容，minCapacity为满足要求的最小容量。  
这里需要注意的是，PriorityQueue的最大容量为：

```java
private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;
```
这是因为不同的虚拟机实现细节不同，有些虚拟机在数组中保留了一些头信息，这么做为了避免发生OOM。  
扩容的具体代码如下：
```java
private void grow(int minCapacity) {
    int oldCapacity = queue.length;
    //如果原始容量小于64的时候，直接扩容为原来的2倍。否则容量变为原来的1.5倍。
    int newCapacity = oldCapacity + ((oldCapacity < 64) ?(oldCapacity + 2) :(oldCapacity >> 1));
    // 扩容后需要对是否达到最大容量进行判断，避免发生OOM。
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    queue = Arrays.copyOf(queue, newCapacity);
}
```
如果扩容后的容量大于了PriorityQueue定义的最大值，则调用`hugeCapacity(minCapacity)` 进行调整：
```java
private static int hugeCapacity(int minCapacity) {
	//如果需求容量都已经超过了int的最大值（溢出值为负数），直接抛出OOM。
    if (minCapacity < 0)
        throw new OutOfMemoryError();
    /*大于了PriorityQueue定义的最大值则取int的最大值，
    否则取PriorityQueue定义的最大值（MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8 ）
    */
    return (minCapacity > MAX_ARRAY_SIZE) ? Integer.MAX_VALUE : MAX_ARRAY_SIZE;
}
```
## 5.添加/取出/删除元素

### 1. 添加元素
添加元素主要有以下两种方法，实质上就是走的`offer(E e)`:
```java
public boolean add(E e) {
    return offer(e);
}

public boolean offer(E e) {
	//被插入的元素不能为null
    if (e == null)
        throw new NullPointerException();
    modCount++;
    int i = size;
    //如果size大于等于了数组的长度，则进行扩容操作
    if (i >= queue.length)
        grow(i + 1);
    size = i + 1;
    if (i == 0)
	    //如果size=0的话直接放入堆顶
        queue[0] = e;
    else
	    //当前元素与上层元素进行比较和调整（siftdown则为当前元素与下层元素进行比较和调整，一般在删除时调用）
        siftUp(i, e);
    return true;
}
```
我们来看看siftup里面做了什么操作：
```java
private void siftUp(int k, E x) {
    if (comparator != null)
        siftUpUsingComparator(k, x);
    else
        siftUpComparable(k, x);
}
```
我们先看看没有comparator的时候，继续跟进`siftUpComparable(k, x)`：
```java
private void siftUpComparable(int k, E x) {
	//这就是前面说的要求所插入的元素必须为实现了Comparable接口的元素，这里要进行比较和调整。
    Comparable<? super E> key = (Comparable<? super E>) x;
    while (k > 0) {
	    //获取父节点索引
        int parent = (k - 1) >>> 1;
        //取得父节点元素
        Object e = queue[parent];
        //判断当前节点与父节点比较结果，默认为小顶堆，所以当当前节点大于等于父节点则跳出循环
        if (key.compareTo((E) e) >= 0)
            break;
        //将父节点作为子节点
        queue[k] = e;
        //将原父节点的位置赋值给k
        k = parent;
    }
    queue[k] = key;
}
```
这里使用到了完全二叉树的特点，假如当前节点位置为i，则它的父节点的位置为i/2取整数部分，它的子节点位置为i*2（左）和i*2+1（右）。这里的循环操作主要是：
比较需要插入的元素A和父元素B，如果A大于等于B，则将A放入B的子节点中；如果A小于B，则将B放入B的子节点中，A暂时作为B父节点（抽象的认为相当于两个交换位置，只是B确定交换，A还要继续与上层比较），然后继续将A与B的父节点C进行比较，重复上述比较操作，直到A大于等于某个节点或者A到堆顶循环停止。  
**siftUpUsingComparator**：只是将`if (key.compareTo((E) e) >= 0)`变为了`if (comparator.compare(x, (E) e) >= 0)`，就是使用你传入的comparator来比较元素，其余操作都一样。

### 2. 取出元素

**peek**：比较简单，就是返回堆顶元素，不进行元素的移除，所以不涉及到逻辑结构的调整：

```java
public E peek() {
    return (size == 0) ? null : (E) queue[0];
}
```
**poll**：返回并移除堆顶元素，所以还要进行逻辑结构的调整，代码如下：
```java
public E poll() {
	//数组为空直接返回null
    if (size == 0)
        return null;
    //获取最后一个元素的索引
    int s = --size;
    modCount++;
    //取出堆顶元素
    E result = (E) queue[0];
    //取出最后一个元素
    E x = (E) queue[s];
    //将最后一个元素的位置变为null
    queue[s] = null;
    if (s != 0)
	    //进行逻辑结构的调整
        siftDown(0, x);
    return result;
}
```
我们来看看移除堆顶元素后，逻辑结构调整的具体实现：
```java
private void siftDown(int k, E x) {
    if (comparator != null)
        siftDownUsingComparator(k, x);
    else
        siftDownComparable(k, x);
}

private void siftDownComparable(int k, E x) {
    Comparable<? super E> key = (Comparable<? super E>)x;
    //这个half保证循环到最下面的非叶子节点
    int half = size >>> 1;
    while (k < half) {
	    //这里假如左子节点是最小值
        int child = (k << 1) + 1; 
        Object c = queue[child];
        int right = child + 1;
        //比较左右节点大小，将当前节点为左右节点中的最小值元素来与传入的元素做比较
        if (right < size &&
            ((Comparable<? super E>) c).compareTo((E) queue[right]) > 0)
            c = queue[child = right];
        if (key.compareTo((E) c) <= 0)
            break;
        queue[k] = c;
        k = child;
    }
    queue[k] = key;
}
```
由于移除堆顶元素后，堆顶暂时为null，所以需要将逻辑结构从堆顶往下调整，所以这里传入的k为0。由于子节点都需要小于父节点，所以代码里面需要比较左右子节点，取出最小值来与传入的元素作比较。这里的循环主要操作为：
假设传入的元素A，堆顶的两个子节点为B，C。现在就是比较A>MIN(B,C)，假如成立，将MIN(B,C)放入堆顶位置，抽象的认为将A放入MIN(B,C)的位置（实际是暂时未放入），然后继续与其子节点进行比较，重复上述操作。直到A小于等于其子节点或者到叶子节点停止循环。
**siftDownUsingComparator(k, x)**：就是使用你传入的comparator来比较元素，其余操作都一样。

### 3. 删除元素

删除指定元素的操作为：

```java
public boolean remove(Object o) {
    int i = indexOf(o);
    if (i == -1)
        return false;
    else {
        removeAt(i);
        return true;
    }
}
```
其中indexOf(o)就是确定该元素在数组中的位置：
```java
private int indexOf(Object o) {
    if (o != null) {
        for (int i = 0; i < size; i++)
	        //这里可能要根据元素的特性决定是否重写equals方法，否则可能得到的结果有偏差
            if (o.equals(queue[i]))
                return i;
    }
    return -1;
}
```
removeAt(i)里面的实现：
```java
private E removeAt(int i) {
    // assert i >= 0 && i < size;
    modCount++;
    int s = --size;
    //如果移除的是最后一个元素，则直接将最后一个元素置为null，不需要调整结构
    if (s == i) 
        queue[i] = null;
    //如果移除的不是最后一个元素，则需要进行逻辑结构调整
    else {
	    //取出最后一个元素，并将其位置上的元素置为null
        E moved = (E) queue[s];
        queue[s] = null;
        //先从被删除元素的位置开始往下调整逻辑结构，i为被删除元素所在位置，moved为数组中的最后一个元素
        siftDown(i, moved);
        //如果下半部分结构未作调整，只是将数组最后一个元素放入了被删除元素的位置，那么再继续往上调整。
        if (queue[i] == moved) {
            siftUp(i, moved);
            if (queue[i] != moved)
                return moved;
        }
    }
    return null;
}
```
该方法的操作和添加/删除差不多，代码注释也大概讲了这个方法的细节，剩下的`siftDown`和`siftUp`这里就不赘述了。
这个方法的返回值主要是用在`iterator`（还未研究），返回值为删除元素前数组中的最后一个元素。

## 6.如何实现大顶堆

这里直接实现自己的Comparator，这里直接给出Java代码，请读者自行尝试：

```java
package queue;

import java.util.Comparator;
import java.util.PriorityQueue;

public class PriorityQueueTest {

    public static void main(String[] args) {
        Comparator<Integer> comparator = (o1,o2) -> o2-o1;
        PriorityQueue<Integer> queue = new PriorityQueue<>(comparator);
        queue.offer(1);
        queue.offer(2);
        queue.offer(3);
        queue.offer(4);
        queue.offer(5);
        queue.offer(6);
        System.out.println(queue);
    }
}
```
输出的结果为：[6, 4, 5, 1, 3, 2]

>*文章写的不好，如果有错误和纰漏的地方还请指正。谢谢支持。*

