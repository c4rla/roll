---
title: 做题笔记04：堆/栈/队列
description: 🎶我排着队，拿着爱的号码牌
slug: "oj-04"
date: 2022-04-06
tags: 
    - OJ
---

## BM42 用两个栈实现队列

栈相当于把队列的一部分元素以相反的方向存储，再使用一个栈把所有元素的顺序反过来就是队列。等队列栈空了，就再做一次相同的事情。

```python
class Solution:
    def __init__(self):
        self.stack1 = []
        self.stack2 = []
    def push(self, node):
        self.stack1.append(node)
    def pop(self):
        if not self.stack1 and not self.stack2: # 队列为空
            return None
        if not self.stack2: # 栈2为空，把栈1所有元素入栈
            while self.stack1:
                self.stack2.append(self.stack1.pop())
        return self.stack2.pop()
```

## BM43 包含min函数的栈

空间换时间，再准备一个栈缓存当前的min就好：

```python
class Solution:
    def __init__(self):
        self.stack = []
        self.stack_min = []
    def push(self, node):
        self.stack.append(node)
        if not self.stack_min:
            self.stack_min.append(node)
        else:
            self.stack_min.append(min(node, self.stack_min[-1]))
    def pop(self):
        self.stack_min.pop()
        return self.stack.pop()
    def top(self):
        return self.stack[-1]
    def min(self):
        return self.stack_min[-1]
```

## BM44 有效括号序列

使用一个栈存储已经出现的左括号：

```python
class Solution:
    def isValid(self , s: str) -> bool:
        stack, pairs = [], {']': '[', ')': '(', '}': '{'}
        for char in s:
            if char in '([{':
                stack.append(char)
            elif char in ')]}':
                if not stack or pairs[char] != stack[-1]:
                    return False
                else:
                    stack.pop()
        return not stack
```

## BM45 滑动窗口的最大值

又是一道难题，首先想到的自然是优先队列，但是时间复杂度 O(nlog(n)) 不能满足要求，只能跑去看题解……

题解的原理其实很简单：希望在下一个滑动窗口中利用前面滑动窗口的最大值信息，即维护一个窗口大小的双向队列。具体而言，因为当前窗口最大值前面的元素，对下一个窗口的最大值毫无影响，因此不需要存储，队列里存储的值即为 [当前窗口的最大值, 前一个数到滑动窗口末的最大值, 前一个数到滑动窗口末的最大值, ...]。

如何形成这样的队列？只要每个新元素在入列之前，把栈中比它小的元素从右边出列即可。窗口开始滑动时，如果队列第一个元素滑出窗口就出列，然后把当前元素入列。

这个思路有点像动态规划，也有点像二叉搜索树的前序遍历……

```python
class Solution:
    def maxInWindows(self , num: List[int], size: int) -> List[int]:
        from collections import deque
        if not num or size == 0 or size > len(num):
            return []
        window = deque([])
        for i in range(size): # 得到第一个窗口的状态
            while window and num[window[-1]] < num[i]:
                window.pop()
            window.append(i)
        result = [num[window[0]]]
        for i in range(size, len(num)):
            if window[0] <= i - size:
                window.popleft()
            while window and num[window[-1]] < num[i]:
                window.pop()
            window.append(i)
            result.append(num[window[0]])
        return result
```

## BM46 最小的K个数

最简单的一题（x

```python
class Solution:
    def GetLeastNumbers_Solution(self , input: List[int], k: int) -> List[int]:
        import heapq
        return heapq.nsmallest(k, input)
```

其实和 `sorted(input)[:k]` 没什么区别😂

## BM47 寻找第K大

上面搞错了，这才是最简单的一题（x

```python
class Solution:
    def findKth(self , a: List[int], n: int, K: int) -> int:
        return sorted(a)[-K]
```

试着写了一下题解快速排序的代码，结果第8个测试用例超出限制了……所以不要浪费生命，less is more！

## BM48 数据流中的中位数

依然是考察堆排序。准备一个大顶堆存放小元素，一个小顶堆存放大元素，并且保持大顶堆所有元素都小于小顶堆所有元素。显然中位数必然在这两个队的队顶元素之间。添加元素时，可以先加入大堆，再从大堆取出最大值加入小堆（为了维持相对大小关系），也可以反过来。

注意 Python 里没有最大堆，所以放入和取出最大堆元素的时候都加了负号。

```python
import heapq
class Solution:
    def __init__(self):
        self.max_h, self.min_h = [], []
    def Insert(self, num):
        if len(self.min_h) == len(self.max_h): # 两堆相等，优先加入小堆
            heapq.heappush(self.min_h, -heapq.heappushpop(self.max_h, -num))
        else: # 加入大堆
            heapq.heappush(self.max_h, -heapq.heappushpop(self.min_h, num))
    def GetMedian(self):
        if len(self.min_h) == len(self.max_h):
            return (self.min_h[0] - self.max_h[0]) / 2.0
        else:
            return self.min_h[0]
```

## BM49 表达式求值

刷新了中等题的难度下限（x

```python
class Solution:
    def solve(self , s: str) -> int:
        return eval(s)
```

还是认真做一下吧，题解的方法像在写有限状态机：

1. 准备两个栈，一个放操作数，一个放运算符。为了兼顾第一个字符为 +- 的情形，操作数的栈中先放个 0；
2. 顺序扫描字符串，如果是数字，继续读取，直到第一个不是数字的字符为止，把数字压入操作数栈；
3. 如果是左括号，压入运算符栈；
4. 如果是右括号，对操作数和运算符进行出栈计算，然后把结果压入操作数栈，这个动作一直持续到操作数栈栈顶元素为左括号为止，左括号出栈；
5. 如果是操作符，则和前一个操作符的运算优先级进行比较，如果小于等于前一个操作符的优先级，则出栈进行计算，一直持续到前一个操作符优先级小于当前操作符优先级为止，或者运算符栈为空；
6. 操作符还有另一种情况：(+,+- 这种两个运算符相连的情形（前一个可以是括号），此时为了处理方便，需要给操作数栈插入一个 0；
7. 对于其他的情形（比如空格），直接跳过，遍历完成后计算并返回结果。

```python
class Solution:
    def solve(self , s: str) -> int:
        nums, ops, i, s_len, last_is_ops = [], [], 0, len(s), True
        prior, allnum = {'+':1, '-': 1, '*': 2}, set(str(i) for i in range(10))
        func = {'+': lambda x,y: x+y, '-': lambda x,y: y-x, '*': lambda x,y: x*y}
        while i < s_len:
            if s[i] in allnum: # 可能有多位数字
                num = 0
                while i < s_len and s[i] in allnum:
                    num, i = num * 10 + int(s[i]), i + 1
                nums.append(num)
                i, last_is_ops = i - 1, False
            elif s[i] == '(': # 出现左括号
                ops.append(s[i])
                last_is_ops = True
            elif s[i] == ')': # 出现右括号，出栈并计算结果，last_is_ops==False 保持不变
                while ops and ops[-1] != '(':
                    nums.append(func[ops.pop()](nums.pop(), nums.pop()))
                ops.pop()
            elif s[i] in prior: # 出现运算符，如果优先级<=上一个，对上一个运算符进行计算
                if last_is_ops: # 处理 (+, +- 这种特殊情形
                    nums.append(0)
                while ops and ops[-1] != '(' and prior[ops[-1]] >= prior[s[i]]:
                    nums.append(func[ops.pop()](nums.pop(), nums.pop()))
                ops.append(s[i])
                last_is_ops = True
            i = i + 1
        while ops: # 把剩余的计算完，一般只剩下 a+b*c,a-b*c
            nums.append(func[ops.pop()](nums.pop(), nums.pop()))
        return nums[-1]
```

写了 30 行代码，感觉像过了半个世纪😂
