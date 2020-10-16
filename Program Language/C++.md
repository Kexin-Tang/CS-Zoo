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
