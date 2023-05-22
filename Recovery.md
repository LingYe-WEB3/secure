# 第十七关：Recovery

### 本关知识点：
```
1. 利用etherscan查找历史交易
```

## 1. 题目要求
` 如果您能从丢失的的合约地址中找回(或移除)，则顺利通过此关`

## 2. 代码功能解读
![[2023-05-21_205407.png]]


## 3. 漏洞分析
```
找到忘记的合约地址
	1. 我们可以通过区块链浏览器查看所有调用了generateToken方法的人，找到忘记地址的玩家创建的地址。
	2. 调用合约的destroy方法发送余额给玩家。
```

## 4. 攻击方法

 ``在区块链中查询到玩家创建的合约地址
 ``调用destroy方法转给player


## 5. 攻击调用图
![[Pasted image 20230521211055.png]]
![[Pasted image 20230521211113.png]]

实例化这个地址
![[Pasted image 20230521211206.png]]

将合约余额取出来给player
![[Pasted image 20230521211306.png]]

成功
![[Pasted image 20230521211357.png]]

## 6. 知识点分析


如果对大家有用，请点赞；如果喜欢，请订阅加点赞，会一直更新~