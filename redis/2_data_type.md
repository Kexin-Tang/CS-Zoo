# String

Redis的string是***SDS(Simple Dynamic String)***, 它通过存储字符串长度来标明结束位置, 而不是通过判断`\0`等方式.

```
buffer => ['R', 'e', 'd', 'i', 's', '\0', '', '']
len => 5
free => 3
```
`len`标记内容长度, `free`标记可使用的剩余空间, `buffer`是预先分配的内存空间.

## 特点
1. O(1)获得长度信息
2. 避免溢出 &rarr; 1MB以下double扩容, 1mb以上每次扩容1MB
3. 二进制安全(Binary Safe) &rarr; 以数组长度而不是`\0`决定终止位置, 可以存储更复杂的内容
4. 惰性空间回收(Lazy Space Reclamation) &rarr; string变短时不会马上回收已分配空间, 而是跟踪free的区域以便于后续再次扩容

## 高级知识

字符串可以保存三种类型: int, embstr和raw.

如果是int, 那么会直接把该值放到redisObject的`ptr`属性中.
![encoding=int](./pic/2_1.png)

如果是较短的字符串, 那么redisObject和SDS将会通过一次内存分配函数来分配一块连续的内存空间.
![encoding=embstr](./pic/2_2.png)

如果是较长的字符串, 那么redisObject和SDS将会通过两次内存分配.
![encoding=raw](./pic/2_3.png)

其优缺点如下:
1. embstr可以减少分配和回收的次数, 同时因为是连续内存空间所以效率更高.
2. 因为其连续内存空间, 所以实际上可以认为是**只读**字符串, 当需要扩容时需要变成raw再操作.

## 应用场景

### 计数器
因为单线程原子操作, 所以可以直接使用int类型来保存数据.

### 分布式锁
可以实现只有key不存在的时候才设置value, 所以相当于:
* key存在说明不能插入, 因为锁被hold
* key不存在说明可以插入, 因为锁没有被hold

### 共享session信息
用户登陆的时候可能会访问不同的服务器, 如果把session信息放在服务器, 就会导致可能丢失session信息. 如果将session信息放在redis, 无论哪个服务器都将访问redis获取session信息.

---

# Hash

用于存储Object, 本身其value就是一个新的key-value pair.



---

# List

---

# Set

---

# Sorted Set (ZSet)

---

# Bitmaps

---

# HyperLogLogs

---

# Stream