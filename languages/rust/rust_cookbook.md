# Crate

---

# Trait

---

# Attribute

---


# 数据类型

## Scalar (标量)

### 数字
比较特殊的是两点:
1. `let ten=1_0;`
2. `let ten=10u8;`

### bool

### 字符
与C比较像，用单引号`'`的是字符，双引号`"`的是字符串。但是rust中字符是4 byte，不是存储ASCII，而是各种utf8，比如中英文和emoji。

## Compound (复合量)

### tuple

```rust
let tup: (u8, f32, bool) = (10, 3.14, true);

let (i, f, b) = tup;    // 解构 destruct

println!("{}", tup.1);  // 3.14
```

### array

与C中数组一样，定长，后面会提供一种类似C++的vector，可变长。

```rust
let arr1: [u8, 2] = [1, 2];
let arr2 = [1, 2, 3];
let arr3 = [0; 3];  // [0, 0, 0]
```

---

# 循环

## loop

### 带标签的嵌套loop

```rust
fn main() {
    let i: u8 = 0;
    'outer': loop {
        let j: u8 = 0;
        loop {
            if j == 2 {
                break;
            }

            if i == 2 {
                break 'outer';
            }

            j += 1;
        }
        i += 1;
    }
}
```

### 返回值

```rust
fn main() {
    let i: u8 = 0;
    let j = loop {
        if i == 5 {
            break i*10;
        }
        i += 1;
    }
}
```

## while
比较常见，和其他语言类似，不赘述。

## for
和 python 比较类似，不是`for(init; condition; update)`的格式，而是`for i in range`

```rust
fn main() {
    let v: [u8, 5] = [1, 2, 3, 4, 5];
    for i in v {
        println!("{}", i);  // 顺序
    }

    for i in (0..4).rev() {
        println!("{}", v[i]);   // 倒序
    }
}
```

---

# 所有权 (Ownership)

所有程序都必须管理其运行时使用计算机内存的方式。一些语言中具有垃圾回收机制，在程序运行时不断地寻找不再使用的内存(Java, Python)；在另一些语言中，程序员必须亲自分配和释放内存(C/C++)。Rust 则选择了第三种方式：通过所有权系统管理内存，编译器在编译时会根据一系列的规则进行检查。如果违反了任何这些规则，程序都不能编译。在运行时，所有权系统的任何功能都不会减慢程序。

请牢记以下三个规则：
1. Rust 中的每一个值都有一个被称为其 所有者（owner）的变量。
2. 值在任一时刻有且只有一个所有者。
3. 当所有者（变量）离开作用域，这个值将被丢弃。

比如说对于 *String* 类型而言，其会分配堆空间以存储字符串信息，而不是像字符串一样存储在栈上，因此，我们可以对 *String* 类型进行修改。对于C++这样的语言而言，当我们`malloc`后，变量便一直存在内存中，直到我们使用`free`释放；而对于rust，当超过作用域后自动被释放（可以理解为`}`执行之前使用了`drop`去释放资源）。

```rust
{
    let s = String::from("hello");  // alloc s
    ...
}   // drop s
// unknown s
```

## 移动与克隆

### 移动

```rust
let s1 = String::from("hello");
let s2 = s1;
println!("{}", s1); // error
```

在python中执行第一句时，堆中有一个 *"hello"* 空间，然后 s1 指向该空间。执行第二句时，变量 s2 同样指向该空间（即 s1 和 s2 实际上是同一空间的两个名字）。

<img src="https://kaisery.github.io/trpl-zh-cn/img/trpl04-02.svg">

在python的垃圾回收策略中，这并不是一个问题。但是对于rust，退出时会删除 s1 和 s2，即同一个内存位置删除了两次，这是会出错的。为了避免这个问题，在实现堆指针的赋值时，相当于移动了控制权，此处 s2 作为新的 s1，而原来的 s1 被清除。

<img src="https://kaisery.github.io/trpl-zh-cn/img/trpl04-04.svg">

### 克隆

```rust
let s1 = String::from("hello");
let s2 = s1.clone();
```

<img src="https://kaisery.github.io/trpl-zh-cn/img/trpl04-03.svg">

相当于python里的深拷贝，即内存中再次新建一块区域，这两个区域互不干扰。

### 总结

注意，移动和克隆只会针对**需要存储在堆上的数据类型**，比如对于整数，是不会有任何区别的（即没有深浅拷贝一说）。

```rust
let x: u8 = 10;
let y = x;  // x and y are both valid
```

Rust 有一个叫做 *Copy* trait 的特殊注解，可以用在类似整型这样的存储在栈上的类型上。如果一个类型实现了 *Copy* trait，那么一个旧的变量在将其赋值给其他变量后仍然可用。Rust 不允许自身或其任何部分实现了 *Drop* trait 的类型使用 *Copy* trait。如果我们对其值离开作用域时需要特殊处理的类型使用 *Copy* 注解，将会出现一个编译时错误。

## 所有权与函数

```rust
fn main() {
    let s0 = String::from("abc");
    let s1 = give_ownership(s0);    // 此时s0的所有权移动到s1，s0不再有效
    // println!("{}", s0);          // error
    println!("{}", s1);             // "abc"  

    takes_ownership(s1);            // s1 的值移动到函数里 ...
                                    // ... 所以到这里不再有效
    // println!("{}", s1);          // error

    let x = 5;
    makes_copy(x);                  // int类型并不存储在堆中，所以所有权不会被drop
    println!("{}", x);
}

fn takes_ownership(some_string: String) {
    println!("{}", some_string);
} // 这里，some_string 移出作用域并调用 `drop` 方法

fn give_ownership(some_string: String) -> String {
    return some_string;
}

fn makes_copy(some_integer: i32) {
    println!("{}", some_integer);
} // 这里，some_integer 移出作用域，并不调用 `drop` 方法

```

从上面的例子可以看出，函数同样满足所有权的概念。如果将一个存储于堆内的变量传入到新的函数域，并且并不将其返回，那么退出该函数域的时候就会清除堆内数据（和C++, Java, Python均不同）；除非进行返回，相当于所有权转让/移动。

还有一种不通过return进行所有权转让/移动的方法，那就是**引用**。

---

# 引用与借用

<img src="https://kaisery.github.io/trpl-zh-cn/img/trpl04-05.svg">

## 借用

借用别人的东西，就必须原封不动的还回去，这意味着我们即使拿到了数据，不会在退出作用域时被删除，但是我们也不能修改对应的内容。

```rust
fn main() {
    let s1 = String::from("hello");

    let len = calculate_length(&s1);

    println!("The length of '{}' is {}.", s1, len);
}

fn calculate_length(s: &String) -> usize {
    // 我们可以获取内存中的信息，但是不允许修改
    // s.push_str("world");
    s.len()
}
```

## 引用

通过将变量设置为可变，然后传递一个可变引用，即可实现对内容的修改同时保证所有权。

```rust
fn main() {
    let mut s1 = String::from("hello");

    let len = calculate_length(&mut s1);

    println!("The length of '{}' is {}.", s1, len);
}

fn calculate_length(s: &mut String) -> usize {
    s.push_str("world");
    s.len()
}
```

但是请注意:
* 不能对固定量使用可变引用，但是可以对可变量使用固定引用
* **在同一时间只能有一个对某一特定数据的可变引用**
* **不能在拥有不可变引用的同时拥有可变引用**

```rust
fn problem1() {
    let mut s = String::from("abc");
    let p1 = &mut s;
    let p2 = &mut s;    // 可以，此时所有权转移
    println!("{} {}", *p1, *p2);    // error, p1所有权已经转移了，p1是无效变量
}
fn problem2() {
    let mut s = String::from("hello");
    let p1 = &s;
    let p2 = &s;        // 可以拥有多个不可变引用
    let p3 = &mut s;    // 不可出现同时有可变以及不可变引用
}
```

这个限制的好处是 Rust 可以在编译时就避免数据竞争。**数据竞争（data race）**类似于竞态条件，它可由这三个行为造成：

* 两个或更多指针同时访问同一数据。
* 至少有一个指针被用来写入数据。
* 没有同步数据访问的机制。

数据竞争会导致未定义行为，难以在运行时追踪，并且难以诊断和修复；Rust 避免了这种情况的发生，因为它甚至不会编译存在数据竞争的代码！

注意**一个引用的作用域从声明的地方开始一直持续到最后一次使用为止**。(请注意是最后一次使用为止)

```rust
fn func1() {
    let mut s = String::from("abc");
    let p1 = &mut s;
    println!("{}", *p1);
    let p2 = &mut s;    // 可以，因为在println之后，p1没有再使用过，引用的作用域失效
}
fn func2() {
    let mut s = String::from("abc");
    let p1 = &mut s;
    println!("{}", *p1);
    p1.push_str("def");
    let p2 = &mut s;    // 可以，因为在push_str之后，p1没有再使用过，引用的作用域失效
}
fn func3() {
    let mut s = String::from("abc");
    let p1 = &mut s;
    let p2 = &mut s;    // 可以，因为p1的作用域就是定义，之后并未使用，所以定义p2后，p1失效，相当于所有权转移了
}
```

## 悬垂引用（Dangling References）

在具有指针的语言中，很容易通过释放内存时保留指向它的指针而错误地生成一个 悬垂指针（dangling pointer），所谓悬垂指针是其指向的内存可能已经被分配给其它持有者。相比之下，在 Rust 中编译器确保引用永远也不会变成悬垂状态：当你拥有一些数据的引用，编译器确保数据不会在其引用之前离开作用域。

---

# Slice

slice string 是 String 中一部分值的引用，其圈定的范围与python的语法类似。如果想要定义 slice 的类型，可以使用 `&str`。(注意，`&str`是不可变引用！)

```rust
let s = String::from("hello world");
let s1: &str = &s[..5];   // hello
let s2: &str = &s[6..];   // world
let s3: &str = &s[1..3];  // el

let s4: &str = &s;        // &str == &String[..]
let s5: &str = &s1[..];
```

> 注意，如果直接`let s = "hello";`，那么 s 为 &str 类型，不可变。

对于非string类型，slice同样适用，比如对于array，其类型为`&[u8]`。

```rust
let arr: [u8; 5] = [0: 5];
let sub: &[u8] = &arr[..3];  // [0, 0, 0]
```

---

# Struct

和其他语言中面向对象类似，只不过是先定义成员/属性，再通过类似实现接口的方式添加方法。

对于结构体而言，只能设置整个结构体都是 mut，不能单独设置某一个属性，但是可以设置某些属性为 pub。

## 基本定义和初始化

```rust
struct User {
    username: String,
    age: u8,
    activate: bool
}

fn main() {
    let u1 = User {
        username: String::from("tom"),
        age: 18u8,
        activate: true
    };

    let u2 = User {
        username: String::from("bob"),
        age: u1.age,
        activate: u1.activate
    };

    let u3 = User {
        username: u1.username,
        age: 20u8,
        activate: false
    };

    println!("{}", u1.username);    // error，因为u1.username是string，所有权已经转移，要想正常访问只能使用clone

    let u4 = User {
        username: u2.username.clone(),
        age: u1.age,
        activate: u2.activate
    };

    println!("{}", u2.username);    // bob

    let u5 = User {
        age: 20u8,
        ..u2    // 将剩余属性直接从u2转移过来
    }           // 相当于 u5.xxx = u2.xxx

    println!("{}", u2.username);    // error, 由于 ..u2 相当于赋值，所以所有权移动了
}
```

## 方法 Method

方法就是位于某个上下文（如结构体）内的函数，其作用仍然是函数，只不过需要通过指明特定的上下文，从而调用。

### 普通方法

普通方法的第一个参数永远是`self`相关，而为了将方法放入特定的上下文，我们可以使用`impl`。在`impl`的大括号内的函数均视为绑定到特定结构体的方法。

```rust
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    fn area(&self) -> u32 {
        self.width * self.height
    }
}

fn main() {
    let rec = Rectangle {10, 10};
    let area = rec.area();
}
```

在上述代码中，`Self`指代了`Rectangle`，`&self`指代了调用时的对象(此处为`rec`)，即`self: &Self`。如果希望能够更改结构体的属性，则需要使用`&mut self`作为第一个参数。

> 在c/c++中对于结构体，我们使用`.`访问，对于结构体指针，我们使用`->`访问。而在rust中，实现了 **自动引用与解引用**，即结构体和结构体指针均可以使用`.`去访问，当使用 `object.something()` 调用方法时，Rust 会自动为 object 添加 &、&mut 或 * 以便使 object 与方法签名匹配。

### 静态方法

其实rust中并没有静态这个概念，但是我们可以让某个方法作用于整个结构体，而非针对某个特定对象（又或者说针对于该结构体类型的所有对象）。具体实现时，我们第一个参数不是`self`相关，那么我们调用时就不能通过某个对象进行调用，而需要使用`structName::method(..)`进行调用。

```rust
struct Rectangle {
    w: u8,
    h: u8
}

impl Rectangle {
    fn show() {
        println!("this is a rectangle");
    }
}

fn main() {
    let rec = Rectangle {10, 10};
    Rectangle::show();
    // rec.show();  error
}
```

---

# 枚举

---

# match & if let


---

# 宏 Macros

宏（Macro）指的是 Rust 中一系列的功能：使用 `macro_rules!` 的 **声明（Declarative）宏**，和三种 **过程（Procedural）宏**：

* **自定义 `#[derive]` 宏** 在结构体和枚举上指定通过 derive 属性添加的代码
* **类属性（Attribute-like）宏** 可用于任意项的自定义属性
* **类函数宏** 看起来像函数不过作用于作为参数传递的 token

## 宏 VS 函数

从根本上来说，宏是一种为写其他代码而写代码的方式，所有的这些宏以展开的方式来生成比你所手写出的更多的代码。

一个函数标签必须声明函数参数个数和类型。相比之下，宏能够接受不同数量的参数：用一个参数调用 `println!("hello")` 或用两个参数调用 `println!("hello {}", name)` 。而且，宏可以在编译器翻译代码前展开，例如，宏可以在一个给定类型上实现 trait 。而函数则不行，因为函数是在运行时被调用，同时 trait 需要在编译时实现。

实现一个宏而不是一个函数的缺点是宏定义要比函数定义更复杂，因为你正在编写生成 Rust 代码的 Rust 代码。由于这样的间接性，宏定义通常要比函数定义更难阅读、理解以及维护。

宏和函数的最后一个重要的区别是：在一个文件里调用宏之前必须定义它，或将其引入作用域，而函数则可以在任何地方定义和调用。

---