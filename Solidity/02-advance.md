# 函数重载

Solidity 是允许函数重载的，但不允许修饰器重载！

```solidity
contract Overloading {
  function saySomething() public pure returns (string memory) {
    return "nothing";
  }

  function saySomething(string  memory something) public pure returns (string memory) {
    return something;
  }
}
```

在调用重载函数时，会把输入的实参与入参的类型匹配！如果出现多个匹配的重载参数，则会报错。

```solidity
// 如果调用 f(50)，则因为 50 既可以转为 uint8 也可以转为 uint256，会报错
contract F {
  function f(uint8 _in) public pure returns (uint8 out) {
    out = _in;
  }

  function f(uint256 _in) public pure returns (uint256 out) {
    out = _in;
  }
}
```

# import

- 通过源文件相对位置导入：`import './father.sol';`
- 通过源文件网址导入网络中的合约：`import 'https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/utils/Address.sol';`
- 通过 npm 目录导入：`import '@openzeppelin/contracts/access/Ownable.sol';`
- 通过全局符号导入特定合约：`import {Father} from './father.sol';`

# library

库合约相比普通合约的不同之处：

1. 不能存在状态变量
2. 不能继承或被继承
3. 不能接收 ETH
4. 不可以被销毁

```solidity
library Strings {
  // ...
}

contract UseLibrary {
  // 使用库合约中函数的方式一：user for  
  using Strings for uint256;
  function getString1(uint256 _number) public pure returns (string memory) {
    return _number.toHexString();
  }

  // 使用库合约中函数的方式二：通过库合约名称调用库函数
  function getString2(uint256 _number) public pure returns (string memory) {
    return Strings.toHexString(_number);
  }
}
```

如 [Strings 合约](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/utils/Strings.sol)

# 接收&发送ETH

## 接收

Solidity 支持两种特殊的回调函数：`receive()`、`fallback()`。当向合约发送交易时，如果没有指定某个函数，就会触发这两个特殊函数：

```
     Ether is sent to contract：
              is msg.data empty?
              /                \
             yes               no
             /                  \
        has receive()?       has fallback()?
         /   \                   /
       yes   no                /
      /        \             /
    receive()  has fallback()?
                  /   \
                yes    no
                /       \
         fallback()     error
```

- receive

一个合约最多有一个 `receive()` 函数，且仅用于接收 ETH。定义时不需要 `function` 关键字，且不允许有任何参数，不能返回任何值，必须包含 `external` 和 `payable` 关键字！！

当合约接收 ETH 时，就会触发 `receive()` 。该函数最好不要执行太多逻辑，如果太复杂可能会触发 Out of Gas。

```solidity
contract Receive {
  event Received(address sender, uint value);

  receive() external payable {
    emit Received(msg.sender, msg.value);
  }
}
```

- fallback

`fallback()` 会在调用合约不存在的函数时被触发，可用于接收 ETH，也可用于代理合约。声明时同样不需要 `function` 关键字，必须有 `external`，一般也会有 `payable`。

```solidity
contract Fallback {
  event fallbackCalled(address sender, uint value, bytes data);

  fallback() external payable {
    emit fallbackCalled(msg.sender, msg.value, msg.data);
  }
}
```

## 发送

Solidity 有三种方法向其他合约发送 ETH：`transfer()`、`send()`、`call()`，推荐使用 `call()`。

```solidity
contract SendETH {
  // payable 使得在部署的时候可以转 ETH 进去！
  constructor() payable{}
  // 接收 ETH 时触发
  receive() external payable {}

  error SendFailed();
  error CallFailed();

  // 利用 transfer 发送 ETG：接收方地址.transfer(转账 ETH 金额)
  // transfer() 的 gas 限制时 2300，足够用于转账，但对方合约的 fallback() 或 receive() 不能实现太复杂的逻辑
  // transfer() 如果转账失败，会自动 revert
  function transferETH(address payable _to, uint256 amount) external payable {
    _to.transfer(amount);
  }

  // 利用 send 发送 ETH：接收方地址.send(转账 ETH 金额)
  // send() 的 gas 限制时 2300，同样对方合约的 fallback() 或 receive() 不能实现太复杂的逻辑
  // send() 如果转账失败，不会自动 revert
  // send() 返回值时 bool，代表转账是否成功，所以还需要额外的代码处理逻辑
  function sendETH(address payable _to, uint256 amount) external payable {
    bool success = _to.send(amount);
    if (!success) {
      revert SendFailed();
    }
  }

  // 利用 call 发送 ETH：接收方地址.call{value: 转账 ETH 金额}("")
  // call() 没有 gas 限制，可以支持对方合约的 fallback() 或 receive() 实现复杂的逻辑
  // call() 如果转账失败，不会自动 revert
  // call() 的返回值时 (bool, data)，其中第一个参数代表转账是否成功，所以还需要额外的代码处理逻辑
  function callETH(address payable _to, uint256 amount) external payable {
    (bool success,) = _to.call{value: amount}("");
    if (!success) {
      revert CallFailed();
    }
  }
}
```

