---
title: LRU 页面替换算法的实现
date: 2021-08-24
tag: 算法
categories: 刷题
top: false
mathjax: false
---

## LRU 页面替换算法实现

### 0. LRU 简介

**LRU** 算法是一种常见的页面替换算法，计算机的存储是一种金字塔体系（hirarchy）越位于顶端，访问速度越快，但容量越小价格越贵，比如位于最顶端的就是寄存器（register），现如今主流设备的寄存器的容量不过 8 个字节（64 bits）而在底层的硬盘、磁带等可以轻松做到十多个 TB 的容量。高速缓存少但是处理的任务不会少，为了最大限度的提升 CPU 和 外部交换数据的速度，所以在内存不够的时候需要淘汰适当的内容最大化的提升效率。LRU 算法就是最接近最优解的方法。

下面通过一个容量为 3 个页大小的内存来演示 LRU 的工作原理，设访问序列是 [7, 0, 1, 2, 0, 3, 0, 4]。内存将会像下图一样变化。

![](https://img-blog.csdnimg.cn/20191109174241708.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JlbG9uZ3RvY29kZQ==,size_16,color_FFFFFF,t_70)

我们可以看到 LRU 工作时，内存部分像一个特殊的栈。特殊之处在于 1) 当加入页的 key 指向的内存已存在内容时要更新内容并将其置于最新的位置；2) 当访问已存在内存时也许更新该内容到最新的位置。自然的，我们想到利用添加其他方法的栈去实现 LRU。不过这是有问题的。问题是如果只用一个栈去存储的话，那么存和取每次都需要按照时间排序，会有大量内存拷贝操作，性能肯定是不能接受的，因为 LRU 的业务场景就是管理内存与 CPU 进行大量高速交换的，自然十分要求时间效率。所以我们需要用其他的数据结构实现相关功能。其中一种较为简单的实现方法是利用哈希表和双向链表实现。

### 1. 哈希表加双向链表原理介绍

通过哈希表可以做到以 O(1) 复杂度查找 key 所对应的 value。而通过一个双向链表，可以以 O(1) 复杂度操作头尾以及相邻的元素，相关原理的原理图如下所示

![](https://img-blog.csdnimg.cn/20191109175403434.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JlbG9uZ3RvY29kZQ==,size_16,color_FFFFFF,t_70)

哈希表可以认为是一个可以快速查找的指针索引，双向链表的头尾代表了这个状态下历史先后顺序。下面通过一张图反映刚才的访问序列下双向链表的变化

![](https://img-blog.csdnimg.cn/20191109175418934.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JlbG9uZ3RvY29kZQ==,size_16,color_FFFFFF,t_70)

### 2. 代码实现

* 结点定义，需要定义键 key 和值 value，以及双向链表必备的前驱指针 pre 和后继指针 next。

```java
package LRU;

public class DLinkedNode {
    String key;
    int value;
    DLinkedNode pre;
    DLinkedNode next;
}
```

* LRU 缓存类编写，成员变量有哈希表（注意线程安全问题），双向链表的头尾结点，计数变量和容量

```java
		// cache 是一个线程安全的哈希表，用来从 key 快速找到缓存。
    private Hashtable<String, DLinkedNode> cache = new Hashtable<>();
    // count 是当前缓存占用数，capacity 是缓存容量。
    private int count;
    private int capacity;
    // 定义头结点和尾结点去建立双向链表
    private DLinkedNode head, tail;
```

* LRU 缓存类的初始化方法，负责完成定义容量和建立一个空的双向链表

```java
// 初始化成员变量，建立指定容量和双向链表
    public LRUCache(int capacity) {
        this.count = 0;
        this.capacity = capacity;
        this.head = new DLinkedNode();
        this.head.pre = null;
        this.tail = new DLinkedNode();
        this.tail.next = null;
        this.head.next = this.tail;
        this.tail.pre = this.head;
    }
```

* LRU 缓存类的私有方法，实际上是双向链表的一系列方法和 LRU 缓存类的一个特殊方法，将访问或更新过的结点放到双向链表头部

```java
		// 在头部添加结点
		private void addNode(DLinkedNode node) {
        node.pre = head;
        node.next = head.next;
        head.next.pre = node;
        head.next = node;
    }

		// 移除指定结点
    private void removeNode(DLinkedNode node) {
        DLinkedNode pre = node.pre;
        DLinkedNode next = node.next;
        pre.next = next;
        next.pre = pre;
    }

		// 将访问或更新过的结点放到双向链表头部
    private void moveToHead(DLinkedNode node) {
        this.removeNode(node);
        this.addNode(node);
    }
		
		// 得到尾部的结点
    private DLinkedNode popTail() {
        DLinkedNode res = tail.pre;
        this.removeNode(res);
        return res;
    }
```

* LRU 缓存类的核心方法 get，负责从键 key 访问内存内容。

```java
		// get 方法去访问内存内容
    public int get(String key) {
        DLinkedNode node = cache.get(key);
        if (node == null) { // 如果未找到，返回错误值 -1
            return -1;
        }
        this.moveToHead(node);  // 将该结点移动到头部（头部到尾部是按照从新到旧保存数据）
        return node.value;
    }
```

* LRU 缓存类核心方法 set，负责更新键 key 对应的内容或者新加键 key 和内容 value

```java
// set 方法将内容放入内存。
    public void set(String key, int value) {
        DLinkedNode node = cache.get(key);  // 探测 key 指针处是否有内容
        if (node == null) { // 若 key 指向地址为空
            DLinkedNode newNode = new DLinkedNode();
            newNode.key = key;
            newNode.value = value;
            this.cache.put(key, newNode);
            this.addNode(newNode);
            ++count;
            if (count > capacity) {
                DLinkedNode tail = this.popTail();
                this.cache.remove(tail.key);
                --count;
            }
        } else {
            node.value = value;
            this.moveToHead(node);
        }
    }
```

### 3. 一点疑问

这个代码总的来说符合思维的直觉，不过有一点不是特别清楚，我认为双向链表的结点没有必要存键 key，只需要存内容 value 就可以了。但是市面上的讲解视频什么的都存了键 key，不知道为什么。

### 4. Redis 中 LRU 算法的实现原理

如果按照HashMap和双向链表实现，需要额外的存储存放 next 和 prev 指针，牺牲比较大的存储空间，显然是不划算的。所以Redis采用了一个近似的做法，就是随机取出若干个key，然后按照访问时间排序后，淘汰掉最不经常使用的，具体分析如下：

为了支持LRU，Redis 2.8.19中使用了一个全局的LRU时钟，server.lruclock，定义如下，

```java
#define REDIS_LRU_BITS 24
unsigned lruclock:REDIS_LRU_BITS; /* Clock for LRU eviction */
```


默认的LRU时钟的分辨率是1秒，可以通过改变REDIS_LRU_CLOCK_RESOLUTION宏的值来改变，Redis会在serverCron()中调用updateLRUClock定期的更新LRU时钟，更新的频率和hz参数有关，默认为100ms一次，如下，

```java
#define REDIS_LRU_CLOCK_MAX ((1<<REDIS_LRU_BITS)-1) /* Max value of obj->lru */
#define REDIS_LRU_CLOCK_RESOLUTION 1 /* LRU clock resolution in seconds */

void updateLRUClock(void) {
    server.lruclock = (server.unixtime / REDIS_LRU_CLOCK_RESOLUTION) &
                                                REDIS_LRU_CLOCK_MAX;
}
```

server.unixtime是系统当前的unix时间戳，当 lruclock 的值超出REDIS_LRU_CLOCK_MAX时，会从头开始计算，所以在计算一个key的最长没有访问时间时，可能key本身保存的lru访问时间会比当前的lrulock还要大，这个时候需要计算额外时间......

相关更多内容可以在 CSDN 上查到。
