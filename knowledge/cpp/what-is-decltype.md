# decltype 介绍

`decltype` 是 C++11 引入的一个关键字，用来 **在编译期获取表达式的类型**。他不会计算表达式，只是 **推导表达式的类型**。

```cpp
decltype(expression)
```

---

# 基本用法

## 获取变量 / 表达式类型
```cpp
int a = 10;
decltype(a) b = 20; // 🟰 int b = 20;
decltype(a + b) c; // 🟰 int c;
```

## 获取函数返回值类型
一般用于 template 中。因为传入参数类型未知，所以返回类型也未知，这个时候使用 **Trailing Return Type**。
```cpp
template<typename T, typename U>
auto function(T t, U u) -> decltype(t + u) {
    return t + u;
}

// ❌ 不能写在前面，因为变量还未定义
// decltype(t + u) fucntion(T t, U u) {...};
```

## 保留引用类型
`decltype` 会保留表达式的引用属性。
```cpp
int a = 10;
int& ref = a;
decltype(ref) b = a; // b 的类型是int&
```

---

# auto vs decltype
| | auto | decltype |
| :---: | :---: | :---: |
| 是否保留引用 | 否 | 是 |
| 是否保留 const | 否    | 是 |
| 是否需要初始化 | 需要 | 不需要 |

所以 `auto` 看值， `decltype` 看表达式。

```cpp
int a = 10;
int& ref = a;
const int b = 10;

auto xa = a;          // int
decltype(ref) ya = a; // int&

auto xb = b;        // int
decltype(b) yb = b; // const int
```

---

# `decltype((var))` 是引用类型

```cpp
int a = 10;
decltype(a) b;   // int
decltype((a)) c; // int&
```