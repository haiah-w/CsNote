深度优先搜索

使用DFS的场景：
- 全排列问题
- 路径问题，穷举每一条路径，到达终点/阻塞，再回溯继续别的路径；（BFS也可）

DFS方法：递归 + 回溯
```go
dfs = func(start, count int) {
    // 终止条件
    if count >= k {
      return
    }
    for i := start; i <= n; i++ {
        // 根据具体场景，处理当前
        count++
        dfs(i+1, count)
        // 回溯：返回时，则回到上一个状态
    }
}
```
