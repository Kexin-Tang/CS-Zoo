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
