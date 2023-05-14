[TOC]
# 第一关：Fallback


### 本关知识点：
```
1. ether的单位转换
2. 如何触发回调方法

```


## 1. 题目要求
`获取合约所有权，并取出合约中的钱`

## 2. 代码功能解读
```
1. 构造器初始化Fallback合约，部署者赋值给变量owner，并给合约部署者1000个以太的初始贡献值。
2. contribute是一个支付方法，每次发送的eth必须小于0.001 ether，且当玩家发出的总数大于合约owner时，玩家成为owner。
3. getContribution方法查看调用者的捐赠总数。
4. withdraw方法，只有owner才能调用，将合约所有ether发送给owner。
5. receive时一个可支付的回调方法，当玩家发送的余额大于0，且在合约中有过捐赠，则可以成为owner。
```

## 3. 漏洞分析
```
攻击点在于如何成为owner，因为我们成为owner之后就可以获得所有钱。
	通过触发receive()这个回调方法，可以让我们成为owner。
	成为owner后取钱。
```

## 4. 攻击方法
```
1. 通过调用一次contribute()方法和一次receive()方法，让自己成为合约拥有者。
2. 调用withdraw取出合约中的所有钱
```

## 5. 攻击调用图
![[Pasted image 20230511111609.png]]

## 6. 知识点分析

在本关中，通过对合约的阅读发现，我们的攻击需要触发receive并发送一定的value。
1. 如何触发receive
	在solidity的0.6.0版本后，定义了关键字receive为接收以太函数。
	-   如果有 `receive` 函数,向合约转账不管是否调用数据，都会触发receive，且receive为一个payable方法。
	-  当调用的数据没有被其他函数匹配时，才会调用fallback，也可以声明为payable方法，但是只有在receive没有被匹配的时候才会调用fallback。
2. ether的单位转换
	以太币的最小单位是wei，在我们传入的value没有货币单位的时候
		1 wei == 1
		1 gwei == 100000000 wei
		1 ether  == 10000000000000000 wei
		1 ether ==  100000000 gwei
