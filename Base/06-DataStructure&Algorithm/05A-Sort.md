| [排序算法](https://mp.weixin.qq.com/s?__biz=MzIxMjE5MTE1Nw==&mid=2653200809&idx=1&sn=44ed67f5382b0aea78867b41e92bf3e3&chksm=8c99d373bbee5a653932f01581a8cacbbeaf565b71b7df4698af43d5eabc75e3443d3c80e0ed&scene=0&xtrack=1&key=288743bececaa03fc6424b2c947f5a7d1ec00b8fa3e0af3c6d3ae6a42bd536a5c9e4bcea5d107716dd9de263e4d3e48eb2952ce4c96cb63b5a543efd586eeb62c3253d4688627d9e3c6adb502aca90c2&ascene=1&uin=MjcyNTczMDYwNw%3D%3D&devicetype=Windows+10&version=62070158&lang=zh_CN&exportkey=AyObLMTiCXGdcc%2FPDJL9WOo%3D&pass_ticket=V%2FmcUGgioqB06NFRa5GOypXKE5cdK%2BDHgcKbyW2kRK4CWMdJPcVYybFKdCcKVnHe) | 时间复杂度                       | 稳定性 | 原地排序 |
| ------------------------------------------------------------ | -------------------------------- | ------ | -------- |
| [冒泡排序](https://mp.weixin.qq.com/s?__biz=MzIxMjE5MTE1Nw==&mid=2653194666&idx=1&sn=69ce32870c0b981c40b1e124fbb6bba8&chksm=8c99fb70bbee72668cad223892ad362525d215e7f936458f99dd289eb82981099359310e9e54&scene=21#wechat_redirect) | $O(n^2)$                         | ✓      | ✓        |
| [选择排序](https://mp.weixin.qq.com/s?__biz=MzIxMjE5MTE1Nw==&mid=2653198991&idx=1&sn=7f98d59898a911e1425baa6cc180c598&chksm=8c99e855bbee61439086680ceefef33c56038c5d552ae64c1d6135abe467b617aa62f4934f36&scene=21#wechat_redirect) | $O(n^2)$                         | ✘      | ✓        |
| [插入排序](https://mp.weixin.qq.com/s?__biz=MzIxMjE5MTE1Nw==&mid=2653199343&idx=1&sn=a5491fa908e45e6117423d9ba5062611&chksm=8c99e935bbee60232aacb7c2b74961a24e7b86d44bf98357c597ad277a8eb15639c1de7034d9&scene=21#wechat_redirect) | $O(n^2)$                         | ✓      | ✓        |
| [归并排序](https://mp.weixin.qq.com/s?__biz=MzIxMjE5MTE1Nw==&mid=2653200029&idx=1&sn=51ecebafb9ff77baf3de71bdc4f67b78&chksm=8c99ec47bbee6551b0377b97e26670c4895d0c934051e4aa927e62bf9b64996b6e1f7459edfe&scene=21#wechat_redirect) | $O(n*logn)$                      | ✓      | ✘        |
| [快速排序](https://mp.weixin.qq.com/s?__biz=MzIxMjE5MTE1Nw==&mid=2653195042&idx=1&sn=2b0915cd2298be9f2163cc90a3d464da&chksm=8c99f9f8bbee70eef627d0f5e5b80a604221abb3a1b5617b397fa178582dcb063c9fb6f904b3&scene=21#wechat_redirect) | $O(n*logn)$                      | ✘      | ✓        |
| [堆排序](https://mp.weixin.qq.com/s?__biz=MzIxMjE5MTE1Nw==&mid=2653195208&idx=1&sn=e3d6559402148458f0a4993b47d8bc6f&chksm=8c99f912bbee7004625a0b204acc8484acbdf4f1b18953e7ff5acbea958ec002d8c8ea072792&scene=21#wechat_redirect) | $O(n*logn)$                      | ✘      | ✓        |
| [桶排序](https://mp.weixin.qq.com/s?__biz=MzIxMjE5MTE1Nw==&mid=2653195582&idx=1&sn=1e7ece4e48c20fb994e2cefdcbdce4c5&chksm=8c99ffe4bbee76f23d16ac1e0c7feeb16654ebb75e40d92c911bffa113059f52ce4508281a55&scene=21#wechat_redirect) | $O(n)$，分桶数量是 n             | ✓      | ✘        |
| [计数排序](https://mp.weixin.qq.com/s?__biz=MzIxMjE5MTE1Nw==&mid=2653195533&idx=1&sn=02918dc51b07837ce1119f00d7900dbc&chksm=8c99ffd7bbee76c1d2e2e9b198259795285ec2c305d3613a5e39622195fd1c32bb6dbe52fa08&scene=21#wechat_redirect) | $O(n+k)$，k 是原始数组的数据范围 | ✓      | ✘        |
| [基数排序](https://mp.weixin.qq.com/s?__biz=MzIxMjE5MTE1Nw==&mid=2653200371&idx=1&sn=94c1882b9156bd96fa6da20c7995850e&chksm=8c99ed29bbee643f292c3d06995825a657d0c93cbabc4cc41a1a4f4073fdb663ecdc6d1d9685&scene=21#wechat_redirect) | $O(k*n)$，k 代表维度             | ✓      | ✘        |

其中 $O(n^2)$ 的排序算法仅适合小规模数据的排序，而$O(n*logn)$ 的算法则更适合大规模数据的排序，更常用。

对于排序算法执行效率的分析，一般会从这几个方面来衡量：

1. 最好情况、最坏情况、平均情况时间复杂度，以及最好/坏情况对应的要排序的原始数据是什么样的；
2. 时间复杂度的系数、常数 、低阶；
3. 比较次数和交换(或移动)次数。

原地排序(Sorted in place)算法：特指空间复杂度是 $O(1)$ 的排序算法。

稳定性：若待排序的序列中存在值相等的元素，排序后，相等元素间原有的先后顺序不变。

假设从小到大为有序：

有序度：数组中具有有序关系的元素对的个数。如：

```
数据：2, 4, 3, 1, 5, 6 这组数据的有序度为 11，因为有序的元素对共有 11 个：
(2,4)、(2,3)、(2,5)、(2,6)、(4,5)、
(4,6)、(3,5)、(3,6)、(1,5)、(1,6)、(5,6)
```

完全有序的数组的有序度称为满有序度。如：

```
数据：1, 2, 3, 4, 5, 6 其有序度为 n*(n-1)/2 = 15
```

逆序度与有序度正好相反。逆序度 = 满有序度 - 有序度

排序就是在增加有序度减少逆序度，最后达到满有序度。

以下内容，如不特殊强调，均为从小到大排序！



# O(n^2)

冒泡、插入排序的元素比较、交换次数取决于原始数组的有序程度，在此基础上，插入排序的性能略高于冒泡排序，因为插入排序的交换操作会比冒泡排序少。

而选择排序的元素交换次数是固定的，和原始数组的有序程度无关。

**当原始数组接近有序时，插入排序性能最优；当原始数组大部分元素无序时，选择排序性能最优。**

## Bubble Sort

[冒泡排序](https://mp.weixin.qq.com/s?__biz=MzIxMjE5MTE1Nw==&mid=2653194666&idx=1&sn=69ce32870c0b981c40b1e124fbb6bba8&chksm=8c99fb70bbee72668cad223892ad362525d215e7f936458f99dd289eb82981099359310e9e54&scene=21#wechat_redirect)有两种操作：比较、交换。**相邻的元素两两比较，根据大小来交换元素的位置**。每交换一次，有序度加一，无论算法怎么修改，交换的次数总是确定的！即为逆序度，也就是 $n*(n-1)/2 - 初始有序度$。

```go
func BubbleSort(nums []int, len int) {
	if nums == nil || len < 2 {
		return
	}
	for i := len - 1; i >= 1; i-- {
		// 假定未排序区间内元素有序
		sorted := true
		for j := 0; j < i; j++ {
			if nums[j] > nums[j+1] {
				nums[j], nums[j+1] = nums[j+1], nums[j]
				// 发生了交换，则说明未排序区间内元素无序
				sorted = false
			}
		}
		if sorted {
			break  // 未排序区间元素有序，所有元素已排序完成，退出循环
		}
	}
}
```

- 冒泡过程仅涉及相邻数据的交换操作，空间复杂度为 $O(1)$，是一个原地排序算法；
- 相邻两个元素相等时不做交换，排序前后的顺序不变，是一个稳定的排序算法；
- 时间复杂度
  - 最好：数据是有序的，只需一次冒泡操作，为 $O(n)$
  - 最坏：数据是倒序的，要进行 n 次冒泡操作，为 $O(n^2)$
  - 平均：$O(n^2)$



## Selection Sort

将数组分为有序和无序区，但[选择排序](https://mp.weixin.qq.com/s?__biz=MzIxMjE5MTE1Nw==&mid=2653198991&idx=1&sn=7f98d59898a911e1425baa6cc180c598&chksm=8c99e855bbee61439086680ceefef33c56038c5d552ae64c1d6135abe467b617aa62f4934f36&scene=21#wechat_redirect)每次会**从无序区中找到最小元素**，将其放到有序区末尾。

```go
func SelectionSort(nums []int, len int) {
	if nums == nil || len < 2 {
		return
	}
	for i := 0; i < len-1; i++ {
		minIndex := i
		// 查找 [i, len-1] 范围内最小值的索引
		for j := i + 1; j < len; j++ {
			if nums[j] < nums[minIndex] {  // 注意这里是和 minIndex 对应的值比较！
				minIndex = j
			}
		}
		// 每次移动i，最多仅发生一次交换操作
		nums[minIndex], nums[i] = nums[i], nums[minIndex]
	}
}
```

- 空间复杂度为 $O(1)$，是一个原地排序算法
- 不是一个稳定的排序算法，如对 `5, 8, 5, 2, 9` 排序
- 时间复杂度：
  - 最好：数据是有序的，为 $O(n^2)$
  - 最坏：数据是倒序的，为 $O(n^2)$
  - 平均 $O(n^2)$

选择排序改进了冒泡排序，可以将必要的交换次数从$O(n^2)$减少到$O(n)$，不过比较次数仍是$O(n^2)$。所以时间复杂度仍是$O(n^2)$。当比较的数据项个数$n$很大时，比较的次数是主要的，所以虽然选择排序和冒泡排序都运行了$O(n^2)$时间，但选择排序更快，因为它进行的交换少很多。



## Insertion Sort

将数组分为有序和无序两个区间，最开始有序区间只有第一个元素，每次取无序区间的元素在有序区间找到合适的位置插入，并保证有序区间一直有序，重复这个过程，直到无序区间元素为空，结束。

[插入排序](https://mp.weixin.qq.com/s?__biz=MzIxMjE5MTE1Nw==&mid=2653199343&idx=1&sn=a5491fa908e45e6117423d9ba5062611&chksm=8c99e935bbee60232aacb7c2b74961a24e7b86d44bf98357c597ad277a8eb15639c1de7034d9&scene=21#wechat_redirect)也包含元素的比较和移动两种操作：

- 不同的查找插入点方法(无序区的元素和有序区元素比较时是从有序区的头到尾或从尾到头)，元素的比较次数是有区别的。
- 但对于一个给定的初始序列，移动操作的次数总是固定的，就等于逆序度。

```go
// 想象玩扑克抽牌的过程，每抽到一张牌 x 就和前面的牌比较，
//   1. x 大则不做任何操作，并结束
//   2. x 小则交换，然后再和更前面的牌比较，再进行 1 或 2 的操作
func InsertionSort(nums []int, len int) {
	if nums == nil || len < 2 {
		return
	}
	// [0,i)是有序区
  // 无序区的第一个元素依次与有序区的元素从后往前比较
	for i := 1; i < len; i++ { 
		for j := i - 1; j >= 0; j-- {
			if nums[j] > nums[j+1] {
				nums[j], nums[j+1] = nums[j+1], nums[j]
			}
		}
	}
}
```

- 空间复杂度为 $O(1)$，是一个原地排序算法
- 是一个稳定的排序算法
- 时间复杂度
  - 最好：数据是有序的，还需要进行比较，但每个元素仅比较一次即可，为 $O(n)$
  - 最坏：数据是倒序的，为 $O(n^2)$
  - 平均 $O(n^2)$

一共$n$个元素，在第一次排序最多比较一次，第二次排序最多比较两次，以此类推，最后一次比较$n-1$次，因此有：$1+2+3+...+n-1 = n*(n-1)/2$，因为在每次排序发现插入点之前，平均只有全体数据项的一半真的进行了比较，所以除以$2$，得到$n*(n-1)/4$。

# O(nlogn)

归并排序和堆排序的时间复杂度稳定在 $O(nlogn)$，而快排的平均时间复杂度为 $O(nlogn)$ 但最坏时间复杂度为 $O(n^2)$。

而由于二叉堆的父子节点在内存中并不连续，对 CPU 缓存不友好，堆排序会比其他两种排序性能略低，

## Merge Sort

[归并排序](https://mp.weixin.qq.com/s?__biz=MzIxMjE5MTE1Nw==&mid=2653200029&idx=1&sn=51ecebafb9ff77baf3de71bdc4f67b78&chksm=8c99ec47bbee6551b0377b97e26670c4895d0c934051e4aa927e62bf9b64996b6e1f7459edfe&scene=21#wechat_redirect)使用的是分治思想。分治，顾名思义，就是分而治之，将一个大问题分解成小的子问题来解决。小的子问题解决了，大问题也就解决了。可使用递归实现。

![](../../images/merge-sort.jpg)

归并排序的递推公式：

```
p...r 代表了数组下标从 p 到 r 的数据：
mergeSort(p...r) = mergeSort(p...q) + mergeSort(q+1...r)
终止条件：
q >= r  不再继续分解
```

归并排序的步骤：

1. 将数组分为两部分，并分别采用递归的归并排序(各自排序好)；
2. 假设左侧部分使用指针 a 从左往右移动，右侧部分使用指针 b 从左往右移动，并准备一个辅助数组；
3. 比较指针 a 和指针 b 指向的元素的大小，小的填入辅助数组，并将对应指针向后移动；
4. 当某一指针移动到该侧的最后，将另一侧的剩余元素直接全部填入辅助数组；
5. 将辅助数组的内容拷贝到原数组

代码：

```go
func MergeSort(nums []int, len int) {
	if nums == nil || len < 2 {
		return
	}
	mergeSort(nums, 0, len-1)
}

func mergeSort(nums []int, start, end int) {
	if start >= end {  // 终止条件
		return
	}
	middle := start + (end-start)>>1
	mergeSort(nums, start, middle)
	mergeSort(nums, middle+1, end)
	merge(nums, start, middle, end)
}

func merge(nums []int, start, middle, end int) {
	temp := make([]int, end-start+1)
	pointA, pointB := start, middle+1
	i := 0
	for pointA <= middle && pointB <= end {
		if nums[pointA] <= nums[pointB] {
			temp[i] = nums[pointA]
			pointA++
		} else {
			temp[i] = nums[pointB]
			pointB++
		}
		i++
	}
	for pointA <= middle {
		temp[i] = nums[pointA]
		i++
		pointA++
	}
	for pointB <= end {
		temp[i] = nums[pointB]
		i++
		pointB++
	} 
	copy(nums[start:end+1],temp) // 由于是左闭右开区间，所以是 [start:end+1]
}
```

- 归并排序是一个稳定的排序算法
- 空间复杂度为 $O(n)$，尽管每次合并操作都需要申请额外的内存空间，但在合并完成之后，临时开辟的内存空间就被释放掉了。在任意时刻，CPU 只会有一个函数在执行，也就只会有一个临时的内存空间在使用。临时内存空间最大也不会超过 n 个数据的大小，所以空间复杂度是 $O(n)$，并不是原地排序
- 时间复杂度：

假设对 n 个元素进行归并排序需要时间 $T(n)$，那分解成两个子数组排序的时间都是 $T(n/2)$。由于 `merge()` 函数合并两个有序子数组的时间复杂度是 $O(n)$。所以，归并排序的时间复杂度的计算公式就是：

```
T(1) = C； n=1时，只需要常量级的执行时间，所以表示为C。
T(n) = 2*T(n/2) + n； n>1
		 = 2*(2*T(n/4) + n/2) + n = 4*T(n/4) + 2*n
		 = 4*(2*T(n/8) + n/4) + 2*n = 8*T(n/8) + 3*n
		 ......
		 = 2^k * T(n/2^k) + k * n
```

当 $T(n/2^k)=T(1)$ 时，得到 $k=log_2n=logn$ 。将 k 值代入上面的公式，得到 $T(n)=Cn+nlog2n$ 。如果用大 O 标记法来表示的话，$T(n) = O(nlogn)$。所以归并排序的时间复杂度是 $O(nlogn)$。



## Quick Sort

[快排](https://mp.weixin.qq.com/s?__biz=MzIxMjE5MTE1Nw==&mid=2653195042&idx=1&sn=2b0915cd2298be9f2163cc90a3d464da&chksm=8c99f9f8bbee70eef627d0f5e5b80a604221abb3a1b5617b397fa178582dcb063c9fb6f904b3&scene=21#wechat_redirect)利用的也是分治思想。

1. 对数组**分区**，小于原数组最后一个值的放在左侧，大于的放在右侧；
2. 对左侧和右侧分别继续进行经典快排

```go
func QuickSort(nums []int, len int) {
	if nums == nil || len < 2 {
		return
	}
	quickSort(nums, 0, len-1)
}

func quickSort(nums []int, low, high int) {
	if low >= high {
		return
	}
	q := partition(nums, low, high)
	quickSort(nums, low, q-1)
	quickSort(nums, q+1, high)
}
// 获取分区点的索引
func partition(nums []int, low, high int) int {
	pivot := nums[high]  // 以最后一个元素作为分区点
	var i = low
	// j 从左往右开始移动，当 nums[j] < pivot 时，交换索引 i 和 j 各自对应的值
	for j := low; j < high; j++ {
		if nums[j] < pivot {
			nums[i], nums[j] = nums[j], nums[i]
			i++
		}
	}
	// 此时索引 i 对应的值就是从左往右第一个大于 pivot 的值
	// 交换 i 和 high(即 pivot) 对应的值，则 i 就是分区点的索引
	nums[i], nums[high] = nums[high], nums[i]
	return i
}
```

快排是原地排序，但并不是一个稳定的算法。

快排的问题：如果数组本身有序，那么每次都是在用最大/小值进行分区，此时时间复杂度为$O(n^2)$。即快排和数据状况是有关的！

如果每次都用数组中值居中的值进行分区(很难做到)，左右侧数据量都是$n/2$，用master公式可得时间复杂度为$O(nlogn)$。

以上是两种极端情况，如果 pivot 随机得到的话，在大部分情况下的时间复杂度都可以做到 $O(nlogn)$，只有在极端情况下，才会退化到 $O(n^2)$。

**快排的优化**：选择合适的分区点 pivot 即可，那么该如何选呢？以下是两种常用的方法：

- 三数取中：从区间的首、尾、中间，分别取出一个数，然后对比大小，取这 3 个数的中间值作为分区点。如果要排序的数组比较大，那“三数取中”可能就不够了，可能要“五数取中”或者“十数取中”。
- 随机：随机取 pivot。此时的额外空间复杂度为 $O(logN)$：
  - 最好情况：使用中间的数来分区，这样递归对左侧快排时(同样使用中间的数分区)，一共使用$logN$的空间记录分区点，而对右侧快排时，左侧已经完成，之前的额外空间会先释放，然后右侧的排序也会使用和左侧相同的额外空间，所以最多为$O(logN)$的额外空间。
  - 最差情况：每次都随机到数组的一侧端点来分区，这样是$O(N)$的额外空间。
  - 所以额外空间复杂度也是一个概率，但是期望为$O(logN)$。



## Heap Sort

[堆排序](https://mp.weixin.qq.com/s?__biz=MzIxMjE5MTE1Nw==&mid=2653195208&idx=1&sn=e3d6559402148458f0a4993b47d8bc6f&chksm=8c99f912bbee7004625a0b204acc8484acbdf4f1b18953e7ff5acbea958ec002d8c8ea072792&scene=21#wechat_redirect)见[10-Heap](10-Heap.md)的排序部分



# 线性时间复杂度

线性排序(Linear sort)算法的时间复杂度都是线性的，桶排序、计数排序、基数排序都是线性排序。线性排序的时间复杂度低，使用场景比较特殊。



## Bucket Sort

[桶排序](https://mp.weixin.qq.com/s?__biz=MzIxMjE5MTE1Nw==&mid=2653195582&idx=1&sn=1e7ece4e48c20fb994e2cefdcbdce4c5&chksm=8c99ffe4bbee76f23d16ac1e0c7feeb16654ebb75e40d92c911bffa113059f52ce4508281a55&scene=21#wechat_redirect)：将要排序的数据分到几个有序的桶里，每个桶里的数据再单独进行排序(通常使用快排)，桶内排完序之后，再把每个桶里的数据按照顺序依次取出，组成的序列就是有序的了。

共计 n 个元素，均匀分到 m 个桶中，每个桶有 k=n/m 个元素，每个桶使用快排，m 个桶的时间复杂度就是 $O(m*k*logk)=O(n*log(n/m))$，当 m 接近于 n 时，桶排序的时间复杂度就接近于 $O(n)$。

桶排序是一个稳定排序，但并不是原地排序。

![](../../images/bucket-sort.jpg)

桶排序对数据有严格的要求：

- 要排序的数据需要很容易就能分成 m 个桶
- 桶与桶之间有天然的大小顺序，这样每个桶内的数据都排序完之后，桶与桶之间的数据不需要再进行排序
- 数据在各个桶之间的分布是比较均匀的，在极端情况下，如果数据都被划分到一个桶里，就退化为 $O(nlogn)$ 了

**桶排序比较适合用在外部排序中**。所谓的外部排序就是数据存储在外部磁盘中，数据量比较大，内存有限，无法将数据全部加载到内存中。

如：假设有 10GB 的订单数据，要按订单金额排序，但内存只有几百 MB，可以通过桶排序，将订单数据分成多个桶，每次将一个桶加载到内存中进行快排，然后写入一个文件，每个桶对应一个文件，并且按照金额范围的大小顺序编号命名，最后按照文件编号，从小到大依次读取每个小文件中的订单数据，并将其写入到一个文件中，那这个文件中存储的就是按照金额从小到大排序的订单数据了。



## Counting Sort

[计数排序](https://mp.weixin.qq.com/s?__biz=MzIxMjE5MTE1Nw==&mid=2653195533&idx=1&sn=02918dc51b07837ce1119f00d7900dbc&chksm=8c99ffd7bbee76c1d2e2e9b198259795285ec2c305d3613a5e39622195fd1c32bb6dbe52fa08&scene=21#wechat_redirect)：当要排序的 n 个数据，所处的范围并不大的时候，比如最大值是 k，我们就可以把数据划分成 k 个桶。每个桶内的数据值都是相同的，省掉了桶内排序的时间。时间复杂度为 $O(n)$。

**计数排序只能用在数据范围不大的场景中，如果数据范围 k 比要排序的数据 n 大很多，就不适合用计数排序了。而且，计数排序只能给非负整数排序，如果要排序的数据是其他类型的，要将其在不改变相对大小的情况下，转化为非负整数。**

```go
func CountingSort(nums []int, len int) {
	if nums == nil || len < 2 {
		return
	}
	// 获取切片中的最大值
	max := nums[0]
	for i := range nums {
		if nums[i] > max {
			max = nums[i]
		}
	}
	temp := make([]int, max+1)
	for i := range nums {
		temp[nums[i]]++
	}
	for i := 1; i <= max; i++ {
		temp[i] += temp[i-1]
	}
	ret := make([]int, len)
  // 这里从后往前取数可以保证该算法是稳定的，从前往后的话就不稳定了
	for i := len-1; i >= 0; i-- {
		index := temp[nums[i]] - 1
		ret[index] = nums[i]
		temp[nums[i]]--
	}
	copy(nums, ret)
}
```



## Radix Sort

如果要比较两个手机号码的大小，可以从后往前逐位比较，当经过 11 次(手机号是 11 位的)比较后，手机号码就是有序的了。

上面就是一个基数排序的例子，基数排序必须借助一个稳定算法来实现，否则这种思路就是错误的。因为如果是非稳定排序算法，那最后一次排序只会考虑最高位的大小顺序，完全不管其他位的大小关系，那么低位的排序就完全没有意义了。

对每一位的比较可通过桶排序或计数排序，其复杂度可以做到 $O(n)$，假设数据有 k 位，则总的复杂度就是 $O(k*n)$。

实际上，有时候要排序的数据并不都是等长的，可以把所有的单词补齐到相同长度，位数不够的可以在后面补“0”，因为根据 ASCII 值，所有字母都大于“0”，所以补“0”不会影响到原有的大小顺序。这样就可以继续用基数排序了。

[基数排序](https://mp.weixin.qq.com/s?__biz=MzIxMjE5MTE1Nw==&mid=2653200371&idx=1&sn=94c1882b9156bd96fa6da20c7995850e&chksm=8c99ed29bbee643f292c3d06995825a657d0c93cbabc4cc41a1a4f4073fdb663ecdc6d1d9685&scene=21#wechat_redirect)对要排序的数据是有要求的，需要可以分割出独立的“位”来比较，而且位之间有递进的关系，如果 a 数据的高位比 b 数据大，那剩下的低位就不用比较了。除此之外，每一位的数据范围不能太大，要可以用线性排序算法来排序，否则，基数排序的时间复杂度就无法做到 $O(n)$ 了。



# 综合排序

- 对小规模数据进行排序，可以选择时间复杂度是 $O(n^2)$ 的算法；
- 对大规模数据进行排序，时间复杂度是 $O(nlogn)$ 的算法更加高效。

所以，为了兼顾任意规模数据的排序，一般都会首选时间复杂度是 $O(nlogn)$ 的排序算法来实现排序函数。

归并排序可以做到平均情况、最坏情况下的时间复杂度都是 $O(nlogn)$，但空间复杂度位 $O(n)$，这对于小数据量的排序，如 1、2KB 等时，额外需要 1、2KB 的内存空间，但如果数据量太大，如 100MB 时，就额外需要 100MB，这就不合适了。

