合约的基本结构：

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.4;

contract HelloWorld {
  string public greet = "HelloWorld";
}
```

- `// SPDX-License-Identifier: MIT` 声明开源协议版本；
- `pragma solidity ^0.8.4;` 声明 solidity 版本大于等于 0.8.4 且小于 0.9.0
    - 等价于：`pragma solidity >=0.8.4 <0.9.0;`

# 数据类型

## 原生数据类型

```solidity
contract PrimitiveDataType {
  // boolean 默认值为 false
  bool public boo = true;

  // uint8, uint16 ... uint256
  uint8 public u8 = 1;
  uint public u256 = 456;
  uint public u = 123;

  // int8, int16 ... int256
  int8 public i8 = - 1;
  int public i256 = 456;
  int public i = - 123;
  // int 的最小和最大值
  int public minInt = type(int).min;
  int public maxInt = type(int).max;

  // 地址类型，默认值为 0x0000000000000000000000000000000000000000
  // 存储一个 20byte 的以太坊地址，有两种类型：普通地址、payable 地址（可以转账）  
  address public addr = 0xCA35b7d915458EF540aDe6068dFe2F44E8fa733c;
  address payable public addr1 = payable(addr);
  uint256 public balance = addr1.balance;
}
```

- boolean 类型的运算符：`!`(逻辑非)，`&&`(逻辑与)，`||`(逻辑或)，`==`(等于)，`!=`(不等于)
- int、uint 类型的运算符，除了通用的外：`**`(幂)，
- `delete a` 可以让变量 a 的值变为默认值

## 字节数组

- 定长字节数组：`byte`、`bytes8`、`bytes32`，属于值类型！
- 不定长字节数组：`bytes`，属于引用类型！

```solidity
contract ByTes {
  bytes32 public _byte32 = "Mini Solidity";
  bytes1 public _byte = _bytes32[0];
}
```

定长的字节数组存储数据时，消耗的 gas 较少。

## 枚举

```solidity
contract Enum {
  // 使用名称来代替从 0 开始的 uint
  enum ActionSet {Buy, Hold, Sell}
  ActionSet buy = ActionSet.Buy;

  // 显式将 enum 转为 uint
  function enumToUint() external view returns (uint) {
    return uint(buy);
  }
}
```

# 修饰符

可见性修饰符

- `public`：修饰函数时该函数合约内外部均可见，也可以用于修饰状态变量，`public` 变量会自动生成 getter 函数
- `private`：修饰函数时该函数只能从本合约内部访问，继承本合约的子合约也不能访问，也可以用于修饰状态变量
- `external`：修饰函数时该函数只能从合约外部访问，内部不可以调用（不过可以用 `this.函数名()` 来调用）
- `internal`：修饰函数时该函数只能从合约内部访问，继承的字合约也可以用，也可以用于修饰状态变量

其他修饰符（仅用于修饰函数）

- `pure`：不读取也不改变状态变量
- `view`：函数内部的状态变量是只读的，不可变更
- `payable`：带有该修饰符的函数才可以向合约转入 ETH
    - 接受 ETH 的合约一定要有提取功能，否则资产会被锁死在里面！！

# 函数

函数签名的形式：`function functionName (parameterTypes) {internal|external|public|private} [pure|view|payable] [returns (<return types>)]`

- 函数可见性修饰符，默认为 `internal`

```solidity
contract FunctionTypes {
  uint256 public number = 5;

  function minus() internal {
    number = number - 1;
  }

  // 合约内的函数可以调用内部（internal）函数
  function minusCall() external {
    minus();
  }

  // 能给合约支付 ETH 的函数
  function minusPayable() external payablle returns (uint256 balance) {
    minus();
    balance = address(this).balance;
  }

  // 如果再加上 pure 修饰符，则会报错，因为 pure 是不能读取和改变状态变量的
  // 如果加上 view 修饰符，也会报错，view 可以读取但是不能改变状态变量
  function add() external {
    number = number + 1;
  }

  // 这样是不会报错的
  function addView() external view returns (uint256 new_number) {
    new_number = number + 1;
  }
}
```

- `returns`：用于声明函数返回
- `return`：返回指定变量

```solidity
contract FunctionReturn {
  // 命名式返回，会自动返回这些值，不需要加 return
  function returnNames() public pure returns (uint256 _number, bool _bool, uint256[3] memory _arr) {
    _number = 2;
    _bool = true;
    _arr = [uint256(3), 2, 1];
  }

  function read() public {
    // 可以读取部分返回值，不需要的部分留空
    (, boo2,) = returnNames();
  }
}
```

## 构造函数 constructor

每个合约可以定义一个 constructor 函数，并在部署合约的时候自动运行一次。可以用来初始化合约的一些参数。 

```solidity
contract ConstructorTest {
  address public owner;
  constructor() {
    owner = msg.sender;
  }
}
```

## 修饰器 modifier

```solidity
contract ModifierTes {
  address public owner;
  // 定义函数修饰器，并在需要的函数上声明该函数修饰器
  // 在函数上声明后，每次执行函数前，先运行修饰器中的代码
  modifier onlyOwner {
    require(msg.sender == owner);
    _; // 如果上一行检查通过，则继续运行函数主体
  }


  function changeOwner(address _newOwner) external onlyOwner {
    owner = _newOwner;
  }
}
```



# 数据存储

## 引用类型

### 数组 array

数组分为定长数组、动态数组

```solidity
contract Arr {
  // 定长数组，声明时指定数组长度
  uint[8] array1; // 默认值未 [0, 0, 0, 0, 0, 0, 0, 0]
  bytes1[5] array2;

  // 动态数组声明时不用指定数组长度
  uint[] arr1;
  bytes1[] arr2;
  bytes arr3; // 该类型较为特殊，也是数组

  function f() public {
    // memory 修饰的动态数组可以使用 new 操作符来创建，但必须声明长度！且声明后长度不可变
    uint[] memory arr4 = new uint[](5); // 5 是在指定长度
    bytes memory arr5 = new bytes(9);

    // 使用 [] 来初始化数组，且里面每个元素的类型以第一个元素为准，如果没有指定类型的话，默认为最小单位的该 type
    // 如，[1, 2, 3] 中所有元素就是 uint8 类型，因为 int 的默认最小单位类型就是 uint8
    // 所以下面必须对第一个值进行显式的类型转换，否则默认的类型 uint8 会报错
    g([uint(1), 2, 3]);
  }

  function g(uint[3] memory) public pure {
    // 如果创建的是动态数组，就需要一个一个元素的赋值
    uint[] memory x = new uint[](2);
    x[0] = 1;
    x[1] = 2;
  }

  // string 的底层其实是 bytes
  function str(string memory _name) public {
    _name = "cat";
  }
}
```

数组的成员：

- `length`：`memory` 数组的长度在创建后是固定的
- `push()`：动态数组和 `bytes` 拥有该成员，可以在数组最后添加一个 0 元素
- `push(x)`：动态数组和 `bytes` 拥有该成员，可以在数组最后添加一个 x 元素
- `pop()`：：动态数组和 `bytes` 拥有该成员，可以移除数组最后一个元素

### 结构体 struct

```solidity
contract Struct {
  // 声明结构体
  struct Student {
    uint256 id;
    uint256 score;
  }

  Student student; // 初始化一个结构体
  // 给结构体赋值的两种方式
  function initStudent() external {
    // 1. 创建一个 storage 的结构体引用
    // 这里将一个合约 storage 赋值给本地 storage，会创建引用，修改时会将原对象一起修改。赋值规则见下面
    Student storage _student = student;
    _student.id = 11;
    _student.score = 100;

    // 2. 直接引用状态变量的结构体
    student.id = 11;
    student.score = 99;
  }
}
```

### 映射 mapping

```solidity
contract Map {
  // 声明一个 key 类型为 uint，value 类型为 address 的 mapping
  mapping(uint => address) public id2Address;

  function f() public {
    id2Address[0] = 0xCA35b7d915458EF540aDe6068dFe2F44E8fa733c;
  }
}
```

- key 的类型必须是 Solidity 默认的类型，不能用自定义的结构体！
- value 的类型可以是自定义类型
- 存储位置必须是 `storage`！不能用于 `public` 函数的参数或返回中！！
- `mapping` 不存储 key，而是采用 keccak256(key) 当作 offset 来存取 value
- Ethereum 会定义所有未使用的空间为 0，所以未赋值的 key 在查询时得到的是初始值 0。
    - 上面的 `public` 变量 id2Address 会自动生成 getter 方法，如果查询 id2Address[1] 的值，得到会是
      0x0000000000000000000000000000000000000000

## 存储位置

Solidity 中的引用类型：array、struct、mapping，在使用时必须声明数据存储的位置！

Solidity 中有三类数据存储位置：`storage`、`memory`、`calldata`，不同存储位置的 gas 成本不同。

- `storage`：状态变量默认都是 `storage`，存储在链上
- `memory`：函数中的参数和临时变量一般使用，存储在内存中，不上链
- `calldata`：类似 `memory`，存储在内存中，但是 `calldata` 声明的变量不能修改！一般用于函数的参数

## 赋值规则

- 合约状态变量 `storage` 赋值给函数中的本地 `storage`，会创建引用！

```solidity
contract Storage2Storage {
  uint[] x = [1, 2, 3];

  function f() public {
    uint[] storage xx = x;
    xx[0] = 100; // 这里修改也会影响到 x[0]
  }
}
```

- `storage` 赋值给 `memory`，会创建独立的副本！

```solidity
contract Storage2Memory {
  uint[] x = [1, 2, 3];

  function f() public {
    uint[] memory xx = x;
    xx[0] = 100; // 修改不会影响到 x[0]
  }
}
```

- `memory` 赋值给 `memory` 会创建引用！
- 其他情况下，变量赋值给 `storage` 会创建独立的副本。

# 变量作用域

Solidity 中变量按作用域分为：状态变量、局部变量、全局变量。

- 状态变量 State variable：数据存储在链上，所有合约函数都能访问。声明位置位于合约内、函数外
- 局部变量 Local variable：仅在函数执行过程中有效的变量，函数退出后，变量无效。声明位置位于函数内
- 全局变量 Global variable：都是 Solidity 预留的关键字，可以在函数内不声明直接使用

完整的全局变量列表见[全局变量](https://learnblockchain.cn/docs/solidity/units-and-global-variables.html#special-variables-and-functions)

```solidity
contract Variables {
  // 状态变量
  uint public x = 1;
  uint public y;

  function bar() external pure returns (uint) {
    // 局部变量
    uint xx = 1;
    uint yy = 2;
    uint zz = xx + yy;
    return zz;
  }

  function global() external view returns (address, uint, bytes memory) {
    address sender = msg.sender;
    uint blockNum = block.number;
    bytes memory data = msg.data;
    return send, blockNum, data;
  }
}
```

# constant 常量 & immutable 不变量

`constant`、`immutable` 这两种变量会更省 gas，因为数据会直接保存到合约的字节码中，而不会存储到链上。

只有变量可以声明 `constant`、`immutable`，而 `string` 和 `bytes` 虽然可以声明为 `constant`，但不能声明`immutalble`！

```solidity
contract Const {
  // constant 变量必须在声明时初始化，之后再也不能改变
  uint256 constant CONSTANT_NUM = 10;
  string constant CONSTANT_STRING = "0xAa";
  bytes constant CONSTANT_BYTES = "fms5cmS";
  address constant CONSTANT_ZERO_ADDRESS = 0x0000000000000000000000000000000000000000;

  // immutable 变量可以在 constructor 中初始化，之后不能改变！
  uint256 public immutable IMMUTABLE_NUM = 9999999;
  address public immutable IMMUTABLE_ADDRESS;
  uint256 public immutable IMMUTABLE_BLOCK;
  uint256 public immutable IMMUTABLE_TEST;

  // 在 constructor 中初始化 immutable 变量
  constructor() {
    IMMUTABLE_ADDRESS = address(this);
    IMMUTABLE_BLOCK = block.number;
    IMMUTABLE_TEST = test();
  }

  function test() public pure returns (uint256) {
    uint256 what = 9;
    return what;
  }
}
```

# 控制流

```solidity
contract Control {
  // 三元运算符
  function ternary(uint _x) public pure returns (uint) {
    return _x < 10 ? 1 : 2;
  }

  function foo(uint x) public pure returns (uint) {
    if (x < 10) {
      return 0;
    } else if (x < 20) {
      return 1;
    } else {
      return 2;
    }
  }

  function loop() public pure {
    // for loop
    for (uint i = 0; i < 10; i++) {
      if (i == 3) {
        continue;
      }
      if (i == 5) {
        break;
      }
    }

    // while loop
    uint j;
    while (j < 10) {
      j++;
    }

    // do-while loop
    uint sum = 0;
    uint i = 0;
    do {
      sum += i;
      i++;
    }
    while (i < 10);
  }
}
```

插入排序：

```solidity
contract InsertSort {
  // 插入排序 正确版
  function insertionSort(uint[] memory a) public pure returns (uint[] memory) {
    // note that uint can not take negative value
    for (uint i = 1; i < a.length; i++) {
      uint temp = a[i];
      uint j = i;
      while ((j >= 1) && (temp < a[j - 1])) {
        a[j] = a[j - 1];
        j--;
      }
      a[j] = temp;
    }
    return (a);
  }
}
```

# 事件

Solidity 中的事件 `event` 是 EVM 上的日志抽象，应用程序可以通过 RPC 接口订阅和监听事件，同时 `event` 也是 EVM 上比较经济的数据存储方式，每个大概消耗 2000 gas，而链上存储一个新的变量至少需要 20000 gas。

在 [ERC20 接口](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC20/IERC20.sol)中定义的 Transfer 事件：

```solidity
interface IERC20 {
  // 声明事件 event
  // indexed 标记的变量是用于检索事件的索引，在 EVM 上单独作为一个 topic 进行存储和索引
  // 每个 indexed 变量大小为固定的 256bit
  // topic[0] 是事件的 keccak256 哈希，topic[1]、topic[2]、topic[3] 存储了带 indexed 标识的变量的 keccak256 哈希
  // 不带 indexed 标识的变量会存储在事件的 data 中，data 是不能被直接检索的，但可以存储任意大小的数据
  // data 部分的变量在存储上消耗的 gas 相比 topic 更少
  event Transfer(address indexed from, address indexed to, uint256 value);
}
```

当需要时，使用 `emit Transfer(from, to, amount)` 这样的语法来释放事件。

# 继承

- `virtual`：父合约中的函数，如果希望字合约重写，需要加上该关键字
- `override`：子合约重写了父合约中的函数，需要加上该关键字

## 简单继承

```solidity
contract Grandpa {
  event Log(string msg);

  function hip() public virtual {emit Log("Grandpa");}

  function pop() public virtual {emit Log("Grandpa");}
}

// x is y 来说明 x 继承 y
contract Father is Grandpa {
  // 重写的函数，除此之外还有一个继承的函数 pop 没有重写，所以还是输出的事件内容没有变化
  function hip() public virtual override {emit Log("father");}
  // 自定义的函数
  function father() public virtual {emit Log("father");}
}
```

## 多重继承

```solidity
// 继承时，需要按从上往下的顺序排！不然会报错
// 如果某个函数在多个继承的合约中都存在，在子合约中必须重写！且必须在 override 后面加上所有父合约的名字！！
contract BX is Grandpa, Father {
  function hip() public virtual override(Grandpa, Father) {emit Log("bx");}

  function pop() public virtual override {emit Log("bx");}
}
```

## 修饰器的继承

```solidity
contract Base {
  // 也需要加上 virtual
  modifier exactDivBy2And3(uint _a) virtual {
    require(_a % 2 == 0 && _a % 3 == 0);
    _;
  }
}

contract Identifier is Base {
  // 也需要加上 override，当前合约也可以直接使用父合约的修饰器，而不重写
  modifier exactDivBy2And3(uint _a) override {
    _;
    require(_a % 2 == 0 && _a % 3 == 0);
  }
}
```

## 构造器的继承

```solidity
abstract contract A {
  uint public a;
  constructor(uint _a) {
    a = _a;
  }
}

// 方式一：
contract B is A {
  constructor(_uint _b) A(_c * _c) {}
}
// 方式二：继承时声明父构造函数的参数
contract C is A(1) {}
```

## 调用父合约的函数

```solidity
contract CallFather is Grandpa, Father {
  // 方式一：通过 父合约名.函数名 来调用
  function callParent() public {
    Grandpa.pop();
  }

  // 方式而：通过 super.函数名 来调用最近的父合约函数
  // 如 这里调用的就是 Father 的，而不是 Grandpa 的
  function callParentSuper() public {
    super.pop();
  }
}
```

# abstract & interface

抽象合约：如果合约中至少有一个未实现的函数，则必须将该合约标记为 `abstract`！且未实现的函数需要加上 `virtual` 以便子合约重写。

```solidity
abstract contract InsertionSort {
  function insertionSort(uint []memory a) public pure virtual returns (uint[] memory);
}
```

接口类似抽象合约不实现任何功能：
1. 不能包含状态变量
2. 不能包含构造函数
3. 不能继承除接口外的其他合约
4. 所有函数都必须是 `external` 且不能有函数体
5. 继承接口的合约必须实现接口定义的功能

[ERC20 接口](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC20/IERC20.sol)

# 异常

## error

Solidity 0.8 后增加了 `error`，可以方便省 gas 地向用户暴露操作失败的原因。可以在合约外定义异常！

```solidity
// 自定义 error
  error TransferNotOwner();

contract Err {
  mapping(uint256 => address) public _owners;

  function transferOwner1(uint256 tokenId, address newOwner) public {
    if (_owners[tokenId] != msg.sender) {
      // 必须搭配 revert 使用！
      revert TransferNotOwner();
    }
    _owners[tokenId] = newOwner;
  }
}
```

## require

Solidity 0.8 之前使用 `require` 来跑出异常。但是 gas 会随着描述异常的字符串长度增加，比 `error` 的消耗高。

```solidity
contract Err {
  mapping(uint256 => address) public _owners;

  function transferOwner1(uint256 tokenId, address newOwner) public {
    require(_owners[tokenId] == msg.sender, "Transfer not owner");
    _owners[tokenId] = newOwner;
  }
}
```

## assert

`assert` 一般用于写 debug，因为它无法解释抛出异常的原因。

```solidity
contract Err {
  mapping(uint256 => address) public _owners;

  function transferOwner1(uint256 tokenId, address newOwner) public {
    assert(_owners[tokenId] == msg.sender);
    _owners[tokenId] = newOwner;
  }
}
```
