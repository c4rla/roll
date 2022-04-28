---
title: 做题笔记01：链表
description: 面试八股之生发液101
slug: "oj-01"
date: 2022-04-01
tags: 
    - OJ
---

## BM1 反转链表

1. 把链表第一个结点变成最后一个（node.next=null），前往下一个结点；
2. 把该结点的 next 改成前一个结点；
3. 继续前往下一个结点，直到最后一个结点为止。

```python
class Solution:
    def ReverseList(self , head: ListNode) -> ListNode:
        pre_node = None
        while head:
            next_node = head.next
            head.next = pre_node
            pre_node = head
            head = next_node
        return pre_node
```

## BM2 链表内指定区间反转

沿用上一题的代码，枯燥而且容易出错……

做完 BM3 回头看发现忘了给输入 m,n 做检查，~~whatever~~。

```python
class Solution:
    def reverseBetween(self , head: ListNode, m: int, n: int) -> ListNode:
        # 0. 设置 0 结点
        dummy = ListNode(0)
        dummy.next = head
        # 1. 找到两端，即第 m-1, n+1 个结点
        left = dummy
        for _ in range(m-1):
            left = left.next
        right = head
        for _ in range(n):
            right = right.next
        # 2. 开始反转，head 指向第 m 个结点
        pre_node, head = right, left.next
        while head != right:
            next_node = head.next
            head.next = pre_node
            pre_node = head
            head = next_node
        # 3. 接头
        left.next = pre_node 
        return dummy.next
```

## BM3 链表中的节点每k个一组翻转

感觉被针对了，失去了吐槽的力气，没有十年脑溢血想不出这样的题目……

```python
class Solution:
    def reverseKGroup(self , head: ListNode, k: int) -> ListNode:
        if head == None or head.next == None:
            return head
        # 0. 同样先写 dummy
        dummy = ListNode(0)
        dummy.next = head
        # 1. 计算需要反转多少次
        cur = head
        length = 0
        while cur != None:
            length = length + 1
            cur = cur.next
        count = length // k
        # 2. 同样先找两端
        left = dummy
        right = head
        if count > 0:
            for _ in range(k):
                right = right.next
        # 3. 开始循环反转
        for i in range(1, count+1):
            pre_node = right
            if count > i:
                for _ in range(k-1):
                    pre_node = pre_node.next
            head = left.next
            while head != right:
                next_node = head.next
                head.next = pre_node
                pre_node = head
                head = next_node
            left.next = pre_node 
            # 前往后面 k 个，注意此时是断开状态（画图更容易理解），需要给 right 左边补 dummy 作为 left
            if count > i:
                left = ListNode(0)
                left.next = right
                for _ in range(k):
                    right = right.next
        return dummy.next                    
```

从题解学到一种双指针技巧：head 指针一直指在第一个结点上，left 指针一直指向第 0 个结点，每次交换时 head.next.next = left.next 然后 head.next 和 left.next 分别前进一位。

结合图形可能更容易理解（比如想像成一串珠子），每次交换把第一个结点的当前 next 结点移到最左边（即第 0 个结点的右边一位），然后第一个结点向右前进一位，第 0 个结点保持不动，不难看到只需要移动 k-1 次。

```python
class Solution:
    def reverseKGroup(self , head: ListNode, k: int) -> ListNode:
        if head == None or head.next == None:
            return head
        # 0. 同样先写 dummy
        dummy = ListNode(0)
        dummy.next = head
        # 1. 计算链表长度
        length = 0
        while head != None:
            length = length + 1
            head = head.next
        # 2. 同样先找左端
        left = dummy
        head = dummy.next
        # 3. 开始循环反转
        while length >= k:
            for j in range(k-1):
                next_node = head.next
                head.next = next_node.next
                next_node.next = left.next
                left.next = next_node
            length = length - k
            left = head
            head = left.next
        return dummy.next
```

## BM4 合并两个排序的链表

比较大小，小的进链表，指针移到下一个，到结尾就收工。

```python
class Solution:
    def Merge(self , pHead1: ListNode, pHead2: ListNode) -> ListNode:
        # 老套路，先创建 dummy
        dummy = ListNode(0)
        head = dummy
        while pHead1 != None and pHead2 != None:
            if pHead1.val > pHead2.val:
                head.next = pHead2
                pHead2 = pHead2.next
            else:
                head.next = pHead1
                pHead1 = pHead1.next
            head = head.next
        head.next = (pHead2 if pHead2 else pHead1)
        return dummy.next
```

## BM5 合并 k 个已排序的链表

学习使用 Python 的优先队列（二叉堆），不过效果不咋样，和直接 sort 的效率没明显差异。

需要注意的是，Python 的二叉堆在比较 tuple 的大小时，如果 tuple 第一个结点相等，就接着比较第二个结点，如果 tuple 第二个结点不能比较大小，就会报错 `TypeError: '<' not supported between instances of xxx and xxx`，所以还是使用了 index 作为比较的依据。

```python
class Solution:
    def mergeKLists(self , lists: List[ListNode]) -> ListNode:
        import heapq
        dummy = ListNode(0)
        cur = dummy
        h = [(head.val, idx, head) for idx, head in enumerate(lists) if head]
        heapq.heapify(h)
        while h:
            _, idx, head = heapq.heappop(h)
            cur.next = head
            cur, head = cur.next, head.next
            if head:
                heapq.heappush(h, (head.val, idx, head))
        return dummy.next
```

## BM6 判断链表中是否有环

熟悉的快慢双指针“追及问题”。

```python
class Solution:
    def hasCycle(self , head: ListNode) -> bool:
        slow, fast = head, head
        while fast and fast.next:
            slow, fast = slow.next, fast.next.next
            if slow == fast:
                return True
        return False   
```

## BM7 链表中环的入口结点

也是快慢双指针“追及问题”。假设相遇时慢指针走了 a 步，快指针走了 2a 步，则 2a-a=a 是环长 z 的整数倍，同时也是到环入口的距离 x + 环入口到相遇点的距离 y + 环长 z 的整数。所以 x + y 是环长的整数倍，只要把快指针放回起点逐个遍历（慢指针在 y 处），两个指针再次相遇时就是在入口起点。如果再加个计数器，就可以求出到环入口的长度。

```python
class Solution:
    def EntryNodeOfLoop(self, head):
        slow, fast = head, head
        while fast and fast.next:
            slow, fast = slow.next, fast.next.next
            if slow == fast:
                fast = head
                while slow != fast:
                    slow, fast = slow.next, fast.next
                return slow
        return None
```

## BM8 链表中倒数最后 k 个结点

依然是双指针问题。只要让一个指针先走 k 步就好。

```python
class Solution:
    def FindKthToTail(self , head: ListNode, k: int) -> ListNode:
        slow, fast = head, head
        for _ in range(k):
            if not fast:
                return fast
            fast = fast.next
        while fast:
            slow, fast = slow.next, fast.next
        return slow
```

## BM9 删除链表的倒数第 n 个节点

和 BM8 的唯一区别是求倒数 k+1 个结点，所以要设置 dummy。

```python
class Solution:
    def removeNthFromEnd(self , head: ListNode, n: int) -> ListNode:
        dummy = ListNode(0)
        dummy.next = head
        slow, fast = dummy, head
        for _ in range(n):
            if not fast:
                return fast
            fast = fast.next
        while fast:
            slow, fast = slow.next, fast.next
        slow.next = slow.next.next
        return dummy.next
```

## BM10 两个链表的第一个公共结点

注意链表是一种特殊的数据结构，每个结点可能有很多个前驱，但后继只能有一个。如果两个链表有公共结点，说明从某个结点开始，这两个链表完全一致。因此可以先算出两个链表的长度差 d，让较长的链表先走 d 步，两个指针相遇时就是公共结点。

除此之外，题解还给出一种方法：把两个链表接起来遍历，A=A+B，B=B+A。因为会经过 None，所以不会出现死循环。

```python
class Solution:
    def FindFirstCommonNode(self , head_1 , head_2 ):
        if (not head_1) or (not head_2):
            return None
        ptr_1, ptr_2 = head_1, head_2
        while ptr_1 != ptr_2:
            ptr_1 = (ptr_1.next if ptr_1 else head_2)
            ptr_2 = (ptr_2.next if ptr_2 else head_1)
        return ptr_1
```

## BM11 链表相加(二)

这种题目有什么存在的必要吗？？？？？？最后还是心平气和使用 BM1 的代码（省略）……

```python
class Solution:
    def addInList(self , head_1: ListNode, head_2: ListNode) -> ListNode:
        head_1, head_2 = self.ReverseList(head_1), self.ReverseList(head_2)
        dummy = ListNode(0)
        head = dummy
        carrier = 0
        while head_1 or head_2:
            sumup = (head_1.val if head_1 else 0) + (head_2.val if head_2 else 0) + carrier
            head.next = ListNode(sumup % 10)
            carrier = sumup // 10
            head = head.next
            head_1 = (head_1.next if head_1 else None)
            head_2 = (head_2.next if head_2 else None)
        if carrier:
            head.next = ListNode(carrier)
        return self.ReverseList(dummy.next)
```

## BM12 单链表的排序

不想手写 merge sort……使用 BM5 的二叉堆吧：

```python
class Solution:
    def sortInList(self , head: ListNode) -> ListNode:
        import heapq
        idx, h, dummy = 0, [], ListNode(0)
        cur = head
        while cur:
            heapq.heappush(h, (cur.val, idx, cur))
            cur = cur.next
            idx = idx + 1
        cur = dummy
        while h:
            cur.next = heapq.heappop(h)[2]
            cur = cur.next
        cur.next = None
        return dummy.next
```

## BM13 判断一个链表是否为回文结构

回文结构即正向和反向遍历结果完全相同的结构。由于链表只能正着走，可以考虑先找到中点，把后半截链表使用 BM1 反过来，再逐个进行比较。找中点也可以使用快慢双指针。

虽然不难，但是 debug 了好久，好丢人……

```python
class Solution:
    def isPail(self , head: ListNode) -> bool:
        if (not head) or (not head.next): # 长度为 0 或 1
            return True
        dummy = ListNode(0)
        dummy.next = head
        slow, fast = dummy, dummy
        while fast and fast.next:
            slow, fast = slow.next, fast.next.next
        if not fast: # 链表长度为奇数，此时 slow 在中点
            fast = self.ReverseList(slow)
        else: # 链表长度为偶数，此时 slow 在前半段末
            fast = self.ReverseList(slow.next)
        slow = head
        while slow and fast and slow.val == fast.val: # 不要忘记比较的是 .val！！！
            slow, fast = slow.next, fast.next
        return ((not slow) or (not fast))
```

## BM14 链表的奇偶重排

怎么又是双指针……~~JOJO 我不做人啦！~~

多指针的题目往往很难进行调试（所以指针越少越好！），需要对状态进行仔细分析。例如，当 while 循环结束后，如果链表结点是奇数个（odd 是最后一个结点，odd.next == even == None），只需要把 even_head 接好；如果链表结点是偶数个（odd.next == even != None 是最后一个结点, even.next == None），同样只需要接上 even_head，所以最后的处理没有任何问题。

```python
class Solution:
    def oddEvenList(self , head: ListNode) -> ListNode:
        if (not head) or (not head.next) or (not head.next.next): # 长度为 0,1,2 不需要处理
            return head
        odd = head
        even, even_head = head.next, head.next
        while even and even.next:
            odd.next = even.next
            odd = odd.next
            even.next = odd.next
            even = even.next
        odd.next = even_head
        return head
```

## BM15 删除有序链表中重复的元素-I

写了太多双指针，已经成为巴甫洛夫的狗了：

```python
class Solution:
    def deleteDuplicates(self , head: ListNode) -> ListNode:
        slow, fast = head, head
        while fast:
            fast = fast.next
            while fast and fast.val == slow.val:
                fast = fast.next
            slow.next = fast
            slow = slow.next
        return head
```

看题解才发现一个指针就足够了：

```python
class Solution:
    def deleteDuplicates(self , head: ListNode) -> ListNode:
        if not head:
            return head
        cur = head
        while cur.next:
            if cur.next.val == cur.val:
                cur.next = cur.next.next
            else:
                cur = cur.next
        return head
```

## BM16 删除有序链表中重复的元素-II

看上去很简单，做起来无从下手。仔细想想，这道题其实要三个相邻的指针：如果后面两个指针的值相同，则一直前进到不同为止，然后最前面的指针的 next 指向最后的指针（不同的值），直到中间的指针指到 None 为止，这又会产生越界问题……要仔细区分所有状态，小心小心再小心！

```python
class Solution:
    def deleteDuplicates(self , head: ListNode) -> ListNode:
        if head == None or head.next == None:
            return head
        dummy = ListNode(0)
        dummy.next = head
        slow, middle = dummy, head
        while middle:
            if middle.next: # middle 不是最后一个元素
                if middle.val == middle.next.val: # 出现重复
                    while middle.next and middle.val == middle.next.val:
                        middle = middle.next
                    # 停止有两种可能：middle 是最后一个元素，或者出现第一个不等
                    if middle.next: # 出现不等
                        slow.next = middle.next
                        middle = middle.next
                    else: # middle 是最后一个元素，middle.next = None
                        slow.next = None
                        return dummy.next
                else: # 没有重复，slow 和 middle 分别前进一格
                    slow = slow.next
                    middle = middle.next
            else: # middle 是最后一个元素，且和前一个元素没有重复
                slow.next = middle
                return dummy.next
```

可以再精简一下判断逻辑，不过可读性似乎变差了：

```python
class Solution:
    def deleteDuplicates(self , head: ListNode) -> ListNode:
        if head == None or head.next == None:
            return head
        dummy = ListNode(0)
        dummy.next = head
        slow, middle = dummy, head
        while middle and middle.next: # middle 不是最后一个元素，也不为空
            if middle.val == middle.next.val: # 出现重复
                while middle.next and middle.val == middle.next.val:
                    middle = middle.next
                # 停止有两种可能：middle 是最后一个元素，或者出现第一个不等
                # 无论是哪一种可能，都是相同的代码
                slow.next = middle.next
                middle = middle.next
            else: # 没有重复，slow 和 middle 分别前进一格
                slow = slow.next
                middle = middle.next
        if middle: # middle 是最后一个元素，且和前一个元素没有重复
            slow.next = middle
        return dummy.next
```

其实更容易理解的方法是使用 hash 或者常数排序，不过空间复杂度就做不到 O(1) 了。收工～
