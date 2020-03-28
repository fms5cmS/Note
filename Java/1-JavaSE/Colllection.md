# 前置知识

## 其他

数组的实际规模（元素个数）与其容量的比值 λ（即 size/capacity），也称作**装填因子**（loadfactor），它是衡量空间利用率的重要指标。

- 向量

		- 内部数组所占物理空间的容量，若在向量的生命期内不允许调整，则称作静态空间管理策略。很遗憾，该策略的空间效率难以保证。一方面，既然容量固定，总有可能在此后的某一时刻，无法加入更多的新元素——即导致所谓的**上溢（overflow）**。需要扩容。

	- 向量的实际规模可能远远小于内部数组的容量。比如在连续的一系列操作过程中，若删除操作远多于插入操作，则装填因子极有可能远远小于100%，甚至非常接近于0。当装填因子低于某一阈值时，我们称数组发生了**下溢（underflow）**。这不是必须解决的问题，但在注重空间利用率时，也有必要适当缩容。

- 数组：

  - 每个元素都通过下标唯一指代，而且我们可以直接访问到任一元素。这里所说的**“访问”包含读取、修改等基本操作**，而**“直接”则是指这些操作都可以在常数时间内完成。**

基于数组实现的数据结构，允许通过下标，在常数时间内找到目标对象，然而，一旦需要对这类数据结构进行修改，那么无论是插入还是删除，都需要耗费线性时间。

基于链表实现的数据结构，可以在常数的时间内插入或删除元素，但是为了找到特定次序的元素，却不得不花费线性时间。



## Collections

`Collection`是`Map`和`List`所实现的接口；`java.util.Collections`是一个工具类。

**Collections**

- 排序操作
  - `binarySearch(List<? extends Comparable<? super T>> list, T key)`：（容器有序）使用二叉搜索算法搜索指定对象的指定列表；
  - `reverse(List) `:反转 `List` 中的元素顺序;
  - `shuffle(List)` :洗牌，对`List` 集合元素进行随机排序 
  - `sort(List)`：根据元素自然顺序对指定 `List`集合元素按升序排列            
  - `sort(List,Comparator) `:根据指定的`Comparator`产生的顺序对`List`集合元素进行排列  
  - `swap(List,int i,int j)` :将指定`List`集合中的 i 处元素和 j 处元素进行交换 
- 查找、替换
  - `Object max(Collection)`:根据元素的自然顺序，返回给定的集合中的最大元素            
  - `Object max(Collection,Comparator)`:根据 `Comparator`指定的顺序，返回给定集合中的最大元素
  - `Object min(Collection)`
  - `Object min(Collection,Comparator)`
  - `int frequency(Collection,Object)`:返回指定集合中指定元素的出现次数
  - `boolean replaceAll(List list , Object oldVal , Object newVal)`:使用新值替换`List`对象的所有旧值
- 获取线程安全的`List`对象,使用`synchronizedList()`方法：`Collections.synchronizedList(List list)` 



## 排序

排序是常做的操作，要想进行排序，需要有比较规则。如：1<2<3<4，所以按从大到小的排序，结果为：{4，3，2，1}。

java内置类是根据什么进行比较的：

1. `Integer`：根据基本数据类型大小；
2. `char`：根据 Unicode 编码顺序；
3. `String`：如果其中一个是另一个的子串，返回长度之差；否则返回第一个不相等的Unicode码之差；
4. `java.util.Date`：根据日期的长整型数比较 。

Q1：如何告诉标注某个类是**可比较**的？

答：让该类实现`java.lang.Comparable`接口，然后重写`compareTo()`方法（即自定义比较规则）。这样，该类就是可比较的，且可以通过调用`compareTo()`方法来对两个该类的实例进行比较。如：`integer1.compareTo(integer2)`。

Q2：如何提供一个额外地业务排序类？

答：让一个类实现`java.util.Comparator<T>`接口，然后重写`compare(T o1,T o2)`方法（自定义比较规则）。这样就使得排序类与实体类解耦，可以应对不同的排序规则。`compare(T o1,T o2)`方法：如果返回值为负数，则o1排在o2前面；返回值为0，o1等于o2；返回值为正数，o1排在o2后面。

java 中的数据结构可以借助 `Collections`工具类的排序方法来排序：

1. `public static <T extends Comparable<? super T>> void sort(List<T> list)`不指定比较器，则默认升序排列；
2. `public static <T> void sort(List<T> list, Comparator<? super T> c)`按照指定比较器 c 自定义的比较方式来排序。

- 排序容器（这两种数据结构都是使用平衡二叉查找树实现的）：
  - `TreeSet`：**数据元素可以排序**且不可重复。底层默认实现自然排序（升序） 
    - 自定义排序比较器：`public TreeSet(Comparator<? super K> comparator)`
    - 注意：在添加数据时排序，数据修改不会影响原来的顺序。这样的话，修改后数据可能会重复，但`Set`中的元素不能重复，所以在使用`TreeSet`过程中，不要修改数据。为确保无法修改数据，可将元素的数据设置为`final`，构造时进行赋值。 
  - `TreeMap`：
    - 确保 Key 可以排序
    - 自定义Key 比较器：`public TreeMap(Comparator<? super K> comparator)`



# Fail

## Fail-Fast

modCount 用来记录 ArrayList 结构发生变化的次数。==结构发生变化是指添加或者删除至少一个元素的所有操作，或者是调整内部数组的大小，仅仅只是设置元素的值不算结构发生变化==。

在进行序列化或者迭代等操作时，需要比较操作前后 modCount 是否改变，如果改变了需要抛出 ==ConcurrentModificationException==。

注意：这里异常的抛出条件是检测到`modCount！= expectedmodCount`这个条件。如果集合发生变化时修改`modCount`值刚好又设置为了`expectedmodCount`值，则异常不会抛出。因此，不能依赖于这个异常是否抛出而进行并发操作的编程，这个异常只建议用于检测并发修改的bug。

场景：`java.util`包下的集合类都是快速失败的，不能在多线程下发生并发修改（迭代过程中被修改）。 



## Fail-Safe

采用安全失败机制的集合容器，在遍历时不是直接在集合内容上访问的，而是先复制原有集合内容，在拷贝的集合上进行遍历。

原理：由于迭代时是对原集合的拷贝进行遍历，所以在遍历过程中对原集合所作的修改并不能被迭代器检测到，所以不会触发`ConcurrentModificationException`。

缺点：基于拷贝内容的优点是避免了`ConcurrentModificationException`，但同样地，迭代器并不能访问到修改后的内容，即：迭代器遍历的是开始遍历那一刻拿到的集合拷贝，在遍历期间原集合发生的修改迭代器是不知道的。

场景：`java.util.concurrent`包下的容器都是安全失败，可以在多线程下并发使用，并发修改。 



# 1. ArrayList

继承于 `AbstractList`，实现了 

- `List`接口：是一个数组队列，提供了相关添加、删除、修改、遍历等功能。 
- `RandomAccess`接口：提供随机访问功能。该接口是用来被List实现，为List提供快速访问功能（通过元素序号获取元素对象）的。 
- `Cloneable`接口：覆盖了clone()函数，能被克隆（浅拷贝）。克隆失败会产生`InternalError`。
- `java.io.Serializable`接口：支持序列化。 

无法存储基本数据类型，需要先自动装箱，而这是有性能消耗的，如果很关注性能，且希望使用基本数据类型，可以使用数组；

多维数据时使用数组会比较直观`Object[][] arr`，而使用`ArrayList`的话则是`ArrayList<ArrayList> arr`。



**对比**：

`AbstractList`底层是基于动态数组实现的，相比于普通的数组，`ArrayList`的容量是可以动态增长的。

和`Vector`的区别：

1. `Vector`是同步的，开销比`ArrayList`大，访问速度更慢。

2. `Vector`的自动扩容是扩容为原来的2倍，而`ArrayList`是原来的1.5倍。

3. `Voctor` 底层数据结构和 `ArrayList` 类似,也是一个动态数组存放数据。不过是在 `add()` 方法的时候使用 `synchronize` 进行同步写数据，但是开销较大，所以 `Vector` 是一个同步容器并不是一个并发容器。 

和`LinkedList`的区别：

1. `ArrayList `基于动态数组实现，`LinkedList` 基于双向循环链表实现；

2. `ArrayList `支持随机访问，`LinkedList `不支持；

3. `LinkedList `在任意位置添加删除元素更快。 



**源码部分**

默认初始容量Capacity = 10； 

底层实现：`transient Object[] elementData`;由于是基于动态数组实现的，所以并不是所有的空间都被使用。因此使用了 `transient` 修饰，可以防止被自动序列化。 

重要属性：`protected transient int modCount = 0`;  继承自父类。详见 失败部分

扩容机制的核心函数：

```java
private void grow(int minCapacity) {
  int oldCapacity = elementData.length;
  int newCapacity = oldCapacity + (oldCapacity >> 1); //先扩容为原来的1.5倍
  if (newCapacity - minCapacity < 0)	//比较新容量和所需最小容量的大小，取较大值
    newCapacity = minCapacity;
  if (newCapacity - MAX_ARRAY_SIZE > 0)	//比较新容量和ArrayList定义的最大容量的大小
    newCapacity = hugeCapacity(minCapacity);
  // minCapacity is usually close to size, so this is a win:
  elementData = Arrays.copyOf(elementData, newCapacity);//将原来的数据复制到扩容后的数组中
}
```

```java
private static int hugeCapacity(int minCapacity) {    //比较所需容量和ArrayList定义的最大容量
  if (minCapacity < 0) // overflow
    throw new OutOfMemoryError();
  // 所需容量 > 最大容量，结果为 Integer 的最大值，否则为最大容量
  return (minCapacity > MAX_ARRAY_SIZE)? Integer.MAX_VALUE:  MAX_ARRAY_SIZE;  
}
```

 `ArrayList` 的主要消耗是数组扩容以及在指定位置添加数据，所以使用时最好是指定大小，尽量减少扩容。更要减少在指定位置插入数据的操作。 

移除操作：`remove(index)`是通过将指定索引后面的元素向前移动一位来完成移除，然后把原本的最后一个元素置 null （便于GC回收 ），返回原本 index 索引处的值；

   `remove(Object)`是遍历所有元素，然后借助私有方法 `fastRemove(index)`（该方法的实现与remove(index)是一样的，但不会返回原本移除的值）实现的。

   注意：==null 和 非空数据的判断是不同的。空值采用 == 来判断；非空数据采用 equals() 方法来判断==。

自定义了序列化和反序列化。使得只有被使用的数据才会序列化。

```java
private void writeObject(java.io.ObjectOutputStream s)
  throws java.io.IOException{
  // Write out element count, and any hidden stuff
  int expectedModCount = modCount;
  s.defaultWriteObject();

  // Write out size as capacity for behavioural compatibility with clone()
  s.writeInt(size);

  // Write out all elements in the proper order.
  for (int i=0; i<size; i++) {	//只序列化了被使用的数据
    s.writeObject(elementData[i]);
  }

  if (modCount != expectedModCount) {
    throw new ConcurrentModificationException();
  }
}


private void readObject(java.io.ObjectInputStream s)
  throws java.io.IOException, ClassNotFoundException {
  elementData = EMPTY_ELEMENTDATA;

  // Read in size, and any hidden stuff
  s.defaultReadObject();

  // Read in capacity
  s.readInt(); // ignored

  if (size > 0) {
    // be like clone(), allocate array based upon size not capacity
    int capacity = calculateCapacity(elementData, size);
    SharedSecrets.getJavaOISAccess().checkArray(s, Object[].class, capacity);
    ensureCapacityInternal(size);

    Object[] a = elementData;
    // Read in all elements in the proper order.
    for (int i=0; i<size; i++) {
      a[i] = s.readObject();
    }
  }
}
```

参考：<https://yq.aliyun.com/articles/563043?spm=a2c4e.11163080.searchblog.38.81652ec1EgGVei> 



# 2. LinkedList

继承于 `AbstractSequentialList` ，实现了 

- `List` 接口
-  `Deque` 接口：使其具有队列的特性
- `Cloneable`接口：覆盖了clone()函数，能被克隆（浅拷贝）。
- `java.io.Serializable`接口 

底层是双向链表结构，支持高效的插入和删除操作。是线程不安全的。 

每次插入、删除都是移动指针，所以效率较高。 

每次查找都需要遍历查询，效率较低

**源码部分**

重要属性：`transient Node<E> first;`和`transient Node<E> last;`所以头节点和尾节点的名称是固定的。当链表长度为 1 时，first 和 last 是同一个节点。 

add 方法：

```java
public boolean add(E e) {
  linkLast(e);
  return true;
}
public void addLast(E e) {  //这两种方法的底层实现是相同的
  linkLast(e);
}
public void addFirst(E e) {
  linkFirst(e);
}
//下面两个方法多次交换分别是为了保证头节点是 first ，尾节点是 last
//可以通过画图的方式理解（注意：堆栈分开画，引用的赋值会将地址赋值给对方）
private void linkFirst(E e) {      
  final Node<E> f = first;                            // 给原先的第一个节点重命名为 f
  final Node<E> newNode = new Node<>(null, e, f);     // 新增一个头节点
  first = newNode;                                    // 给头节点起名为 first
  if (f == null)     //判断原先的链表是否为空链表
    last = newNode;    //原先为空链表，故 first 和last 是同一个节点
  else
    f.prev = newNode;    //原先不是空链表，所以将第二个节点的前驱结点指向 first （此时first为newNode）
  size++;
  modCount++;
}

void linkLast(E e) {
  final Node<E> l = last;
  final Node<E> newNode = new Node<>(l, e, null);
  last = newNode;
  if (l == null)
    first = newNode;
  else
    l.next = newNode;
  size++;
  modCount++;
}
```

在指定位置添加元素

```java
public void add(int index, E element) {
  checkPositionIndex(index);
  if (index == size)
    linkLast(element);
  else
    linkBefore(element, node(index));
}
/**
     * Returns the (non-null) Node at the specified element index.
     */
Node<E> node(int index) { //获取指定索引处的节点（注意，调用此方法前需要先检查索引，保证该索引处不为null）
  // assert isElementIndex(index);  //将范围二分
  if (index < (size >> 1)) { //如果索引值小于链表的一般，从头节点开始遍历    
    Node<E> x = first;
    for (int i = 0; i < index; i++)
      x = x.next;
    return x;
  } else {	//如果索引值大于链表大小的一半，那么将从尾结点开始遍历
    Node<E> x = last;
    for (int i = size - 1; i > index; i--)
      x = x.prev;
    return x;
  }
}
/**
     * Inserts element e before non-null Node succ.
     */
void linkBefore(E e, Node<E> succ) {
  // assert succ != null;
  final Node<E> pred = succ.prev;
  final Node<E> newNode = new Node<>(pred, e, succ);
  succ.prev = newNode;
  if (pred == null)
    first = newNode;
  else
    pred.next = newNode;
  size++;
  modCount++;
}
```

插入集合

```java
public boolean addAll(int index, Collection<? extends E> c) {
  checkPositionIndex(index);  //检查索引界限，必须在 [0,size] 区间
  Object[] a = c.toArray();     //将集合转为数组
  int numNew = a.length;
  if (numNew == 0)
    return false;        //集合为空时，插入失败
  Node<E> pred, succ;    //该指定位置的前驱 pred、后继节点succ
  if (index == size) {                        
    succ = null;        //插入位置为链表的末尾
    pred = last;         //pred为尾节点，succ为空
  } else {               //插入位置不在链表的末尾
    succ = node(index); //succ 为该索引处的节点 （插入时从该索引起之后的所有节点都要后移）
    pred = succ.prev; //pred 为该索引处节点的前驱节点 （此处之前的节点都不会发生位置变化）
  }
  for (Object o : a) {
    @SuppressWarnings("unchecked") E e = (E) o;        
    Node<E> newNode = new Node<>(pred, e, null);//新增一个节点，并将其前驱节点指向 pred
    if (pred == null){
      //如果 pred 是空，说明插在了首位，重置 first 节点
      first = newNode; 
    }else{
      //如果插在之间的位置，将 pred 的后继节点指向新增的节点
      pred.next = newNode;  
  	}
    //更新 pred 便于在遍历的过程中逐个链接接下来的元素
    pred = newNode;  
  }             
  //这里只是将前一段后插入的链接到了一起，但是后一段和插入的并没有连接
  if (succ == null) {
    last = pred;   //succ 为空，说明插在了末尾，重置 last 节点
  } else {  //否则，将插入的链表与先前的链表连接起来
    pred.next = succ;                                    
    succ.prev = pred;
  }
  size += numNew;
  modCount++;
  return true;
}
```

get方法，较为简单，如下：

- 获取头节点的四个方法 :

  - `public E getFirst() ` ：链表为空时，抛出异常 
  - `public E element()`：借助getFirst实现，链表为空时，抛出异常
  - `public E peek()`：链表为空时，返回null
  - `public E peekFirst()`：链表为空时，返回null 

- 获取尾节点的方法： 

  - `public E getLast()`：链表为空时，抛出异常
  - `public E peekLast()`：链表为空时，返回null 

  根据对象获取索引（从头遍历找：indexOf(Object o) ；从尾遍历找：lastIndexOf(Object o)） 

```java
public int indexOf(Object o) {//注意：该方法中对于 null 和非null的引用类型的判断，和ArrayList一致
  int index = 0;
  if (o == null) {
    for (Node<E> x = first; x != null; x = x.next) {//注意这里的写法！！可以对应写出 lastIndexOf(Object o) 方法
      if (x.item == null)
        return index;
      index++;
    }
  } else {
    for (Node<E> x = first; x != null; x = x.next) {
      if (o.equals(x.item))
        return index;
      index++;
    }
  }
  return -1;
}
```

- 删除（remove/pop）方法：
  - 删除头节点：`removeFirst()；remove() ；pop()`  后两个方法借助第一个方法实现
  - 删除尾节点
    - `public E removeLast()`在链表为空时将抛出`NoSuchElementException`
    - `public E pollLast()`在链表为空时返回null
  - 删除指定元素：`public boolean remove(Object o)` 借助` unlink(Node x)`方法实现
  - 删除指定位置的元素：`public E remove(int index) `借助 `unlink(Node x)`方法实现
  - `unlink(Node x) `

```java
E unlink(Node<E> x) {
  // assert x != null;
  final E element = x.item;
  final Node<E> next = x.next;
  final Node<E> prev = x.prev;

  //删除前驱指针
  if (prev == null) {
    first = next;
  } else {
    prev.next = next;
    x.prev = null;
  }

  //删除后继指针
  if (next == null) {
    last = prev;
  } else {
    next.prev = prev;
    x.next = null;
  }
  x.item = null;
  size--;
  modCount++;
  return element;
}
```

栈操作：入栈 push；出栈 pop

单向队列操作：获取头节点 peek、element；弹出头节点 poll ；移除头节点 remove；在尾部添加节点offer。

双向队列操作：在头部添加 offerFirst；在尾部添加 offerLast；获取头节点 peekFirst；获取尾节点 peekLast 。

参考：<https://yq.aliyun.com/articles/569200?spm=a2c4e.11153940.blogcont563043.24.7f4a14fe6iJXbF> 

# 3. HashMap

继承于`java.util.AbstractMap`，实现了：

- `Map`接口
- `Cloneable`接口：覆盖了clone()函数，能被克隆（浅拷贝）。
- `java.io.Serializable`接口 

`HashMap`最多只允许一条记录的键为 null，允许多条记录的值为 null。非线程安全。如果想保证线程安全，可以使用 `ConcurrentHahMap` 代替，而不是线程安全的 `HashTable`，因为`HashTable`基本已经被淘汰。

**初始化注意：initialCapacity = (需要存储的元素个数 / 负载因子) + 1。注意负载因子（即loaderfactor）默认为 0.75， 如果暂时无法确定初始值大小，请设置为 16（即默认值）。**

推荐使用 entrySet 遍历 Map 类集合 KV ，而不是 keySet 方式进行遍历。

| 集合类            | Key           | Value         | Super       | 说明                   |
| ----------------- | ------------- | ------------- | ----------- | ---------------------- |
| Hashtable         | 不允许为 null | 不允许为 null | Dictionary  | 线程安全               |
| ConcurrentHashMap | 不允许为 null | 不允许为 null | AbstractMap | 锁分段技术（JDK8:CAS） |
| TreeMap           | 不允许为 null | 允许为 null   | AbstractMap | 线程不安全             |
| HashMap           | 允许为 null   | 允许为 null   | AbstractMap | 线程不安全             |



**源码部分**

==JDK1.8以前==：

- 底层是数组和单向链表结合再一起使用，也就是链表散列。通过 key 的`hashCode`来计算 hash 值，当`hashCode`相同时，通过“拉链法”解决冲突。 
- 如果定位到的数组位置不含链表（当前`entry`的`next`指向null）,那么对于查找，添加等操作很快，仅需一次寻址即可；如果定位到的数组包含链表，对于添加操作，其时间复杂度依然为O(1)，因为最新的`entry`会插入链表头部（因为后插入的`entry`被查找的可能性更大），仅需要简单改变引用链即可，而对于查找操作来讲，此时就需要遍历链表，然后通过key对象的`equals()`方法逐一比对查找。
- `HashMap`的容量是有限的。当经过多次元素插入，使得`HashMap`达到一定饱和度时，Key映射位置发生冲突的几率会逐渐提高。这时候，`HashMap`需要扩展它的长度，也就是进行**Resize**。rehash 是 resize 的子过程。
  -  当 **HashMap.Size   >=  Capacity * LoadFactor时，会进行Resize**。 步骤：
    - 1.扩容——> 创建一个新的Entry空数组，长度是原数组的2倍。 
    - 遍历原Entry数组，把所有的Entry重新Hash到新数组。 

==JDK1.8以后==：

![HashMap](https://github.com/fms5cmS/MyNote/raw/master/images/Collection-HashMap.png)

- 解决哈希冲突的方式有了较大变化，当**链表长度大于阈值（默认为 8）时，将链表转化为红黑树，利用红黑树快速增删查改的特点提高`HashMap`的性能**。 

- 几个静态常量：

  ```java
  static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16
  static final int MAXIMUM_CAPACITY = 1 << 30;
  static final float DEFAULT_LOAD_FACTOR = 0.75f;//默认的装填因子=0.75
  static final int TREEIFY_THRESHOLD = 8;//// 当桶(bucket)上的结点数大于这个值时会转成红黑树
  static final int UNTREEIFY_THRESHOLD = 6; // 当桶(bucket)上的结点数小于这个值时树转链表
  static final int MIN_TREEIFY_CAPACITY = 64;
  ```

- `transient Node<K,V>[]  table`：哈希桶数组。==长度必须是 2 的幂次（一定是合数）==，这是非常规设计。（常规设计是将桶的大小设计为素数，而素数导致冲突的概率要小于合数）。HashMap 的这种非常规设计是为了在取模和扩容时做优化，同时为了减少冲突，HashMap 定位哈希桶索引时，也加入了高位参与运算的过程。

-  装填（加载）因子` loadFactor` ：控制数组存放数据的疏密程度，`loadFactor`越趋近于1，那么 数组中存放的数据(entry)也就越多，也就越密，也就是会让链表的长度增加，`load Factor`越小，也就是趋近于0。**`loadFactor`太大导致查找元素效率低，太小导致数组的利用率低，存放的数据会很分散。`loadFactor`的默认值为 0.75f 是官方给出的一个比较好的临界值。 　向量实际规模与其内部数组容量的比值（即 size/capacity）。**

-  阈值 **`threshold`** = capacity * loadFactor ：所能容纳的 key-value 对极限，当 size >= threshold 时，需要对数组扩容resize，扩容后的容量是之前的两倍。 

- 用于链表的静态内部类 `Node<K,V>` 中除了有 `hash`、`key`、`value`，还有一个指向下一节点的`Node next`。

- 还有用于红黑树节点的静态内部类` TreeNode<K,V>`。  

- `tableSizeFor(int cap)`方法根据给定的数字返回一个 2 的幂次。



Q1：为什么==长度必须为2的幂==？（注意： `index =  HashCode（Key） &  （Length - 1）`见`putVal()`方法）

答：数组下标是从0开始计算，所以最大下标为`length-1`。如果`length`为2的幂，那么`length-1`的二进制位后面都为1， 这种情况下，`index`的结果等同于`HashCode`后几位的值。只要输入的`HashCode`本身分布均匀，Hash算法的结果就是均匀的。 





高并发下的`HahMap`： 

多线程操作`HashMap`会出现死循环。        

`Hashmap`的`Resize`包含扩容和`ReHash`两个步骤，`ReHash`在并发的情况下可能会形成链表环。        

高并发下使用`ConcurrentHashMap`。 



详见：

<https://mp.weixin.qq.com/s/HzRH9ZJYmidzW5jrMvEi4w>；

<https://mp.weixin.qq.com/s/dzNq50zBQ4iDrOAhM4a70A>；

<https://yq.aliyun.com/articles/569204?spm=a2c4e.11153940.blogcont569200.25.707b8789jfvCrP>；<https://yq.aliyun.com/articles/259186?spm=a2c4e.11163080.searchblog.91.4cf92ec1L56nc7> 



# 零散

## Vector类

继承于`java.util.AbstractList`,实现了;

- `List`

- `RandomAccess`

- `Cloneable`

- `java.io.Serializable`

底层实现：`protected Object[] elementData`。

容量增量：`protected int capacityIncrement`，当Vector溢出时自动增长的容量，如果该值小于等于0，则每次扩容时容量为原来的2倍。见下面的扩容方法。

构造器：

1. 空构造器，此时默认的初始容量为 10；
2. 指定初始容量；
3. 指定初始容量和每次 Vector 溢出时增长的容量；
4. 将一个 Collection 放入Vector。

扩容方法：

```java
private void grow(int minCapacity) {
  // overflow-conscious code
  int oldCapacity = elementData.length;
  int newCapacity = oldCapacity + ((capacityIncrement > 0) ?
                                   capacityIncrement : oldCapacity);
  if (newCapacity - minCapacity < 0)
    newCapacity = minCapacity;
  if (newCapacity - MAX_ARRAY_SIZE > 0)
    newCapacity = hugeCapacity(minCapacity);
  elementData = Arrays.copyOf(elementData, newCapacity);
}
```

Vector 的很多方法都是加了 `synchronized` 同步锁的，线程安全，但使用时开销较大，效率低。在实际中使用较少。



## Stack类

继承于` java.util.Vector`，没有实现任何接口。底层和 `Vector`是相同的，都是基于动态数组实现。

Stack 是 LIFO（last-in-first-out）的。

五个新增方法：

| 方法                                       | 功能                                                         |
| ------------------------------------------ | ------------------------------------------------------------ |
| `public E push(E item)`                    | 入栈（将一个元素添加到栈顶）                                 |
| `public synchronized E pop()`              | 出栈（将栈顶的元素移除，并作为返回值返回）                   |
| `public synchronized E peek()`             | 查看栈顶的元素，不会将其移除                                 |
| `public boolean empty()`                   | 判断栈是否为空                                               |
| `public synchronized int search(Object o)` | 获得指定元素在栈中的位置（注意：索引为0的元素返回为1，以此类推。如果不在栈中，返回-1）|



## Queue接口

继承于`java.util.Collection`，除了 Collection 的基本操作以外，新增了 插入、获取、查看操作，且都有两种实现方式，一种在操作失败时抛出异常，另一种会返回 null 或 false。

| 失败时 | 抛出异常           | 特殊值                     |
| ------ | ------------------ | -------------------------- |
| 插入   | `boolean add(E e)` | `boolean offer(E e)` |
| 获取   | `E element()`      | `E peek()`                 |
| 移除   | `E remove()`       | `E poll()`                 |



## Deque接口

继承于 Queue 接口。

双端队列，支持队列两端的插入、移除、获取操作。每个方法又有 失败抛出异常、返回特殊值 两种形式，所以共计十二种新增方法。

|            | 失败抛出异常                              | 返回特殊值                    |
| ---------- | ----------------------------------------- | ----------------------------- |
| 插入首元素 | ` addFirst(e) `、` push(e)`               | ` offerFirst(e) `             |
| 插入末元素 | ` addLast(e) `、` add(e)`                 | ` offerLast(e)`、` offer(e) ` |
| 移除首元素 | ` removeFirst()`、` remove() `、` pop() ` | ` pollFirst() `、` poll()`    |
| 移除末元素 | ` removeLast()`                           | ` pollLast()`                 |
| 获取首元素 | ` getFirst()`、` element() t`             | ` peekFirst() `、` peek()`    |
| 移除末元素 | ` getLast() `                             | ` peekLast() `                |

