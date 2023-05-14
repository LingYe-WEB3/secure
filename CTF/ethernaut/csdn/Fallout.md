[toc]
# 第二关：Fallout
### 本关知识点：
```
1. 声明构造器的方法
```

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

## 6. 知识点分析
构造函数是一个只会在部署合约的时候运行一次的函数。在本关中试图用和合约名称相同的函数充当构造函数，编写错误导致了可以让其他人重复调用。
- 在Solidity 0.4.22之前，构造函数不使用 `constructor` 而是使用与合约名同名的函数作为构造函数而使用，由于这种旧写法容易使开发者在书写时发生疏漏,于是在solidity的0.4.22版本开始之后，用constructor来定义构造函数，有效防止构造函数的误写和提高了合约易读性。