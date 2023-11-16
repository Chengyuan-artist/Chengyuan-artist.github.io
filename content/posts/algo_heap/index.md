---
title: "二叉堆，二项堆和斐波那契堆"
subtitle: ""
date: 2023-11-15T20:43:01+08:00
lastmod: 2023-11-15T20:43:01+08:00
draft: true
author: ""
authorLink: ""
description: ""

tags: [Algorithm]
categories: [Algorithm]

hiddenFromHomePage: false
hiddenFromSearch: false

featuredImage: ""
featuredImagePreview: ""

toc:
  enable: true
math:
  enable: true
lightgallery: false
license: ""
---

<!--more-->

## 优先队列

### 动机

某些情况下，我们需要频繁地**动态**地取出一个集合S中的最小元素/最大元素。

这里**动态**指我们需要在集合S上执行插入/删除/更改键值操作。

问题在于如何组织数据结构使这些操作更高效。

### 操作定义

以最小优先队列为例：

| 方法名               | 描述                                                         |
| -------------------- | ------------------------------------------------------------ |
| MakeHeap()           | to create a new heap H                                       |
| Insert(H, x)         | to insert into H an element x together with its priority     |
| ExtractMin(H)        | to extract the element with the highest priority;            |
| DecreaseKey(H, x, k) | to decrease the priority of element x                        |
| Union(H1 , H2 )      | return a new heap containing all elements of heaps H1 and H2 , and destroy the input heaps |

### 应用举例

- Dijkstra单源最短路算法：需要ExtractMin
- Prim最小生成树算法：需要ExtractMin与DecreaseKey
- Why we need Union?

## 数组/链表实现

我们可以用数组或链表快速实现一个简单版本的优先队列。

由于本篇偏向理论分析，因此更在意数据结构的**逻辑结构**而非物理结构。对以下两种实现方式而言，可以认为数组的逻辑结构和链表相同（实际上二者复杂度分析也相同），因此都用链表代替。

### 无序链表实现

<img src="./images/image-20231115215858165.png" alt="image-20231115215858165" style="zoom: 50%;" />

每次都在首部插入新元素；提取最小值时，遍历链表找到最小值并更改链表指针将其从链表中删除。

（数组版本）每次在尾部插入新元素；提取最小值时，遍历数组找到最小值并将其与尾部元素交换位置，减小长度计数。

复杂度：

- 插入操作：\\(O(1)\\)
- 提取最小值：$O(n)$

### 有序链表实现

<img src="./images/image-20231115220444807.png" alt="image-20231115220444807" style="zoom:50%;" />

插入新元素时，遍历链表至第一个大于其的元素位置处，在此之前插入；提取最小值直接返回首部，并将其删除（更改头指针）。

（数组版本）倒序维护数组大小，插入新元素时，遍历链表至第一个小于其的元素位置处，在此之前插入，后续元素后移；提取最小值时直接返回尾部，并将其删除（减小长度计数）。

复杂度：

- 插入操作：$O(n)$
- 提取最小值：$O(1)$

### 问题

是否存在使得插入和提取最小值时间复杂度更均衡/更优的数据结构？

## 二叉堆

观察有序链表实现，插入操作实际上每次都在进行插入排序以维持整个链表的有序，然而这属于工作过头了，我们只需要维护一个最小值。

- 有序链表的每个指针指向的元素都大于等于自身，数据结构限制过于严格，因此需要**放松限制**。
- 但限制不能**过分放松**，否则就会像无序链表那样，元素间没有任何限制，导致提取操作复杂度上升。

二叉堆：

- 逻辑结构：一颗二叉树，且父结点小于子结点。所有结点除最后一层外按照完全二叉树排列。

<img src="images/image-20231115224930406.png" alt="image-20231115224930406" style="zoom: 67%;" />

- 物理结构：为了便于随机读取以及减少指针开销。结点以完全二叉树编号为下标放入数组中。

  <img src="images/image-20231115225848350.png" alt="image-20231115225848350" style="zoom: 67%;" />

  因此对结点$i$ ，左子结点下标为$2i$，右子结点下标为$2(i + 1)$，父结点下标为$ (i-1)/2$（向下取整）。下标越界时，表示结点不存在。

操作：

- 插入：将新元素置于二叉堆尾部，依次向上调整，使得最小堆性质（父结点小于子结点）保持。由于二叉堆高为$log_2n$（$n$为元素个数），因此时间复杂度为$O(logn)$。
- 提取最小值：将堆顶元素与堆尾元素交换，减小堆数组长度计数，从根结点开始依次向下调整，使得最小堆性质保持。时间复杂度为$O(logn)$。
- `DecreaseKey(x, k)`：将下标`x`处元素值降低为`k`。从该元素开始依次向上调整。时间复杂度为$O(logn)$。

