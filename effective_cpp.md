# explicit

编译器会对**只有一个不确定参数**的**构造函数**采用隐式类型转换

```cpp
class Object {
public:
    int _val;
    int _size;
    Object(int val, int size = 10) : _val(val), _size(size) {};
}

Object o = 10;  // ok, 10 被隐式转换为Object类型
```

通过加`explicit`能避免这个问题

```cpp
class Object {
public:
    int _val;
    int _size;
    explicit Object(int val, int size = 10) : _val(val), _size(size) {};
}

Object o = 10;  // error
Object o(10);   // ok
```

---

# decltype & 返回类型后置（trailing-return-type)

`auto`必须要求右侧有内容且 cannot use in non-static member variable

```cpp
auto a; // error

class Object {
public:
    static auto size = 10;
    auto num = 20;  // error
}
```

`decltype(expr)`更加灵活，且可以和`auto`配合用于 **返回类型后置（trailing-return-type)**

> 注意如果expr是函数，那么需要包含函数名，括号以及内部的参数

```cpp
int& func(int a, int b);
int&& func(void);

decltype(10+20) a = 20;     // int a = 20
decltype(func(10, 20)) b;   // int& b
decltype(func()) c;         // int&& c

template<typename T, typename K>
auto add(T a, K b) -> decltype(a+b) {
    return a + b;
}

int a = 10;
double b = 10.0;
bool c = true;
add(a, b);  // decltype(10 + 10.0) -> double
add(a, c);  // decltype(10 + true) -> int
```

---

# enum

```cpp
enum Type {
    INT,
    DOUBLE,
    BOOL
}

// 根据 type 选择输出的 res 类型
switch (type) {
    case Type::INT {
        int res = val;
        break;
    }
    case Type::DOUBLE {
        double res = val;
        break;
    }
    case Type::BOOL {
        bool res = val;
        break;
    }
    default {
        string res = val;
        break;
    }
}
```

---

#  使用using代替typedef

```cpp
typedef uint unsigned int;
using uint = unsigned int;  // 通过赋值更容易看出结构

// ----

template<typename T>
using map_str_t = map<string, T>;

map_str_t<int> map1;

// ----

typedef void (*func_t)(int, int);
using func_t = void (*)(int, int);  // 通过赋值更容易看出结构
```

---

# 常使用 const !

1. 普通变量加const代表元素不可被更改
2. 函数参数加const代表该函数不可修改传入参数的内容
3. 类内方法加const代表该方法不会修改类的成员变量

## const 指针

> `const char* p` &rarr; const data
> `char* const p` &rarr; const pointer

## const 迭代器

> `const vector<int>::iterator` &rarr; 表明迭代器只能指向某一个位置,不可移动
> `vector<int>::const_iterator` &rarr; 表明迭代器指向的内容不可变,但是可以移动
> ```cpp
> const vector<int>::iterator it1 = nums.begin();
> *it1 = 10;    // ok
> it1++;        // error
>
> vector<int>::const_iterator it2 = nums.begin();
> *it2 = 10;    // error
> it2++;        // ok
> ```

## mutable

标记为mutable的变量可以在const函数中被改变

```cpp
class Object {
public:
    char* buffer;
    mutable size_t length;
    mutable bool is_valid;
    ...
    size_t size() const {
        if (!is_valid) {
            length = strlen(buffer);
            is_valid = true;
        }
        return length;
    }
}
```

---

# static

1. static变量的生命周期是整个程序, 且只会在第一次被执行时初始化
2. class 的 static 变量是类共享而非某个对象独有
3. class 的 static 方法只能使用 static 变量或其他 static 方法

```cpp
class Object {
public:
    static int common;
    static void update() {
        return ++common;
    }
}

int Object::common = 0;
```

当static变量在多个不同的作用域内, 且某些static变量需要使用其他static变量的时候, 会遇到问题. 因为多个程序编译和执行的顺序无法预计和固定, 所以可能出现某个static需要使用另一个没有初始化过的static变量, 那么就会报错.

```cpp
// file.cpp
class FileSystem {
    ...
}
extern FileSystem fs;
```

```cpp
// local.cpp
class Local {
    Local () {
        int size = fs.get_size();   // 要求一定要初始化过fs
    }
}
```

解决的方法就是把static变量变为non-static函数的返回值, 这样只有我们call函数的时候才会涉及到static的初始化, 但是函数调用的顺序我们是可以控制的, 因此我们可以控制static的初始化顺序.

```cpp
// file.cpp
class FileSystem {
    ...
}

FileSystem& init_fs() {
    static FileSystem fs;
    return fs;
} 
```

```cpp
// local.cpp
class Local {
    Local () {
        int size = init_fs().get_size();    // ok, 一定会调用函数, 从而保证fs一定被先初始化过
    }
}
```

---

# 构造函数

```cpp
class Object {
public:
    int _val;
    
    Object();                               // default constructor
    Object(int val);                        // converting constructor 
    Object(const Object& rhs);              // copy constructor
    Object& operator=(const Object& rhs);   // copy assignment
    Object(const Object&& rhs);             // move constructor
}
```

每当新创建一个对象的时候一定调用constructor.

```cpp
Object a;
Object b(a);    // copy constructor
Object c = b;   // copy constructor, 因为c是新的, 只能被构造
c = a;          // copy assignment, 因为c已经存在, 此处是赋值
```

> 在调用构造函数时, 更推荐使用 member initialization list 进行“初始化”而非“赋值”. 另一个有趣的事实是: 初始化的顺序是固定的, 先父类再子类, class内的变量按照定义顺序初始化.

## copy assignment

注意: operator= 中需要注意先判断是否和自身相等(证同)

## 子类copy函数需要记得copy父类的内容

在子类copy assignment或者copy constructor中, 由于某些成员是父类的, 所以我们也不能忘了他们.

```cpp
//  父类
class Base {    
public:
    ...
    Base(const Base& rhs);
    Base& operator=(const Base& rhs);
    ...
private:
    int base_var;
    struct base_struct;
};

// 子类
class Derive : Base {   
public:
    ...
    Derive(const Derive& rhs);
    Derive& operator=(const Derive& rhs);
    ...
private:
    int derive_var;
};
```

```cpp
Derive::Derive(const Derive& rhs) : Base(rhs) { // 一定要记得调用Base的copy constructor
    derive_var = ...;
}

Derive& Derive::operator=(const Derive& rhs) {
    if (&rhs == this) {     // 证同
        return *this;
    }
    Base::operator=(rhs);   // 一定要记得调用Base的copy assignment
    derive_var = ...;
}
```

## 禁止copy

“如果子类不定义构造函数, 那么默认构造函数会尝试调用父类的对应构造函数”, 于是我们可以创建一个父类, 其中copy相关的函数放入private中并且不做任何实现. 那么对于其他类, 如果不想支持copy, **只需要继承这个父类, 并且不实现任何copy相关函数**以保证会使用默认的.

```cpp
class Uncopy {
public:
    ...
private:
    Uncopy(const Uncopy& rhs);              // 只声明不定义 
    Uncopy& operator=(const Uncopy& rhs);
}

class Object: private Uncopy {
public:
    ...         // 正常定义其他函数, 不要定义copy assignment & copy constructor
private:
    ...
}
```

---

# virtual & =0

* 没有 virtual &rarr; 根据引用类型或指针类型选择方法; 有 virtual &rarr; 根据引用和指针指向的对象选择方法
* 在基类中将派生类会重新定义的方法设置为virtual, 这样在派生类中重定义时会自动生成virtual
* virtual关键字只用于类声明中, 而不用于定义, 在定义时通过className加以区分即可
* **如果某个类是基类, 其会有子类, 那么需要使用虚析构函数(类内至少有一个virtual函数的时候才需要定义virtual析构函数)**, 否则建议不要使用virtual析构函数, 因为会增加额外的虚函数表开销

> polymorphic base class 应该声明virtual析构函数, 如果class带有任何virtual函数, 他就应该拥有virtual析构函数; 如果某个 base class 不是为了 polymorphic, 就不需要virtual析构函数.

> 如果使用了virtual, 对象会额外保存一些信息 &rarr; vptr (virtual table pointer) 会指向一个函数指针构成的数组 vtbl (virtual table). 每个带有virtual的class都会有对应的vtbl. 当调用virtual函数的时候, 实际被调用的函数取决于vptr所指向的vtbl.

> 不要在构造函数或者析构函数中调用virtual方法

`virtual void func() = 0;`代表纯虚函数, 有纯虚函数的class不能被实例化, 只能被继承.

---

# override vs overload

override是子类继承父类的virtual函数, overload是同名但函数签名不同.

---

# 按值返回 vs 按引用返回

1. 如果返回的是class, 那么按值返回一个副本, 会默认调用copy constructor去赋值, 效率低
2. 按引用返回的时候一定保证返回的东西不会在函数退出后被解构, 比如static和放在heap里的
3. 按引用返回的内容可以作为左值, 按值返回的只能做右值. 所以如果需要连续调用, 一定要按引用返回

> * 比如对于string类而言, `string& operator=(const string& rhs)` 返回的是引用, 因为可能有 `str_a = str_b = str_c`
> * 但是 `string operator+(const string& rhs)` 返回的是值, 因为调用的方法是 `str_res = str_a + str_b`, 即会赋值给一个新的

```cpp
// return by value
vector<int> return_by_value(int a, int b) {
    vector<int> res = {a, b};
    return res;
}

// return by reference
vector<int>& return_by_reference(int a, int b) {
    vector<int>* res = new vector<int> {a, b};
    return *res;
}

// return by pointer
vector<int>* return_by_pointer(int a, int b) {
    vector<int>* res = new vector<int> {a, b};
    return res;
}

vector<int> value = return_by_value(1, 2);  // ok
return_by_value(1, 2)[0] = 3;               // error

vector<int> reference = return_by_reference(1, 2);  // ok
return_by_reference(1, 2)[0] = 3;                   // ok, [3, 2]

vector<int>* pointer = return_by_pointer(1, 2);     // ok
(*return_by_pointer(1, 2))[0] = 3;                  // ok, [3, 2]
```

> 对于assignment赋值函数(eg. =, +=, etc), 一定按引用返回

---

# smart pointer

smart pointer的好处在于被指向的资源不需要手动去delete, 会自动调用析构函数(意味着最好不要指向C风格内容, 否则没有正确的析构函数可以调用)

> 因为是指针, 使用 -> 去访问指向资源的方法和成员; 使用 . 去访问智能指针自带的方法.

## auto_ptr / unique_ptr

只能有一个智能指针指向同一个资源, 区别在于:
1. auto_ptr 可以copy, 结果就是老的变为null, 新的获得控制权
2. unique_ptr 不能copy只能move, move之后老的自然被销毁

```cpp
auto_ptr<Object> p1(new Object(1));
auto_ptr<Object> p2(new Object(2));
p1->show(); // Obj(1)
p2->show(); // Obj(2)

p2 = p1;

p1->show(); // null
p2->show(); // Obj(1)
```

```cpp
unique_ptr<Object> p1(new Object(1));
unique_ptr<Object> p2(new Object(2));
p1->show(); // Obj(1)
p2->show(); // Obj(2)

p2 = p1;    // error
p2 = move(p1);

p1->show(); // null
p2->show(); // Obj(1)
```

## shared_ptr

可以有多个智能指针指向同一个资源, 会有一个计数器记录某个资源被多少个指针指向, 当指针数量为0, 就会消除资源.

```cpp
shared_ptr<Object> p1(new Object(1));
shared_ptr<Object> p2(new Object(2));
shared_ptr<Object> p3 = p1;

cout << p1.use_count(); // 2 (p1, p3)
cout << p2.use_count(); // 1 (p2)
```

### 在函数中传递shared_ptr

因为本身是一个指针, 所以都是按值传递 (对于shared_ptr指向的内容, 相当于按指针传递)

### deleter

有时候shared_ptr某个资源利用率为0的时候, 我们并不想删除它, 而是做额外操作. 比如说我们有一个锁, 资源上没有锁的时候, 我们应该unlock而不是删除资源.

```cpp
void unlock(Object* obj) {
    cout << obj << " is unlocked" << endl;
};

int main() {
    ...
    shared_ptr<Object> p(new Object(1), unlock); // 计数器为0时调用unlock而非调用Object析构函数
}   // Object(1) is unlocked
```

这里要求删除器：
1. 可以是function object, 函数指针, Lambda表达式, bind/functor等等均可.
2. **返回值是void，参数是shared_ptr类型的指针**
3. 从形参看出, 删除器以传值的方式传入, 所以要求**删除器要是可拷贝的**
4. 删除器不要抛出异常

## 注意只会调用`delete`而非`delete[]`

因此使用智能指针指向C类型的数组是一个bad idea.

```cpp
unique_ptr<int> p(new int[10]); // bad
unique_ptr<vector<int>> p(new vector<int>);
```

---

# constexpr

<s>常量表达式和非常量表达式的计算时机不同，非常量表达式只能在程序运行阶段计算出结果；而常量表达式的计算往往发生在程序的编译阶段，这可以极大提高程序的执行效率，因为表达式只需要在编译阶段计算一次，节省了每次程序运行时都需要计算一次的时间。

`constexpr`关键字的功能是使指定的常量表达式获得在程序编译阶段计算出结果的能力，而不必等到程序运行阶段。
> 注：获得编译阶段得出结果的能力不代表一定在编译期间被执行</s>