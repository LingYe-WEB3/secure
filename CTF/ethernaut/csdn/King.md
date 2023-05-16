# 第九关：King
[toc]
### 本关知识点：
```
主动让函数调用失败，进行revert。
```

## 1. 题目要求
`阻止其他人重获王位来通过这一关 `

## 2. 代码功能解读
![[Pasted image 20230516141044.png]]


## 3. 漏洞分析
```
阻止其他人成为king，那么在玩家当国王的时候，拒绝其他人的付款就可以完成挑战。
	通过在king合约的回退函数中直接调用revert(),或者耗尽gas来达到无法付款的目的。

```

## 4. 攻击方法
``` java
fallback() external payable{ revert(); }
```

## 5. 攻击调用图

部署攻击合约，并放入了1 ether。在攻击的时候传入受害者合约地址就行。
![[Pasted image 20230516152204.png]]

攻击后，攻击合约地址就成为了king，并且其他人无法成为king，通过本关
![[Pasted image 20230516152409.png]]



调用攻击函数后查看，合约拥有者是攻击合约，并其他人无法成为king。
## 6. 知识点分析
	在发送外部交易的时候是危险的，对方的操作可以使方法执行失败，造成不可挽回的损失。
	解决方法: 将之前的king可获得的金额写入合约，让他们自己来取，防止有人恶意成为king。


