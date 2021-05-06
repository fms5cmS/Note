# Binary Search

```go
func BinarySearch(nums []int, target int) int {
	low, high := 0, len(nums)-1
	// 1.这里的循环退出条件必须是 low<=high
	for low <= high {
		// 2.这里不写成 (low+high)/2 是为了防止 low+high 溢出，使用位运算符可提高性能
		// 注意，Go 中位运算符的优先级和乘除运算符是同级的！而其他语言中位运算符则低于加减
		mid := low + (high-low)>>1
		// 3.注意 high、low 更新的值，并不是等于 mid
		// 否则，当 high=low=mid 时，就会陷入死循环
		if nums[mid] == target {
			return mid
		} else if nums[mid] > target {
			high = mid - 1
		} else {
			low = mid + 1
		}
	}
	return -1
}
```

- 二分查找需要按照下标随机访问元素，所以其依赖于顺序表结构，也即数组，如果是其他数据结构，则时间复杂度很高！
- 二分查找针对的是**有序数据**！
- 数据量太小不适合二分查找，顺序遍历即可
  - 注意：如果数据之间的比较操作非常耗时，不管数据量大小，都推荐使用二分查找。
- 数据量太大也不适合二分查找，因为二分查找底层依赖于数组，数组为了支持随机访问的特性，要求内存空间**连续**，对内存的要求比较苛刻。
- 二分查找更适合处理静态数据，也就是没有频繁的数据插入、删除操作。
- **能用二分查找解决的，绝大部分我们更倾向于用散列表或者二叉查找树，二分查找更适合用在“近似”查找问题，如后面的二分查找的变形问题！！**

递归形式的二分查找：

```go
func BinarySearchRecursive(nums []int, target int) int {
	return bSearch(nums, 0, len(nums)-1, target)
}

func bSearch(nums []int, low, high, target int) int {
	if low > high {
		return -1
	}
	mid := low + (high-low)>>1
	if nums[mid] == target {
		return mid
	} else if nums[mid] > target {
		return bSearch(nums, low, mid-1, target)
	} else {
		return bSearch(nums, mid+1, high, target)
	}
}
```



# 变形问题

- 查找第一个等于给定值的元素

```go
func FirstElement(nums []int, target int) int {
	low, high := 0, len(nums)-1
	for low <= high {
		mid := low + (high-low)>>1
		if nums[mid] == target {
			if mid == 0 || nums[mid-1] != target {
				return mid
			}
			high = mid - 1
		} else if nums[mid] > target {
			high = mid - 1
		} else {
			low = mid + 1
		}
	}
	return -1
}
```

---

- 查找最后一个等于给定值的元素

```go
func LastElement(nums []int, target int) int {
	low, high := 0, len(nums)-1
	for low <= high {
		mid := low + (high-low)>>1
		if nums[mid] == target {
			if mid == len(nums)-1 || nums[mid+1] != target {
				return mid
			}
			low = mid + 1
		} else if nums[mid] > target {
			high = mid - 1
		} else {
			low = mid + 1
		}
	}
	return -1
}
```

---

- 查找第一个大于等于给定值的元素

```go
func FirstGreaterOrEqualElement(nums []int, target int) int {
	low, high := 0, len(nums)-1
	for low <= high {
		mid := low + (high-low)>>1
		if nums[mid] >= target {
			high = mid - 1
		} else {
			low = mid + 1
		}
	}
	return -1
}
```

---

- 查找最后一个小于等于给定值的元素

```go
func LastLessOrEqualElement(nums []int, target int) int {
	low, high := 0, len(nums)-1
	for low <= high {
		mid := low + (high-low)>>1
		if nums[mid] <= target {
			if mid == len(nums)-1 || nums[mid+1] > target {
				return mid
			}
			low = mid + 1
		} else {
			high = mid - 1
		}
	}
	return -1
}
```

