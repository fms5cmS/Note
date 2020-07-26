栈是一种“操作受限”的线性表，只允许在一端(栈顶)插入、删除数据，且具有 LIFO(Last In First Out，后进先出) 特性。

栈 ADT 的操作主要有：

- `Push(E)`：入栈，向栈顶压入数据
- `Pop()`：出栈，从栈顶取出数据
- `Peek()`：获取栈顶元素

对于入栈操作来说，最好情况时间复杂度是 O(1)，最坏情况(空间不足时需要重新申请内存和数据搬移)时间复杂度是 O(n)，

栈的应用：

- 递归
- 树和图的深度优先搜索
- 十进制数转二进制数
- 撤销
- 用栈实现**表达式求值**功能（使用两个栈，分别保存操作数、运算符）
  - 从左向右遍历表达式
    - 遇到数字，压入操作数栈
    - 遇到运算符，与运算符栈的栈顶元素比较
      - 当前运算符优先级 > 栈顶运算符优先级，当前运算符入栈
      - 当前运算符优先级 <= 栈顶运算符优先级，从操作数栈中取出两个元素，然后取出栈顶运算符对两个元素进行计算，将结果压入操作数栈，并继续比较当前运算符与新的栈顶运算符
  - 遍历完成后，根据两个栈剩余的元素进行计算即可得到表达式的结果

- 函数调用栈
  - 从调用函数到被调用函数，对于数据而言，变化的是作用域，所以，只要保证每进入一个新的函数，都是一个新的作用域就可以。而这个用栈实现很方便。
  - 在进入被调用函数的时候，分配一段栈空间给这个函数的变量，在函数结束的时候，将栈顶复位，正好回到调用函数的作用域内。
- 用栈实现**括号匹配**功能（使用一个栈保存未匹配的左括号）
  - 扫描到左括号，就入栈
  - 扫描到右括号，就出栈，并比较出栈元素（左括号）与右括号是否匹配
  - 括号不匹配的情形：
    - 出栈元素与右括号不匹配，如出栈的是 [，而右括号则是 )
    - 右括号没有进行匹配的元素（右括号比左括号多）
    - 右括号扫描结束后栈中不为空（左括号比右括号多）
  - 所有括号扫描结束后，栈为空，则所有的括号都是相匹配的
- 浏览器的前进、后退功能
  - 把首次浏览的页面依次压入栈 X 中（X 的栈顶元素代表的页面为当前所在页面）
  - 点击后退时，从 X 出栈并放入栈 Y 中
  - 点击前进时，从 Y 中出栈并放入 X 中
  - 当 X、Y 中都有元素，并跳转到一个新的页面，需要将 Y 清空，然后将新的页面放入 X 中！！

# 自定义栈

数组实现的栈称为顺序栈，链表实现的栈称为链式栈。

- 顺序栈：0 下标处为栈底、末尾为栈顶；
- 链式栈：头节点为栈底，尾节点为栈顶。

```go
type ArrayStack struct {
	data []interface{}
	top  int   // // 栈顶指针
}

func (s ArrayStack) String() string {
	if len(s.data) == 0{
		return "nothing"
	}
	return fmt.Sprint("Stack: ", s.data, " top")
}

func NewStack() ArrayStack {
	return ArrayStack{
		data: make([]interface{}, 0),
		top:  -1,
	}
}

func (s *ArrayStack) Push(x interface{}) {
	s.data = append(s.data, x)
	s.top++
}

func (s *ArrayStack) Pop() interface{} {
	ret := s.data[s.top]
	s.data = s.data[0:s.top]
	s.top--
	return ret
}

func (s *ArrayStack) Peek() interface{} {
	return s.data[s.top]
}

func (s *ArrayStack) Size() interface{} {
	return len(s.data)
}
```

# 栈实现队列

底层使用两个栈(in、out)，每次入队操作将元素压入 in 中，出队操作则从 out 中弹出

- 每次 `Enqueue()`：将元素直接压栈入 in
- 每次 `Dequeue()`：
  - 当 out 中没有元素时将 in 中元素**全部**出栈并逐个入栈到 out，然后再从 out 出栈
  - 当 out 中有元素时，直接从 out 中出栈即可

```go
type MyQueue struct {
	in, out stack.Stack  // 这里使用的是上面自定义的栈
}

func (this *MyQueue) String() string {
	if this.Empty() {
		return "nothing"
	}
	inSize := this.in.Size()
	if this.out.Size() == 0 {
		for i := 0; i < inSize; i++ {
			this.out.Push(this.in.Pop())
		}
	}
	ret := this.out.String()
	ret = strings.Replace(ret, "Stack", "tail", -1)
	ret = strings.Replace(ret, "top", "front", -1)
	return ret
}

func Constructor() MyQueue {
	return MyQueue{
		in:  stack.NewStack(),
		out: stack.NewStack(),
	}
}
// 入队操作
func (this *MyQueue) Enqueue(x int) {
	this.in.Push(x)
}
// 出队操作
func (this *MyQueue) Dequeue() int {
	inSize := this.in.Size()
	if this.out.Size() == 0 {
		for i := 0; i < inSize; i++ {
			this.out.Push(this.in.Pop())
		}
	}
	return this.out.Pop()
}
// 获取队首元素
func (this *MyQueue) Front() int {
	inSize := this.in.Size()
	if this.out.Size() == 0 {
		for i := 0; i < inSize; i++ {
			this.out.Push(this.in.Pop())
		}
	}
	return this.out.Peek()
}

func (this *MyQueue) Empty() bool {
	return this.out.Size() == 0 && this.in.Size() == 0
}
```

入队操作的时间复杂度为 $O(1)$；

出队操作的最坏时间复杂度为 $O(n)$，但是在一系列操作中，最坏情况不会每次都发生，所以最坏时间复杂度是远大于实际的复杂度的，这里使用**摊还分析**来分析**均摊时间复杂度**：

每执行 n 次入队操作，下一次执行出队操作时，由于要将 in 中的元素先出栈再入栈到 out 中，所以这次出队的时间复杂度为 $O(2*n)$，此时 out 中还剩下 n-1 个元素，那么之后的 n-1 次出队操作的时间复杂度就是 $O(1)$，那么均摊下来，这共计 n 次的出队操作的均摊时间复杂度就是 $O(1)$。

其实，单独考虑每个元素的出队操作，都只有有限步操作：出栈(in)一次、入栈(out)一次、出栈(out)一次，所以出队操作的均摊时间复杂度就是$O(1)$。

查看队首元素的时间复杂度和出队操作的时间复杂度分析相同。

# 最小元素栈

```go
// 155. 最小栈：设计一个支持 push，pop，top 操作，并能在常数时间内检索到最小元素的栈。
type MinStack struct {
	// 栈顶指针
	top  int
	// 存放元素、存放当前栈中的最小值
	data,min  []int
}

func Constructor() MinStack {
	return MinStack{
		data: make([]int, 0),
		min:  make([]int, 0),
		top:  -1,
	}
}
// 第一次压栈时，同时向 data 和 min 中压入第一个元素；
// 其他时候，data 正常压栈，min 有所区别：
//    比较 min 中栈顶元素与当前压入 data 中元素 X 的大小，
//		1. 如果 X 较小，则向 min 中也压入 X
//		2. 如果 min 的栈顶元素较小，则重复压入 min 的栈顶元素
func (this *MinStack) Push(x int) {
	this.top++
	this.data = append(this.data, x)
	if this.top != 0 && x > this.min[this.top-1] {
		this.min = append(this.min, this.min[this.top-1])
		return
	}
	this.min = append(this.min, x)
}
// 每次 data 进行 pop 操作时，min 也要进行 pop 操作，
// 这样 min 的栈顶元素就始终是当前 data 中所有元素的最小值
func (this *MinStack) Pop() {
	this.data = this.data[:this.top]
	this.min = this.min[:this.top]
	this.top--
}
// 获取 data 的栈顶元素
func (this *MinStack) Top() int {
	return this.data[this.top]
}
// 获取 min 的栈顶元素即可
func (this *MinStack) GetMin() int {
	return this.min[this.top]
}
```
