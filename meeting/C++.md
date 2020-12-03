所有普通类内成员函数都不能作为函数指针的参数，因为在编译阶段，原函数的定义会发生变化
```c++
class Solution{
bool comp(int a, int b) { return a<b; }   // 在编译后会变成 bool comp(Solution* this, int a, int b)!!
...
}
```


capacity Vs size
```
capacity返回的是容器的大小
size返回的是元素的个数
一般而言，capacity>=size，即水杯>=水的量
```
