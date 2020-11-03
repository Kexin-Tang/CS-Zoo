
数据结构 | 查找复杂度 | 插入/删除复杂度
:---: | :---: | :---:
数组 | O(n) | O(n)
链表 | O(n) | O(1)
跳表 | O(log n) | O(log n)

### 跳表(skip list)

<a href="https://sm.ms/image/yE1lmepBY3O4z5J" target="_blank"><img src="https://i.loli.net/2020/11/03/yE1lmepBY3O4z5J.jpg" ></a>

* 元素必须有序 *(因为有序，相当于多了一些信息，那么对于一维的链表，想要改进即可以升维，而跳表正是如此)*

* 用于取代平衡树(AVL Tree)和二分查找

* 第k级nodes个数为n/2<sup>k</sup>.设置索引最多有h级，最高索引有两个Node，那么n/2<sup>h</sup>=2，即h=log(n)-1
