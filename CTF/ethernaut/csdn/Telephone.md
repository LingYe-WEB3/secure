# 第四关：Telephone
### 本关知识点：
```
1. tx.origin和msg.sender的区别。
```

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