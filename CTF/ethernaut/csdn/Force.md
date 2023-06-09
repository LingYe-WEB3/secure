# 第七关：Force
### 本关知识点：
```
向没有payable方法的合约发送以太
```

## 1. 题目要求
`使合约的余额大于0 `

## 2. 代码功能解读
`空合约`
![[Pasted image 20230515214331.png]]

## 3. 漏洞分析
```
向空合约发送eth
	普通没有payable方法的合约无法接收eth，但是selfdestruct()方法可以强制发送给其他地址。
```

## 4. 攻击方法
```
1. 创建一个拥有eth的合约，并编写一个自毁函数，接收地址设置为关卡实例地址。
```

## 5. 攻击调用图
部署的时候向合约传入1wei或者更多，然后通过attack函数调用受害者合约地址，就可以发现攻击者合约中没有以太。
![[Pasted image 20230515215825.png]]
![[Pasted image 20230515215612.png]]

## 6. 知识点分析
在合约通过balance来做计算的时候要注意，防止黑客操纵合约balance发动攻击。