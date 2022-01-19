# leetcode求根节点到叶节点数字之和题解

关键点：
1. 当前路径和=父路径和×10+当前节点值；
2. 如果当前节点是叶子节点，则求和、结束，否则继续递归遍历左右子节点。

```go
func sumNumbers(root *TreeNode) int {
	var sum int
	sumPath(&sum, 0, root)
	return sum
}

func sumPath(sum *int, parent int, root *TreeNode) {
	if root == nil {
		// 边界条件处理
		return
	}
	// 当前路径和=父路径和*10+当前节点值
	current := parent*10 + root.Val
	if root.Left == nil && root.Right == nil {
		// 如果是叶子节点，则求和
		*sum += current
		return
	}
	// 不是叶子节点，则继续递归遍历
	sumPath(sum, current, root.Left)
	sumPath(sum, current, root.Right)
}
```
