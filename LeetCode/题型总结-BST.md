# BST

遍历BST常见的有三种方法

## 递归

递归是非常简单和直白的方法，时间复杂度 O(N)，空间复杂度 O(H)。

```python
def inorder(root):
    def dfs(node):
        if node:
            dfs(node.left)
            print(node.val)
            dfs(node.right)

# 以下是更简单的写法
def inorder(r: TreeNode) -> List[int]:
    return inorder(r.left) + [r.val] + inorder(r.right) if r else []
```

## stack

使用栈是一样的，就是模拟了递归方法，好处是因为使用循环，所以可以在到达某个位置后停下来，处理后再根据条件决定是否进行下一步（比如说实现iter迭代器）。时间复杂度 O(N)，空间复杂度 O(H)。

```python
def inorder(root):
    stack = []
    cur = root
    while cur==root or len(stack)>0:
        while cur:          # 走到最左侧（当前分枝最小值）
            stack.append(cur)
            cur = cur.left
        cur = stack.pop()
        print(cur.val)
        cur = cur.right     # 走到右边
```

## Morris

Morris方法的时间复杂度是 O(N)，空间复杂度是 O(1)，其思想是利用了叶子结点的 None 指针，让他们指向下一个应该去往的位置。

首先我们定义一个 `cur` 代表着现在的位置，然后使用 `pre` 去寻找到当前位置前头的节点，然后让 `pre` 的某个 None 节点指向 `cur`。

我们使用第一遍遍历去构建整个树，让叶子结点的None指针指向下一个该去的位置。第二遍遍历去得到内容，然后并修改指针回到原来的样子。

```python
def inorder(root):
    cur = root
    while cur:
        if cur.left is None:    # 找到了最左侧
            print(cur.val)
            cur = cur.right
        else:
            pre = cur.left
            while pre.right and pre.right!=cur: # 寻找小于cur的最大值（左子树的最右侧）
                pre = pre.right

            # 如果下一个不是指向cur，那么意味着这个节点还在第一个phase，需要构建这个树
            if pre.right != None:   
                pre.right = cur # 设置指向
                cur = cur.left  # cur移动，进入下一轮
            # 此时pre指向了小于cur的最大值，如果pre的下一个是cur，意味着在phase 2
            # 我们需要断开连接恢复结构，然后输出当前值，再去往右侧
            else:                   
                pre.right = None
                print(cur.val)
                cur = cur.right
```

---

# 类似题型

* [538. Convert BST to Greater Tree](https://leetcode.com/problems/convert-bst-to-greater-tree/)
* [99. Recover Binary Search Tree](https://leetcode.com/problems/recover-binary-search-tree/)