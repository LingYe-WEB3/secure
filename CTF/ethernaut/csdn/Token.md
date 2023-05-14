# 第五关：Token
### 本关知识点：
```
不同的合约版本之间的升级改动
```

## 1. 题目要求
`通过某种方法可以增加你手中的 token 数量,你就可以通过这一关,当然越多越好 `

## 2. 代码功能解读
```
1. 构造器初始化部署者的balances为_initialSupply，且等于totalSupply。
2. transfer(address _to, uint _value)方法执行玩家balances中的数量转移，发送的数量必须小于等于玩家在合约中balances中的数量
3. balanceOf(address _owner)方法查看地址的余额。
```


## 3. 漏洞分析
```
增加玩家的balances数量
	在solidity版本0.8.0之前，数学运算会有溢出问题，这里利用溢出来达到增加余额的目的。
	
```

## 4. 攻击方法
```
1. 调用transfer方法，value填写为21，让计算产生下溢。
```

## 5. 攻击调用图
![[Pasted image 20230511145928.png]]
