要想学会设计模式还是得先学会UML

要想学会UML就先看看其中类之间的六种关系吧

https://design-patterns.readthedocs.io/zh_CN/latest/read_uml.html

generalization泛化关系 空心实箭头

is-a关系 属于继承和泛化

realize实现关系 空心箭头虚线

aggregation聚合关系	空心菱形  整体和部分不是强依赖的，整体不存在了部分依然存在

composition组合关系	实心菱形 整体和部分是强以来的，整体不存在部分也不会存在

association关联关系	一条直线	关联关系默认不强调方向

dependency依赖关系	---> 表示A依赖于B；他描述一个对象在运行期间会用到另一个对象的关系

----------------

附mermaid语法

```mermaid
graph LR
	A --text--> B
	B -.- A
	c --> A
	B --> c
```

```
​```mermaid
	graph 流程图方向(TB上到下/BT下到上/RL右到左/LR左到右/TD同TB)
	流程图内容
​```
```

```mermaid
graph TB
id[带文本的矩形]
id4(带文本的圆角矩形)
id1{带文本的菱形}
id2((带文本的圆形))
id3>带文本的不对称的矩形]

```

```mermaid
graph LR
    A[A] --> B[B] 
    A1[A] --- B1[B] 
    A4[A] -.- B4[B] 
    A5[A] -.-> B5[B] 
    A7[A] ==> B7[B] 
    A2[A] -- 描述 --- B2[B] 
    A3[A] -- 描述 --> B3[B] 
    A6[A] -. 描述 .-> B6[B] 
    A8[A] == 描述 ==> B8[B] 
```

```mermaid
graph TB
    c1-->a2
    subgraph one
    a1-->a2
    end
    subgraph two
    b1-->b2
    end
    subgraph three
    c1-->c2
    end
```

```mermaid
graph LR
    start[开始] --> input[输入A,B,C]
    input --> conditionA{A是否大于B}
    conditionA -- YES --> conditionC{A是否大于C}
    conditionA -- NO --> conditionB{B是否大于C}
    conditionC -- YES --> printA[输出A]
    conditionC -- NO --> printC[输出C]
    conditionB -- YES --> printB[输出B]
    conditionB -- NO --> printC[输出C]
    printA --> stop[结束]
    printC --> stop
    printB --> stop
```