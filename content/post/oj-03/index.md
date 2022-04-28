---
title: 做题笔记03：二叉树
description: 讨厌递归！
slug: "oj-03"
date: 2022-04-03
tags: 
    - OJ
---

## BM23 二叉树的前序遍历

继续记模板！

```python
class Solution:
    def preorderTraversal(self , root: TreeNode) -> List[int]:
        result = []
        self.preOrder(root, result)
        return result
        
    def preOrder(self, root, result):
        if not root:
            return None
        else:
            result.append(root.val)
            self.preOrder(root.left, result)
            self.preOrder(root.right, result)
```

## BM24 二叉树的中序遍历

直接使用递归超出系统限制了，虽然可以使用作弊器：

```python
import sys
sys.setrecursionlimit(100000)
```

但还是学习一下非递归的写法吧：

```python
class Solution:
    def inorderTraversal(self , root: TreeNode) -> List[int]:
        result = []
        self.inOrder(root, result)
        return result
    
    def preOrder(self, root, result): # 前序，“根左右”
        if not root:
            return None
        stack = [root]
        while stack:
            top = stack.pop()
            result.append(top.val)
            if top.right: # 优先加入右结点，因为是栈，左结点会先出来
                stack.append(top.right)
            if top.left:
                stack.append(top.left)

    def inOrder(self, root, result): # 中序，“左根右”
        if not root:
            return None
        stack = []
        while root: # 左
            stack.append(root)
            root = root.left
        while stack: # root 是当前访问的结点
            node = stack.pop() # 根
            result.append(node.val)
            root = node.right # 右
            while root:
                stack.append(root)
                root = root.left
```

## BM25 二叉树的后序遍历

后序的非递归版本并不好写，最简单的做法似乎是把前序遍历反转一下：

```python
class Solution:
    def postorderTraversal(self , root: TreeNode) -> List[int]:
        result = []
        self.postOrder(root, result)
        return result[::-1]
    
    def postOrder(self, root, result): # 后序1，前序“根右左”，反转在上面
        if not root:
            return None
        stack = [root]
        while stack:
            top = stack.pop()
            result.append(top.val)
            if top.left:
                stack.append(top.left)
            if top.right:
                stack.append(top.right)

    def postOrder2(self, root, result): # 后序2，“左右根”
        if not root:
            return None
        stack = [root]
        while stack:
            top = stack.pop() # 根
            if top:
                stack.append(top)
                stack.append(None) # 标记此结点已经访问过
                if top.right: # 右
                    stack.append(top.right)
                if top.left: # 左
                    stack.append(top.left)
            else:
                result.append(stack.pop().val)
```

## BM26 求二叉树的层序遍历

相对简单，使用队列一层一层扫描即可：

```python
class Solution:
    def levelOrder(self , root: TreeNode) -> List[List[int]]:
        if not root:
            return
        from collections import deque
        result, queue = [], deque([root])
        while queue:
            tmp = []
            for _ in range(len(queue)):
                node = queue.popleft()
                tmp.append(node.val)
                if node.left:
                    queue.append(node.left)
                if node.right:
                    queue.append(node.right)
            result.append(tmp)
        return result
```

比较有趣的是递归的写法，然而直接 Stack Overflow……

## BM27 按之字形顺序打印二叉树

层序遍历+判断正负，也很简单。

```python
class Solution:
    def Print(self , root: TreeNode) -> List[List[int]]:
        if not root:
            return
        from collections import deque
        result, queue, tik = [], deque([root]), True
        while queue:
            tmp = []
            for _ in range(len(queue)):
                node = queue.popleft()
                tmp.append(node.val)
                if node.left:
                    queue.append(node.left)
                if node.right:
                    queue.append(node.right)
            if tik:
                result.append(tmp)
            else:
                result.append(tmp[::-1])
            tik = (not tik)
        return result
```

## BM28 二叉树的最大深度

依然是层序遍历。

```python
class Solution:
    def maxDepth(self , root: TreeNode) -> int:
        if not root:
            return 0
        from collections import deque
        queue, depth = deque([root]), 0
        while queue:
            for _ in range(len(queue)):
                node = queue.popleft()
                if node.left:
                    queue.append(node.left)
                if node.right:
                    queue.append(node.right)
            depth = depth + 1
        return depth
```

## BM29 二叉树中和为某一值的路径(一)

本来以为很简单，但写起来意外不顺手，最后还是基于后序遍历的代码修改。只要再加一个栈，就可以获得完整的路径。

```python
class Solution:
    def hasPathSum(self , root: TreeNode, sumup: int) -> bool:
        if not root:
            return False
        stack, count = [root], 0
        while stack:
            node = stack.pop()
            if node:
                stack.extend([node, None])
                count += node.val
                if node.right:
                    stack.append(node.right)
                if node.left:
                    stack.append(node.left)
            else:
                node = stack.pop()
                if (not node.left) and (not node.right) and count == sumup: # 叶子结点
                    return True
                else:
                    count -= node.val
        return False
```

## BM30 二叉搜索树与双向链表

中序遍历，二叉树的题目写起来意外棘手……

```python
class Solution:
    def Convert(self , root ):
        if not root:
            return None
        stack, pre = [], None # 使用两个指针记录状态
        while root: # 左
            stack.append(root)
            root = root.left
        while stack: # root 是当前访问的结点
            node = stack.pop() # 根
            root = node.right # 右
            node.left = pre
            if pre:
                pre.right = node
            else:
                head = node
            pre = node
            while root:
                stack.append(root)
                root = root.left
        return head
```

## BM31 对称的二叉树

仔细想想，好像任何一种遍历方式都可以，只要把左右颠倒过来遍历，再比较即可。

这里使用层序遍历，不过要注意，元素为空也是一种需要比较的状态……

```python
class Solution:
    def isSymmetrical(self , root: TreeNode) -> bool:
        if not root:
            return True
        queue_left, queue_right = [root], [root]
        while queue_left and queue_right:
            node_left = queue_left.pop()
            node_right = queue_right.pop()
            if node_left and node_right: # 同时存在
                if node_left.val != node_right.val:
                    return False
                else:
                    queue_left.append(node_left.left) # 空状态也要记录
                    queue_left.append(node_left.right)
                    queue_right.append(node_right.right)
                    queue_right.append(node_right.left)
            elif (not node_left) and (not node_right): # 同时为空
                pass
            else:
                return False
        return (not queue_left) and (not queue_right)
```

## BM32 合并二叉树

同样是任何一种遍历方式都可以，依然是层序：

```python
class Solution:
    def mergeTrees(self , t1: TreeNode, t2: TreeNode) -> TreeNode:
        if not t1:
            return t2
        if not t2:
            return t1
        head = TreeNode(t1.val + t2.val)
        queue1, queue2, queue = [t1], [t2], [head]
        while queue1 and queue2:
            node1, node2, node = queue1.pop(), queue2.pop(), queue.pop()
            if node1.left and node2.left:
                node.left = TreeNode(node1.left.val + node2.left.val)
                queue.append(node.left)
                queue1.append(node1.left)
                queue2.append(node2.left)
            elif node1.left:
                node.left = node1.left
            elif node2.left:
                node.left = node2.left
            else:
                pass
            if node1.right and node2.right:
                node.right = TreeNode(node1.right.val + node2.right.val)
                queue.append(node.right)
                queue1.append(node1.right)
                queue2.append(node2.right)
            elif node1.right:
                node.right = node1.right
            elif node2.right:
                node.right = node2.right
            else:
                pass
        return head
```

## BM33 二叉树的镜像

爱上了层序……

```python
class Solution:
    def Mirror(self , root: TreeNode) -> TreeNode:
        if not root:
            return root
        queue = [root]
        while queue:
            node = queue.pop()
            node.left, node.right = node.right, node.left # 原地镜像
            if node.left:
                queue.append(node.left)
            if node.right:
                queue.append(node.right)
        return root
```

## BM34 判断是不是二叉搜索树

又是中序遍历……

```python
class Solution:
    def isValidBST(self , root: TreeNode) -> bool:
        if not root:
            return False
        stack, pre = [], -2**32-1
        while root:
            stack.append(root)
            root = root.left
        while stack:
            node = stack.pop()
            if not node.val > pre:
                return False
            else:
                pre = node.val
            root = node.right
            while root:
                stack.append(root)
                root = root.left
        return True
```

## BM35 判断是不是完全二叉树

又是层序遍历。

```python
class Solution:
    def isCompleteTree(self , root: TreeNode) -> bool:
        if not root:
            return True
        from collections import deque
        queue, flag = deque([root]), False # 标记是否出现叶子结点
        while queue:
            for _ in range(len(queue)):
                node = queue.popleft()
                if node:
                    if flag: # 之前有空结点
                        return False
                    else:
                        queue.extend([node.left, node.right])
                else: # 出现空结点
                    flag = True
        return True # 遍历完成，没有中途退出，是完全二叉树
```

## BM36 判断是不是平衡二叉树

美好的一天从一道“简单”题开始😭

### 方法一：递归（Credit：官方解答）

其实就是要写一个递归计算二叉树深度的函数，发现不平衡立刻返回 False：

```python
class Solution:
    def IsBalanced_Solution(self , root: TreeNode) -> bool:
        return self.height(root) >= 0 # 能求出高度就是平衡二叉树
    
    def height(self, root: TreeNode) -> int:
        if not root: # 空结点，高度自然是 0
            return 0
        else:
            leftHeight = self.height(root.left)
            rightHeight = self.height(root.right)
            if leftHeight == -1 or rightHeight == -1 or abs(leftHeight - rightHeight) > 1:
                return -1
            else:
                return max(leftHeight, rightHeight) + 1
```

### 方法二：后序遍历

类似 BM29，使用一个字典存储每个结点的深度：

```python
class Solution:
    def IsBalanced_Solution(self , root: TreeNode) -> bool:
        if not root:
            return True
        stack, height = [root], {None: 0}
        while stack:
            root = stack.pop()
            if root:
                stack.extend([root, None])
                if root.left: # 不小心写反了！
                    stack.append(root.left)
                if root.right:
                    stack.append(root.right)
            else:
                node = stack.pop()
                if abs(height[node.left] - height[node.right]) > 1:
                    return False
                else:
                    height[node] = max(height[node.left], height[node.right]) + 1
        return True
```

## BM37 二叉搜索树的最近公共祖先

偷懒了，怎么简单怎么写。

```python
class Solution:
    def lowestCommonAncestor(self , root: TreeNode, p: int, q: int) -> int:
        while root:
            if p < root.val and q < root.val:
                root = root.left
            elif p > root.val and q > root.val:
                root = root.right
            else:
                return root.val
```

## BM38 在二叉树中找到两个节点的最近公共祖先

就很朴素：

1. 后序遍历二叉树，找到前往两个结点的路径；
2. 按顺序逐个比较路径中的每个结点，找到最后一个相同的并返回。

奇怪的是，如果用数组保存 TreeNode 的话，在比较相等的时候会出问题……

```python
class Solution:
    def lowestCommonAncestor(self , root: TreeNode, o1: int, o2: int) -> int:
        from os.path import commonprefix
        if not root:
            return None
        stack, path, path1, path2 = [root], [], [], []
        while stack:
            node = stack.pop()
            if node:
                stack.extend([node, None])
                path.append(node.val)
                if node.right:
                    stack.append(node.right)
                if node.left:
                    stack.append(node.left)
            else:
                node = stack.pop()
                if node.val == o1:
                    path1 = path.copy() # 浅复制
                if node.val == o2:
                    path2 = path.copy()
                path.pop()
        return commonprefix([path1, path2])[-1]
```

## BM39 序列化二叉树

标准答案是按前序和中序遍历做的，还是喜欢层序（因为官方使用的就是层序），虽然不难，但是很长……

```python
class Solution:
    def Serialize(self, root):
        if not root:
            return '{}'
        from collections import deque
        result, queue = '', deque([root])
        while any(queue):
            for _ in range(len(queue)):
                node = queue.popleft()
                if node:
                    queue.extend([node.left, node.right])
                    if result:
                        result += "," + str(node.val)
                    else:
                        result += "{" + str(node.val)
                else:
                    queue.extend([None, None])
                    result += ",#"
        return result + "}"
            
    def Deserialize(self, s):
        if s == "{}":
            return None
        from collections import deque
        data = s[1:-1].split(',')
        root = TreeNode(0)
        queue = deque([root])
        for s in range(len(data)):
            if data[s] != '#':
                head = queue.popleft()
                head.val = int(data[s])
                if 2 * (s + 1) < len(data): # 编号为 N 的结点子结点编号为 2N, 2N+1，对应的数组坐标为 N-1
                    if data[2 * (s + 1) - 1] != '#':
                        head.left = TreeNode(0)
                        queue.append(head.left)
                    if data[2 * (s + 1)] != '#':
                        head.right = TreeNode(0)
                        queue.append(head.right)
                else: # 到达最底层
                    pass
            else: # 空结点
                pass
        return root
```

## BM40 重建二叉树

明明这题更难一点😭题解的做法是递归：

1. 找到前序遍历的第一个结点，即为根结点；
2. 找到中序遍历中根结点所在的位置，根结点左边的结点即为左子树，右边的结点即为右子树；
3. 同样对前序遍历的结点进行左右子树的切分，递归做下去即可。

```python
class Solution:
    def reConstructBinaryTree(self , pre: List[int], vin: List[int]) -> TreeNode:
        if not len(pre) or not len(vin):
            return None
        root = TreeNode(pre[0])
        root_index = vin.index(pre[0])
        root.left = self.reConstructBinaryTree(pre[1:root_index+1], vin[0:root_index])
        root.right = self.reConstructBinaryTree(pre[root_index+1:], vin[root_index+1:])
        return root
```

如果不使用递归，考虑使用栈作为辅助存储：

1. 从前序遍历读取第一个结点，栈为空，新建结点后放入栈中；
2. 从前序遍历中读取下一个结点，计算中序索引，并与栈中的结点在中序中的索引大小进行比较；
3. 如果小于栈最后一个结点的索引，说明是栈最后一个结点的左子结点，入栈；
4. 如果大于栈最后一个结点的索引，说明这个结点一定是栈中某个元素的右结点，以反方向遍历栈，找到栈中中序索引比结点索引小且距离最小的第一个结点，就是所求的祖先。

终于写出来了，然而效率并不如递归……

```python
class Solution:
    def reConstructBinaryTree(self , pre: List[int], vin: List[int]) -> TreeNode:
        if not len(pre) or not len(vin):
            return None
        stack, vin_index, tmp = [], dict((j,i) for i,j in enumerate(vin)), []
        for val in pre:
            if not stack: # 刚开始 栈为空
                stack.append(TreeNode(val))
                tmp.append(val)
            else: # 栈不为空，开始比较结点
                while stack[-1].val != val: # 结点未入栈
                    if vin_index[val] < vin_index[stack[-1].val]: # 小于最后一个结点，为左子结点
                        stack[-1].left = TreeNode(val)
                        stack.append(stack[-1].left)
                        tmp.append(val)
                    else: # 是某个祖先的右结点
                        closest_index = -1 # 中序遍历栈中离结点最近的左边结点坐标
                        for node in reversed(stack):
                            if vin_index[node.val] < vin_index[val]:
                                if vin_index[node.val] > closest_index:
                                    closest_index = vin_index[node.val]
                                    if closest_index == vin_index[val] - 1:
                                        break
                                else: # 似乎可以再优化一下……
                                    pass
                            else: # 遇到右边的结点，停止
                                break
                        while vin_index[stack[-1].val] != closest_index:
                            stack.pop()
                            tmp.pop()
                        stack[-1].right = TreeNode(val)
                        stack.append(stack[-1].right)
                        tmp.append(val)
        return stack[0]
```

## BM41 输出二叉树的右视图

又是层序遍历！收工👏

```python
class Solution:
    def solve(self , pre: List[int], vin: List[int]) -> List[int]:
        tree = self.reConstructBinaryTree(pre, vin)
        if not tree:
            return []
        from collections import deque
        queue, result = deque([tree]), []
        while queue:
            for _ in range(len(queue)):
                node = queue.popleft()
                val = node.val
                if node.left:
                    queue.append(node.left)
                if node.right:
                    queue.append(node.right)
            result.append(val)
        return result
```