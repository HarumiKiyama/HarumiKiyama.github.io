+++
title = "Hash Table"
date = 2020-03-05
publishDate = 2020-02-29T00:00:00+08:00
tags = ["Algorithms"]
draft = false
creator = "Emacs 26.3 (Org mode N/A + ox-hugo)"
+++
哈希表是什么
<!--more-->

<div class="ox-hugo-toc toc">
<div></div>

<div class="heading">Table of Contents</div>

- [前言](#前言)
- [拉链法](#拉链法)
- [开放地址法](#开放地址法)

</div>
<!--endtoc-->


## 前言 {#前言}

哈希表(`hash table`)是一种非常常用的数据结构, 它是以 _key-value_ 的形式存储数据
的, 所谓的 _key-value_ 指的是, 任一 _key_ 都对应到内存中的某个位置. 可以认为哈希
表就是一种高级数组, 可以通过给定的 _key_ 计算出数组对应的下标. 由于这个特性, 哈
希表的插入速度是 \\(\Theta(1)\\), 查找速度也是 \\(\Theta(1)\\).

使用 _key_ 计算出对应下标的这个函数叫作哈希函数(`hash function`). 最简单的一个哈
希函数便是把 _key_ 转换成字符串, 然后返回这个字符串的长度.如下

```python
def hash(key):
    return len(str(key))
```

早期的 _php_ 在对核心函数生成哈希表时, 使用就是使用函数名长度作为哈希值的, 这个
也是 _php_ 经常被人诟病的一个点.为什么使用字符串长度作为哈希值会有问题呢? 具体原
因是这样的, 为了防止两个不同的 _key_ 生成同样的哈希值(这一现象被成为哈希碰撞
(`collision`)), 早期 _php_ 的核心函数中函数命名非常的不一致, 导致要记忆常用的一
些函数也变成了一种负担.[^fn:1]

由此, 一个新的问题出现了, 既然使用字符串长度作为哈希值会有哈希碰撞的问题, 那用其
他的方法就不会吗? 办法是有的, 这种办法的核心思路就是在足够大的空间中生成哈希值,
举个例子. 如果我们假定 _key_ 由小写的英文组成的字符串, 那么按照下述方法生成的哈
希值是一定不 会有碰撞的.

```python
def hash(key):
    length = len(key)
    rv = 0
    for i in range(length-1, -1, -1):
        rv += (ord(key[i])- ord('a')) * 26 ** (length-1-i)
    return rv
```

这个方法的实质是将一串小写英文字符串视作26进制的整数, 并将这个整数化为10进制. 以
比较简单的字符串 "abc" 为例, `hash("abc")` 返回的结果为 `28`. 如果基于这样的哈希
函数来实现我们的不会碰撞的哈希表的话, 将会是这样的.

```python
class HashTable:
    def __init__(self):
        self._l=[]

    def hash(self,key):
        length = len(key)
        rv = 0
        for i in range(length-1, -1, -1):
            rv += (ord(key[i])- ord('a')) * 26 ** (length-1-i)
        return rv

    def __setitem__(self, key, val):
        hash_v = self.hash(key)
        if len(self._l) <= hash_v:
            for _ in range(hash_v-len(self._l)+1):
                self._l.append(0)
        self._l[hash_v] = val

    def __getitem__(self, key):
        hash_v = self.hash(key)
        return self._l[hash_v]
```

**注意, 我们这里限制了key为小写英文字符构成的字符串**

事实上 **小写英文字符构成的字符串** 的这个限制是可以取消的, 把这个算法推广就是:

如果在内存无限的情况下, 我们可以根据 _key_ 得到一个整数, 然后我们可以说, 该整数
就是这个key所对应的下标, 那这样, 确实是可以实现一个不会碰撞的哈希表. 但是问题在
于在物理世界哈希表的空间是有限的, 于是哈希碰撞便是不可避免的. 现实当中的哈希表要
不就是在一开始就分配好内存的大小, 要不就是动态更新内存的大小. 并且在根据 _key_
得到哈希值之后, 会对这个哈希值根据哈希表的容量取余或者根据哈希表的容量减一进行与
操作. (一般是进行与操作因为 _bitwise operation_ 的会更加快)
比如在 _Python_ 当 中每个新创建的 _dict_ 会有8个 _slot_ (即是说储存哈希元素的列
表长度), 然后会把哈希值和 _dict_ 的长度减一做一个与操作得到下标, 在储存的元素大
于哈希表长度的 \\(\frac{2}{3}\\) 时会对哈希表进行一次扩容
[^fn:2]

为了解决哈希碰撞的问题, 目前大致有两种解决方案:

1.  拉链法(`open hashing`)
2.  开放地址法(`open addressing`)


## 拉链法 {#拉链法}

所谓的链地址法就是说, 既然哈希碰撞是一定会发生的, 那么在每个哈希值对应的下标处储
存的不是该哈希值对应的元素, 而是一个链表, 在链表中的每个储存了, _key_, _value_
的信息, 查询链表就是在哈希值对应的下标中, 遍历整个链表, 比对每个 _key_ 如果有和
查询的 _key_ 一致的节点就返回. 而插入操作也是类似的, 先通过查询得到该 _key_ 是否
已经存在的信息, 如果有就更新那个节点, 如果没有就在链表的最后插入一个新的节点. 代
码如下[^fn:3]

```c
const int SIZE = 1000000;
const int M = 999997;
struct HashTable {
  struct Node {
    int next, value, key;
  } data[SIZE];
  int head[M], size;
  int f(int key) { return key % M; }
  int get(int key) {
    for (int p = head[f(key)]; p; p = data[p].next)
      if (data[p].key == key) return data[p].value;
    return -1;
  }
  int modify(int key, int value) {
    for (int p = head[f(key)]; p; p = data[p].next)
      if (data[p].key == key) return data[p].value = value;
  }
  int add(int key, int value) {
    if (get(key) != -1) return -1;
    data[++size] = (Node){head[f(key)], value, key};
    head[f(key)] = size;
    return value;
  }
};
```


## 开放地址法 {#开放地址法}

开放地址法的实现会相对于拉链法要难, 其解决哈希碰撞的思路是, 先根据哈希值得到对应
的下标, 然后如果对应的下标没有元素就在下标处插入 _key_, _value_, 如果对应的下标
已经被占据了, 那么一般有两种策略来解决这个问题, 一种是线性查找(`linear
probing`), 即以该下标为起始点, 遍历哈希表, 查找是否有空位置, 另一种是平方探测法
(`quadratic probing`), 即以该下标为起点, 在第 \\(i\\) 次查找时, 每次增加 \\(i^2\\) 直到
找到为空位置.
关于线性查找和平方查找的优劣是学术界中一个经典的问题, 这里就不做展开了, 具体可以
看这两篇文章
[^fn:4]<sup>, </sup>[^fn:5].
但是比较奇怪的是 _Python_ 选择了这两种常见方法之外的方法, 随机查找(`random
probe`) 来解决哈希碰撞的问题. 主要的原因是因为 _Python_ 的哈希函数本身的随机性不
太好, 如 `map(hash, (0,1,2))` 的值为
`[0,1,2]`[^fn:6].

[^fn:1]: <https://zhuanlan.zhihu.com/p/27288770>
[^fn:2]: <https://stackoverflow.com/questions/327311/how-are-pythons-built-in-dictionaries-implemented>
[^fn:3]: <https://oi-wiki.org/ds/hash/>
[^fn:4]: <https://www.cnblogs.com/hongshijie/p/9432838.html>
[^fn:5]: <https://www.cnblogs.com/hongshijie/p/9432838.html>
[^fn:6]: <https://hg.python.org/cpython/file/52f68c95e025/Objects/dictobject.c#l33>
