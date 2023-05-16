# 第六关：Delegation
### 本关知识点：
```
1. delegatecall调用得上下文环境
2. msg.data的数据内容
```

## 1. 题目要求
`成为合约的拥有者owner `

## 2. 代码功能解读
![[Pasted image 20230515140705.png]]

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

## 6. 知识点分析
- delegatecall调用常发生在代理合约中，利用delegatecall能够将存储代码和逻辑代码分离，从而实现使项目更容易维护。但是要尤其注意存储的位置，否则容易发生槽位覆写等问题。
- msg.data是完整的 calldata 数据，我们在calldata中传入的数据是0xdd365b8b，所以根据delegatecall的调用，他会去Delegate中找到函数选择器等于dd365b8b的方法，并在Delegatetion的上下文中执行。