
# 概述
本项目主要总结自己在使用C++刷leetcode的过程中，经常用到的操作、函数和方法


# C++
### 函数
<table>
    <tr>
        <td>类型</td> 
        <td>函数</td> 
        <td>用法</td>
   </tr>
    <tr>
        <td rowspan="4">字符串函数</td>    
        <td>stoi(string)</td>  
        <td>将输入字符串转换为int</td>
    </tr>
    <tr>  
        <td>to_string(val)</td>  
        <td>将输入内容转换为string</td> 
    </tr>
    <tr>
        <td>isalpha, isdigit, isalnum</td>
        <td>检查是否为字母、数字等</td>
    </tr>
    <tr>
        <td>equal(it1.begin(), it1.end(), it2.begin(), ...)</td>
        <td>比较两个迭代器内容和长度，均相同才返回true</td>
    </tr>
    <tr>
        <td rowspan="6">查找函数</td>
        <td>lower_bound(it.begin(), it.end(), val)</td>
        <td>查找第一个不小于val的值</td>
    </tr>
    <tr>
        <td>upper_bound(it.begin(), it.end(), val)</td>
        <td>查找第一个大于val的值</td>
    </tr>
    <tr>
        <td>find(it.begin(), it.end(), val)</td>
        <td>查找第一个等于val的值，没有找到则返回it.end()</td>
    </tr>
    <tr>
        <td>find_if(it.begin(), it.end(), func)</td>
        <td>查找符合func中定义的值</td>
    </tr>
    <tr>
        <td>find_end(it.begin(), it.end(), subit.begin(), subit.end())</td>
        <td>查找一个子序列，并返回最后出现的位置</td>
    </tr>
    <tr>
        <td>search(it.begin(), it.end(), subit.begin(), subit.end())</td>
        <td>查找一个子序列，并返回首次出现的位置</td>
    </tr>
    <tr>
        <td rowspan="4">修改函数</td>
        <td>reverse(it.begin(), it.end())</td>
        <td>用于将数据进行翻转</td>
    </tr>
    <tr>
        <td>容器.pop/push_back()</td><td>在尾部插入/删除一个元素</td>
    </tr>
    <tr>
        <td>容器.insert()</td>
        <td>在指定位置插入指定的内容</td>
    </tr>
    <tr>
        <td>replace(it.begin(), it.end(), oldVal, newVal)</td>
        <td>将迭代器内的oldVal替换为newVal</td>
    </tr>
    <tr>
        <td rowspan="3">排序函数</td>
        <td>容器.sort(it.begin(), it.end())</td>
        <td>进行排序，默认为升序</td>
    </tr>
    <tr>
        <td>容器.is_sorted(it.begin(), it.end())</td>
        <td>是否按照升序排列好了</td>
    </tr>
    <tr>
        <td>nth_element(it.begin(), nth, it.end())</td>
        <td>将第n小的数据找到并放在对应的位置，保证前面的数据都比他小，后面的数据都比他大</td>
    </tr>
    <tr>
        <td rowspan="6">删除函数</td>
        <td>remove(it.begin(), it.end(), val)</td>
        <td>* 是链表的话，则断开指针连接<br/>* 是数组(vector)的话，使用将后面的元素前挪来覆盖，并不会改变长度或删除元素，需要配合erase使用</td>
    </tr>
    <tr>
        <td>容器.erase(it.pos1, it.pos2)</td><td>删除数据</td>
    </tr>
</table>



##### 字符串函数
* stoi
```c++
// input        ->  输入的字符串
string s="12345";
stoi(s);    // "12345" -> 12345
```
* to_string
```c++
int a=100;
to_string(a);   // 100 -> "100"
```

* isalpha(char c)<br/> isdigit(char c)<br/> isalnum(char c)

* equal()
```c++
vector<string> arr1={"one", "two", "three"};
vector<string> arr2={"two", "three", "four"};
// 三参数equal
cout<< equal(arr1.begin()+1, arr1.end(), arr2.begin()); // 返回为true，因为前一个迭代器只有两个数据，所以该equal函数只比较两个元素长度，即arr2只有前两个元素参与比较
cout<< equal(arr1.begin(), arr1.end(), arr2.begin()+1); // 返回为false，因为前者长度比后者长，必定错误

// 四参数equal
cout<< equal(arr1.begin()+1, arr1.end(), arr2.begin(), arr2.end()); // false，两个指定区间长度不同，且内容也不同
```

##### 查找函数
* bound家族
> * lower_bound (itBegin, itEnd, val)
>> ```c++
>> // 在范围[begin, end)中，查找不小于val的第一个值，不存在则返回it.begin()
>> ForwardIterator lower_bound (ForwardIterator first, ForwardIterator end, const T& val)
>> ```
> * upper_bound(itBegin, itEnd, val)
>> 在set和map中有lower_bound()和upper_bound()的成员函数，效率更高。即set.lower_bound(val)
* find家族
> * find()
>> ```c++
>> // 由于底层代码是由==实现的，所以迭代器必须对操作符==有效
>> vector<int> arr={1,10,50,42,31};
>> vector<int> it;
>> it = find(arr.begin(), arr.end(), 22);
>> if(it==arr.end())
>> {
>>     cout<< "没有该元素"<< endl;
>> }
>> ```
> * find_if()
>> ```c++
>> InputIterator find_if (InputIterator first, InputIterator last, UnaryPredicate pred);
>> ```
> * find_if_not()

> * find_end()
>> 查找子序列在序列中出现的最后一个位置
> * search()
>> 查找子序列在序列中出现的第一个位置


##### 修改函数
* reverse(it.begin(), it.end())
* pop/push_back()
* insert家族
> * vec.insert(vec.begin(), val)
>> 在头部插入一个val
> * vec.insert(vec.end(), arr.begin(), arr.end())
>> 在vec的后面插入arr，即生成vec+arr


* replace家族
> * replace(it.begin(), it.end(), oldVal, newVal)
>> 将会把迭代器区间内的oldVal替换为newVal
> * replace_copy(it.begin(), it.end(), itNew.begin(), oldVal, newVal)
>> 将会把迭代器区间内容进行替换，并保存到新的容器
> * relpace_if(it.begin(), it.end(), func, newVal)
>> 将会把func为True的内容进行替换
##### 排列函数
* 容器.sort()
```c++
// 第三个参数可以不用，default为升序
void sort (RandomAccessIterator first, RandomAccessIterator last, Compare comp);
```
> * 提供了stable_sort进行稳定性排序
> * 提供了partial_sort (first, middle, last)， 从 [first,last) 范围内，筛选出 middle-first 个最小的元素并排序存放在 [first，middle) 区间中 
* 容器.is_sorted()
* nth_element()
```c++
vector<int> arr={1,2,3,4,5};
nth_element(arr.begin(), arr.begin()+3, arr.end());
// 最终输出可能为2,1,3,5,4，只要3对了就OK
```

##### 删除函数
* erase家族
```c++
string s="hello world";
s.erase(5);     // 删除第5个位置之后的所有内容
s.erase(5, 2);  // 删除第5个位置之后的2个内容
s.erase(s.begin()+5); // 当传入iterator时，删除iterator所指向的内容（一个值）
s.erase(s.begin(), s.begin()+5);    // 删除iterator间的内容
```
* remove家族
> * remove(it.begin(), it.end(), val)
>> ```c++
>> list<int> a={1,6,0,3,4,2,0};
>> vector<int> v={1,6,0,3,4,2,0};
>> // 下面两种方法结果一样
>> // list是双向链表，可以使用.remove(val)
>> // vector是数组类型，使用remove(it, it, val)时，会将后面的元素挪上前覆盖原来的参数，但是并不会删除元素，即vector长度仍不变，那么最后的数据将会被保留
>> a.remove(0);
>> v.erase(remove(v.begin(), v.end(), 0), v.end());
>> ```
>>> 如果vector为{1,2,0,4,0,5}，那么不使用erase直接使用remove将会导致1,2,4,5,(0,5)最后两位被保留，没有删除，因此vector长度仍不变
> * remove_copy(it.begin(), it.end(), itNew.begin(), val)
>> 会将原数据删除，并复制到一个新的容器之中

:top:[Back to top](#概述):top:

---

### 类

类名 | 作用
:---: | :---:
[stringsteam](#stringsteam) | 完成任意类型的转换
[unordered_map](#unordered_map) | hash表实现快速查找
[struct](#struct) | 结构体相关
[queue](#queue) | 队列相关


#### stringsteam

```c++
#include <sstream>
#include <iostream>
#include <string>

int main()
{
    using namespace std;
    string as="100";
    int a;
    int b = 200;
    stringstream itos;
    itos<< as;  // 将数据放入stream中
    itos>> a;   // 以另一种格式读取出来
    cout<< a+b<< endl;  // 输出300
}
``` 
#### unordered_map

* it->first == key; it->second == second
* map[key] = value
* find()用于查找key,没找到返回map.end()
```c++
#include <unordered_map>
...

vector<int> twoSum(vector<int>& nums, int target)
{
    unordered_map<int, int> num_map;
    for(int i=0; i<nums.size(); i++)
    {
        auto it = num_map.find(target-nums[i]);
        
        if(it!=num_map.end())
            return vector<int> {it->second, i};
        else
            num_map[nums[i]] = i;
    }
    return vector<int> {};
}
```

#### struct

与C语言中只能设置结构体内成员不同，在C++中可以和类一样，声明构造函数，直接调用构造函数即可
```c++
struct Node
{
    int val;
    Node* next;
    Node() : val(0), next(nullptr) {}   // 注意，不能用";"
    Node(int v) : val(v), next(nullptr) {}
    ...
}

int main()
{
    Node a = Node();
    Node* pb = new Node(2);
}
```

#### queue

###### 普通队列

名称 | 功能
:---: | :---:
queue.push(x) | 将x插入队列尾部
queue.pop() | 将队首弹出（不返回值）
queue.size() | 队列大小
queue.front() / queue.back() | 返回队首/队尾

###### 优先队列 priority_queue

名称 | 功能
:---: | :---:
queue.top | 返回队首
queue.size() | 返回数据大小
queue.empty() | 返回bool指明是否为空
queue.push(x) / queue.pop() | 插入/弹出数据(*对优先队列的数据进行增删时，会自动进行排序!!*)
```c++
// 使用时注意，默认为大顶堆
// 函数原型：priority_queue<Type, Container, Functional>
priority_queue<int, vector<int>, less<int> > a;     // 大顶堆，队首大
priority_queue<int> b;                              // 与a相同
priority_queue<int, vector<int>, greater<int> > c;  // 小顶堆，队首小
```

:top:[Back to top](#概述):top:

---
