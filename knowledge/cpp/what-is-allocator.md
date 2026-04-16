# Allocator 是什么

在 C++ 里，**allocator**是一个用来管理内存分配与释放的抽象层。最经典的使用场景是 **STL 容器**，这些容器并不是直接写死用 `new` / `delete` 来管理元素内存，而是通过 allocator 来完成底层内存操作。

> **“容器如何申请、释放、构造、销毁对象的一套策略接口。”**

它的核心意义有两个：

1. **把“内存管理策略”从“数据结构逻辑”中分离出来**
2. **允许用户自定义内存分配方式**，比如：
   - 内存池
   - 栈上 arena
   - 共享内存
   - NUMA-aware 分配
   - 对齐分配
   - 调试型分配器（统计、跟踪、检测泄漏）

---

# 为什么需要 Allocator

如果没有 allocator，容器内部可能直接这么做：

```cpp
T* p = new T[n];
delete[] p;
```

这样做的问题很多：

- 容器和内存管理策略强耦合
- 无法灵活替换内存来源
- 不方便性能优化
- 无法统一做内存统计/调试
- 难以支持特殊场景（共享内存、固定地址池、无锁池等）

Allocator 的出现，使 STL 容器具备了更强的可扩展性。

---

# Allocator 解决的到底是什么问题

## 原始内存管理

- 申请一块**未构造对象**的原始内存
- 释放这块原始内存

> ⚠️ 这一步不是“创建对象”，只是拿到足够大的字节区域。

```cpp
#include <memory>
#include <iostream>

int main() {
    std::allocator<int> alloc;

    int* p = alloc.allocate(3);   // 只分配内存，不初始化

    std::construct_at(p, 10);     // 在特定位置初始化
    std::construct_at(p + 1, 20);
    std::construct_at(p + 2, 30);

    for (int i = 0; i < 3; ++i) { // 在特定位置销毁对象
        std::destroy_at(p + i);
    }

    alloc.deallocate(p, 3);      // 回收内存空间
}
```

这个过程分成四步：

1. `allocate(3)`：拿原始内存
2. `construct_at(...)`：逐个构造对象
3. `destroy_at(...)`：逐个析构对象
4. `deallocate(...)`：释放原始内存

## 对象生命周期管理

以前 allocator 接口里还会负责：

- `construct` &rarr; 在已分配的原始内存上构造对象
- `destroy` &rarr; 显式销毁对象

现代 C++ 中通常通过 `std::allocator_traits` + placement new 来完成这部分。

---

# `std::allocator`

```cpp
template<class T>
struct allocator {
    using value_type = T;

    T* allocate(std::size_t n);
    void deallocate(T* p, std::size_t n);
};
```

> 容器不用 `new[]` / `delete[]`的一个很重要的原因是：**容器经常需要“分配一块空间，但只构造其中一部分元素”**。而`new[N]`会直接分配N个对象大小的空间并构造N个对象。


> [!NOTE]
> `new`包含空间分配和对象构造；`::operator new`只包含空间分配。通常来说可以认为`::operator new`是`allocate()`的底层实现。
---

# `std::allocator_traits`

标准库提供的一层统一适配器，用来屏蔽不同 allocator 的细节。你可以把它理解成：“容器不要直接假设 allocator 长什么样，而是统一通过 allocator_traits 去调用”。

```cpp
std::allocator_traits<Alloc>
```

它的作用主要是：
- 给所有 allocator 提供统一接口
- 如果 allocator 自己有特殊 construct/destroy，就用它
- 如果没有，就退回标准默认行为（本质上就是 placement new / 显式析构）

## 常见接口

```cpp
using traits = std::allocator_traits<Alloc>;

T* p = traits::allocate(alloc, n);
traits::deallocate(alloc, p, n);

traits::construct(alloc, p, args...);
traits::destroy(alloc, p);
```

容器实现时不需要假设 allocator 一定有完整接口。只要 allocator 满足最基本要求，traits 就能尽量补齐。详情请见[自定义allocator基本要求](#自定义allocator基本要求)

## allocator 的模板拷贝构造

```cpp
template<typename U>
MyAllocator(const MyAllocator<U>&) {}
```

这是为了支持 allocator 的**rebind**。

因为容器内部不一定只给 `T` 分配内存。例如某些容器节点结构可能不是 `T` 本身，而是内部节点类型。因此 allocator 需要能从 `Allocator<T>` 转成 `Allocator<U>`。

> 比如我有链表`LinkedListNode<int>`，那么T是int，但是初始化的是Node节点，其中他的value是int类型。

---

# `rebind`

因为容器接受到的类型是 T，但是内部不一定给 T 分配内存。
> 比如我有链表`LinkedListNode<int>`，那么T是int，但是初始化的是Node节点，其中他的value是int类型。

在旧式 allocator 模型里，经常写：

```cpp
template<typename U>
struct rebind {
    using other = MyAllocator<U>;
};
```

意思是“把当前 allocator 重新绑定到另一种类型 `U` 上。”

例如：

- 当前 allocator 是 `MyAllocator<int>`
- 容器内部要给 `Node` 分配内存
- 就通过 `rebind` 拿到 `MyAllocator<Node>`

现代 C++ 中通常通过 `allocator_traits<Alloc>::rebind_alloc<U>` 来做：

```cpp
using NodeAlloc = std::allocator_traits<Alloc>::template rebind_alloc<Node>;
```

> [!NOTE]
> 此处使用`::template`是因为`rebind_alloc`是一个成员模版函数，且前面的部分并非一个已知类型，而是使用的模版类型。
> ```cpp
> using A = std::allocator_traits<int>;
> using B = std::allocator_traits<A>::rebind_alloc<Node>;
>
> template<typename Alloc>
> struct X {
>   using NodeAlloc = std::allocator_traits<Alloc>::template rebind_alloc<Node>;
> };
> ```

---

# 状态型 allocator vs 无状态 allocator

## 无状态 allocator（stateless allocator）

allocator 自己不保存任何额外信息，比如只是简单调用`::operator new`等全局的函数，那么每个allocator的实例都是一样的。相当于**只是定义了规则**。

```cpp
template<typename T>
struct MyAlloc {
    using value_type = T;

    T* allocate(std::size_t n) {
        return static_cast<T*>(::operator new(n * sizeof(T)));
    }

    void deallocate(T* p, std::size_t) {
        ::operator delete(p);
    }
};

// a，b是一样的，没有成员变量，没有特殊的配置，都是直接操作内存
MyAlloc<int> a;
MyAlloc<int> b;
```

## 状态型 allocator（stateful allocator）

allocator 对象内部带着一些数据，这些数据会影响它怎么分配内存，例如：
- 持有内存池指针
- 持有 arena 指针
- 持有共享内存句柄
- 持有 NUMA 节点信息
- 持有统计上下文

相当于**不只是规则，还有上下文**。

```cpp
struct MemoryPool {
    void* allocate(std::size_t bytes);
    void deallocate(void* p);
};

template<typename T>
struct PoolAlloc {
    using value_type = T;
    MemoryPool* pool;

    PoolAlloc(MemoryPool* p) : pool(p) {}

    T* allocate(std::size_t n) {
        return static_cast<T*>(pool->allocate(n * sizeof(T)));
    }

    void deallocate(T* p, std::size_t) {
        pool->deallocate(p);
    }
};

// a，b不一样，因为一个基于pool1，一个基于pool2
PoolAlloc<int> a(&pool1);
PoolAlloc<int> b(&pool2);
```

---

# allocator propagation（传播规则）

当一个容器发生 拷贝赋值/移动赋值/swap 时，allocator 要不要跟着一起传播？

标准通过一些 traits 来控制：

- `propagate_on_container_copy_assignment` &rarr; `a = b`
- `propagate_on_container_move_assignment` &rarr; `a = move(b)`
- `propagate_on_container_swap` &rarr; `swap(a, b)`

它们通常是 `std::true_type` 或 `std::false_type`。
- 如果 propagation 是 true，那么在对应操作里，a 的 allocator 可以被替换成 b 的 allocator，因此 a 就能用和 b 一样的内存管理上下文去管理那块内存
- 如果 propagation 是 false，那么 a 保留自己的 allocator，这时它不一定能直接接管 b 的内存

比如两个 vector 如果执行：

```cpp
a = std::move(b);
```

- `a` 能不能应该接管 `b` 的内存
- 如果 allocator 无状态那么应该可以
- 如果 allocator 有状态，且`pool1 != pool2`，那么`a.allocator`理论上不知道如何管理pool2

## `is_always_equal`
任意两个该 allocator 实例都可视为等价，可以互相释放对方分配的内存。这通常适用于无状态 allocator。

```cpp
struct MyAlloc {
    using is_always_equal = std::true_type;
};
```

如果 allocator 总是等价，容器在 move / swap 等场景可以做更多优化，因为它不需要担心“这块内存只能被某个特定 allocator 回收”。

---

# Polymorphic Memory Resource (PMR)

C++ 17 新特性，详情参考[PMR](./what-is-PMR.md)。

---

# Allocator 常见使用场景

**性能优化**

- 减少系统调用
- 降低 malloc/free 开销
- 提升 cache locality
- 减少碎片
- 提升批量分配效率

**资源隔离**

不同模块用不同 allocator：

- 便于统计
- 便于限额
- 便于追踪泄漏
- 便于故障定位

**特殊内存来源**

- 共享内存
- huge page
- NUMA 节点本地内存
- GPU / pinned memory（通常需要更专门接口）
- 固定映射区域

**调试与可观测性**

- 统计总分配次数
- 统计峰值内存
- 标记调用来源
- 做越界检测
- 检测双重释放

> [!NOTE]
> 粗略统计容器占用的动态内存。
>
> ```cpp
> #include <atomic>
> #include <iostream>
> #include <memory>
> #include <vector>
> 
> struct AllocStats {
>     std::atomic<std::size_t> bytes{0};
> };
> 
> template<typename T>
> struct CountingAllocator {
>     using value_type = T;
> 
>     AllocStats* stats = nullptr;
> 
>     CountingAllocator() = default;
>     explicit CountingAllocator(AllocStats* s) : stats(s) {}
> 
>     template<typename U>
>     CountingAllocator(const CountingAllocator<U>& other) : stats(other.stats) {}
> 
>     T* allocate(std::size_t n) {
>         std::size_t b = n * sizeof(T);
>         if (stats) stats->bytes += b;
>         return static_cast<T*>(::operator new(b));
>     }
> 
>     void deallocate(T* p, std::size_t n) {
>         std::size_t b = n * sizeof(T);
>         if (stats) stats->bytes -= b;
>         ::operator delete(p);
>     }
> };
> ```

---

# 自定义allocator基本要求

最小要求通常包括：

```cpp
template<typename T>
struct MyAlloc {
    using value_type = T;

    MyAlloc() = default;

    template<typename U>
    MyAlloc(const MyAlloc<U>&);

    T* allocate(std::size_t n);
    void deallocate(T* p, std::size_t n);
};
```

通常还建议提供：

- 比较运算符 `==` / `!=`
- `is_always_equal`
- propagation traits
- noexcept 保证（能写则写）
- 与 `allocator_traits` 兼容的行为

---

# Allocator 异常

考虑 vector 扩容：

1. allocate 新内存
2. 逐个 move/copy 构造新元素
3. 如果中途抛异常：
   - 销毁已经构造的新元素
   - deallocate 新内存
   - 保留旧内存和旧元素不动

这说明 allocator 必须和对象构造/销毁逻辑严密配合，它往往和RAII一起出现。
