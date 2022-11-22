# DFS

深度优先搜索，属于搜索算法，暴力枚举；

# 适用场景

- 全排列、排列组合
- 岛屿问题，由一个点，通过DFS遍历所有关联节点；
- 路径问题，穷举每一条路径，到达终点/阻塞，再回溯继续别的路径；（BFS也可）

# 思路

[dfs、回溯专题刷题总结 - 力扣（LeetCode）](https://leetcode.cn/circle/article/sZEj8D/)

[深度优先搜索/遍历（DFS） - 力扣（LeetCode）](https://leetcode.cn/circle/article/l6ghwb/)

DFS方法：递归 + 回溯

```go
dfs = func(start, count int) {
    // 终止条件
    if count >= k {
      return
    }
    for i := start; i <= n; i++ {
        // 根据具体场景，处理当前，也可以剪枝
        count++
        dfs(i+1, count)
        // 回溯：返回时，则回到上一个状态
        count--
    }
}
```
