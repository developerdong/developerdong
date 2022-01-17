# leetcode合并两个有序数组题解

第一版实现如下：

```go
func merge(nums1 []int, m int, nums2 []int, n int) {
	// 任何一个切片为nil的情况都无法处理，直接返回
	if nums1 == nil || nums2 == nil {
		return
	}
	// 如果第一个切片长度为0，则直接把第二个切片的所有元素拷贝过去
	if m == 0 {
		// 虽然修改内部的nums1不会影响外面实参的SliceHeader，但因为共享底层数组，所以只要不发生扩容，还是可以体现效果
		nums1 = append(nums1[:0], nums2...)
		return
	}
	// 比较插入
	index1 := 0
	for index2 := 0; index2 < n; index2++ {
		for index1 < m+index2 {
			if nums2[index2] < nums1[index1] {
				break
			}
			index1++
		}
		nums1 = append(nums1[:index1], append([]int{nums2[index2]}, nums1[index1:m+index2]...)...)
	}
}
```

第一版实现从前往后比较插入，每次都需要移动很多元素，虽然通过了测试，
但总感觉不优雅。下班坐地铁的时候突然想到可以从后往前比较，
这样就不需要移动元素了，所以第二版实现如下：

```go
func merge(nums1 []int, m int, nums2 []int, n int) {
	// 任何一个切片为nil的情况都无法处理，直接返回
	if nums1 == nil || nums2 == nil {
		return
	}
	// 比较插入
	index, index1, index2 := m+n-1, m-1, n-1
	for index >= 0 {
		if index1 >= 0 && index2 >= 0 {
			if nums1[index1] < nums2[index2] {
				nums1[index] = nums2[index2]
				index2--
			} else {
				nums1[index] = nums1[index1]
				index1--
			}
		} else if index1 >= 0 {
			nums1[index] = nums1[index1]
			index1--
		} else if index2 >= 0 {
			nums1[index] = nums2[index2]
			index2--
		} else {
			break
		}
		index--
	}
}
```

关键思想就是从后往前比较合并。
