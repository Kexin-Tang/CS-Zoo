# list
* 多维list初始化
```py
l = [[0 for i in range(n)] for j in range(m)]   # 正确
t = [[0]*n]*m   # 错误，改变一个元素会改变一整行或一整列
```

* 翻转 -- `list.reverse()`或`list[:,:,-1]`

# dict
* 排序
```py
sort(dict.keys(), reverse=False)
sort(dict.items(), key=lambda item: item[1])
```
* 初始化
```py
# 如果key(i)不存在，则设为0，否则获取其value
data = {}
data.setdefault(i, 0)
```


# 无穷
```py
infinite = float("inf")
```

# 判断类型
* 使用`isinstance()`
```py
l = []
isinstance(l, list)         # true
isinstance(l, [int, str])   # false
```

# 堆(heapq)模块

* 基于pure python实现的小根堆
```py
class hp:
    def __init__(self):
        self.heap = []
    
    def _move_down(self, index=0):
        '''
        将堆顶位置的元素(值比较大)合理地向下移动插入到小根堆中
        '''
        n = len(self.heap)
        while index*2+1 < n:
            left = index*2+1
            right = index*2+2
            parent = index
            small = parent
            # 如果左边子节点更小，那么把标记标定在左子节点
            if self.heap[left] < self.heap[small]:
                small = left
            # 如果右子节点存在且更小，那么把标记标定在右子节点
            if right < n and self.heap[right] < self.heap[small]:
                small = right
            # 如果左右节点都比当前的大，那么找到了合适的位置，退出调整
            if small == parent:
                break
            
            self.heap[index], self.heap[small] = self.heap[small], self.heap[index]
            index = small
    
    def _move_up(self, index):
        '''
        将堆底位置的元素(值比较小)合理地向上移动插入到小根堆中
        '''
        while index>0:
            parent = (index-1)//2
            # 如果堆的上一层比该元素小，那么不用调整了
            if self.heap[parent] < self.heap[index]:
                break
            else:
                self.heap[parent], self.heap[index] = self.heap[index], self.heap[parent]   # 交换
                index = parent  # 更新到上一层
    
    def pop(self):
        '''
        弹出堆顶元素，并调整小根堆
        '''
        last = len(self.heap)-1
        self.heap[0], self.heap[last] = self.heap[last], self.heap[0]   # 交换堆顶和堆底，并把堆底弹出
        res = self.heap[last]
        self.heap.pop()
        self._move_down()   # 调整小根堆，把交换上去的栈顶调整下来
        return res
    
    def push(self, x):
        '''
        向堆中插入一个元素并调整小根堆
        '''
        self.heap.append(x)
        self._move_up(len(self.heap)-1)
```

* 使用`heapq`库<b>默认为小顶堆</b>
> | 函数 | 功能 |
> | :---: | :---: |
> | heappush(heap, x) | 将x压入堆 |
> | heappop(heap)   | 弹出堆顶 |
> | heapify(list)   | list&rarr;heap |
> | heapreplace(heap, x) | 弹出堆顶并压入x<br>heappop + heappush |
> | nlargest(n, iter) | 返回iter中n个最大元素 |
> | nsmallest(n, iter) | 返回iter中n个最小元素 |

# collections
* 使用`defaultdict`使默认字典的键和值可以不存在
```py
from collections import defaultdict
dict1 = defaultdict(int)    # { : 0}
dict1[1] += 1               # {1: 1}

dict2 = defaultdict(list)   # {       : [      ]}
dict2["name"].append("tom") # {"name" : ["tom",]}
```

* 使用`Counter`可以计算列表中各种元素的个数，并按照从多到少进行排列
```py
from collections import Counter
data = ['tom', 'tom', 'jack', 'mary', 'tom', 'mary']
cnt = Counter(data) # {'tom':3, 'mary':2, 'jack':1}
```

* 使用`deque`可以将list&rarr;双端队列
```py
from collections import deque
l = deque([1,2,3])

l.appendleft(0) # [0,1,2,3]
l.append(4)     # [0,1,2,3,4]

l.popleft()     # [1,2,3,4]
l.pop()         # [1,2,3]
```

# itertools
* `combinations`生成的子列表不重复，第一个参数为iterable对象，第二个参数为子列表的长度
```py
from itertools import combinations
data = list(combinations('ABC',2)) # ['AB', 'AC', 'BC']
```
* `permutations`生成的子列表包括重复，第一个参数为iterable对象，第二个参数为子列表的长度
```py
from itertools import permutations
data = list(permutations('ABC',2)) # ['AB', 'AC', 'BC', 'BA', 'CA', 'CB']
```

# bisect
* bisect采用二分查找法返回排序数组指定值的下标，如果指定值不存在，返回应该插入的下标位置<b>注意，使用二分查找要求序列有序</b>
    * `bisect.insort(list, data)` -- 在List中二分插入data
    * `bisect.bisect(list, data)` -- 如果data在list中存在，则返回其位置；如果data不在List中存在，则返回data应该插入的位置

<i>(上述为函数都有两个后缀为`_left`和`_right`的函数，用于处理出现相同元素时，是插入左侧还是插入右侧)</i>

# 全局变量
* 对于可变变量(list, dict, set)，可以直接视为全局
* 对于不可变变量(int, string, tuple)，需要使用关键字`global`，且必须注意<b>不可用于递归</b>

# ord & chr
* `ord()`返回ASCII码
* `chr()`返回ASCII符号