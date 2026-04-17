# C++ PMR 详解（Polymorphic Memory Resource）

## 1. 什么是 PMR

PMR 是 **Polymorphic Memory Resource**，是 C++17 引入的一套内存资源体系，位于：

```cpp
#include <memory_resource>
```

它的核心目标是：

> **把“内存从哪里分配”这件事，从容器类型里抽出来，改成在运行时决定。**

你可以把它理解成：

- 传统 allocator：**编译期**决定内存分配策略
- PMR：**运行时**决定内存分配策略

这让 C++ 的容器在很多工程场景里更灵活，尤其适合：

- arena / region 分配
- 对象池
- 请求级临时内存
- 高频小对象分配
- 内存统计与隔离
- 自定义分配策略注入

---

## 2. 为什么会有 PMR

在传统 STL allocator 模型里，容器类型会把 allocator 写进模板参数：

```cpp
std::vector<int, MyAllocator<int>>
```

这会带来几个问题：

### 2.1 容器类型会变化

```cpp
std::vector<int, A1>
std::vector<int, A2>
```

这是两个不同类型。

这意味着：

- 接口传递变复杂
- 不同 allocator 的容器很难统一处理
- 工程里切换内存策略不够灵活

### 2.2 allocator 策略被绑定在类型系统里

很多时候我们只是想：

- 这次请求用 arena1
- 下次请求用 arena2
- 某模块用 poolA
- 某模块用 poolB

但我们并不想因此让容器类型也发生变化。

### 2.3 传统 allocator 写起来比较重

自定义 allocator 往往需要处理：

- `value_type`
- `rebind`
- `allocator_traits`
- propagation
- equality
- 状态传递

在很多实际场景下，程序员真正想要的是：

> “给我一个容器，让它用这块内存资源。”

PMR 正是为这个目标设计的。

---

## 3. PMR 的核心思想

PMR 的思路是：

1. 把“分配内存”的逻辑抽象成一个运行时对象：`memory_resource`
2. 用 `polymorphic_allocator` 把这个资源适配给 STL 容器
3. 提供 `pmr::vector`、`pmr::string` 等方便直接使用
4. 提供几种常见内存资源实现，比如 monotonic buffer 和 pool resource

你可以把它理解成四层：

### 第 1 层：抽象接口
- `std::pmr::memory_resource`

### 第 2 层：allocator 适配器
- `std::pmr::polymorphic_allocator<T>`

### 第 3 层：容器别名
- `std::pmr::vector<T>`
- `std::pmr::string`
- 其他 `pmr` 容器

### 第 4 层：具体资源实现
- `std::pmr::monotonic_buffer_resource`
- `std::pmr::unsynchronized_pool_resource`
- `std::pmr::synchronized_pool_resource`

---

## 4. 一个先看结论的直觉理解

传统 allocator 更像：

> “这个容器的内存策略是类型的一部分。”

PMR 更像：

> “这个容器的内存策略是构造时塞进去的一个资源对象。”

所以：

```cpp
std::pmr::vector<int> a(&pool1);
std::pmr::vector<int> b(&pool2);
```

这里 `a` 和 `b` 是**同一种类型**，但底层用的是不同内存资源。

这是 PMR 最重要的优势之一。

---

## 5. `std::pmr::memory_resource`

这是 PMR 的核心抽象基类。

你可以把它理解成：

> **“运行时多态的内存分配接口。”**

概念上它像这样：

```cpp
class memory_resource {
public:
    void* allocate(std::size_t bytes, std::size_t alignment);
    void deallocate(void* p, std::size_t bytes, std::size_t alignment);
    bool is_equal(const memory_resource& other) const noexcept;
};
```

标准实现内部实际上通过虚函数完成：

- `do_allocate`
- `do_deallocate`
- `do_is_equal`

也就是说：

- 不同的 resource 可以覆写这些函数
- 调用时通过虚函数动态分派
- 因此“真正的分配策略”在运行时决定

---

## 6. `memory_resource` 解决了什么问题

有了 `memory_resource`，你就可以：

- 保持容器类型不变
- 仅通过传不同的 resource 指针切换内存策略
- 在工程里把“内存来源”变成配置/上下文，而不是类型爆炸

例如：

```cpp
std::pmr::memory_resource* r = std::pmr::get_default_resource();
```

这是一种默认全局资源。

你也可以把自己的 resource 传给容器。

---

## 7. `std::pmr::polymorphic_allocator<T>`

这是 PMR 和 STL 容器之间的桥梁。

它的作用是：

> **把 `memory_resource*` 包装成一个 allocator，让标准容器可以直接使用。**

它内部可以粗略理解为持有一个：

```cpp
std::pmr::memory_resource* res_;
```

当容器调用：

- `allocate`
- `deallocate`
- `construct`
- `destroy`

时，`polymorphic_allocator` 会把“分配/释放内存”委托给这个 resource。

---

## 8. 为什么叫 polymorphic allocator

因为它背后依赖的是 `memory_resource` 的**运行时多态**。

同样是：

```cpp
std::pmr::polymorphic_allocator<int>
```

但它内部可能指向：

- 默认资源
- monotonic resource
- unsynchronized pool resource
- synchronized pool resource
- 用户自定义 resource

也就是说：

- allocator 类型不变
- 具体策略运行时变

这就是 “polymorphic” 的来源。

---

## 9. `std::pmr::vector`

它不是一个新容器，而是一个别名：

```cpp
namespace std::pmr {
    template<class T>
    using vector = std::vector<T, std::pmr::polymorphic_allocator<T>>;
}
```

所以：

> **`std::pmr::vector<T>` 本质就是使用 `polymorphic_allocator<T>` 的 `std::vector<T>`。**

它和普通 `std::vector` 接口几乎一样，最大不同点是构造时可以传 resource：

```cpp
std::pmr::vector<int> v(&pool);
```

---

## 10. `std::pmr::string`

类似地，它本质上是：

```cpp
std::basic_string<char, std::char_traits<char>,
                  std::pmr::polymorphic_allocator<char>>
```

也就是：

> **一个使用 PMR allocator 的 string。**

这在工程里特别有价值，因为字符串往往是动态分配热点。

很多系统里：

- JSON 解析
- 协议解析
- 日志拼接
- 文本处理
- 请求上下文组装

都包含大量短期字符串分配。

这时让 `pmr::string` 使用请求级 arena / monotonic resource，会很有效。

---

## 11. PMR 的几种核心 resource

PMR 最常见的几种资源实现是：

- `monotonic_buffer_resource`
- `unsynchronized_pool_resource`
- `synchronized_pool_resource`

它们代表三种不同风格的分配策略。

---

## 12. `std::pmr::monotonic_buffer_resource`

这是 PMR 中最常见、也最值得掌握的资源之一。

它的核心思想是：

> **只往前分配，不单独回收。**

也就是：

- 分配时从一块连续区域往后 bump pointer
- 单个对象 `deallocate` 通常什么都不做
- 整个 resource 销毁时统一回收全部内存

这也叫：

- arena allocation
- region allocation
- bump allocation

---

## 13. `monotonic_buffer_resource` 的特点

### 优点

- 分配极快
- 实现简单
- cache locality 往往较好
- 很适合短生命周期对象批量分配

### 缺点

- 不能高效单独释放对象
- 如果生命周期管理不匹配，可能浪费内存
- 长生命周期场景下不适合滥用

---

## 14. 适用场景

特别适合：

- 单次请求生命周期内的临时对象
- 解析器 AST
- 编译器中间数据结构
- 一次构建、整体销毁的数据
- 批处理任务中的 scratch memory

它特别适合这种模式：

> “在一个阶段里分配很多对象，阶段结束后整体释放。”

---

## 15. `monotonic_buffer_resource` 示例

```cpp
#include <memory_resource>
#include <vector>
#include <iostream>

int main() {
    std::byte buffer[1024];
    std::pmr::monotonic_buffer_resource pool(buffer, sizeof(buffer));

    std::pmr::vector<int> v(&pool);

    for (int i = 0; i < 10; ++i) {
        v.push_back(i);
    }

    for (int x : v) {
        std::cout << x << ' ';
    }
}
```

这里：

- `pool` 优先使用栈上的 `buffer`
- `v` 的动态内存从 `pool` 来
- 当 `pool` 生命周期结束时，整体回收

---

## 16. 为什么 `monotonic_buffer_resource` 快

因为它不需要像通用 allocator 那样：

- 维护复杂空闲链表
- 做频繁的小块回收
- 每次都走系统分配器

很多时候它只是：

1. 看当前 buffer 还够不够
2. 如果够，直接返回一段对齐后的地址
3. bump 指针往后移动

这在性能敏感路径上很有吸引力。

---

## 17. `std::pmr::unsynchronized_pool_resource`

这是一个**非线程安全**的池式资源。

它的核心思想是：

> **把不同大小的小对象分桶缓存，减少频繁向上游资源申请/归还内存。**

适合大量小对象的反复分配和释放。

和 monotonic 不同的是：

- monotonic 更像“只增不减”
- pool resource 更像“有回收、有复用的对象池”

---

## 18. `unsynchronized_pool_resource` 的特点

### 优点

- 小块分配/释放效率高
- 减少系统 allocator 压力
- 适合反复分配释放的小对象
- 通常比直接使用默认 allocator 更高效

### 缺点

- 非线程安全
- 有池管理额外开销
- 生命周期过长时可能保留较多缓存块

---

## 19. 适用场景

适合：

- 单线程或外部保证线程隔离
- 大量小对象
- 容器频繁增删
- 字符串、节点、元数据反复创建销毁

例如：

- parser 的节点池
- 单线程服务组件
- 高频对象缓存
- 临时 map/list/set node 分配

---

## 20. `std::pmr::synchronized_pool_resource`

它和前一个类似，但区别是：

> **它是线程安全的。**

所以你可以理解成：

- `unsynchronized_pool_resource`：单线程版本，快
- `synchronized_pool_resource`：多线程版本，安全但更贵

---

## 21. `synchronized_pool_resource` 的特点

### 优点

- 线程安全
- 在多线程共享场景下更容易使用
- 对大量小对象分配有池化优势

### 缺点

- 有同步成本
- 在高竞争场景下可能成为瓶颈
- 单线程里通常不如 unsynchronized 直接

---

## 22. pool resource 和 monotonic 的区别

这是 PMR 里最容易被问到的点之一。

### monotonic_buffer_resource
- 只增不减
- 单次释放通常无意义
- 整体销毁时统一回收
- 适合“整批分配，整批销毁”

### pool_resource
- 有对象回收和复用
- 适合大量小对象反复分配/释放
- 更像经典对象池/size class pool

所以：

> **monotonic 更像 arena**  
> **pool_resource 更像 allocator cache / object pool**

---

## 23. upstream resource 是什么

很多 PMR resource 自己并不是“最终的内存来源”，而是从一个 **upstream resource** 再拿大块内存。

比如：

- pool resource 需要拿大块 chunk
- monotonic buffer 用完初始 buffer 后也可能继续向上游申请

默认上游资源通常是：

```cpp
std::pmr::get_default_resource()
```

但你也可以自己指定。

这意味着 PMR 资源是可以层层组合的。

---

## 24. 资源组合的思路

例如你可以这样理解：

- 最底层：默认 resource（最终向系统拿内存）
- 中间层：pool resource（做对象池）
- 上层：容器从 pool resource 拿内存

或者：

- 最底层：默认 resource
- 中间层：monotonic resource（arena）
- 上层：请求里的所有 `pmr::vector/string/map` 共用这一块 arena

所以 PMR 体系很适合做分层内存管理。

---

## 25. `get_default_resource()` / `set_default_resource()`

PMR 提供默认资源接口：

```cpp
std::pmr::memory_resource* r = std::pmr::get_default_resource();
```

以及：

```cpp
std::pmr::memory_resource* old = std::pmr::set_default_resource(new_r);
```

这让你可以替换全局默认 PMR resource。

但工程上需要谨慎，因为这可能影响很多依赖默认 resource 的组件。

---

## 26. `new_delete_resource()`

这是一个 PMR resource，语义上可以理解为：

> **底层走普通 `new/delete` 的 resource。**

如果你没有特殊资源，最终很多路径会落到它这里或等价行为。

它经常作为：

- 默认上游资源
- 自定义 resource 的后备资源

---

## 27. `null_memory_resource()`

这是一个永远无法分配成功的 resource。

如果调用它的 `allocate`，通常会抛 `std::bad_alloc`。

它有时用于：

- 限制不允许 fallback
- 调试资源耗尽行为
- 验证某段代码是否真的没有越界申请额外内存

比如你给 monotonic 一个固定 buffer，再把 upstream 设成 `null_memory_resource()`，那超出 buffer 就直接失败。

---

## 28. 一个典型 PMR 使用模式：请求级 arena

这是 PMR 非常经典的用法。

例如服务端处理一个请求时，会构造一个 monotonic resource：

```cpp
std::byte local_buf[4096];
std::pmr::monotonic_buffer_resource req_mem(local_buf, sizeof(local_buf));
```

然后这一整个请求中的临时对象都使用它：

- `pmr::string`
- `pmr::vector`
- `pmr::unordered_map`

当请求结束时，直接销毁 `req_mem`，所有临时内存一起释放。

这会带来几个好处：

- 减少碎片
- 减少小对象分配开销
- 提高 locality
- 内存释放逻辑非常简单

---

## 29. 为什么 PMR 对字符串特别有价值

字符串是工程里很常见的性能热点，因为：

- 数量多
- 生命周期复杂
- 常常短小但分配频繁

例如一个请求可能产生很多：

- header key/value
- JSON field
- 日志字段
- 临时拼接文本

如果这些都用普通 heap allocator，分配释放会很碎。  
如果这些都用请求级 PMR 资源，收益可能明显。

---

## 30. PMR 的一个重要特点：资源指针是运行时状态

传统 allocator 经常强调：

- stateful allocator
- propagation
- allocator equality

而 PMR 本质上把“状态”统一收敛成了：

```cpp
memory_resource*
```

也就是说：

- 哪块内存资源
- 生命周期由谁控制
- 是否共享
- 是否线程安全

很多都通过这个 resource 对象表达。

---

## 31. PMR 和传统 allocator 的关系

PMR 并不是完全绕开 allocator。

实际上：

- STL 容器依旧通过 allocator 接口工作
- PMR 只是提供了 `polymorphic_allocator`
- 这个 allocator 再把调用转发给 `memory_resource`

所以你可以理解成：

> **PMR 是建立在 allocator 模型之上的一个更实用的运行时资源体系。**

---

## 32. PMR 比传统 allocator 好在哪

### 1）容器类型不变
不同 resource 的 `pmr::vector<int>` 仍然是同一个类型。

### 2）运行时灵活
同一种容器可以按上下文选择不同 resource。

### 3）更适合 arena / pool 场景
这类需求天然更接近“资源对象”而不是“模板类型”。

### 4）组合性更强
resource 可以有 upstream，可以层层包装。

### 5）更适合工程落地
很多项目并不想为了不同内存策略引入一堆 allocator 类型。

---

## 33. PMR 的代价和注意点

PMR 很实用，但不是免费午餐。

### 33.1 运行时多态开销
`memory_resource` 基于虚函数，因此存在一定动态分派成本。

通常这个成本相对内存分配本身不算大，但在极致场景下仍需评估。

### 33.2 生命周期管理要小心
resource 不拥有所有使用它的对象生命周期逻辑，但容器/对象会依赖它。

比如：

- `pmr::vector` 还活着
- 但背后的 `monotonic_buffer_resource` 已经销毁

这会出严重问题。

所以：

> **resource 的生命周期必须覆盖所有使用它的 PMR 对象。**

### 33.3 不是所有场景都适合 monotonic
如果对象频繁独立释放，而不是整体销毁，那 monotonic 可能会浪费很多内存。

### 33.4 线程安全要分清
- `monotonic_buffer_resource`：通常不提供跨线程同步语义
- `unsynchronized_pool_resource`：非线程安全
- `synchronized_pool_resource`：线程安全但更贵

---

## 34. 一个 `pmr::string` + monotonic 的简单例子

```cpp
#include <memory_resource>
#include <string>
#include <iostream>

int main() {
    std::byte buf[1024];
    std::pmr::monotonic_buffer_resource mem(buf, sizeof(buf));

    std::pmr::string s1("hello", &mem);
    std::pmr::string s2("world", &mem);

    s1 += " ";
    s1 += s2;

    std::cout << s1 << '\n';
}
```

注意这里字符串内容扩展时用的动态内存来自 `mem`。

---

## 35. 一个 `pmr::vector` + pool resource 的例子

```cpp
#include <memory_resource>
#include <vector>
#include <iostream>

int main() {
    std::pmr::unsynchronized_pool_resource pool;
    std::pmr::vector<int> v(&pool);

    for (int i = 0; i < 100; ++i) {
        v.push_back(i);
    }

    std::cout << v.size() << '\n';
}
```

虽然 `vector<int>` 这个例子不一定最能体现 pool 的优势，但它展示了基本用法。  
对小对象频繁申请释放的节点型容器时，pool resource 通常更有表现空间。

---

## 36. 自定义 `memory_resource`

你也可以继承 `std::pmr::memory_resource`，实现自己的策略。

典型场景：

- 内存统计
- debug allocator
- NUMA-aware resource
- huge-page resource
- 共享内存 resource
- 限额 resource
- trace resource

概念上大致像这样：

```cpp
#include <memory_resource>
#include <iostream>

class LoggingResource : public std::pmr::memory_resource {
private:
    std::pmr::memory_resource* upstream_;

protected:
    void* do_allocate(std::size_t bytes, std::size_t alignment) override {
        std::cout << "allocate " << bytes << " bytes\n";
        return upstream_->allocate(bytes, alignment);
    }

    void do_deallocate(void* p, std::size_t bytes, std::size_t alignment) override {
        std::cout << "deallocate " << bytes << " bytes\n";
        upstream_->deallocate(p, bytes, alignment);
    }

    bool do_is_equal(const std::pmr::memory_resource& other) const noexcept override {
        return this == &other;
    }

public:
    explicit LoggingResource(std::pmr::memory_resource* upstream =
                                 std::pmr::get_default_resource())
        : upstream_(upstream) {}
};
```

然后：

```cpp
LoggingResource log_res;
std::pmr::vector<int> v(&log_res);
```

这样你就能观察容器的分配行为。

---

## 37. `do_is_equal` 是干什么的

这是 `memory_resource` 里一个容易忽略但重要的接口。

它的作用是：

> 判断两个 resource 是否可视为等价。

等价通常意味着：

- 一方分配出来的内存，另一方也可以合法回收
- 或者它们本质上代表同一个资源语义

最简单的实现通常是：

```cpp
return this == &other;
```

也就是“只有同一个对象才相等”。

但某些资源包装器也可能选择更复杂的等价判断。

---

## 38. PMR 和 allocator propagation 的关系

你前面学过 allocator propagation。  
PMR 本质上也没有让这个问题消失，只是把焦点转移到了 resource 上。

因为 `polymorphic_allocator` 的状态核心就是：

```cpp
memory_resource*
```

所以很多语义问题最终都会落到：

- 两个 allocator 背后的 resource 是否相同/等价
- 操作时是否能直接接管底层内存
- 生命周期是否安全

---

## 39. PMR 在高性能系统中的典型价值

### 39.1 请求级内存
一个请求内所有临时对象共享 arena，结束后整体回收。

### 39.2 解析器 / 编译器
AST、token、symbol 等中间对象通常很适合 monotonic。

### 39.3 高频字符串处理
大量 `pmr::string` 避免碎片化和频繁小分配。

### 39.4 池化场景
大量小对象频繁 new/delete，pool resource 很有帮助。

### 39.5 内存可观测性
自定义 resource 可以统计模块内存使用情况。

---

## 40. PMR 什么时候特别值得考虑

值得考虑的信号包括：

- 程序里小对象分配极多
- 请求生命周期清晰
- 字符串/容器临时对象很多
- 默认堆分配成为瓶颈
- 想做模块级内存隔离/统计
- 想用 arena 或 pool，但又希望复用标准容器

---

## 41. PMR 什么时候不一定值得

不一定适合的情况：

- 程序规模很小
- 内存分配不是热点
- 团队对对象生命周期控制不强
- 容器对象会长时间跨上下文存活
- 场景简单，普通 allocator 足够

因为 PMR 引入后，你需要更认真地管理 resource 生命周期和对象归属。

---

## 42. PMR 和 `std::vector` / `std::string` 迁移时的注意点

从普通容器迁移到 `pmr` 容器时，要特别注意：

### 1）类型变了
`std::string` 和 `std::pmr::string` 不是同一个类型。

### 2）resource 生命周期
所有 `pmr` 对象都不能比 resource 活得更久。

### 3）跨模块接口
如果接口里直接暴露 `pmr::string` / `pmr::vector`，要明确资源语义。

### 4）拷贝/移动行为
对象移动后是否仍然绑定原 resource，需要理解其 allocator 语义。

---

## 43. 面试高频：PMR 和传统 allocator 的区别

你可以这样答：

### 传统 allocator
- 分配策略通常通过模板参数决定
- 不同 allocator 往往导致容器类型不同
- 更偏编译期策略注入

### PMR
- 分配策略通过 `memory_resource` 在运行时决定
- 同一种 `pmr::vector<T>` 可以用不同资源
- 更适合 arena、pool、请求级内存管理

---

## 44. 面试高频：什么时候用 monotonic，什么时候用 pool

### monotonic_buffer_resource
用于：

- 临时对象
- 生命周期一致
- 整体销毁
- 极致快分配

### pool_resource
用于：

- 小对象反复分配释放
- 想复用已释放块
- 分配和释放交错频繁

一句话：

> **monotonic 适合“一锅端”**  
> **pool 适合“反复周转”**

---

## 45. 面试中可以怎么介绍 PMR

如果面试官问：

> “请介绍一下 C++ PMR。”

你可以这样回答：

PMR 是 C++17 引入的多态内存资源体系，核心组件是 `std::pmr::memory_resource` 和 `std::pmr::polymorphic_allocator`。它把内存分配策略从传统 allocator 的模板参数中抽离出来，改成运行时通过 resource 指针决定。这样同一种 `pmr::vector` / `pmr::string` 容器类型可以绑定不同内存资源，特别适合 arena、pool、请求级临时内存和内存统计等场景。常见资源有 `monotonic_buffer_resource`，适合整体分配整体释放；还有 `unsynchronized_pool_resource` 和 `synchronized_pool_resource`，适合大量小对象反复分配释放。

---

## 46. 一个简明脑图

### PMR 核心
- `memory_resource`：资源接口
- `polymorphic_allocator`：allocator 适配器
- `pmr::vector/string`：方便使用的容器别名
- `monotonic/pool resource`：具体资源策略

### 关键价值
- 容器类型不因为 allocator 改变
- 运行时切换内存策略
- 特别适合 arena / pool / request-local memory

### 常见资源选择
- 生命周期一致、整体释放：`monotonic_buffer_resource`
- 小对象高频分配释放：`pool_resource`

---

## 47. 一句话总结

> **PMR 是 C++17 提供的一套“运行时可切换的内存资源体系”，它让标准容器能够方便地使用 arena、pool 和自定义内存策略。**

---

## 48. 学习 PMR 的建议顺序

建议按这个顺序掌握：

1. 先理解传统 allocator 的局限
2. 理解 `memory_resource` 是运行时资源接口
3. 理解 `polymorphic_allocator` 是桥梁
4. 先会用 `pmr::vector` / `pmr::string`
5. 重点掌握 `monotonic_buffer_resource`
6. 再理解 `unsynchronized_pool_resource` / `synchronized_pool_resource`
7. 最后看自定义 resource 和工程场景

---

## 49. 常见术语对照

- PMR = Polymorphic Memory Resource
- arena / region = 整体分配整体释放风格
- pool = 池化复用风格
- upstream resource = 上游资源
- default resource = 默认资源
- polymorphic allocator = 运行时资源适配到 allocator 接口

---

## 50. 最简速记版

### PMR 是什么
- C++17 内存资源体系
- 运行时决定内存分配策略

### 核心组件
- `memory_resource`
- `polymorphic_allocator`
- `pmr::vector`
- `pmr::string`

### 常见 resource
- `monotonic_buffer_resource`：只增不减，整体释放
- `unsynchronized_pool_resource`：非线程安全对象池
- `synchronized_pool_resource`：线程安全对象池

### 为什么有用
- 容器类型不变
- 适合 arena / pool / request memory
- 方便自定义和组合内存策略
