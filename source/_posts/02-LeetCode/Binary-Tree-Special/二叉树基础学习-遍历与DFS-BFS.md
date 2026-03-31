---
title: 二叉树基础学习：遍历、DFS与BFS深度解析
date: 2026-03-30T21:09:00+11:00
tags: [二叉树, 遍历算法, DFS, BFS, 前序遍历, 中序遍历, 后序遍历, LeetCode, Python]
categories: 02-LeetCode/Binary-Tree-Special
---

# 二叉树基础学习：遍历、DFS与BFS深度解析

> 本文记录二叉树基础学习的第一天，涵盖二叉树遍历的三种顺序（前序、中序、后序）、DFS递归实现、BFS层序遍历，以及LeetCode相关题目解法。

## 1. 二叉树基础概念

### 二叉树定义
二叉树是每个节点最多有两个子节点的树结构，通常称为左子节点和右子节点。

```python
class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val
        self.left = left
        self.right = right
```

### 二叉树示例
```
       1
      / \
     2   3
    / \   \
   4   5   6
```

## 2. 深度优先遍历（DFS）的三种顺序

### 核心理解
**你的理解完全正确！** 遍历顺序取决于**对根节点的操作放在不同位置**：

```python
def traverse(root):
    if not root:
        return
    
    # 前序遍历：根 -> 左 -> 右
    # 在这里操作root
    
    traverse(root.left)   # 遍历左子树
    
    # 中序遍历：左 -> 根 -> 右  
    # 在这里操作root
    
    traverse(root.right)  # 遍历右子树
    
    # 后序遍历：左 -> 右 -> 根
    # 在这里操作root
```

### 2.1 前序遍历（Preorder Traversal）
**顺序**：根节点 → 左子树 → 右子树

#### 递归实现
```python
def preorderTraversal(root):
    result = []
    
    def dfs(node):
        if not node:
            return
        result.append(node.val)  # 先访问根节点
        dfs(node.left)           # 再遍历左子树
        dfs(node.right)          # 最后遍历右子树
    
    dfs(root)
    return result
```

#### 迭代实现（栈）
```python
def preorderTraversal_iterative(root):
    if not root:
        return []
    
    result = []
    stack = [root]
    
    while stack:
        node = stack.pop()
        result.append(node.val)  # 访问当前节点
        
        # 注意：先右后左，因为栈是LIFO
        if node.right:
            stack.append(node.right)
        if node.left:
            stack.append(node.left)
    
    return result
```

### 2.2 中序遍历（Inorder Traversal）
**顺序**：左子树 → 根节点 → 右子树

#### 递归实现
```python
def inorderTraversal(root):
    result = []
    
    def dfs(node):
        if not node:
            return
        dfs(node.left)           # 先遍历左子树
        result.append(node.val)  # 再访问根节点
        dfs(node.right)          # 最后遍历右子树
    
    dfs(root)
    return result
```

#### 迭代实现（栈）
```python
def inorderTraversal_iterative(root):
    result = []
    stack = []
    curr = root
    
    while curr or stack:
        # 遍历到最左边的节点
        while curr:
            stack.append(curr)
            curr = curr.left
        
        # 弹出栈顶节点并访问
        curr = stack.pop()
        result.append(curr.val)
        
        # 转向右子树
        curr = curr.right
    
    return result
```

### 2.3 后序遍历（Postorder Traversal）
**顺序**：左子树 → 右子树 → 根节点

#### 递归实现
```python
def postorderTraversal(root):
    result = []
    
    def dfs(node):
        if not node:
            return
        dfs(node.left)           # 先遍历左子树
        dfs(node.right)          # 再遍历右子树
        result.append(node.val)  # 最后访问根节点
    
    dfs(root)
    return result
```

#### 迭代实现（栈 - 反转前序遍历）
```python
def postorderTraversal_iterative(root):
    if not root:
        return []
    
    result = []
    stack = [root]
    
    while stack:
        node = stack.pop()
        result.append(node.val)  # 访问当前节点
        
        # 注意：先左后右，因为结果要反转
        if node.left:
            stack.append(node.left)
        if node.right:
            stack.append(node.right)
    
    return result[::-1]  # 反转得到后序遍历
```

## 3. 广度优先遍历（BFS / 层序遍历）

### 3.1 基础层序遍历（不知道层数）

```python
from collections import deque

def levelOrderTraverse(root):
    if not root:
        return
    
    q = deque()
    q.append(root)
    
    while q:
        cur = q.popleft()
        # 访问当前节点
        print(cur.val)
        
        # 把当前节点的左右子节点加入队列
        if cur.left:
            q.append(cur.left)
        if cur.right:
            q.append(cur.right)
```

**特点**：
- 按层遍历所有节点
- 不知道每个节点所在的层数
- 适用于只需要访问节点值的场景

### 3.2 知道层数的层序遍历

```python
from collections import deque

def levelOrderTraverse_with_depth(root):
    if not root:
        return
    
    q = deque()
    q.append(root)
    depth = 1  # 记录当前遍历到的层数（根节点视为第1层）
    
    while q:
        sz = len(q)  # 当前层的节点数
        for i in range(sz):
            cur = q.popleft()
            # 访问当前节点，同时知道它所在的层数
            print(f"depth = {depth}, val = {cur.val}")
            
            # 把当前节点的左右子节点加入队列
            if cur.left:
                q.append(cur.left)
            if cur.right:
                q.append(cur.right)
        depth += 1  # 进入下一层
```

**特点**：
- 知道每个节点所在的层数
- 通过`sz = len(q)`获取当前层的节点数
- 适用于需要按层处理节点的场景

### 3.3 使用State类记录深度信息

```python
from collections import deque

class State:
    def __init__(self, node, depth):
        self.node = node
        self.depth = depth

def levelOrderTraverse_with_state(root):
    if not root:
        return
    
    q = deque()
    # 根节点的深度是1
    q.append(State(root, 1))
    
    while q:
        cur_state = q.popleft()
        cur_node = cur_state.node
        cur_depth = cur_state.depth
        
        # 访问当前节点，同时知道它的深度
        print(f"depth = {cur_depth}, val = {cur_node.val}")
        
        # 把当前节点的左右子节点加入队列
        if cur_node.left:
            q.append(State(cur_node.left, cur_depth + 1))
        if cur_node.right:
            q.append(State(cur_node.right, cur_depth + 1))
```

**特点**：
- 每个节点存储对应的深度信息
- 可以处理深度不同的节点
- 适用于需要深度信息的复杂场景

## 4. LeetCode题目实战

### 4.1 144. 二叉树的前序遍历

#### 题目描述
给你二叉树的根节点 `root` ，返回它节点值的 **前序遍历**。

#### 递归解法
```python
class Solution:
    def preorderTraversal(self, root: Optional[TreeNode]) -> List[int]:
        result = []
        
        def dfs(node):
            if not node:
                return
            result.append(node.val)  # 前序：先访问根节点
            dfs(node.left)
            dfs(node.right)
        
        dfs(root)
        return result
```

#### 迭代解法
```python
class Solution:
    def preorderTraversal(self, root: Optional[TreeNode]) -> List[int]:
        if not root:
            return []
        
        result = []
        stack = [root]
        
        while stack:
            node = stack.pop()
            result.append(node.val)
            
            # 先右后左，保证左子树先被访问
            if node.right:
                stack.append(node.right)
            if node.left:
                stack.append(node.left)
        
        return result
```

### 4.2 94. 二叉树的中序遍历

#### 题目描述
给定一个二叉树的根节点 `root` ，返回它的 **中序遍历**。

#### 递归解法
```python
class Solution:
    def inorderTraversal(self, root: Optional[TreeNode]) -> List[int]:
        result = []
        
        def dfs(node):
            if not node:
                return
            dfs(node.left)           # 中序：先遍历左子树
            result.append(node.val)  # 再访问根节点
            dfs(node.right)          # 最后遍历右子树
        
        dfs(root)
        return result
```

#### 迭代解法
```python
class Solution:
    def inorderTraversal(self, root: Optional[TreeNode]) -> List[int]:
        result = []
        stack = []
        curr = root
        
        while curr or stack:
            # 遍历到最左边的节点
            while curr:
                stack.append(curr)
                curr = curr.left
            
            # 弹出栈顶节点并访问
            curr = stack.pop()
            result.append(curr.val)
            
            # 转向右子树
            curr = curr.right
        
        return result
```

### 4.3 145. 二叉树的后序遍历

#### 题目描述
给你一棵二叉树的根节点 `root` ，返回其节点值的 **后序遍历**。

#### 递归解法
```python
class Solution:
    def postorderTraversal(self, root: Optional[TreeNode]) -> List[int]:
        result = []
        
        def dfs(node):
            if not node:
                return
            dfs(node.left)           # 后序：先遍历左子树
            dfs(node.right)          # 再遍历右子树
            result.append(node.val)  # 最后访问根节点
        
        dfs(root)
        return result
```

#### 迭代解法（反转前序遍历）
```python
class Solution:
    def postorderTraversal(self, root: Optional[TreeNode]) -> List[int]:
        if not root:
            return []
        
        result = []
        stack = [root]
        
        while stack:
            node = stack.pop()
            result.append(node.val)
            
            # 先左后右，因为结果要反转
            if node.left:
                stack.append(node.left)
            if node.right:
                stack.append(node.right)
        
        return result[::-1]  # 反转得到后序遍历
```

## 5. 遍历顺序对比与记忆技巧

### 遍历顺序总结
| 遍历方式 | 访问顺序 | 递归代码位置 | 应用场景 |
|---------|---------|-------------|---------|
| 前序遍历 | 根 → 左 → 右 | 递归函数开头 | 复制二叉树、序列化 |
| 中序遍历 | 左 → 根 → 右 | 递归函数中间 | 二叉搜索树排序 |
| 后序遍历 | 左 → 右 → 根 | 递归函数结尾 | 删除二叉树、计算高度 |

### 记忆技巧
1. **"前"序遍历**：先访问**根**节点（前=前面）
2. **"中"序遍历**：根节点在**中间**访问
3. **"后"序遍历**：最后访问**根**节点（后=后面）

### 代码位置记忆
```python
def traverse(root):
    if not root:
        return
    
    # 前序遍历：在这里操作
    traverse(root.left)
    # 中序遍历：在这里操作  
    traverse(root.right)
    # 后序遍历：在这里操作
```

## 6. DFS与BFS对比

### 深度优先搜索（DFS）
**特点**：
- 使用栈（递归或显式栈）
- 沿着树的深度遍历节点
- 适合路径查找、回溯问题
- 空间复杂度：O(h)，h为树高

**适用场景**：
- 需要遍历所有路径
- 查找特定路径
- 树的序列化

### 广度优先搜索（BFS）
**特点**：
- 使用队列
- 按层遍历节点
- 适合最短路径、层相关问题
- 空间复杂度：O(w)，w为树的最大宽度

**适用场景**：
- 层序遍历
- 最短路径问题
- 按层处理节点

## 7. 常见问题与解答

### Q1：为什么中序遍历对二叉搜索树特别有用？
**A**：二叉搜索树的中序遍历结果是有序的。对于BST，中序遍历可以得到升序序列。

### Q2：如何选择DFS还是BFS？
**A**：
- 需要**最短路径**或**按层处理** → 选择BFS
- 需要**遍历所有可能路径**或**深度相关** → 选择DFS
- 树很深但宽度不大 → DFS更节省空间
- 树很宽但深度不大 → BFS更合适

### Q3：三种遍历方式的时间复杂度是多少？
**A**：都是O(n)，其中n是节点数。每个节点只访问一次。

### Q4：迭代和递归实现哪个更好？
**A**：
- **递归**：代码简洁，容易理解，但可能栈溢出（树很深时）
- **迭代**：更可控，不会栈溢出，但代码稍复杂
- 实际中根据具体情况选择

## 8. 扩展学习

### 8.1 莫里斯遍历（Morris Traversal）
空间复杂度O(1)的中序遍历算法，利用线索二叉树。

### 8.2 迭代统一写法
三种遍历的迭代统一实现，通过标记法。

### 8.3 N叉树的遍历
将二叉树遍历扩展到N叉树。

## 9. 练习题

### 基础练习
1. 实现二叉树的三种遍历（递归+迭代）
2. 实现层序遍历的三种变体
3. 计算二叉树的高度（使用后序遍历）

### LeetCode进阶
1. **102. 二叉树的层序遍历** - 按层返回结果
2. **107. 二叉树的层序遍历 II** - 自底向上层序遍历
3. **103. 二叉树的锯齿形层序遍历** - Z字形遍历

## 10. 学习总结

### 今日掌握要点
✅ **二叉树三种遍历顺序**：前序、中序、后序  
✅ **遍历顺序核心**：根节点操作位置决定遍历顺序  
✅ **DFS递归实现**：简洁明了，理解递归思想  
✅ **BFS层序遍历**：三种实现方式，适应不同需求  
✅ **LeetCode实战**：完成144、94、145三道基础题  

### 关键理解
1. **遍历的本质**：以不同的顺序访问树中的节点
2. **递归思想**：将问题分解为子问题（左子树、右子树）
3. **栈与队列**：DFS用栈（LIFO），BFS用队列（FIFO）
4. **空间复杂度**：DFS与树高相关，BFS与树宽相关

### 下一步学习方向
1. **二叉搜索树**：利用中序遍历特性
2. **树的构建**：根据遍历结果重建二叉树
3. **树的高度/深度**：后序遍历的应用
4. **路径问题**：DFS在路径查找中的应用

---

*通过今天的学习，你已经掌握了二叉树遍历的核心概念。记住：遍历顺序取决于对根节点的操作位置，这是理解三种遍历方式的关键。继续练习，将这些知识应用到更多二叉树问题中！*