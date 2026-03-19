# 为什么需要smart pointer

防止内存泄漏，smart pointer可以自动管理动态内存的生命周期。其核心思想是: **把资源释放绑定到对象生命周期中，利用RAII自动回收资源**。

```cpp
void function() {
    int* ptr = new int(100);
    if (...) {
        ...  // if throw error, it won't run delete
    }
    delete ptr;
}
```

---

# RAII

RAII (Resource Acquisition Is Initialization) 资源在对象构造时获得，在对象解构时释放。常见的RAII的资源有socket, mutex lock, database connection等。

详情请参考[RAII](./what-is-RAII.md)。

---

# 类别

## `unique_ptr`
### 唯一拥有
当指针生命周期结束时会自动delete。
```cpp
std::unique_ptr<int> p = std::make_unique<int>(42);
```

> [!NOTICE]
> 推荐使用`std::make_unique<int>(42);`而不是`std::unique_ptr<int>(new int(42))`。主要原因是因为`make_unique<T>`更安全。
> ```cpp
> foo(make_unique<T>(), make_unique<U>());
> foo(unique_ptr<T>(new T()), unique_ptr<U>(new U()));
> ```
> 第二个不安全，因为编译器可能先`new T()`，然后unique_ptr还没指向对象，编译器就去执行`new U()`，如果此时异常那么T就内存泄漏了。`make_unique<T>`更像一个atomic operation，保证了指针指向内容。

### 不能拷贝
```cpp
std::unique_ptr<int> p1 = std::make_unique<int>(42);
std::unique_ptr<int> p2 = p1; // ❌
```

### 可以移动

```cpp
std::unique_ptr<int> p1 = std::make_unique<int>(42);
std::unique_ptr<int> p2 = std::move(p1); // 此时p1是nullptr
```

### 作为函数参数/返回值
#### 按值传递 &rarr; 接管所有权
```cpp
void func(std::unique_ptr<int> p) {
    std::cout << *p << std::endl;
}

auto p = std::make_unique<int>(42);
func(std::move(p)); // 之后p就是nullptr了，因为已经移交给func，func退出时析构了p，释放了资源
```

#### 按引用传递 &rarr; 借用所有权
```cpp
void func(std::unique_ptr<int>& p) {
    std::cout << *p << std::endl;
}

auto p = std::make_unique<int>(42);
func(p); // 之后p仍然有效
```

#### 按原始指针传递 &rarr; 无法使用智能指针的方法，只能使用raw pointer的操作
```cpp
void func(int* p) {
    std::cout << *p << std::endl;
}

auto p = std::make_unique<int>(42);
func(p.get());
```

#### 返回值 (按值返回)

一般都是按值返回，调用者接管所有权。
```cpp
std::unique_ptr<int> make_value() {
    return std::make_unique<int>(42);
}

auto p = make_value(); // p接管了临时对象的所有权
```

#### 自定义deleter
```cpp
// ⚠️ 此处使用函数指针类型而非函数类型，即 &fclose 而非 fclose
std::unique_ptr<FILE, decltype(&fclose)> fp(fopen(...), &fclose);
```

> [!NOTICE]
> 函数类型 != 函数指针类型
> ```cpp
> int add(int x, int y) { return x + y; }
> 
> int(int, int) p = add; // ❌，函数类型不能当作对象类型
> int (*p)(int, int) = add; // ✅，函数指针类型可以是对象类型
> auto p = add; // 推导为 int(*)(int, int)
> ```
> 函数本体不是一块"可拷贝存放到变量里"的普通对象

## `shared_ptr`

## `weak_ptr`