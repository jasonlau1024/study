## Python核心基础

Python的核心语言元素：变量、类型、运算符、表达式、分支结构、循环结构等。

### 概述

#### 发展历程

1991年2月：第一个Python编译器（同时也是解释器）诞生，它是用C语言实现的（后面），可以调用C语言的库函数。

1994年1月：Python 1.0正式发布。

2000年10月：Python 2.0发布。增加了完整的[垃圾回收](https://zh.wikipedia.org/wiki/%E5%9E%83%E5%9C%BE%E5%9B%9E%E6%94%B6_(%E8%A8%88%E7%AE%97%E6%A9%9F%E7%A7%91%E5%AD%B8))，提供了对[Unicode](https://zh.wikipedia.org/wiki/Unicode)的支持。

2008年12月：Python 3.0发布。为尽可能的兼容Python2，Python 3.x的很多新特性后来也被移植到Python 2.6/2.7版本中。

#### 版本说明

**版本号A.B.C：**

- A：表示出现大改动或不向后兼容的更改。
- B：表示功能更新，出现新功能。
- C：表示小的改动，如：修复某个BUG。

#### 优缺点

Python的优点很多，简单的可以总结为以下几点。

1. 简单和明确，做一件事只有一种方法。
2. 学习曲线低，跟其他很多语言相比，Python更容易上手。
3. 开放源代码，拥有强大的社区和生态圈。
4. 解释型语言，天生具有平台可移植性。
5. 对两种主流的编程范式（面向对象编程和函数式编程）都提供了支持。
6. 可扩展性和可嵌入性，例如在Python中可以调用C/C++代码。
7. 代码规范程度高，可读性强，适合有代码洁癖和强迫症的人群。

Python的缺点主要集中在以下几点。

1. 执行效率稍低，因此计算密集型任务可以由C/C++编写。
2. 代码无法加密，但是现在很多公司都不销售卖软件而是销售服务，这个问题会被弱化。
3. 在开发时可以选择的框架太多（如Web框架就有100多个），有选择的地方就有错误。

#### 应用领域

​	目前Python在Web应用开发、云基础设施、DevOps、网络数据采集（爬虫）、数据分析挖掘、机器学习等领域都有着广泛的应用，因此也产生了Web后端开发、数据接口开发、自动化运维、自动化测试、科学计算和可视化、数据分析、量化交易、机器人开发、自然语言处理、图像识别等一系列相关的职位。

#### 解释器

官方的Python解释器是用C语言实现的，也是使用最为广泛的Python解释器，通常称之为`CPython`。除此之外，还有Java语言实现的`Jython`等。

#### 代码注释

1. 单行注释 - 以#和空格开头的部分
2. 多行注释 - 三个引号开头，三个引号结尾

#### 开发工具

1）**IPython - 交互式编程工具**

```shell
# 安装 python3.6
yum install -y python36
# 安装 pip3
yum install -y python36-setuptools
yum install -y python36-pip
## pip 安装
pip install ipython
## pip3 安装
pip3 install ipython
## 软连接
find / -name ipython3
ln -s /usr/local/bin/ipython3  /usr/bin/ipython3
## 启动 IPython
ipython
```

2）**PyCharm - Python开发神器**



### 变量

#### 常见变量类型

- 整形：Python中可以处理任意大小的整数（Python2.x中有int和long两种类型的整数，Python 3.x中整数只有int这一种）

- 浮点型：浮点数即小数，支持科学计数法。
- 字符串型：字符串是以单引号或双引号括起来的任意文本（以及三引号）
- 布尔型：布尔值只有`True`、`False`两种值，在Python中，可以直接用`True`、`False`表示布尔值，也可以通过布尔运算表示，如：3 == 5 布尔值为False。
- 复数型：形如`3+5j`，跟数学上的复数表示一样，唯一不同的是虚部的`i`换成了`j`。（不常用了解即可）

#### 变量命名

**命令规则：**

- 硬性规则：
  - 变量名由字母、数字和下划线构成（不能以数字开头）。
  - 大小写敏感
  - 不要跟关键字（有特殊含义的单词，后面会讲到）和系统保留字（如函数、模块等的名字）冲突。

- PEP 8要求：
  - 用小写字母拼写，多个单词用下划线连接。
  - 受保护的实例属性用单个下划线开头。
  - 私有的实例属性用两个下划线开头。

#### 变量使用

**变量的定义&运算：**

```shell
a = 321
b = 123
print(a + b)
print(a - b)
```

**type()检查变量类型：**

```SHELL
a = 100
b = 12.345
d = 'hello, world'
e = True
print(type(a)) # <class 'int'>
print(type(b)) # <class 'float'>
print(type(d)) # <class 'str'>
print(type(e)) # <class 'bool'>
```

#### 内置函数-变量类型转换

- `int()`：将一个数值或字符串转换成整数，可以指定进制。
- `float()`：将一个字符串转换成浮点数。
- `str()`：将指定的对象转换成字符串形式，可以指定编码。
- `chr()`：将整数转换成该编码对应的字符串（一个字符）。
- `ord()`：将字符串（一个字符）转换成对应的编码（整数）。

实例1.输入两个整数并输出两个整数的算术运算

```shell
a = int(input('a = '))
b = int(input('b = '))
print('%d + %d = %d' % (a, b, a + b))
print('%d - %d = %d' % (a, b, a - b))
```

**扩展说明：**print函数的占位符语法

`'%string' % (value1,value2,...)`

字符串之后的`%`后面跟的变量值会替换掉占位符然后输出到终端中。

- `%d`是整数的占位符
- `%f`是小数的占位符
- `%%`表示百分号

#### 运算符

下表按照优先级从高到低的顺序列出了所有的运算符：

| 运算符                                                       | 描述                           |
| ------------------------------------------------------------ | ------------------------------ |
| `[]` `[:]`                                                   | 下标，切片                     |
| `**`                                                         | 指数                           |
| `~` `+` `-`                                                  | 按位取反, 正负号               |
| `*` `/` `%` `//`                                             | 乘，除，模，整除               |
| `+` `-`                                                      | 加，减                         |
| `>>` `<<`                                                    | 右移，左移                     |
| `&`                                                          | 按位与                         |
| `^` `\|`                                                     | 按位异或，按位或               |
| `<=` `<` `>` `>=`                                            | 小于等于，小于，大于，大于等于 |
| `==` `!=`                                                    | 等于，不等于                   |
| `is`  `is not`                                               | 身份运算符                     |
| `in` `not in`                                                | 成员运算符                     |
| `not` `or` `and`                                             | 逻辑运算符                     |
| `=` `+=` `-=` `*=` `/=` `%=` `//=` `**=` `&=` `|=` `^=` `>>=` `<<=` | （复合）赋值运算符             |

> 注：在实际开发中，如果搞不清楚运算符的优先级，可以使用括号来确保运算的执行顺序。

实例1.赋值运算符和复合赋值运算符

```shell
a = 10
b = 3
a += b # 相当于：a = a + b
a *= a + 2 # 相当于：a = a * (a + 2)
```

实例2.比较运算符（关系运算符）、逻辑运算符和身份运算符

```shell
flag0 = 1 == 1	# True
flag1 = 3 > 2	# True
flag2 = flag0 and flag1	# False
flag3 = flag1 or flag2	# False
flag4 = not (1 != 2)	# False
flag1 is True	# True
flag2 is not False	# False
```

### 分支结构

#### if 语句

关键字：`if`、`elif`和`else`

`if ... else`

```python
if xx:
    xx
else:
    xx
```

`if ... elif ... else`

```python
if xx:
    xx
elif:
    xx
else:
    xx
```

if 嵌套语句

```python
if xx:
    xx
else:
    xx
    if xx:
        xx
    else:
        xx
```

> 注：Flat is better than nested. 嵌套结构的嵌套层次多了之后会严重的影响代码的可读性，能使用扁平化的结构就不要使用嵌套。

### 循环结构

重复的执行某条或某些指令。

Python中构造循环结构有两种做法，一种是`for-in`循环，一种是`while`循环。

#### for-in 循环

如果明确的知道循环执行的次数或者要对一个容器进行迭代，推荐使用`for-in`。

实例1.计算1~100求和的结果

```python
sum = 0
for x in range(101):
    sum += x
print(sum)
```

***扩展补充：***

`range()`函数`range(start, stop[, step])`

- `start`：计数从 start 开始。默认是从 0 开始。
- `stop`: 计数到 stop 结束，但不包括 stop。
- `step`：步长，默认为1。

#### while 循环

如果要构造不知道具体循环次数的循环结构，推荐使用`while`循环。

实例1."猜数字"的小游戏

```python
"""
猜数字游戏
计算机出一个1~100之间的随机数由人来猜
计算机根据人猜的数字分别给出提示大一点/小一点/猜对了
""""
import random

answer = random.randint(1, 100)
counter = 0
while True:
    counter += 1
    number = int(input('请输入: '))
    if number < answer:
        print('大一点')
    elif number > answer:
        print('小一点')
    else:
        print('恭喜你猜对了!')
        break
print('你总共猜了%d次' % counter)
if counter > 7:
    print('你的智商余额明显不足')
```

***扩展补充：***

1. `break`关键字，终止当前所在循环，退回上一个循环或退出循环。
2. `continue`关键字，放弃本轮循环，直接让循环进入下一轮。

实例2.嵌套循环-九九乘法表

```python
for i in range(1, 10):
    for j in range(1, i + 1):
        print('%d*%d=%d' % (i, j, i * j), end='\t')
    print()
```

#### 练习题

##### 输入一个正整数判断是不是素数

> 提示：素数指只能被1和自身整除且大于1的整数

```python
from math import sqrt

num = int(input('请输入一个正整数: '))
end = int(sqrt(num))
is_prime = True
for x in range(2, end + 1):
    if num % x == 0:
        is_prime = False
        break
if is_prime and num != 1:
    print('%d是素数' % num)
else:
    print('%d不是素数' % num)
```

##### 输入两个正整数，计算它们的最大公约数和最小公倍数

```python
x = int(input('x = '))
y = int(input('y = '))
# 如果x大于y就交换x和y的值
if x > y:
    # 通过下面的操作将y的值赋给x, 将x的值赋给y
    x, y = y, x
# 从两个数中较的数开始做递减的循环
for factor in range(x, 0, -1):
    if x % factor == 0 and y % factor == 0:
        print('%d和%d的最大公约数是%d' % (x, y, factor))
        print('%d和%d的最小公倍数是%d' % (x, y, x * y // factor))
        break
```

##### 打印三角形图案

```python
row = int(input('请输入行数: '))
for i in range(row):
    for _ in range(row - i - 1):
        print(' ', end='')
    for _ in range(2 * i + 1):
        print('*', end='')
    print()
```

### 程序逻辑构造

把用人类自然语言描述的算法（解决问题的方法和步骤）翻译成Python代码。

#### 寻找水仙花数

***说明：***水仙花数（也称为超完全数字不变数、自恋数、自幂数、阿姆斯特朗数），它是一个3位数，该数字每个位上数字的立方之和正好等于它本身，如：1^3 + 5^3+ 3^3=153。

```python
"""
找出所有水仙花数

Version: 0.1
Author: Jason
"""

for num in range(100, 1000):
    low = num % 10
    mid = num // 10 % 10
    high = num // 100
    if num == low ** 3 + mid ** 3 + high ** 3:
        print(num)
```

> **知识点：**通过整除和求模运算分别找出了一个三位数的个位、十位和百位。

#### 百钱白鸡问题

***说明：***百钱百鸡是我国古代数学家[张丘建]在《算经》一书中提出的数学问题：鸡翁一值钱五，鸡母一值钱三，鸡雏三值钱一。百钱买百鸡，问鸡翁、鸡母、鸡雏各几何？翻译成现代文是：公鸡5元一只，母鸡3元一只，小鸡1元三只，用100块钱买一百只鸡，问公鸡、母鸡、小鸡各有多少只？

```python
for x in range(0, 20):
    for y in range(0, 33):
        z = 100 - x - y
        if 5 * x + 3 * y + z / 3 == 100:
            print('公鸡: %d只, 母鸡: %d只, 小鸡: %d只' % (x, y, z))
```

> 知识点：这里使用了**穷举法**，也称**暴力搜索法**。通过一项一项的列举备选解决方案中所有可能的候选项并检查每个候选项是否符合问题的描述，最终得到问题的解。该方法看起来比较笨拙，但对于运算能力非常强大的计算机来说，通常都是一个可行的甚至是不错的选择，而且问题的解如果存在，这种方法一定能够找到它。

#### CRAPS赌博游戏

***说明：***CRAPS又称花旗骰，是美国拉斯维加斯非常受欢迎的一种的桌上赌博游戏。该游戏使用两粒骰子，玩家通过摇两粒骰子获得点数进行游戏。简单的规则是：玩家第一次摇骰子如果摇出了7点或11点，玩家胜；玩家第一次如果摇出2点、3点或12点，庄家胜；其他点数玩家继续摇骰子，如果玩家摇出了7点，庄家胜；如果玩家摇出了第一次摇的点数，玩家胜；其他点数，玩家继续要骰子，直到分出胜负。

```python
"""
Craps赌博游戏
我们设定玩家开始游戏时有1000元的赌注
游戏结束的条件是玩家输光所有的赌注

Version: 0.1
Author: 骆昊
"""
from random import randint

money = 1000
while money > 0:
    print('你的总资产为:', money)
    needs_go_on = False
    while True:
        debt = int(input('请下注: '))
        if 0 < debt <= money:
            break
    first = randint(1, 6) + randint(1, 6)
    print('玩家摇出了%d点' % first)
    if first == 7 or first == 11:
        print('玩家胜!')
        money += debt
    elif first == 2 or first == 3 or first == 12:
        print('庄家胜!')
        money -= debt
    else:
        needs_go_on = True
    while needs_go_on:
        needs_go_on = False
        current = randint(1, 6) + randint(1, 6)
        print('玩家摇出了%d点' % current)
        if current == 7:
            print('庄家胜')
            money -= debt
        elif current == first:
            print('玩家胜')
            money += debt
        else:
            needs_go_on = True
print('你破产了, 游戏结束!')
```

#### 练习题

##### 生成**斐波那契数列**的前20个数

***说明：***斐波那契数列（Fibonacci sequence），又称黄金分割数列，是意大利数学家莱昂纳多·斐波那契（Leonardoda Fibonacci）在《计算之书》中提出一个在理想假设条件下兔子成长率的问题而引入的数列，所以这个数列也被戏称为&quot;兔子数列&quot;。斐波那契数列的特点是数列的前两个数都是1，从第三个数开始，每个数都是它前面两个数的和，形如：1, 1, 2, 3, 5, 8, 13, 21, 34, 55, 89, 144, ...。斐波那契数列在现代物理、准晶体结构、化学等领域都有直接的应用。



##### 找出10000以内的完美数

***说明：***完美数又称为完全数或完备数，它的所有的真因子（即除了自身以外的因子）的和（即因子函数）恰好等于它本身。例如：6（6=1+2+3）和28（28=1+2+4+7+14）就是完美数。完美数有很多神奇的特性，有兴趣的可以自行了解。



##### 输出100以内所有的素数

***说明：***素数指的是只能被1和自身整除的正整数（不包括1）。

