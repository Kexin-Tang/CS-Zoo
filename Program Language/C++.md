# **基本概念**
## 名称空间
* 使用名称空间可以将不同文件中同名的函数，变量区分开来，以免冲突
* 在不同的地方使用该定义，具有不同的生命周期

*(注意：如果使用头文件，在include<xxx.h>时不需要使用名称空间，只有使用include<xxx>时需要使用)*

```c++
using namespace std
// using std::cout
```
## 泛型 Vs 面向对象OOP
* 泛型——忽略不同数据类型之间的差异
* OOP——对某些数据定义统一操作方式
## 类 Vs 对象
* 类Class——定义某些数据和操作
* 对象Object——创建某一个Class的实体
## 函数原型 Vs 函数定义
- 原型——只针对函数的返回值、形式参数等进行定义
- 定义——针对函数的具体实现
```c++
int max(int a, int b);  # 原型
int max(int a, int b)   # 定义
{
  return a>b?a:b;
}
```
## 初始化

使用列表初始化能很好地避免隐式类型转换，即数据无法被截断，如 float->int

```c++
int a = 10;  // traditional init
int b(10);   // C++ type init
int c = {10};// list init
```
## 强制类型转换
```c++
(type) value;
type (value);
static_cast<tpye> (value)
```
## 数组
* 只能在定义的时候对数组初始化，且不能将一个数组赋值给另一个数组
* 可以使用列表初始化
* 只对一部分初始化将会将剩余部分置为0
* 定义数组的时候，数组元素个数必须是常量
```c++
int list[5] = {1, 2, 3};  // list = 1, 2, 3, 0, 0
// int list[5]; list = {1,2,3};  是错误的方式
// int list2[5] = list;          也是错误的方式
```
* 字符串数组中不要忘记最末位有个'\0'也是算入个数的
## String类
- 不用指定长度
- 可以相互赋值
- 可以使用"+"进行拼接
- 可以使用"=="判断是否相同

*(注意：C风格字符串即char[]不能使用"=="判断是否相同，而要用strcmp()!!)*

## 结构体Structure
* 不像C语言，不使用typedef也可以省去structure关键字
* 可以使用列表初始化{}
* 可以将结构体相互赋值
## 联合Union Vs 枚举Enum
- union——存储不同数据类型，且每个类型只有一个
```c++
union Brand
{
  int id_int;
  string id_string;
}
Brand brand;
std::cout<<brand.id_int;
```
- enum
```c++
enum color{red, orange, blue}; // red:0 orange:1 blue:2
// red = 100; 不能对enum进行赋值
// color(4);  不能越界
enum size{small=1, medium=10, large=100};
// size(50);  不报错，但是没有意义。只要idx在定义的[min, max]都不会报错
enum name{tom, alice=0, bob=1};
```
## 动态联编 Vs 静态联编
- 动态联编——在程序运行后才进行内存分配等操作(new操作，虚函数等)
- 静态联编——在程序编译时即进行内存分配
## 堆Heap Vs 栈Stack
- heap——为new的内存来源
- stack——为用户定义的内存来源
## vector、array和数组
- vector == 整合了new&delete的可边长数组
- array == 可直接赋值的定长数组
```c++
vector<int> v(n);  // n 可以是cin>>n;动态分配
array<int, 10> a;  // 若还存在array<int, 10> b;则可以a=b
int i[10];
```
## 基于范围的for循环
```c++
double points[5] = {0.1, 1.2, 6.5, 7.2, 9.0};
for(double x: points)  // 使用该方法只能读出数据 
  cout<< x<<endl;
for(double &y: points)  // 使用&可以修改数据
  y = y*0.1;
```

---


# **指针**
## 创建/消除指针
* *delete只能消除用new创建的指针!!*
```c++
type* pointer = new type;
type i;
type* pi = &i;
delete pointer;    // ok
delete pi;         // not allowed
```
## 指针 Vs 数组名
- 指针是变量(未使用const时)，而数组名是常量，所以指针可以+1操作
- int * p;那么p+1为在p的地址上移动sizeof(int*)的距离
- 数组名是第一个元素的地址，而对数组取地址得到的是整个数组的地址
```c++
short tell[10];
cout<< tell<< endl;  // display the addr of tell[0], maybe 0xFF
cout<< &tell<< endl; // display the addr of tell, maybe 0xFF
cout<< tell+1<<endl;  // 地址增加2
cout<< &tell+1<<endl; // 地址增加2*10
cout<< sizeof(&tell[0])<<endl;    // display 2
cout<< sizeof(tell)<<endl;        // display 2*10
```
## 指针数组 Vs 数组指针
- 指针数组——指针类型的数组
- 数组指针——指向数组的指针
```c++
int* p[10];    // 指针数组，p[10]为数组，内部为int*指针
int (*p)[10];  // 数组指针，*p作为一个指针，指向int[10]
```
## 浅复制 Vs 深复制
* *不要用赋值运算给数组输入数据，而是用strcpy()，但是string可以用=*
```c++
string s1 = "hello C++";
char *p = new char(strlen(s1)+1);
// p = s1; 此时为深复制，即p与s1指向同一个地方，此时创建的原来new的区域失去指针，且更改p或s1会同时更改另一个
// strcpy(p, s1); 此时为浅复制，p与s1地址不同，但是内容相同，修改s1不会更改p
```
## const指针
* *不能将const地址赋给非const指针*
```c++
int a = 10;
const int * p = &a;  // 此时，*p为const，不能通过p改变a的值，但是可以直接改变a的值，如a = 11(√)； *p = 11(×)
int* const t = &a;   // 此时，*t可以更改a的值，但是不能指向别的地址
```

---


# **函数**
## 函数中传递数组
* 数组名 == 数组第一个元素的地址
```c++
int max(int arr[]);    // 此处使用 int arr[] == int* arr
max(myArr, ArrSize);   // 调用时，只要传入数组名，不需要取地址
```
- 推荐显示传递数组的元素个数，而不是在函数中再去计算个数，因为传入的只有第一个元素的地址，无法得到总个数
```plain
int max(int arr[][4], int arrSize);
```
- 传递二维数组时，要指明列数，行数可以不指明。而且此时，列数传去的参数是可以直接当做数据使用的。
- 传递常规变量时，修改的是临时变量；但传递数组时，修改的是原数据
## 返回指针
```c++
int main()
{
  char * p = build(...);
  cout<< p<<endl;
  delete [] p;    // 在build中使用了new[]，所以需要在此处使用delete[]
}
char * build(...)
{
  char * ps = new char[n+1];
  ...    
  return ps;  //退出时，ps作为局部变量被消除，但是ps指向的空间是new的，所以仍然存在，main中通过返回的地址，
}
```
## const函数
```c++
int max(const int arr[]); // 此时该函数只能读arr而不能修改 
```
## 函数指针
- 在函数声明中，将函数名更换为(*函数名)即可，然后再代码中将别的函数名替换进来即可
- 对于单个函数指针，如果定义其类型过于复杂，可以使用auto自动推断其类型
```c++
int max(int arr[]);
int (*p)(int arr[]);
p = max;
int a = max(arr);
int b = (*p)(arr); 
```
- 使用typedef来简化代码
```c++
typedef double* (*p_func)(double);
p_func p1 = f;
double* (*p2)(double) = f;
```
## 内联函数
- 一般直接将定义书写在原本声明处，并加上inline关键字
- 不需要在不同地址间互相跳跃，直接将内容增加到调用的代码内存空间后面
```c++
inline int square(int a) {return a*a;}  //一般内联函数都比较短小
```
## 默认参数
- 只需要在函数声明的时候写出默认即可，函数定义时可以不用管
- 默认参数只能位于最右侧
```c++
int max(int a, int b, int c=0);
int max(int a, int b, int c) {...} 
```
## 函数重载(多态)
- 多个同名函数起到不同作用
- 函数重载的关键是"函数特征标"（参数引用和参数本身是同样的类型！！）
- 函数的返回值可以不同，但是特征标一定要不同！
- 名称修饰(名称矫正)——在编译阶段，针对函数重载的不同格式进行不同的编码，以区分
```c++
int max(int, int);
double max(double, double);
// int max(int &, int &); 并不是重载，和第一个函数冲突 
```
- 在类中，是否const是不同的特征标
```c++
char& operator [](int i);
char& operator [](int i) const;  // 这两个函数是不同的！！
```
## 函数模板
* 对于多种不同类型的数据，进行同样的操作，则使用模板
```c++
template <typename type>    // 当传入的a,b为int时，type==int
void swap(type& a, type& b)
{
  type temp;
  temp = a;
  a = b;
  b = temp;
}
```
* 模板也可以进行重载，只要其函数特征标不同即可
* 模板可以含有多种数据类型
```c++
template <typename t1, typename t2>
void add(t1 a, t2 b) {...}
```
##### 显示具体化
* 当模板无法完成某一类型的操作时，可以使用显示具体化，对某一个类型单独设计操作
```c++
// 在函数声明和定义时，前方加入template<>，并在函数名后加入<类型>即可
template<> void Swap<job>(job& a, job& b);
// 可以缩写成 template<> void Swap(job&, job&);
```
* 优先级：非模板函数 > 显示具体化 > 模板函数
##### 实例化
* 实例化的函数定义仍然满足同名函数的模板，只不过是实例化了对应的参数类型
* 具体化的函数定义是不满足函数模板的，需要进行重载
* 实例化不需要定义，只需要声明；具体化需要声明+定义
```c++
template<> void Swap(job&, job&);  // 具体化
template void Swap(int&, int&);    // 实例化
```
## decltype
* 在使用多数据类型模板中，可能无法确定数据类型
* 使用decltype(x+y) xpy = x + y;即可完成不同类型x,y相加
* 使用decltype((xx)) r = xx;则r为xx类型的引用
* 如果无法确定返回值，则可以使用后置返回(auto func_name(paras) -> type)
```c++
template <typename t1, typename t2>
auto add(t1 a, t2 b) -> decltype(a+b)
{
  return a + b;
}
```

---

# **引用&**
* *被引用的两个变量会指向同样的地址，值也是共享的*
* *必须在声明时就初始化*
```c++
int rat;
int & rodent = rat;
// "int & rodent; rodent=rat;" is invalid
```
+ 可以通过初始化声明来设置引用，但是不能通过赋值来设置
```c++
// 最后p指向bunnies，而rodent仍为rats的引用，为100
int rats = 100;
int * p = &rats;
int & rodent = *p; // rodent与*p共用地址和内容，此后p改变并不影响rodent
int bunnies = 50;
p = &bunnies;
```
+ 可以通过指针或者引用来实现函数对main中数据的改变
+ 在返回引用的时候要注意，千万不能返回一个临时变量！要么返回new过的，要么返回传入的引用参数
+ 由于返回的是引用，所以可以作为左值和右值，如果不想函数作为左值，则需要在返回值处加入const
+ *数组->指针； 结构体->指针/引用； 类->引用*

---

# **内存模型和名称空间**
## 编译
* 使用自己编写的头文件时，用""，用系统自带的头文件时，用<>
* 头文件一般包括：#define、函数声明、内联函数、const常量、结构体声明、类声明
* 在编写头文件时保留好习惯
```c++
#ifndef NAME_H_
#define NAME_H_
// place include file contents here
#endif
```
- 源文件(.cpp) ---编译---> 目标文件(.obj) ---链接---> 可执行文件(.exe)
    - 编译：将C++代码翻译成机器代码
    - 链接：将目标文件、操作系统启动代码和库文件整合成可执行文件
## 作用域和链接性
- 作用域：定义的变量有效的范围
    - 自动持续变量：常见的定义的变量，执行到代码结尾则失去活性，存储在栈中
    - 静态持续变量
        - 未初始化的静态变量均为0
        - static变量无论是否使用，都驻留在内存中，更新时只会更新对应位置的变量
        - 结构体中可用mutable关键词代表const结构体中仍可以改变的成员
    - 函数使用static将被赋予单文件的作用域；不使用关键字或external都被赋予多文件作用域
```c++
int global = 1000;    // 外部链接性，其他文件可通过external访问
static int global_static = 100;   // 内部链接性，只在该文件里有用该文件该文件文件
void function()
{
  void static int local_static = 10;  // 只在函数内有用
}
```
```c++
struct student
{
  string name;
  mutable int age;
}
const student tom;
tom.age ++;        //仍被允许
```
* 链接性
## 名称空间
- 名称空间接名称空间可以是全局的，也可以位于另一个名称空间中，但不能位于代码块中
- using声明——指出能够使用namespace中特定值; using编译——使用namespace所有内容
- using声明更加安全
```c++
namespace tom
{
  string name = "tom";
}
int main()
{
  using namespace tom;  // 使用using编译不报错，但是使用using tom::name则会因为冲突而报错
  string name = "jack";
  cout<< name;          // 输出为jack，局部变量会隐藏全局变量
}
```
- 名称空间可以嵌套
- 可以使用=对名称空间进行命名
```c++
namespace student
{
  namespace tom
  {
    ...
  }
}
```
- namespace TOM = student::tom;则TOM为tom的别名

---

# **类Class**
## 类定义
```c++
class Stock
{
  private:
    char company[30];
    int shares;
    double share_val;
    double total_val;
    void set_tot() {total_val = shares * share_val;}
  public:
    void show();
    void update(double price);
    ...
}
```
* 实现类函数——通过className::funcName来标识类函数，同时类函数可以访问private组件
```c++
void Stock::show()
{
  ...  // 可以使用Stock内的private成员或方法
}
inline void Stock::set_tot()  // 内联函数除了在类声明时定义，也可以在后面定义
{
  ...
}
```
* 不同的对象占用不同的内存空间，但调用的方法是公共的。比如tom.show()和jack.show()都共用Stock::show()，但是两者调用的数据位于不同的内存区域
## 构造函数
- 构造函数与类名相同
- 构造函数没有返回类型，用于初始化类中的private成员，当代码中创建对象的时候会自动调用合适的构造函数进行初始化
- 构造函数的形式参数不能和类的成员同名，否则会有歧义，建议以_结尾作为类的成员名，如name_, age_
- 构造函数的使用
```c++
Stock garment = Stock("Furry Mason", 50, 2.5);
Stock food("World Cabbage", 250, 1.25);
Stock *pstock = new Stock("Electro Games", 18, 1.9);
```
- 在对一个对象进行赋值操作时，会先创建一个临时对象，然后进行值的复制，再将临时变量删除。如果既可以初始化生成对象，又可以赋值生成时，建议使用初始化。
```c++
Stock s1 = Stock("JC", 50, 0.5);  // 使用初始化
Stock s2;                         // 调用默认初始化
// 下面都是赋值操作，不推荐使用
// s2= Stock("KTV", 100, 1.8);
// s2 = s1;
```
- 在构造的时候如果是new不定长度的变量，那么在遇到只需要一个长度的变量时，请使用new[]，即如下，这样可以只需要再析构函数中写delete[]即可
```c++
str = new char[1];    // 而不是str = new char
```
- 如果有多种构造函数，一定要注意使用相同的new的方式，如都为new或都为new[]
- **默认构造函数**
    - 在不传入任何参数的时候，将自动default一些值(*隐式调用默认构造函数时不要加括号！！！！*)
    - 有参数的构造函数进行default设置
    - 新建无参数的构造函数
```c++
Stock::Stock()
{
  company = "";
  share = 0;
  share_val = 0.0;
}
// 或者是在已有的有参数构造函数中给所有形参设置default
// 正确使用默认构造函数
Stock first;
Stock first = Stock();
Stock *pfirst = new Stock;
// 错误使用默认构造函数，下面语句意为second函数返回值为Stock
Stock second();
```
- 列表初始化(*推荐使用，这样调用默认构造函数时不会出错*)
```c++
Stock first {};                  // 调用默认构造函数
Stock second = {"", 100, 0.5};
```
## 析构函数
- 在类名前加上~，即~ClassName()即为析构函数
- 析构函数在对象从内存上被删除时自动调用，不需要用户手动调用
- 如果构造函数使用了new，则析构函数要记得使用delete / delete[]
## const成员函数
```c++
private: 
  const int i;
  ...
public:
  // void show(){std::cout<<i;}此时会报错，因为不能保证函数不修改const int
  void show() const;  // 此时不会报错，这样是const成员函数，表明不会更改成员
```
## this指针
* this指向调用对象的地址
```c++
const double topval(const Stock& s) const
{
  if(s.val>val) return s.val;
  else          return this->val;    // 时刻注意this是指针而不是值
}
```
## 运算符重载
* 运算符重载格式
```c++
returnType operator op(Type params);
```
* 在使用op时，左侧的是调用的对象，右侧的为被作为参数的对象，如果使用成员函数重载，则左侧必为用户定义的类，即a op b == a.operator(b)
- 当使用友元函数重载时，a op b == operator(a, b)
- 运算符重载限制
    - 不能修改优先级
    - 不能创建新的操作符
    - 至少有一个操作数是用户定义的类型(否则会使原有的含义出现歧义)
    - =、[]、->、()这四个只能通过成员函数进行重载(即不能通过友元函数)~~
## 友元函数
- 友元函数能够访问类内数据，但是不属于类作用域
- 使用关键字friend，首先在类声明中声明，*再在类外进行定义，注意定义时不需要加上className::funcName与friend关键字!!*
```c++
class Stock
{
  ...
  public:
    friend Stock operator *(double a, const Stock& t);
}
Stock operator *(double a, const Stock& t)
{
  return t*a;    // 此处已经使用*运算符成员函数重载
}
```
- 如果操作数是一个类元素一个非类元素，通常要使用成员函数重载运算符+友元函数翻转运算符顺序
- 如果友元函数访问两个不同的类，并不意味着需要同时成为两个类的友元，该函数访问哪个类的数据，就需要成为哪个类的友元
- <<操作符重载(*注意该函数为用户类的友元*)
```c++
// 返回值为ostream&能实现多个<<连续使用
ostream& operator <<(ostream& os, className& C)
{
  os<<输出内容;
  return os;
}
```
## 类的自动类型转换和强制类型转换
- 只有一个参数的构造函数可以作为转换函数(*如果有多个参数，但其他参数均有default也可以*)
- 使用explicit作为限制，转换函数只能显式转换而不能隐式转换
```c++
// 有构造函数Stone(int, double) Stone(double) Stone()
Stone myCat;    // 调用隐式构造函数
myCat = 15.4;   // 隐式类型转换调用Stone(double)，在explicit下不行
myCat = Stone(18.5);  // 显式类型转换
myCat = (Stone) 16.4; // 显式类型转换
```
- 如果存在Stone(long)，则上述代码报错，因为存在二义性
- 如果要实现将类类型强制转换成普通类型，则需要使用转换函数
```c++
operator double() const;  // 转换函数在类内声明，无返回值和形参
double a = double(Stone);
```
## 动态内存分配
- 如果在类声明中定义了static类型成员变量，则需要在调用该类的文件的公共区域进行初始化(注意，初始化时并未使用static关键字，且使用了类作用域符)
```c++
/* filename = mystring.h */
class String
{
  ...
  static int num;    // 此处不能初始化，因为声明并不分配内存空间
}
/* file2 */
#include "mystring.h"
int String::num = 0;
int main(){...}
```
##### 复制构造函数
* 当新建一个对象并将其初始化为同类现有对象的时候调用复制构造函数
```c++
// 如果motto是StringBad类，则下列均会调用复制构造函数
StringBad ditto(motto);
StringBad ditto = motto;
StringBad ditto = StringBad(motto);
StringBad* p = new StringBad(motto);
```
* 由于按值传递会引起复制构造函数，所以一般使用按引用传递
* 引起复制构造函数的三种情况：
    * 以传值的方式送入函数作为参数
    * 以传值的方式return
    * 以赋值的方式初始化另一个对象
###### 浅复制
* 逐个复制非静态成员，复制的是成员的值
* 对于new过的部分，浅复制会导致两个类指向同一个内存空间，而当调用析构函数后，该内存空间会被释放，此时另一个类将指向内容不定的区域

![浅复制.png](https://i.loli.net/2020/10/07/lhVa9xTMS6Pq3vH.png)

###### 深复制
* 创建一个字符串的备份，这样两个类指向的内存空间不同

![深复制.png](https://i.loli.net/2020/10/07/Lid1sJ92e7ZPtwa.png)

* 如果类中使用了new初始化成员，则要定义复制构造函数，进行深复制
* 对赋值运算符也要进行重新定义，但是与复制构造函数有差别——(1) 由于被赋值对象可能已有值，所以要先delete[]; (2) 避免赋值给自身，否则先delete[]后就没法赋值了; (3) 函数要返回对象的引用，便于连续赋值.
```c++
StringBad& StringBad::operator =(StringBad& s)
{
  if(this == &s)    // 检查是否是对自身赋值
    return *this;
  delete[] str;     // 删除原有的指向内存 
  len = s.len;
  str = new char[len+1];
  std::strcpy(str, s.str);    // 进行深复制
  return *this;
}
```
###### 赋值函数 Vs 复制构造函数 Vs 构造函数
* 如果对象不存在，且没有使用别的对象进行初始化 -> 构造函数
* 如果对象不存在，用别的对象进行初始化 -> 复制构造函数
* 如果对象存在，用别的对象进行赋值 -> 赋值函数
## 静态成员函数
- 函数声明包括static关键字，但是定义中不能出现static
- 不能通过对象调用静态函数，因为静态函数是属于这个类的所有对象的，而不是属于这个类
- 静态函数只能访问类中的静态变量
```c++
class String
{
  ...
  static int num;
  int len;
  ...
  static int Howmany() {return num;}  // 只能访问num不能访问len
}
int main()
{
  String a;
  String b;
  int n = String::Howmany();  // 不能使用a::Howmany()这种
}
```
## 成员初始化列表
* 只能用于构造函数
* 必须用这种方法初始化非静态常量(非static的const)
* 必须用这种方式初始化引用数据成员
* 数据成员初始化的顺序与出现在类声明的顺序一致，与初始化器的排列顺序无关
```c++
Queue::Queue(int qs): qsize(qs), front(NULL), rear(NULL)
{
  ...
}
```
* 在C++11标准中，可以再声明const和引用变量时直接初始化，如果没有进行成员初始化，则使用该值，否则使用成员初始化列表中的值
```c++
class Queue
{
  ...
  const int qs = 0;    // 如果没使用Queue(int qs)初始化，则qs=0
  ...
}
Queue::Queue(int qs): qs(qs)  // 调用后qs = qs
{
  ...
}
```
## 继承
- 从已有的类派生出新的方法，但同时继承原有类(基类)的特征和方法
```c++
class TableTennisPlayer {...}
class RatedPlayer : public TableTennisPlayer  // 公有继承
{...}
```
- 继承的类需要添加属于自己的成员变量和函数，其可以继承基类的数据，但是只能通过基类函数来访问； 同时需要加入构造函数，同时初始化派生类成员和基类成员
```c++
class RatedPlayer : public TableTennisPlayer
{
  private:
    int rating;
  public:
    RatedPlayer(int r=0, const string& s="none");  // 两种方法同等
    RatedPlayer(int r, const TableTennisPlayer& tp);
}
// 要通过成员列表初始化才能进行对父类的初始化
RatedPlayer::RatedPlayer(int r=0, const string& s="none") : TableTennisPlayer(s)
{
  ...
}
// 如果不使用成员列表初始化，相当于调用父类的默认构造函数
//RatedPlayer::RatedPlayer(int r=0, const string& s="none"){...}
//RatedPlayer::RatedPlayer(int r=0, const string& s="none") : TableTennisPlayer(){...}
```
##### 派生类和基类的特殊关系:
* 派生类可以使用基类方法，条件是方法不是基类私有的
* 基类指针可以指向派生类；基类引用可以是派生类(基类能用的派生类都能用，反之不行)
##### 虚函数
* 由于基类（指针、引用）可作用于派生类，所以会产生歧义，而使用虚函数可以消除这些歧义
* 没有virtual——根据引用类型或指针类型选择方法；有virtual   ——根据引用和指针指向的对象选择方法
* 在基类中将派生类会重新定义的方法设置为virtual，这样在派生类中重定义时会自动生成virtual，但仍建议在派生类中协商virtual
* virtual关键字只用于类声明中，而不用于定义，在定义时通过className加以区分即可
* 如果某个函数在基类和派生类都定义了，则在派生类中运用基类方法时，一定要加入className加以区分，否则会产生递归
* 可以使用基类的指针数组，完成对基类和派生类的访问，通过虚函数自动根据对象进行函数选择，这就是多态性
* 使用虚析构函数，可以正确根据对象调用基类或派生类的析构函数，保证按照正确的顺序进行删除
* 虚函数表的工作原理:

![虚函数.png](https://i.loli.net/2020/10/11/FczQTEbfsDOHqSx.png)

* 构造函数不能是虚函数；基类析构函数应当为虚函数；友元函数不能为虚函数
##### 隐藏
* 在同一个class中定义同名但参数不同的函数是重载
* 在子类和父类中定义同名函数并不是重载（无论参数是否相同），而是会用子类隐藏父类
* 子类和父类中同名且参数相同的函数可以使用virtual来声明
* 返回类型协变——返回值中子类和父类的指针、引用是等价的，并不是隐藏
* 消除隐藏的方法是在子类中把父类被隐藏的函数再声明一遍（或virtual，保证参数完全相同）
##### public、private、protected继承
|    |public|private|protected|
|:----|:----|:----|:----|
|公有成员|公有|私有|保护|
|私有成员|不可直接访问|不可直接访问|不可直接访问|
|保护成员|保护|私有|保护|

* 保护继承——如果想让派生类了解基类，但又不想让外接知道类内成员，则使用保护
* 对数据采用private，对函数采用protected
## 抽象基类(ABC)
* 通常在ABC中声明暂时无法实现的函数为纯虚函数，类似于python中的pass
* 使用纯虚函数的原因就是为了在派生类中重新定义它
* 纯虚函数使用virtual关键字和=0表征
```c++
class Circle
{
  private:
    double x;
    double y;
    double r;
    ...
  public:
    ...
    void Move(int nx, int ny){x=nx; y=ny;}
    virtual double Area() const = 0;    // pure virtual function
    ...
} 
```
* 使用纯虚函数将A->B变为了A->B，A->C的关系，即ABC中存储BC公共的数据和纯虚方法，再在BC内定义所需的数据，并覆盖纯虚方法。可以将ABC看作一种必须实施的接口。
## 继承和动态内存分配
##### 派生类不使用new
* 由于不使用new，则不存在深复制和浅复制，也无需特殊的析构函数，赋值、复制时都能正确调用默认函数，所以基本上不需要做任何显式定义和声明
##### 派生类使用new
* 由于需要释放new的成分，需要显式定义析构函数；同时此时会出现浅复制，需要显式定义复制构造函数和赋值函数
* 析构函数——需要delete派生类的new的东西
* 复制构造函数——需要使用基类（复制）构造函数，可以使用成员列表初始化，将基类成员数据先初始化，再将派生的成员初始化
```c++
hasDMA::hasDMA(const hasDMA& hs) : baseDMA(hs)
{
  style = new char[std::strlen(hs.style)+1];  // style is char* in class
  std::strcpy(style, hs.stley);
}
```
* 赋值函数——需要使用基类的赋值函数，将基类成员赋值后，再赋值派生的成员
```c++
hasDMA& hasDMA::hasDMA(const hasDMA &hs)
{
  if(this==&hs)
    return *this;
  baseDMA::operator=(hs);    // new line
  delete[] style;
  style = new char[std::strlen(hs.style)+1];
  std::strcpy(style, hs.stley);
  return *this;
}
```
##### 友元
* 如果一个函数是子类的友元，如何访问父类成员？（使用baseDMA的同名函数）
* 由于友元无法使用className，如何使用baseDMA函数？（使用强制类型转换）
## 类设计小结
##### 三种默认函数
* 默认构造函数
* 默认复制构造函数（在使用new时需要显式定义）
* 默认赋值函数（在使用new时需要显式定义）
##### 按值传递和按引用传递
* 推荐按引用传递
* 按值传递会引起复制构造函数，可能导致浅复制；在派生类中使用虚函数，父类的引用可以运用到派生类
##### 返回引用和返回值
* 如果返回临时变量，则返回值
* 如果返回传入的引用、new的空间等，使用返回引用，避免调用复制构造函数或者赋值函数
##### 继承
* 构造函数、析构函数、赋值函数不能被继承
* 友元函数无法继承，但可以使用强制类型转换进行子类、父类的访问
* 可以将派生类赋给基类，但反之不一定可以，除非有转换函数
##### 类函数
|**函数**|**能否继承**|**成员还是友元**|**是否默认生成**|**能否为virtual**|**是否有return**|
|:----|:----|:----|:----|:----|:----|
|构造函数|否|成员|是|否|否|
|析构函数|否|成员|是|是|否|
|=|否|成员|是|是|是|
|&|是|任意|是|是|是|
|转换函数|是|成员|否|是|否|
|() [] ->|是|成员|否|是|是|
|友元|否|友元|否|否|是|


---
