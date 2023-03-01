# 分治和回溯

8 Jul 2020

---

## 解决算法问题的乐趣

以LeetCode平台为例

- 问题聚焦，需求明确
- 测试完备，性能敏感

学生时代的际遇：汉诺塔问题

```text
初中时代，同学游戏机里初遇：
求对于n个盘子，移动到最终位置需要的最少步数
```

![hanoi](/hanio1.png)

---

$f(n) = 2 * f(n-1) + 1$

<img src="/hanio2.png" class="h-100">

---

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

> $f(n) = 2f(n-1) + 1$
>
> $f(n) + 1 = 2[f(n-1) + 1]$
>
> > $f(n) + 1$ 是个等比数列， 首项是$f(1) + 1 = 2$, 公比是$2$
>
> $f(n) + 1 = 2^n$
>
> $f(n) = 2^n - 1$

时间复杂度由 _o(n)_ 降为 _O(1)_

```go
func hanioSolutions(n int) int {
    return (1<<n) + 1
}
```

---

分治：分而治之，化整为零

> 二分搜索、快排、归并排序、动态规划等

回溯：穷举尝试，有错就改

> 数组、链表、树、图、二维矩阵的DFS等

---

## 分治思想应用的例子

[215.数组中的第 K 个最大元素](https://leetcode-cn.com/problems/kth-largest-element-in-an-array)

```text
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
```

朴素实现，排序后再求解，时间复杂度 O(nlgn)，空间复杂度 O(1)

```go
func findKthLargest(nums []int, k int) int {
    sort.Sort(sort.Reverse(sort.IntSlice(nums)))
    return nums[k-1]
}
```

---

### 借助堆的解法

借助一个小顶堆，将nums里的元素一一入堆，但需要保持堆的大小最多为k，如果超出k，需要把堆顶元素出堆

```go
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

```

> 时间复杂度O(nlgk),空间复杂度O(k)；并不比朴素实现快多少
>
> IntHeap 实现略，借助标注库 `container/heap` 即可

---

### 快速选择算法

参照快排的思路，可以不必完全排完序

```go
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
```

---

```go
// 随机选择基准元素，大于基准的放在左侧，小于基准的放在右侧
// 返回最终基准元素的索引
func partition(nums []int, left, right int) int {
	pivotIndex := left + rand.Intn(right-left+1)
	pivot := nums[pivotIndex]
	nums[right], nums[pivotIndex] = nums[pivotIndex], nums[right] // 1. 先把基准元放到最后
	storeIndex := left
	for i := left; i < right; i++ { // 2. 所有不小于基准元的元素放到左侧
		if nums[i] >= pivot {
			nums[storeIndex], nums[i] = nums[i], nums[storeIndex]
			storeIndex++
		}
	}
	nums[storeIndex], nums[right] = nums[right], nums[storeIndex] // 3. 基准元放到最终位置
	return storeIndex
}
```

时间复杂度 : 平均情况 O(N)，最坏情况 O(N^2)。空间复杂度 : O(lgn)，栈空间大小。

> 《算法导论》 9.2 期望为线性的选择算法

---

[23. 合并 K 个排序链表](https://leetcode-cn.com/problems/merge-k-sorted-lists)

```text
合并 k 个排序链表，返回合并后的排序链表。请分析和描述算法的复杂度。

示例:

输入:
[
  1->4->5,
  1->3->4,
  2->6
]
输出: 1->1->2->3->4->4->5->6
```

这里只涉及到“治”，也就是合并，没有涉及"分“；其实可以考虑该问题的全貌：
```text
有一个比较大的链表，需要对其排序
1. 可以先分割成n个小链表，并开辟n个协程对每个小链表排序
2. 最后合并这些小链表，并保持最终大链表有序
```

---

首先实现将两个链表合并的 merge 函数

```go
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
```

---

主体代码-实现 1：

```go
func mergeKLists(lists []*ListNode) *ListNode {
	var r *ListNode
    for _, v := range lists {
    	r = merge(r, v)
    }
    return r
}
```

合并并不均衡，考虑如下情形：

```text
子链表1： 1 -> 5 -> 8
子链表2： 2 -> 6 -> 7
子链表3： 3 -> 9
子链表4： 4 -> 7 -> 8 -> 10
...
```

> 按照当前的实现，会先合并子链表 1 和 2，长度变成 6；再和子链表 3 合并，长度变成 8；再和子链表 4 合并，长度变成 4；...
> 
> _到后边会发现要合并的两个链表，长度非常不均衡，第一个非常长，第二个比较短_

---

优化合并过程：

`先两两合并子链表：1和2合并成链表x，2和4合并成链表y，...；再继续两两合并：x和y合并成链表a ...`

以上合并方法同数组的归并排序合并方法，能有效优化合并时间

```go
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
```

---

### 分治：动态规划

如前边讨论到的汉诺塔问题递推公式，再如：[面试题 08.01. 三步问题](https://leetcode-cn.com/problems/three-steps-problem-lcci)

```text
有个小孩正在上楼梯，楼梯有n阶台阶，小孩一次可以上1阶、2阶或3阶。
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
```

---

经典 dp 问题

```go
// 假设 dp(n)表示到达第 n 个阶梯可能的方式数；
// 考虑倒数第二步：可以在第 n-1、n-2、n-3 这 3 个台阶上，然后一步就可以到达终点
// 所以 dp(n) = dp(n-1) + dp(n-2) + dp(n-3)
// 初始状态容易得到：dp(1), dp(2), dp(3) = 1, 2, 4
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
```

---

## 回溯

穷举尝试，有错就改

```text
一般具体实现为深度优先搜索，可应用到数组、链表、树、图、二维矩阵等多种数据结构相关的问题
```

```go
    // 回溯算法的框架
    result = []
    func backtrack(选择列表,路径):
        if 满足结束条件:
            result.add(路径)
            return
        for 选择 in 选择列表:
            做选择
            backtrack(选择列表,路径)
            撤销选择
```

---

#### 回溯函数的几种写法

回溯算法有一点和二分搜索很像：思想很简单，细节是魔鬼

[面试题 08.09](https://leetcode-cn.com/problems/bracket-lcci)

```text
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
```

---

写法一(框架)：

```go
func generateParenthesis(n int) []string {
	result := make([]string, 0)
	curr := make([]byte, 0, 2*n) // 记录当前构建的一个可能串
	left, right := 0, 0          // 记录当前构建的串里左右括号的个数

    // 回溯函数没有入参，也没有返回值
	var backtrace func()

	// 结果中已经出现left个左括号，right个右括号
	backtrace = func() {
		// ...
	}
	backtrace()
	return result
}
```

---

写法一（backtrace实现）：

```go
	backtrace = func() {
		if left > n || right > left { // 递归结束条件 1
			return
		}
		if len(curr) == cap(curr) { // 等价于： left == right == n； 递归结束条件 2
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
```

---

写法二:

`将 left、right 作为参数，这样可以减少回溯代码`

```go
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
```

---

写法三：

`left，right，cur 都可以作为参数传递`

```go
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
```

---

写法四：

`result 作为回溯函数的返回值`

```go
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
        backtrace(left+1, right, curr+"(") // 尝试追加一个左括号
        backtrace(left, right+1, curr+")") // 尝试追加一个右括号
        return result
    }
    return backtrace(0, 0, "")
}
```

> 这个问题写成写法四意义不大；在记忆化搜索优化回溯过程的场景中有意义，比如下边机器人的例子。

---

## 回溯的优化

[63. 不同路径 II](https://leetcode-cn.com/problems/unique-paths-ii)

```text
一个机器人位于一个 m x n 网格的左上角 （起始点在下图中标记为“Start” ）。
机器人每次只能向下或者向右移动一步。机器人试图达到网格的右下角（在下图中标记为“Finish”）。
现在考虑网格中有障碍物。那么从左上角到右下角将会有多少条不同的路径？
```

![robot](/robot_maze.png)

---

```text
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
```

---

我们先尝试直接递归回溯

```go
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
```

时间复杂度 $O(2^{m+n})$， 空间复杂度 $O(m+n)$，主要为递归栈的空间

---

优化思路一：给递归函数加上备忘录，来避免重复的计算

```go
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
```

---

给递归加上备忘录的做法也叫`记忆化搜索`，是对自顶向下的递归时间上的一个优化，时空复杂度都变成了 $O(mn)$

还可以自底向上用动态规划的方法来解决

```go
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
```

---

## 小结

- 分治：分而治之，化整为零
- 回溯：穷举尝试，有错就改

分治思想应用的例子：

`快排、归并排序、动态规划等`

回溯思想应用的例子：

`数组、链表、树、图、二维矩阵的DFS等`

