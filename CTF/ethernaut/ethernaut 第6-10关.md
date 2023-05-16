# 第六关：Delegation

## 1. 题目要求
`成为合约的拥有者owner `

## 2. 代码功能解读
```
在Delegate方法中
	1. 构造器初始化owner为传入的地址。
	2. pwn()方法设置发送者为owner。
在Delegation合约中
	1. 在构造器中实例化Delegate,并设置部署者为owner;
	2. fallback()回调函数，让delegate底层呼叫calldata
```


## 3. 漏洞分析
```
成为Delegation的owner
	利用delegatecall呼叫不会改变上下文环境的特点，将owner设置为玩家地址。
```

## 4. 攻击方法
```
1. 触发Delegation的fallback方法，msg.data中传入pwn()的函数选择器。
获取方法的函数选择器的方法有很多，我自己使用foundry的cast查看的。0xdd365b8b
```

## 5. 攻击调用图
![[Pasted image 20230511151718.png]]


# 第七关：Force

## 1. 题目要求
`使合约的余额大于0 `

## 2. 代码功能解读
```
本合约是一个空合约
```


## 3. 漏洞分析
```
向空合约发送eth
	普通没有payable方法的合约无法接收eth，但是self可以强制发送给其他地址。
```

## 4. 攻击方法
```
1. 创建一个拥有eth的合约，并编写一个自毁函数，接收地址设置为关卡实例地址。
```

## 5. 攻击调用图



# 第八关：Vault

### 本关知识点：
```

```

## 1. 题目要求
`把locked状态变量设置为true `

## 2. 代码功能解读
```
1. 构造函数初始化的时候传入密码写入私有变量password并设置locked为true。
2. unlock方法通过传入猜测的密码来设置locked为false。
```


## 3. 漏洞分析
```
设置locked为false，那么我们就需要传入正确的密码来完成方法执行。
	在区块链中所有数据都是公开透明的，所以我们可以通过槽位分析来得到passward的变量。	
```

## 4. 攻击方法
```
使用web3的getStorageAt方法来查看slot中的数据。
web3.eth.getStorageAt("0x407d73d8a49eeb85d32cf465507dd71d507100c1", 0) .then(console.log);
```

## 5. 攻击调用图

# 第九关：Telephone

## 1. 题目要求
`阻止其他人重获王位来通过这一关 `

## 2. 代码功能解读
```
1. 一个可支付的构造器，分别设置owner,king为合约部署者，price为发送的value。
2. receive可支付回退函数，当玩家发送的value大于price，或者是owner发送的value，将eth发送给之前的king，并让自己成为新king。
3. _king方法，查看当前的king是谁
```


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



# 第十关：Reentrance

## 1. 题目要求
` 偷走合约的所有资产`

## 2. 代码功能解读
```
1. donate是一个可支付方法，增加了balances中to地址的数量。
2. balanceOf是一个查看地址余额的方法。
3. withdraw(uint _amount)方法是一个玩家提取balances中余额的方法，输入的amount必须小于balances中的余额。
4. receive()回退函数，让合约可以接收其他人发送的eth。
```


## 3. 漏洞分析
```
重入攻击
	在withdraw方法中，合约先将钱发送给对方，然后再扣除余额，我们通过发钱的时候触发回调函数来再次进入合约，完成多次提取相同的余额。
```

## 4. 攻击方法
```
1. 攻击合约调用withdraw发送取出所有的钱，
2. 在合约的fallback方法中再次进入withdraw，请求的amount为1，这样我们可以让合约产生下溢。
3. 最后查看合约有多少钱，直接withdraw全部调走。
```

## 5. 攻击调用图


## 6. 知识点分析
