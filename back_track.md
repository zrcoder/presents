# 算法题目精讲——分治和回溯

8 Jul 2020

zrcoder

love-nankai@163.com

---

* 解决算法问题的乐趣

`以LeetCode平台为例`

- 问题聚焦，需求明确
- 测试完备，性能敏感

学生时代的际遇：汉诺塔问题

`初中时代，同学游戏机里初遇：`

    求对于n个盘子，移动到最终位置需要的最少步数  
    
.image https://raw.githubusercontent.com/zrcoder/presents/main/backtrack/hanio1.png

* 解决算法问题的乐趣
    
`f(n) = 2 * f(n-1) + 1`
    
.image https://raw.githubusercontent.com/zrcoder/presents/main/backtrack/hanio2.png 
---

* 解决算法问题的乐趣

```go
func hanioSolutions(n int) int {
    dp := 1
    for i := 2; i <= n; i++ {
        dp = 2*dp + 1
    }
    return dp
}
```

高中时代，学习了数列的递推公式和通项公式

```
f(n) = 2f(n-1) + 1
f(n) + 1 = 2[f(n-1) + 1]
f(n) + 1 是个等比数列， 首项是f(1) + 1 = 2, 公比是2

f(n) + 1 = 2^n
f(n) = 2^n - 1
```

时间复杂度由 *o(n)* 降为 *O(1)*
    
```go
func hanioSolutions(n int) int {
    return (1<<n) + 1
}
```
---

* 分治和回溯
- 分治：分而治之，化整为零
- 回溯：穷举尝试，有错就改

分治思想应用的例子：

    二分搜索、快排、归并排序、动态规划等
    
回溯思想应用的例子：

    数组、链表、树、图、二维矩阵的DFS等
    
---

* 分治思想应用的例子
.link https://leetcode-cn.com/problems/kth-largest-element-in-an-array 215. 数组中的第K个最大元素
    在未排序的数组中找到第 k 个最大的元素。请注意，你需要找的是数组排序后的第 k 个最大的元素，
    而不是第 k 个不同的元素。
    
    示例 1:
    输入: [3,2,1,5,6,4] 和 k = 2
    输出: 5
    
    示例 2:
    输入: [3,2,3,1,2,4,5,5,6] 和 k = 4
    输出: 4
    
    说明:
    你可以假设 k 总是有效的，且 1 ≤ k ≤ 数组的长度。
    
朴素实现，排序后再求解，时间复杂度O(nlgn)，空间复杂度O(1)

```go
func findKthLargest(nums []int, k int) int {
    sort.Sort(sort.Reverse(sort.IntSlice(nums)))
    return nums[k-1]
}
```
---

* 分治思想应用的例子
借助堆的解法

    借助一个小顶堆，将nums里的元素一一入堆，
    但需要保持堆的大小最多为k，如果超出k，需要把堆顶元素出堆
    最后堆顶元素就是结果

`时间复杂度O(nlgk),空间复杂度O(k)；并不比朴素实现快多少`

    func findKthLargest(nums []int, k int) int {
        h := &IntHeap{}
        for _, v := range nums {
            heap.Push(h, v)
            if h.Len() > k {
                _ = heap.Pop(h)
            }
        }
        return (*h)[0]
    }
    
---

* 分治思想应用的例子
IntHeap相关实现如下

`Go标准库里的heap使用起来不是很方便，其实也可以重新设计，有兴趣可以参看如下博客：`

.link https://github.com/zrcoder/myGo/blob/master/present/2020/heap/readme.md 重新设计 heap 包

    type IntHeap []int
    
    func (h IntHeap) Len() int            { return len(h) }
    func (h IntHeap) Less(i, j int) bool  { return h[i] < h[j] }
    func (h IntHeap) Swap(i, j int)       { h[i], h[j] = h[j], h[i] }
    func (h *IntHeap) Push(x interface{}) { *h = append(*h, x.(int)) }
    func (h *IntHeap) Pop() interface{} {
    	x := (*h)[len(*h)-1]
    	*h = (*h)[:len(*h)-1]
    	return x
    }
    
---

* 分治思想应用的例子
参照快排的思路，可以不必完全排完序

    func findKthLargest(nums []int, k int) int {
        if k < 1 || k > len(nums) {
            return 0
        }
        quickSelect(nums, 0, len(nums)-1, k)
        return nums[k-1]
    }
    
    func quickSelect(nums []int, left, right, k int) {
    	if left == right { // 递归结束条件：区间里仅有一个元素
    		return
    	}
    	pivotIndex := partition(nums, left, right)
    	if pivotIndex+1 == k {
    		return
    	}
    	if pivotIndex+1 > k {
    		quickSelect(nums, left, pivotIndex-1, k)
    	} else {
    		quickSelect(nums, pivotIndex+1, right, k)
    	}
    }
---

* 分治思想应用的例子

    // 随机选择基准元素，大于基准的放在左侧，小于基准的放在右侧
    // 返回最终基准元素的索引
    func partition(nums []int, left, right int) int {
    	pivotIndex := left + rand.Intn(right-left+1)
    	pivot := nums[pivotIndex]    	
    	nums[right], nums[pivotIndex] = nums[pivotIndex], nums[right] // 1. 先把基准元放到最后
    	storeIndex := left    	
    	for i := left; i < right; i++ {                               // 2. 所有不小于基准元的元素放到左侧
    		if nums[i] >= pivot {
    			nums[storeIndex], nums[i] = nums[i], nums[storeIndex]
    			storeIndex++
    		}
    	}    	
    	nums[storeIndex], nums[right] = nums[right], nums[storeIndex] // 3. 基准元放到最终位置
    	return storeIndex
    }
    
时间复杂度 : 平均情况O(N)，最坏情况O(N^2)。空间复杂度 : O(lgn)，栈空间大小。

    《算法导论》 9.2 期望为线性的选择算法
---

* 分治思想应用的例子
.link https://leetcode-cn.com/problems/merge-k-sorted-lists 23. 合并K个排序链表

    合并 k 个排序链表，返回合并后的排序链表。请分析和描述算法的复杂度。
    
    示例:
    
    输入:
    [
      1->4->5,
      1->3->4,
      2->6
    ]
    输出: 1->1->2->3->4->4->5->6

这里只涉及到“治”，也就是合并，没有涉及"分“；其实可以考虑该问题的全貌：

    有一个比较大的链表，需要对其排序
    1. 可以先分割成n个小链表，并开辟n个协程对每个小链表排序
    2. 最后合并这些小链表，并保持最终大链表有序
    
---

* 分治思想应用的例子
首先实现将两个链表合并的merge函数

    func merge(l1, l2 *ListNode) *ListNode {
    	dummy := new(ListNode)
    	for p := dummy; l1 != nil || l2 != nil; p = p.Next {
    		if l1 != nil && l2 != nil && l1.Val < l2.Val || l2 == nil {
    			p.Next = l1
    			l1 = l1.Next
    		} else {
    			p.Next = l2
    			l2 = l2.Next
    		}
    	}
    	l1, dummy.Next = dummy.Next, nil
    	return l1
    }
---

* 分治思想应用的例子

主体代码-实现1：

    func mergeKLists(lists []*ListNode) *ListNode {
    	var r *ListNode
        for _, v := range lists {
        	r = merge(r, v)
        }
        return r
    }
    
实现1的合并并不均衡，考虑如下情形：

    子链表1： 1 -> 5 -> 8
    子链表2： 2 -> 6 -> 7
    子链表3： 3 -> 9
    子链表4： 4 -> 7 -> 8 -> 10
    ...
    
按照当前的实现，会先合并子链表1和2，长度变成6；再和子链表3合并，长度变成8；再和子链表4合并，长度变成4；...

*到后边会发现要合并的两个链表，长度非常不均衡，第一个非常长，第二个比较短*
---

* 分治思想应用的例子
优化合并过程：

    先两两合并子链表：1和2合并成链表x，2和4合并成链表y，...；再继续两两合并：x和y合并成链表a ...
    
以上合并方法同数组的归并排序合并方法，能有效优化合并时间

    LeetCode实测，优化后耗时从220ms降低到了10ms

主体代码-实现2：

    func mergeKLists(lists []*ListNode) *ListNode {
    	n := len(lists)
    	if n == 0 {
    		return nil
    	}
    	for interval := 1; interval < n; interval *= 2 {
    		for i := 0; i+interval < n; i += interval * 2 {
    			lists[i] = merge(lists[i], lists[i+interval])
    		}
    	}
    	return lists[0]
    }
    
---

* 分治思想应用的例子
动态规划，如前边讨论到的汉诺塔问题递推公式，再如：

.link https://leetcode-cn.com/problems/three-steps-problem-lcci 面试题08.01. 三步问题

    三步问题。有个小孩正在上楼梯，楼梯有n阶台阶，小孩一次可以上1阶、2阶或3阶。
    实现一种方法，计算小孩有多少种上楼梯的方式。结果可能很大，你需要对结果模1000000007。
    
    示例1:    
     输入：n = 3 
     输出：4
     说明: 有四种走法
     
    示例2:    
     输入：n = 5
     输出：13
    提示:
    
    n范围在[1, 1000000]之间
---

* 分治思想应用的例子

    假设dp(n)表示到达第n个阶梯可能的方式数；
    考虑倒数第二步：可以在第n-1、n-2、n-3这3个台阶上，然后一步就可以到达终点
    所以dp(n) = dp(n-1) + dp(n-2) + dp(n-3)
    初始状态容易得到：dp(1), dp(2), dp(3) = 1, 2, 4

经典dp问题，代码如下

    func waysToStep(n int) int {
        if n == 1 {
            return 1
        }
        if n == 2 {
            return 2
        }
        if n == 3 {
            return 4
        }
        dp1, dp2, dp3 := 1, 2, 4
        for i := 4; i <= n; i++ {
            dp1, dp2, dp3 = dp2, dp3, (dp1+dp2+dp3)%1000000007
        }
        return dp3
    }
    
---

* 回溯思想应用的例子
穷举尝试，有错就改

    一般具体实现为深度优先搜索，可应用到数组、链表、树、图、二维矩阵等多种数据结构相关的问题

回溯算法的框架
    
    result = []
    func backtrack(选择列表,路径):
        if 满足结束条件:
            result.add(路径)
            return
        for 选择 in 选择列表:
            做选择
            backtrack(选择列表,路径)
            撤销选择
    
---

* 回溯思想应用的例子
.link https://leetcode-cn.com/problems/subsets 78. 子集

    给定一组不含重复元素的整数数组 nums，返回该数组所有可能的子集（幂集）。
    说明：解集不能包含重复的子集。
    
    示例:
    输入: nums = [1,2,3]
    输出:
    [
      [3],
      [1],
      [2],
      [1,2,3],
      [1,3],
      [2,3],
      [1,2],
      []
    ]
---

* 回溯思想应用的例子
    
    func subsets(nums []int) [][]int {
        var result [][]int
        var backtrack func(start int, curr []int)
        backtrack = func(start int, curr []int) {
            result = append(result, copySlice(curr))
            for i := start; i < len(nums); i++ {
                curr = append(curr, nums[i])
                backtrack(i+1, curr)
                curr = curr[:len(curr)-1]
            }
        }
        backtrack(0, nil)
        return result
    }
    
辅助函数copySlice：

    func copySlice(s []int) []int {
        r := make([]int, len(s))
        _ = copy(r, s)
        return r
    }
---

* 回溯思想应用的例子
.link https://leetcode-cn.com/problems/path-sum-ii 113. 路径总和 II

    给定一个二叉树和一个目标和，找到所有从根节点到叶子节点路径总和等于给定目标和的路径。
    说明: 叶子节点是指没有子节点的节点。
    
    示例:
    给定如下二叉树，以及目标和 sum = 22，
    
                  5
                 / \
                4   8
               /   / \
              11  13  4
             /  \    / \
            7    2  5   1
    返回:
    [
       [5,4,11,2],
       [5,8,4,5]
    ]
    
用一个切片path记录遍历的路径，到达叶子节点发现path内元素和为sum则将当期path添加到结果里
---

* 回溯思想应用的例子

    func pathSum(root *TreeNode, sum int) [][]int {
        var result [][]int
        var path []int
        prefixSum := 0
        var dfs func(*TreeNode)
        dfs = func(node *TreeNode) {
            if node == nil {
                return
            }
            path = append(path, node.Val)
            prefixSum += node.Val
            if node.Left == nil && node.Right == nil && prefixSum == sum {
                tmp := make([]int, len(path))
                _ = copy(tmp, path)
                result = append(result, tmp)
            }
            dfs(node.Left)
            dfs(node.Right)
            path = path[:len(path)-1]
            prefixSum -= node.Val
        }
        dfs(root)
        return result
    }
---

* 回溯思想应用的例子
.link https://leetcode-cn.com/problems/n-queens 51. N皇后

    n 皇后问题研究的是如何将 n 个皇后放置在 n×n 的棋盘上，并且使皇后彼此之间不能相互攻击。
    给定一个整数 n，返回所有不同的 n 皇后问题的解决方案。
    
.image https://raw.githubusercontent.com/zrcoder/presents/main/backtrack/8-queens.png
---

* 回溯思想应用的例子

    每一种解法包含一个明确的 n 皇后问题的棋子放置方案，该方案中 'Q' 和 '.' 分别代表了皇后和空位。
    示例:
    输入: 4
    输出: [
     [".Q..",  // 解法 1
      "...Q",
      "Q...",
      "..Q."],
    
     ["..Q.",  // 解法 2
      "Q...",
      "...Q",
      ".Q.."]
    ]
    解释: 4 皇后问题存在两个不同的解法。
    
---

* 回溯思想应用的例子
常规回溯

    红色表示已经放置了Q，黑色表示不能放置Q的位置，蓝色表示可以放置Q的位置
    
.image https://raw.githubusercontent.com/zrcoder/presents/main/backtrack/queen.png
    
    省略第一个Q在第一行第三格和第一行最后一格的图示
    时间复杂度O(n!)；空间复杂度O(n)
    
---

* 回溯思想应用的例子

    func solveNQueens(n int) [][]string {
        var result [][]string
        backtrack(0, makeBoard(n), &result)
        return result
    }

backtrack第一个参数代表行
    
makeBoard返回一个n*n的[][]byte类型矩阵，初始所有元素为'.'， 代码略
---

* 回溯思想应用的例子
我们来看看核心的回溯函数

    // 在行r找到合适的列放置皇后
    func backtrack(r int, board [][]byte, result *[][]string) {
    	if r == len(board) {
    		*result = append(*result, parse(board))
    		return
    	}
    	for c := 0; c < len(board); c++ {
    		if !canSetQueen(board, r, c) {
    			continue
    		}
    		board[r][c] = 'Q' // 在(r, c)这个位置放一个皇后
    		backtrack(r+1, board, result)
    		board[r][c] = '.' // 回溯
    	}
    }

parse函数用来把[][]byte类型的board转换成[]string，代码略
---

* 回溯思想应用的例子
canSetQueen用来判断位置(r, c)处放一个皇后是否会和同行、同列或同斜线已有皇后冲突
    
    func canSetQueen(board [][]byte, r, c int) bool {
        // 根据回溯的主体逻辑，只需要检查正上方、左上斜线和右上斜线有没有皇后冲突
    	var i, j int
    	for i = 0; i < r; i++ { // 头顶一列
    		if board[i][c] == 'Q' {
    			return false
    		}
    	}
    	for i, j = r-1, c-1; i >= 0 && j >= 0; i, j = i-1, j-1 { // 左上斜线
    		if board[i][j] == 'Q' {
    			return false
    		}
    	}
    	for i, j = r-1, c+1; i >= 0 && j < len(board); i, j = i-1, j+1 { // 右上斜线
    		if board[i][j] == 'Q' {
    			return false
    		}
    	}
    	return true
    }
---

* 回溯函数的几种写法

    回溯算法有一点和二分搜索很像：思想很简单，细节是魔鬼

.link https://leetcode-cn.com/problems/bracket-lcci 面试题 08.09

    设计一种算法，打印n对括号的所有合法的组合。    
    说明：解集不能包含重复的子集。    
    例如，给出 n = 3，生成结果为：    
    [
      "((()))",
      "(()())",
      "(())()",
      "()(())",
      "()()()"
    ]
    
---

* 回溯函数的几种写法
写法一：程序框架

    func generateParenthesis(n int) []string {
    	result := make([]string, 0)
    	curr := make([]byte, 0, 2*n) // 记录当前构建的一个可能串
    	left, right := 0, 0          // 记录当前构建的串里左右括号的个数
    	var backtrace func()
    	// 结果中已经出现left个左括号，right个右括号
    	backtrace = func() {
    		// ...
    	}
    	backtrace()
    	return result
    }

`回溯函数没有入参，也没有返回值`
    
---

* 回溯函数的几种写法
            
        backtrace = func() {
            if left > n || right > left { // 递归结束条件1
                return
            }
            if len(curr) == cap(curr) { // 等价于： left == right == n； 递归结束条件2
                result = append(result, string(curr))
                return
            }
            // 尝试追加一个左括号
            left++
            curr = append(curr, '(')
            backtrace()
            // 回溯
            left--
            curr = curr[:len(curr)-1]        
            // 尝试追加一个右括号
            right++
            curr = append(curr, ')')
            backtrace()
            // 回溯
            right--
            curr = curr[:len(curr)-1]
        }
---

* 回溯函数的几种写法
写法二:将left、right作为参数，这样可以减少回溯代码

    func generateParenthesis(n int) []string {
        // ...
    	var backtrace func(int, int)
    	backtrace = func(left, right int) {
    		if left > n || right > left {
    			return
    		}
    		if len(curr) == cap(curr) {
    			result = append(result, string(curr))
    			return
    		}
    		curr = append(curr, '(')
    		backtrace(left+1, right)
    		curr = curr[:len(curr)-1]    
    		curr = append(curr, ')')
    		backtrace(left, right+1)
    		curr = curr[:len(curr)-1]
    	}
    	backtrace(0, 0)
    	return result
    }
---

* 回溯函数的几种写法
写法三：left，right，cur都可以作为参数传递

    func generateParenthesis(n int) []string {
        result := make([]string, 0)   
        
        var backtrace func(left, right int, curr []byte) []string
        backtrace = func(left, right int, curr []byte) []string {
            if left > n || right > left { // 递归结束条件1
                return result
            }
            if len(curr) == 2*n {
                result = append(result, string(curr))
                return result
            }
            // 尝试追加一个左括号
            backtrace(left+1, right, append(curr, '('))
            // 尝试追加一个右括号
            backtrace(left, right+1, append(curr, ')'))
            return result
        }
        return backtrace(0, 0, nil)
    }
---

* 回溯函数的几种写法
写法四：result作为回溯函数的返回值

    func generateParenthesis(n int) []string {
        result := make([]string, 0)
        var backtrace func(int, int, string) []string
        backtrace = func(left, right int, curr string) []string {
            if left > n || right > left {
                return result
            }
            if len(curr) == 2*n {
                result = append(result, curr)
                return result
            }
            // 尝试追加一个左括号
            backtrace(left+1, right, curr+"(")  
            // 尝试追加一个右括号
            backtrace(left, right+1, curr+")")
            return result
        }
        return backtrace(0, 0, "")
    }    
    
    这个问题写成写法四意义不大；在类似后边机器人的例子里会比较有意义
  
---

* 回溯的优化

.link https://leetcode-cn.com/problems/unique-paths-ii 63. 不同路径 II

    一个机器人位于一个 m x n 网格的左上角 （起始点在下图中标记为“Start” ）。    
    机器人每次只能向下或者向右移动一步。机器人试图达到网格的右下角（在下图中标记为“Finish”）。    
    现在考虑网格中有障碍物。那么从左上角到右下角将会有多少条不同的路径？    
    
.image https://raw.githubusercontent.com/zrcoder/presents/main/backtrack/robot_maze.png
---

* 回溯的优化
   
    网格中的障碍物和空位置分别用 1 和 0 来表示。    
    说明：m 和 n 的值均不超过 100。    
    示例 1:    
    输入:
    [
      [0,0,0],
      [0,1,0],
      [0,0,0]
    ]
    输出: 2
    解释:
    3x3 网格的正中间有一个障碍物。
    从左上角到右下角一共有 2 条不同的路径：
    1. 向右 -> 向右 -> 向下 -> 向下
    2. 向下 -> 向下 -> 向右 -> 向右
    
我们先尝试直接递归回溯
---

* 回溯的优化

    func uniquePathsWithObstacles(obstacleGrid [][]int) int {
        if len(obstacleGrid) == 0 || len(obstacleGrid[0]) == 0 {
            return 0
        }
        m, n := len(obstacleGrid), len(obstacleGrid[0])
        if obstacleGrid[0][0] == 1 || obstacleGrid[m-1][n-1] == 1 {
            return 0
        }
        
        var dfs func(r, c int) int // dfs(r, c)表示从位置(r,c)到达终点可行的方法数
        dfs = func(r, c int) int {
            if r >= m || c >= n || obstacleGrid[r][c] != 0 {
                return 0
            }
            if r == m-1 && c == n-1 {
                return 1
            }
            return dfs(r+1, c) + dfs(r, c+1)
        }
        return dfs(0, 0)
    }
    
时间复杂度 O(2^(m+n))， 空间复杂度 O(m+n)，主要为递归栈的空间
    
---

* 回溯的优化
优化思路一：给递归函数加上备忘录，来避免重复的计算

    func uniquePathsWithObstacles(obstacleGrid [][]int) int {
    	// ...    
    	memo := make([][]int, m)
    	for i := range memo {
    		memo[i] = make([]int, n)
    		for j := range memo[i] {
    			memo[i][j] = -1
    		}
    	}
    
    	var dfs func(r, c int) int
    	dfs = func(r, c int) int {
    		// ...
    		if memo[r][c] != -1 {
    			return memo[r][c]
    		}
    		memo[r][c] = dfs(r+1, c) + dfs(r, c+1)
    		return memo[r][c]
    	}
    	return dfs(0, 0)
    }
---

* 回溯的优化

    给递归加上备忘录的做法也叫“记忆化搜索”，是对自顶向下的递归时间上的一个优化
    可以看到时空复杂度都变成了O(m*n)
    
还可以自底向上用动态规划的方法来解决

    其实这个问题更容易想到动态规划的解法
    
---

* 回溯的优化
    
    func uniquePathsWithObstacles(obstacleGrid [][]int) int {
    	// 边界处理代码略
    	m, n := len(obstacleGrid), len(obstacleGrid[0])    	
    	dp := [101][101]int{}    	
    	
    	for r := 0; r < m; r++ {
    		for c := 0; c < n; c++ {
    			if obstacleGrid[r][c] == 1 {
    				continue
    			}
    			if r == 0 && c == 0 {
                    dp[r+1][c+1] = 1
                } else {
                    dp[r+1][c+1] = dp[r][c+1] + dp[r+1][c]
                }
    		}
    	}
    	return dp[m][n]
    }
        
---

* 练习题

.link https://leetcode-cn.com/problems/sudoku-solver 37. 解数独
.link https://leetcode-cn.com/problems/n-queens-ii 52. N皇后 II
.link https://leetcode-cn.com/problems/partition-to-k-equal-sum-subsets 698. 划分为k个相等的子集

.link https://leetcode-cn.com/problems/stickers-to-spell-word 691. 贴纸拼词
.link https://leetcode-cn.com/problems/maximum-students-taking-exam 1349. 参加考试的最大学生数---

---

* 小结
- 分治：分而治之，化整为零
- 回溯：穷举尝试，有错就改

分治思想应用的例子：

    快排、归并排序、动态规划等
    
回溯思想应用的例子：

    数组、链表、树、图、二维矩阵的DFS等
---

* 小结

回溯的写法比较灵活。思想很简单，细节是魔鬼

回溯算法框架

    result = []
    func backtrack(选择列表,路径):
        if 满足结束条件:
            result.add(路径)
            return
        for 选择 in 选择列表:
            做选择
            backtrack(选择列表,路径)
            撤销选择
回溯的优化                
    
    1. 可以用增加备忘录的方式优化时间复杂度
    2. 也可以尝试用自底向上的动态规划方法来代替回溯方法
