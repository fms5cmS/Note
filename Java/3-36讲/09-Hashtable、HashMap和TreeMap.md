# Q

Q：对比`Hashtable`、`HashMap`、`TreeMap`的不同。

A：三者都是常见的`Map`实现，是以键值对的形式存储和操作数据的容器类型。

- `Hashtable`：不推荐使用
  - 本身是同步的，
  - 键、值都不能为`null`，
  - 元素无序，
  - 默认容量为11，不要求底层数组容量一定为2的整数次幂，
  - 扩容后的容量为： 2 * 原始容量 + 1
- `HashMap`：
  - 不是同步的，
  - 只允许一条记录的键为`null`，值可有多个为`null`，
  - 元素无序，
  - 默认容量为16，底层数组容量必须为2的整数次幂，
  - 扩容后容量为原来的2倍，
  - 通常情况下，进行`put`和`get`操作可以达到常数时间的性能，所以是绝大部分利用键值对存取场景的首选；
- ``TreeMap`：基于红黑树的一种提供顺序访问的`Map`，其`get`、`put`、`remove`之类操作都是$O(logN)$的时间复杂度，具体顺序可以由指定的`Comparator`决定或根据键的自然顺序判断
  - 未实现`Comparator`接口时，键不能为`null`
  - 实现`Comparator`接口时，若未对`null`情况进行判断，则键不能为`null`



# 扩展

## 有序Map

`LinkedHashMap`通常提供的是遍历顺序符合插入顺序，实现是通过为条目（键值对）维护一个双向链表。注意，通过特定构造函数，可以创建反映访问顺序的实例，所谓的 put、get、compute 等，都算作“访问”。这种行为适用于一些特定应用场景，如，构建一个空间占用敏感的资源池，希望可以自动将最不常被访问的对象释放掉，就可以利用`LinkedHashMap`提供的机制来实现。

`TreeMap`的整体顺序是由键的顺序决定的，通过`Comparator`或`Comparable`(自然顺序)来决定。



## HashMap源码

==JDK1.8以前==：

- 底层是数组和单向链表结合再一起使用，也就是链表散列。通过 key 的`hashCode`来计算 hash 值，当`hashCode`相同时，通过“拉链法”解决冲突。 
- 如果定位到的数组位置不含链表（当前`entry`的`next`指向null）,那么对于查找，添加等操作很快，仅需一次寻址即可；如果定位到的数组包含链表，对于添加操作，其时间复杂度依然为$O(1)$，因为最新的`entry`会插入链表头部（因为后插入的`entry`被查找的可能性更大），仅需要简单改变引用链即可，而对于查找操作来讲，此时就需要遍历链表，然后通过key对象的`equals()`方法逐一比对查找。
- `HashMap`的容量是有限的。当经过多次元素插入，使得`HashMap`达到一定饱和度时，Key映射位置发生冲突的几率会逐渐提高。这时候，`HashMap`需要扩展它的长度，也就是进行**Resize**。rehash 是 resize 的子过程。
  - 当 **HashMap.Size   >=  Capacity * LoadFactor时，会进行Resize**。 步骤：
    - 1.扩容——> 创建一个新的Entry空数组，长度是原数组的2倍。 
    - 遍历原Entry数组，把所有的Entry重新Hash到新数组。 

==JDK1.8以后==：

其内部结构可以看作是数组(`transient Node<K,V>[] table`)和链表结合组成的符合结构，数组被分成一个个桶(bucket)，通过哈希值决定键值对在这个数组的寻址，哈希值相同的键值对则以链表形式存储。注意：**当链表大小超过阈值(`TREEIFY_THRESHOLD = 8`)时，将链表转化为红黑树，利用红黑树快速增删查改的特点提高`HashMap`的性能；而当桶上的节点数小于阈值(`UNTREEIFY_THRESHOLD = 6`)时，又会将红黑树转为链表**。单纯的链表中查询操作是线性的，会严重影响存取的性能。

![HashMap](https://github.com/fms5cmS/MyNote/raw/master/images/Collection-HashMap.png)

`resize()`：初始化存储表格(当`table==null`或`table.length==0`)或者在容量不满足需求时进行扩容。

阈值 **`threshold`** = capacity * loadFactor ：所能容纳的 key-value 对极限，当 size >= threshold 时，需要对数组扩容resize，扩容后的容量是之前的两倍。 

容量capacity和负载引起loadFactor 决定了可用的桶的数量，空桶太多会浪费空间，如果使用的太满则会严重影响操作性能。

- 初始容量的设置要同时满足以下两个条件：
  - 需要符合计算条件：capacity * loadFactor > 元素数量，所以**初始容量  > 预估元素数量 / loadFactor**
  - 初始容量必须是 2 的幂数。（`tableSizeFor(int cap)`方法根据给定的数字返回一个 2 的幂次）

- 负载因子：
  - 无特别需求，不要更改，默认的负载因子(`DEFAULT_LOAD_FACTOR = 0.75f`)符合通用场景的需求；
  - 如果确实要调整，建议不要设置超过 0.75 的值，因为会显著增加冲突，降低`HashMap`的性能。

- 树化
  - 改造的主要逻辑在`putVal`和`treeifyBin`中
  - 当 bin 的数量大于`TREEIFY_THRESHOLD`时：
    - 如果容量小于`MIN_TREEIFY_CAPACITY`，只进行简单的扩容；
    - 如果容量大于`MIN_TREEIFY_CAPACITY`，进行树化改造。