# leetcode合并K个升序链表题解

[题目描述](https://leetcode-cn.com/problems/merge-k-sorted-lists/)

先说关键点：
1. 边界条件判断，如果链表数组长度小于2，则直接返回；
2. 使用preHead伪节点，也就是最终输出链表中头节点的前一个节点，这个思想在k个一组翻转链表里也有体现，这样在循环时就不用对头节点做特殊判断；
3. 递归分别对链表数组的前后两个部分合并，然后再合并返回的两个链表

代码如下：
```go
/**
 * Definition for singly-linked list.
 * type ListNode struct {
 *     Val int
 *     Next *ListNode
 * }
 */
func mergeKLists(lists []*ListNode) *ListNode {
	// 先进行边界条件判断
	if len(lists) == 0 {
		return nil
	}
	if len(lists) == 1 {
		return lists[0]
	}
	// 使用preHead简化判断逻辑
	preHead := new(ListNode)
	current := preHead
	// 递归
	currentA := mergeKLists(lists[:len(lists)/2])
	currentB := mergeKLists(lists[len(lists)/2:])
	// 循环比较
	for currentA != nil && currentB != nil {
		if currentA.Val < currentB.Val {
			current.Next, currentA = currentA, currentA.Next
		} else {
			current.Next, currentB  = currentB, currentB.Next
		}
		current = current.Next
	}
	// 挂载剩余链表
	if currentA != nil {
		current.Next = currentA
	} else if currentB != nil {
		current.Next = currentB
	}
	return preHead.Next
}
```
