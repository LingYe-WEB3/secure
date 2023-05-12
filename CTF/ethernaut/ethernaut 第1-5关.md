[TOC]
# 第一关：fallback

## 1. 题目要求
`获取合约所有权，并取出合约中的钱`

## 2. 代码功能解读
```
1. 初始化Fallback合约，部署者赋值给变量owner，并给合约部署者1000个以太的初始贡献值。
2. contribute是一个支付方法，每次发送的eth必须小于0.001 ether，且当玩家发出的总数大于合约owner时，玩家成为owner。
3. getContribution方法查看调用者的捐赠总数。
4. withdraw方法，只有owner才能调用，将合约所有ether发送给owner。
5. receive时一个可支付的回调方法，当玩家发送的余额大于0，且在合约中有过捐赠，则可以成为owner。
```

## 3. 漏洞分析
```
攻击点在于如何成为owner，因为我们成为owner之后就可以获得所有钱。
	通过对receive()方法的调用，可以让我们没有大于1000 ether就可以取出所有钱。
```

## 4. 攻击方法
```
1. 通过调用一次contribute()方法和一次receive()方法，让自己成为合约拥有者。
2. 调用withdraw取出合约中的所有钱
```

## 5. 攻击调用图
![[Pasted image 20230511111609.png]]





# 第二关：fallout

## 1. 题目要求
`获取合约所有权`

## 2. 代码功能解读
```
1. Fal1out()方法是一个可支付方法，发送者成为owner，并在allocations中记录他发送的value
2. allocate()方法是一个可支付方法，在allocations中增加发送value的总数。
3. sendAllocation(address payable allocator)方法中，如果allocations记录中大于0，则发送相应的eth给他
4. collectAllocations()是一个只有owner才可以调用的方法，可以获得合约中的所有钱。
5. allocatorBalance(address allocator)方法查看allocator在合约中发送的eth。
```

## 3. 漏洞分析
```
攻击点在于如何成为owner，因为我们成为owner之后就可以获得所有钱。
	本合约的编译版本是^0.6.0，在合约中特别标注了Fal1out()应该是一个constructor。这个方法是constructor的条件是必须与合约名称相同，但是仔细看发现和合约名称并不相同，所以我们可以调用这个方法成为owner。
```

## 4. 攻击方法
```
1. 调用Fal1out()方法成为拥有者。
```

## 5. 攻击调用图
![[Pasted image 20230511111915.png]]





# 第三关：CoinFlip

## 1. 题目要求
`这是一个掷硬币的游戏，你需要连续的猜对结果。完成这一关，你需要通过你的超能力来连续猜对十次。`

## 2. 代码功能解读
```
1. 构造器初始化consecutiveWins为0
2. flip(bool _guess)方法中，不能在同一个区块中调用此方法，当我们输入的_guess猜中计算的答案的时候，consecutiveWins累加，猜错一次就重置次数。
```


## 3. 漏洞分析
```
攻击点在于区块中的数据没有随机性，都是确定的。
	我们可以编写一个方法得到答案，然后再调用flip方法，然后当区块跟新后就再次计算答案，连续十次即可。
```

## 4. 攻击方法
	编写攻击合约，调用攻击合约进行攻击。

``` java

// SPDX-License-Identifier: MIT

pragma solidity ^0.8.0;

  

interface CoinFlipInterface {

    function flip(bool _guess) external returns (bool);

}

  

contract Exploit {

    uint256 FACTOR =

        57896044618658097711785492504343953926634992332820282019728792003956564819968;

    CoinFlipInterface victim;

  

    constructor(address _victim) {

        victim = CoinFlipInterface(_victim);

    }

    function Attack() public returns (bool) {

        bool guess_ = guess();

        return victim.flip(guess_);

    }

    function guess() public view returns (bool) {

        uint256 blockValue = uint256(blockhash(block.number - 1));

        uint256 coinFlip = blockValue / FACTOR;

        bool side = coinFlip == 1 ? true : false;

        return side;

    }

}
```

## 5. 攻击调用图

![[Pasted image 20230511142044.png]]
有时间的朋友可以自己完成十次攻击。
![[Pasted image 20230511141951.png]]



# 第四关：Telephone

## 1. 题目要求
`玩家成为合约的owner`

## 2. 代码功能解读
```
1. 构造器初始化部署者为owner
2. changeOwner(address _owner)方法当tx.origin != msg.sender的时候_owner成为拥有者
```


## 3. 漏洞分析
```
让tx.origin不等于msg.sender
	tx.origin是合约的最开始的调用者是谁
	msg.sender是当前合约的调用者是谁
	
```

## 4. 攻击方法
```
创建一个攻击合约
让攻击合约调用changeOwner方法
```

## 5. 攻击调用图
![[Pasted image 20230511143833.png]]
![[Pasted image 20230511143936.png]]

# 第五关：Token

## 1. 题目要求
`通过某种方法可以增加你手中的 token 数量,你就可以通过这一关,当然越多越好 `

## 2. 代码功能解读
```
1. 构造器初始化部署者的balances为_initialSupply，且等于totalSupply。
2. transfer(address _to, uint _value)方法执行玩家balances中的数量转移，发送的数量必须小于等于玩家在合约中balances中的数量
3. balanceOf(address _owner)方法查看地址的余额。
```


## 3. 漏洞分析
```
增加玩家的balances数量
	在solidity版本0.8.0之前，数学运算会有溢出问题，这里利用溢出来达到增加余额的目的。
	
```

## 4. 攻击方法
```
1. 调用transfer方法，value填写为21，让计算产生下溢。
```

## 5. 攻击调用图
![[Pasted image 20230511145928.png]]