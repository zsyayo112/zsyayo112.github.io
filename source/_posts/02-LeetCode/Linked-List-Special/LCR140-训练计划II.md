---
title: LCR 140. 训练计划II - 找到倒数第k个节点的双指针技巧
date: 2026-03-29T12:18:00+11:00
tags: [链表, 双指针, 倒数第k个节点, 剑指Offer, LeetCode, Python]
categories: 02-LeetCode/Linked-List-Special
---

# LCR 140. 训练计划II - 找到倒数第k个节点的双指针技巧

> 本文分析LCR 140题"训练计划II"（剑指Offer 22题），这是19题"删除倒数第N个节点"的简化版本，只需要找到节点而不需要删除。

## 题目描述

### 题目来源
剑指Offer 22. 链表中倒数第k个节点

### 题目描述
给定一个头节点为 `head` 的链表用于记录一系列核心肌群训练项目编号，请查找并返回倒数第 `cnt` 个训练项目编号。

### 示例
```
输入：head = [2,4,7,8], cnt = 1
输出：8

输入：head = [1,2,3,4,5,6], cnt = 3
输出：4
```

### 说明
- `1 <= cnt <= 链表长度`

## 解题思路

### 核心思想
使用**快慢指针**技巧：
- 快指针 `fast` 先走 `cnt` 步
- 然后快慢指针 `slow` 和 `fast` 同步前进
- 当快指针到达末尾时，慢指针正好在倒数第 `cnt` 个节点

### 与19题的区别
- 19题：需要删除倒数第N个节点，因此需要找到前驱节点（走N+1步）
- 本题：只需要找到倒数第cnt个节点，不需要删除（走cnt步即可）

### 算法步骤
1. 初始化 `slow = fast = head`
2. 快指针先走 `cnt` 步
3. 快慢指针同步前进，直到快指针为 `None`
4. 返回 `slow`

## 代码实现

### 标准解法（快慢指针）
```python
# Definition for singly-linked list.
# class ListNode:
#     def __init__(self, x):
#         self.val = x
#         self.next = None

class Solution:
    def trainingPlan(self, head: Optional[ListNode], cnt: int) -> Optional[ListNode]:
        """
        找到倒数第cnt个节点 - 快慢指针方法
        
        时间复杂度: O(n)，其中n是链表长度
        空间复杂度: O(1)，只使用常数额外空间
        
        算法步骤:
        1. 快指针先走cnt步
        2. 快慢指针同步前进
        3. 当快指针到达末尾时，慢指针在倒数第cnt个节点
        """
        slow = fast = head
        
        # 快指针先走cnt步
        for _ in range(cnt):
            fast = fast.next
        
        # 快慢指针同步前进
        while fast:
            slow = slow.next
            fast = fast.next
        
        return slow
```

### 两趟扫描方法（对比）
```python
class Solution:
    def trainingPlan_two_pass(self, head: Optional[ListNode], cnt: int) -> Optional[ListNode]:
        """
        两趟扫描方法
        1. 第一趟计算链表长度
        2. 第二趟找到倒数第cnt个节点
        
        时间复杂度: O(2n) = O(n)
        空间复杂度: O(1)
        """
        # 第一趟：计算长度
        length = 0
        curr = head
        while curr:
            length += 1
            curr = curr.next
        
        # 第二趟：找到倒数第cnt个节点
        # 正数位置 = length - cnt
        curr = head
        for _ in range(length - cnt):
            curr = curr.next
        
        return curr
```

### 栈方法（对比）
```python
class Solution:
    def trainingPlan_stack(self, head: Optional[ListNode], cnt: int) -> Optional[ListNode]:
        """
        栈方法 - 将所有节点压入栈，然后弹出cnt个
        
        时间复杂度: O(n)
        空间复杂度: O(n)，需要栈存储所有节点
        """
        stack = []
        curr = head
        
        # 将所有节点压入栈
        while curr:
            stack.append(curr)
            curr = curr.next
        
        # 弹出前cnt-1个节点
        for _ in range(cnt - 1):
            stack.pop()
        
        # 栈顶就是倒数第cnt个节点
        return stack[-1] if stack else None
```

## 详细分析

### 快慢指针工作原理

#### 示例1：head = [2,4,7,8], cnt = 1
```
链表: 2 → 4 → 7 → 8 → None

初始化: slow=2, fast=2

fast先走1步:
fast = 4

同步走:
fast=4, slow=2
fast=7, slow=4
fast=8, slow=7
fast=None, slow=8

结果: slow=8 (倒数第1个)
```

#### 示例2：head = [1,2,3,4,5,6], cnt = 3
```
链表: 1 → 2 → 3 → 4 → 5 → 6 → None

初始化: slow=1, fast=1

fast先走3步:
fast = 4

同步走:
fast=4, slow=1
fast=5, slow=2
fast=6, slow=3
fast=None, slow=4

结果: slow=4 (倒数第3个)
```

### 数学原理

设链表长度为 `n`，要找倒数第 `cnt` 个节点：
- 正数位置：`n - cnt`
- 快指针先走 `cnt` 步后，距离末尾：`n - cnt`
- 慢指针从head开始，快指针从第 `cnt+1` 个节点开始
- 两者同步走 `n - cnt` 步后：
  - 快指针到达末尾（None）
  - 慢指针到达第 `(n - cnt) + 1` 个节点，即正数第 `n - cnt + 1` 个
  - 倒数位置：`n - (n - cnt + 1) + 1 = cnt`

### 与19题的对比

#### 19题（删除倒数第N个节点）
```python
# 需要找到前驱节点，所以快指针走N+1步
for _ in range(n + 1):
    fast = fast.next
# 然后同步走，slow停在要删除节点的前一个节点
```

#### 本题（找到倒数第cnt个节点）
```python
# 只需要找到节点本身，所以快指针走cnt步
for _ in range(cnt):
    fast = fast.next
# 然后同步走，slow停在倒数第cnt个节点
```

## 边界条件处理

### 1. 空链表
```python
# 根据题目约束，链表非空且cnt有效
# 但实际中应该检查
if not head:
    return None
```

### 2. cnt等于链表长度
```python
# head = [1,2,3], cnt=3
# fast先走3步：fast = None
# 同步走：不执行（fast为None）
# slow = head = 1，即倒数第3个（正数第1个）
```

### 3. cnt等于1
```python
# head = [1,2,3], cnt=1
# fast先走1步：fast = 2
# 同步走到fast为None
# slow = 3，即倒数第1个
```

### 4. 单节点链表
```python
# head = [1], cnt=1
# fast先走1步：fast = None
# 同步走：不执行
# slow = 1，正确
```

## 性能分析

### 时间复杂度对比

| 方法 | 时间复杂度 | 扫描次数 | 优点 | 缺点 |
|------|------------|----------|------|------|
| 快慢指针 | O(n) | 1趟 | 最优，一趟扫描 | 需要理解指针步数 |
| 两趟扫描 | O(2n) = O(n) | 2趟 | 容易理解 | 需要两次遍历 |
| 栈方法 | O(n) | 1趟 | 逻辑简单 | 空间复杂度O(n) |

### 空间复杂度对比

| 方法 | 空间复杂度 | 说明 |
|------|------------|------|
| 快慢指针 | O(1) | 只使用常数额外空间 |
| 两趟扫描 | O(1) | 只使用常数额外空间 |
| 栈方法 | O(n) | 需要栈存储所有节点 |

### 实际性能考虑
- **链表长度小**：所有方法性能差异不大
- **链表长度大**：快慢指针方法最优
- **内存限制严格**：避免栈方法
- **cnt接近链表长度**：快慢指针方法仍然高效

## 常见错误

### 错误1：步数错误
```python
# ❌ 错误：快指针走cnt-1步
for _ in range(cnt - 1):
    fast = fast.next

# 结果：slow会指向倒数第cnt+1个节点

# ✅ 正确：走cnt步
for _ in range(cnt):
    fast = fast.next
```

### 错误2：忘记检查fast是否为None
```python
# ❌ 错误：在循环中直接访问fast.next
while fast.next:  # 如果fast为None，访问.next报错
    slow = slow.next
    fast = fast.next

# ✅ 正确：检查fast本身
while fast:
    slow = slow.next
    fast = fast.next
```

### 错误3：指针初始化错误
```python
# ❌ 错误：slow和fast从不同位置开始
slow = head
fast = head.next  # 如果cnt=1，这样会出错

# ✅ 正确：都从head开始
slow = fast = head
```

### 错误4：未检查cnt的有效性
```python
# ❌ 错误：假设cnt总是有效的
for _ in range(cnt):
    fast = fast.next  # 如果cnt大于链表长度，这里会报错

# ✅ 正确：根据题目约束cnt有效，但实际中应该检查
if cnt <= 0:
    return head
```

## 测试用例

### 基础测试
```python
def test_trainingPlan():
    solution = Solution()
    
    # 测试用例1：正常情况
    head = ListNode(2, ListNode(4, ListNode(7, ListNode(8))))
    result = solution.trainingPlan(head, 1)
    assert result.val == 8
    
    # 测试用例2：cnt=3
    head = ListNode(1, ListNode(2, ListNode(3, ListNode(4, ListNode(5, ListNode(6))))))
    result = solution.trainingPlan(head, 3)
    assert result.val == 4
    
    # 测试用例3：cnt等于链表长度
    head = ListNode(1, ListNode(2, ListNode(3)))
    result = solution.trainingPlan(head, 3)
    assert result.val == 1
    
    # 测试用例4：单节点
    head = ListNode(1)
    result = solution.trainingPlan(head, 1)
    assert result.val == 1
```

### 边界测试
```python
# 测试长链表
head = ListNode(1)
curr = head
for i in range(2, 101):
    curr.next = ListNode(i)
    curr = curr.next

# 测试各种cnt值
assert solution.trainingPlan(head, 1).val == 100    # 倒数第1个
assert solution.trainingPlan(head, 50).val == 51    # 倒数第50个
assert solution.trainingPlan(head, 100).val == 1    # 倒数第100个

# 测试cnt=2
head = ListNode(1, ListNode(2))
assert solution.trainingPlan(head, 2).val == 1
assert solution.trainingPlan(head, 1).val == 2
```

## 扩展应用

### 1. 找到链表的1/4处节点
快指针走4步，慢指针走1步。

### 2. 找到链表的中位数
结合876题（找中点）的思路。

### 3. 判断链表是否为回文
找到中点后，反转后半部分进行比较。

### 4. 链表重排
找到特定位置节点进行重排操作。

## 算法变体

### 变体1：递归解法
```python
class Solution:
    def trainingPlan_recursive(self, head: Optional[ListNode], cnt: int) -> Optional[ListNode]:
        """
        递归解法
        利用递归栈记录位置
        """
        self.count = 0
        self.result = None
        
        def dfs(node):
            if not node:
                return 0
            
            # 先递归到末尾
            index_from_end = dfs(node.next) + 1
            
            # 如果是倒数第cnt个，记录结果
            if index_from_end == cnt:
                self.result = node
            
            return index_from_end
        
        dfs(head)
        return self.result
```

### 变体2：使用数组记录
```python
def trainingPlan_array(self, head, cnt):
    """
    使用数组记录所有节点，直接访问
    """
    nodes = []
    curr = head
    
    while curr:
        nodes.append(curr)
        curr = curr.next
    
    # 倒数第cnt个节点的索引：len(nodes) - cnt
    return nodes[-cnt] if cnt <= len(nodes) else None
```

### 变体3：快慢指针不同速度
```python
def trainingPlan_variable_speed(self, head, cnt, fast_step=2):
    """
    快指针可以走更多步的通用版本
    """
    slow = fast = head
    
    # 快指针先走cnt步（每次走fast_step步）
    steps = cnt
    while steps > 0 and fast:
        fast = fast.next
        steps -= 1
    
    # 如果cnt大于链表长度
    if steps > 0:
        return None
    
    # 同步前进
    while fast:
        slow = slow.next
        fast = fast.next
    
    return slow
```

## 优化技巧

### 1. 减少变量
```python
def trainingPlan_compact(self, head, cnt):
    fast = slow = head
    for _ in range(cnt):
        fast = fast.next
    while fast:
        fast, slow = fast.next, slow.next
    return slow
```

### 2. 提前检查
```python
def trainingPlan_checked(self, head, cnt):
    # 检查输入有效性
    if not head or cnt <= 0:
        return None
    
    fast = slow = head
    
    # 先走cnt步，检查是否越界
    for _ in range(cnt):
        if not fast:  # cnt大于链表长度
            return None
        fast = fast.next
    
    # 同步走
    while fast:
        slow = slow.next
        fast = fast.next
    
    return slow
```

### 3. 使用生成器（高级）
```python
def trainingPlan_generator(self, head, cnt):
    """
    使用生成器实现，更Pythonic
    """
    def node_generator(node):
        while node:
            yield node
            node = node.next
    
    # 实际上还是需要快慢指针逻辑
    # 这里只是展示另一种思路
    fast = slow = head
    
    # 快指针先走cnt步
    for _ in range(cnt):
        if not fast:
            return None
        fast = fast.next
    
    # 同步走
    while fast:
        slow = slow.next
        fast = fast.next
    
    return slow
```

## 学习要点

### 必须掌握
1. **快慢指针技巧**：理解步进关系和终止条件
2. **倒数与正数的转换**：倒数第k个 = 正数第(n-k+1)个
3. **边界条件处理**：空链表、cnt等于长度等情况

### 建议练习
1. 实现所有变体解法
2. 扩展到删除倒数第N个节点（19题）
3. 结合其他链表问题综合练习

### 面试要点
- 能够解释快慢指针的工作原理
- 能够分析时间/空间复杂度
- 能够处理各种边界条件
- 能够与相关问题对比（如19题）

## 相关题目

### 基础题目
- [0876. 链表的中间结点](https://leetcode.com/problems/middle-of-the-linked-list/) - 快慢指针基础
- [0206. 反转链表](https://leetcode.com/problems/reverse-linked-list/) - 链表操作基础

### 进阶题目
- [0019. 删除倒数第N个节点](https://leetcode.com/problems/remove-nth-node-from-end-of-list/) - 本题的扩展
- [0141. 环形链表](https://leetcode.com/problems/linked-list-cycle/) - 快慢指针检测环

### 剑指Offer系列
- 剑指Offer 06. 从尾到头打印链表
- 剑指Offer 18. 删除链表的节点
- 剑指Offer 24. 反转链表
- 剑指Offer 25. 合并两个排序的链表

---

*本题是快慢指针技巧的典型应用，虽然题目简单，但包含了链表操作的核心思想。通过本题的学习，应该能够熟练掌握快慢指针的使用方法，为解决更复杂的链表问题打下基础。*