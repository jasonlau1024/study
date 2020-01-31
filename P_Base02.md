## 基础进阶

### 函数与模块

#### 函数的作用

编程大师*Martin Fowler*先生曾经说过：“**代码有很多种坏味道，重复是最坏的一种！**”，要写出高质量的代码首先要解决的就是重复代码的问题。

将重复的代码封装到一个称之为“函数”的功能模块中，在需要使用时，“调用”这个“函数”即可。

#### 函数的定义

Python中使用`def`关键字来定义函数，函数名命名规则和变量名一样。

```python
def func_box(参数):	# 相当于数学上说的函数的自变量
    ...
    return xx	# 相当于数学上说的函数的因变量
```

#### 函数的参数

***说明：***函数是绝大多数编程语言中都支持的一个代码的&quot;构建块&quot;，但是Python中的函数与其他语言中的函数还是有很多不太相同的地方，其中一个显著的区别就是Python对函数参数的处理。在Python中，函数的参数可以有默认值，也支持使用可变参数，所以Python并不需要像其他语言一样支持[函数的重载](https://zh.wikipedia.org/wiki/%E5%87%BD%E6%95%B0%E9%87%8D%E8%BD%BD)，因为我们在定义一个函数的时候可以让它有多种不同的使用方式，如下示例：

示例①.

```python
from random import randint


def roll_dice(n=2):
    """摇色子"""
    total = 0
    for _ in range(n):
        total += randint(1, 6)
    return total


def add(a=0, b=0, c=0):
    """三个数相加"""
    return a + b + c


# 如果没有指定参数那么使用默认值摇两颗色子
print(roll_dice())
# 摇三颗色子
print(roll_dice(3))
print(add())
print(add(1))
print(add(1, 2))
print(add(1, 2, 3))
# 传递参数时可以不按照设定的顺序进行传递
print(add(c=50, a=100, b=200))
```

> **知识点：**函数参数的默认值，在调用函数的时候如果没有传入对应参数的值时将使用该参数的默认值。
>
> 可变参数，如：add(*args)，在不确定参数个数的时候，我们可以使用可变参数。

#### 用模块管理函数

***说明：***在同一个.py文件中定义了两个同名函数，由于Python没有函数重载的概念，那么后面的定义会覆盖之前的定义。这在多人协作进行团队开发的时，可能出现命名冲突。Python中每个文件就代表了一个模块（module），我们在不同的模块中可以有同名的函数，在使用函数的时候我们通过`import`关键字导入指定的模块就可以区分到底要使用的是哪个模块中的函数。

```python
# module01.py
def foo():
    print('hello,world!')
# module02.py
def foo():
    print('goodbye,world!')
# test.py
import module01 as m1
import module02 as m2

m1.foo()	# hello,world!
m2.foo()	# goodbye,world!
```

**补充说明：**如果我们导入的模块除了定义函数之外还中有可以执行代码，那么Python解释器在导入这个模块时就会执行这些代码，事实上我们可能并不希望如此，因此如果我们在模块中编写了执行代码，最好是将这些执行代码放入如下所示的条件中，这样的话除非直接运行该模块，if条件下的这些代码是不会执行的，因为只有直接执行的模块的名字才是&quot;\_\_main\_\_&quot;。

```python
# module03.py
def foo():
    pass

def bar():
    pass

if __name__ == '__main__':
    print('call foo()')
    foo()
    print('call bar()')
    bar()
#### 说明 ####
__name__是Python中一个隐含的变量它代表了模块的名字。
只有被Python解释器直接执行的模块的名字才是__main__
# test.py
import module03
#### 说明 ####
导入module03时 不会执行模块中if条件成立时的代码 因为模块的名字是module3而不是__main__
```

#### 变量的作用域

Python查找一个变量时会按照“局部作用域”、“嵌套作用域”、“全局作用域”和“内置作用域”的顺序进行搜索。

- 局部作用域：局部变量（local variable），定义在函数中。
- 嵌套作用域：函数内部的局部变量，相对函数里面嵌套的函数来说。
- 全局作用域：全局变量。
- 内置作用域：Python内置的标识符，如：`input`、`print`、`int`等

**补充说明：**在实际开发中，我们应该尽量减少对全局变量的使用，因为全局变量的作用域和影响过于广泛，可能会发生意料之外的修改和使用，除此之外全局变量比局部变量拥有更长的生命周期，可能导致对象占用的内存长时间无法被[垃圾回收](https://zh.wikipedia.org/wiki/%E5%9E%83%E5%9C%BE%E5%9B%9E%E6%94%B6_(%E8%A8%88%E7%AE%97%E6%A9%9F%E7%A7%91%E5%AD%B8))。事实上，减少对全局变量的使用，也是降低代码之间耦合度的一个重要举措，同时也是对[迪米特法则](https://zh.wikipedia.org/zh-hans/%E5%BE%97%E5%A2%A8%E5%BF%92%E8%80%B3%E5%AE%9A%E5%BE%8B)的践行。减少全局变量的使用就意味着我们应该尽量让变量的作用域在函数的内部，但是如果我们希望将一个局部变量的生命周期延长，使其在定义它的函数调用结束后依然可以使用它的值，这时候就需要使用[闭包]

#### 练习题

##### 实现计算求最大公约数和最小公倍数的函数

```python
def gcd(x, y):
    """求最大公约数"""
    (x, y) = (y, x) if x > y else (x, y)
    for factor in range(x, 0, -1):
        if x % factor == 0 and y % factor == 0:
            return factor

def lcm(x, y):
    """求最小公倍数"""
    return x * y // gcd(x, y)
```

##### 实现判断一个数是不是素数的函数

```python
def is_prime(num):
    """判断一个数是不是素数"""
    for factor in range(2, num):
        if num % factor == 0:
            return False
    return True if num != 1 else False
```

### 字符串及常见数据结构

#### 字符串

由零个或多个字符组成的有限序列。在Python程序中，如果我们把单个或多个字符用单引号或者双引号包围起来，就可以表示一个字符串。

**转义：**在字符串中使用`\`（反斜杠）来表示转义

- `\n`表示换行
- `\t`表示制表符

**字符串运算符：**

- 字符串拼接：`+`运算符来实现。
- 字符串重复：`*`运算符来实现。
- 成员运算：`in`和`not in`来判断一个字符串是否包含另外一个字符串。
- 切片运算：用`[]`和`[:]`运算符从字符串取出某个字符或某些字符。

**内置字符串处理函数：**

- 字符串长度获取
  - `len()`计算字符串长度：`print(len(str1))`
- 字符串大小写处理
  - `capitalize()`字符串首字母大写：`print(str1.capitalize())`
  - `title()`字符串每个单词首字母大写：`print(str1.title())`
  - `upper()`字符串全部转为大写：`print(str1.upper())`

- 字符串匹配
  - `find()`在字符串找到子串的位子：`print(str1.find('or')) # 8`
  - `index()`类似`find()`，但找到子串时会抛出异常。
  - `startswith()`匹配字符串开头：`print(str1.startswith('He'))`
  - `endswith('!')`匹配字符串结尾：`print(str1.endswith('!'))`
  - `isdigit()`判断字符串由数字构成：`print(str2.isdigit())`
  - `isalpha()`判断字符串由字母构成：`print(str2.isalpha())`
  - `isalnum()`判断字符串有数字+字母构成：`print(str2.isalnum())`

**格式化输出字符串：**

```python
a, b = 5, 10
# 方式一：
print('%d * %d = %d' % (a, b, a * b))
# 方式二：
print('{0} * {1} = {2}'.format(a, b, a * b))
# 方式三：(Python 3.6以后，提供更为简洁的书写方式，在字符串前加上字母f)
print(f'{a} * {b} = {a * b}')
```

#### 列表(List)

列表是值的有序序列，每个值都可以通过索引进行标识。

定义列表可以将列表的元素放在`[]`中，多个元素用`,`进行分隔，可以使用`for`循环对列表元素进行遍历，也可以使用`[]`或`[:]`运算符取出列表中的一个或多个元素。

 ```python
####  列表的定义，遍历  ####
# 定义列表
list01 = [1,3,5,7,9]  ==> [1, 3, 5, 7, 100]
# 列表元素重复
list02 = ['hello'] * 3  ==> ['hello', 'hello', 'hello']
# 计算列表长度（元素个数）
print(len(list01))	==> 5
# 下标（索引）运算
print(list01[4])	==> 9
print(list01[-1])	==> 9
print(list01[5])  # IndexError: list index out of range
## 循环遍历列表元素
# 方式一，下标遍历列表元素
for index in range(len(list01)):
    print(list01[index])
# 方式二，直接遍历列表元素
for elem in list01:
    print(elem)
## enumerate函数，同时获得元素索引和值
for index, elem in enumerate(list01):
    print(index, elem)
####  列表元素的添加及移除  ####
# 添加元素
list01.append(100)	==> [1,3,5,7,9,100]
list01.insert(1,200)	==> [1,200,3,5,7,9,100]
# 合并列表
list1 += [1000, 2000]	==> [1, 400, 3, 5, 7, 100, 200, 1000, 2000]
# 移除成员
if 3 in list01:
    list01.remove(3)	==> [1, 400, 5, 7, 100, 200, 1000, 2000]
# 指定索引位置删除元素
list1.pop(0)	==> [400, 5, 7, 100, 200, 1000, 2000]

 ```

***知识点：***

- `len()`：返回对象（字符、列表、元组等）长度

  `len(str)`

- `enumerate()`：将一个可遍历的数据对象(如列表、元组或字符串)组合为一个索引序列，同时列出数据和数据下标，一般用在 for 循环当中。返回 enumerate(枚举) 对象。

  `enumerate(sequence, [start=0])`

  - `sequence`：一个序列、迭代器或其他支持迭代对象。
  - `start`：下标起始位置。

- `append()`：在列表末尾添加新的对象。直接修改原来的列表，无返回值。

  `list.append(obj)`

  - `obj`：添加到列表末尾的对象。

- `insert()`：将指定对象插入列表的指定位置。

  `list.insert(index, obj)`

  - `index`：指定插入索引位置。
  - `obj`：要插入列表中的对象。

- `remove()`：移除列表中某个值的第一个匹配项。

  `list.remove(obj)`

- `pop()`：移除列表中的一个元素（默认最后一个元素），并且返回该元素的值。

  `list.pop([index=-1])`

- `clear()`：清空列表，类似于 `del a[:]`。无返回值。

  `list.clear()`

***列表切片：***

和字符串一样，列表也可以做切片操作，通过切片操作我们可以实现对列表的复制或者将列表中的一部分取出来创建出新的列表。

```python
# 列表拼接
fruits = ['grape', 'apple', 'strawberry', 'waxberry']
fruits += ['pitaya', 'pear', 'mango']
# 列表切片
fruits01 = fruits[1:4]	# apple strawberry waxberry
# 反向切片，倒转列表
fruits02 = fruits[::-1]	# ['mango', 'pear', 'pitaya', 'waxberry', 'strawberry', 'apple', 'grape']
```

***列表排序：***

```python
list01 = ['orange', 'apple', 'zoo', 'internationalization', 'blueberry']
list02 = sorted(list01)
list03 = sorted(list01,key=len,reverse=True)
list04 = sort(list01)
```

***知识点：***

- `sorted()`：对所有可迭代的对象进行排序操作。返回重新排序的列表。

  `sorted(iterable, key=None, reverse=False)`

  - `iterable`：可迭代对象
  - `key`：用来进行比较的元素，指定可迭代对象中的一个元素来进行排序，如：字符串长度。
  - `reverse`：指定排序规则。`reverse = True` 降序 ，默认升序 `reverse = False` 

- **sort 与 sorted 区别：**
  1. sort 是应用在 list 上的方法，sorted 可以对所有可迭代的对象进行排序操作。
  2. sort 直接修改列表，内建函数 sorted 方法返回的是一个新的 list 。

#### 生成式和生成器

***生成器：***生成器是一个返回迭代器的函数，只能用于迭代操作，更简单点理解生成器就是一个迭代器。调用一个生成器函数，返回的是一个迭代器对象。

```python
# 使用列表的生成式语法来创建列表
f = [x for x in range(1, 10)]	# [1, 2, 3, 4, 5, 6, 7, 8, 9]
f = [x + y for x in 'ABC' for y in '12345']
# ['A1', 'A2', 'A3', 'A4', 'A5', 'B1', 'B2', 'B3', 'B4', 'B5', 'C1', 'C2', 'C3', 'C4', 'C5']
# 用列表的生成表达式语法创建列表容器
# 用这种语法创建列表之后元素已经准备就绪所以需要耗费较多的内存空间
f = [x ** 2 for x in range(1, 1000)]
print(sys.getsizeof(f))  # 查看对象占用内存的字节数，9024
print(f)	# 直接打印所有成员
# 请注意下面的代码创建的不是一个列表而是一个生成器对象
# 通过生成器可以获取到数据但它不占用额外的空间存储数据
# 每次需要数据的时候就通过内部的运算得到数据(需要花费额外的时间)
f = (x ** 2 for x in range(1, 1000))
print(sys.getsizeof(f))  # 相比生成式生成器不占用存储数据的空间，88
print(f)	# 错误信息：generator object
for val in f:
    print(val)	# 通过for循环打印所有成员
```

***补充：***除了上面提到的生成器语法，Python中还有另外一种定义生成器的方式，就是通过`yield`关键字将一个普通函数改造成生成器函数。

```python
# 实现一个生成[斐波拉切数列]的生成器
def fib(n):
    a, b = 0, 1
    for _ in range(n):
        a, b = b, a + b
        yield a
def main():
    for val in fid(20):
        print(val)
if __name__ == '__main__':
    main()
```

#### 元组

Python中的元组与列表类似也是一种容器数据类型，可以用一个变量（对象）来存储多个数据，不同之处在于元组的元素不能修改。把多个元素组合到一起就形成了一个元组，它和列表一样可以保存多条数据。

```python
# 定义元组
t = ('骆昊', 38, True, '四川成都')	# ('骆昊', 38, True, '四川成都')
# 获取元组中的元素
print(t[0])	# 骆昊
# 遍历元组中的值
for member in t:
    print(member)
# 重新给元组赋值
t[0] = '王大锤'	# TypeError: 'tuple' object does not support item assignment
# 将元组转换成列表
person = list(t)
print(person)	# ['骆昊', 38, True, '四川成都']
person[0] = '李小龙'	# 列表可以修改它的元素
person[1] = 25
print(person)	# ['李小龙', 25, True, '四川成都']
# 将列表转换成元组
person_tuple = tuple(person)
print(person_tuple)	# ('李小龙', 25, True, '四川成都')
```

***知识点：***

1. 元组转列表：`list()`函数，如，`list01 = list(tuple01)`
2. 列表转元组：`tuple()`函数，如，`tuple02 = tuple(list01)`

***问题探讨：***既然有了列表这种数据结构，为什么还需要元组？

1. 元组中的元素是无法修改的，不可变对象尤其适用于多线程环境。

   不可变对象的优势：

   ①.避免对象状态修改引起的不必要的程序错误，更加容易维护。

   ②.所有线程不能修改不变对象的内部状态，实现线程安全。

   ③.不变对象可以方便的被共享访问。

   总结：如果不需要对元素进行添加、删除、修改的时候，首选元组。

2. 元组在创建时间和占用的空间上面都优于列表。

   ```python
   print(sys.getsizeof([1,2,3,4,5]))	# 104
   print(sys.getsizeof((1,2,3,4,5)))	# 88
   %timeit [1,2,3,4,5]	# 66.2 ns 
   %timeit (1,2,3,4,5)	# 16.8 ns
   ```

   ***知识点：***

   `sys.getsizeof()` ：查看对象的内存占用（字节）

   `%timeit`：ipython下使用，可以测量一行代码多次执行的时间，

   `%time`：可以测量一行代码执行的时间

#### 集合

Python中的集合跟数学上的集合是一致的，不允许有重复元素，而且可以进行交集、并集、差集等运算。

```python
# 创建集合的字面量语法
set1 = {1, 2, 3, 3, 3, 2}
print(set1)	# {1, 2, 3}
# 创建集合的构造器语法
set2 = set(range(1, 10))	# {1, 2, 3, 4, 5, 6, 7, 8, 9}
set3 = set((1, 2, 3, 3, 2, 1))	# {1, 2, 3}
# 创建集合的推导式语法
set4 = {num for num in range(1, 10) if num % 3 == 0 or num % 5 == 0}	# {9, 3, 5, 6}
# 向集合添加元素和从集合删除元素
set1.add(4)	# {1, 2, 3, 4}
set1.update([5, 6])	# {1, 2, 3, 4, 5, 6}
set1.discard(5)	# {1, 2, 3, 4, 6}
if 4 in set1:
    set1.remove(4)	# {1, 2, 3, 6}
# 集合的交集、并集、差集、对称差运算
print(set1, set2)	# {1, 2, 3, 6} {1, 2, 3, 4, 5, 6, 7, 8, 9}
print(set1 & set2)	# {1, 2, 3, 6}  <==> print(set1.intersection(set2))
print(set1 | set2)	# {1, 2, 3, 4, 5, 6, 7, 8, 9}  <==> print(set1.union(set2)) 
print(set1 - set2)	# set()	<==> print(set1.difference(set2))
print(set1 ^ set2)	# {4, 5, 7, 8, 9} <==> print(set1.symmetric_difference(set2))
# 判断子集和超集
print(set1 >= set2)	# False <==> print(set1.issuperset(set2))
print(set1 <= set2)	# True 	<==> print(set1.issubset(set2))
```

#### 字典

字典是另一种可变容器模型，Python中的字典跟我们生活中使用的字典是一样一样的，它可以存储任意类型对象，与列表、集合不同的是，字典的每个元素都是由一个键和一个值组成的“键值对”，键和值通过冒号分开。

```python
# 创建字典的字面量语法
scores = {'骆昊': 95, '白元芳': 78, '狄仁杰': 82}	# {'骆昊': 95, '白元芳': 78, '狄仁杰': 82}
# 创建字典的构造器语法
items1 = dict(one=1, two=2, three=3, four=4)	# {'one': 1, 'two': 2, 'three': 3, 'four': 4}
# 通过zip函数将两个序列压成字典
items2 = dict(zip(['a', 'b', 'c'], '123'))	# {'a': '1', 'b': '2', 'c': '3'}
# 创建字典的推导式语法
items3 = {num: num ** 2 for num in range(1, 5)}	# {1: 1, 2: 4, 3: 9, 4: 16}
# 通过键可以获取字典中对应的值
print(scores['骆昊'])	# 95
if '狄仁杰' in scores:
    print(scores['狄仁杰'])
print(scores.get('狄仁杰'))
# get方法也是通过键获取对应的值但是可以设置默认值
print(scores.get('武则天', 60))
# 对字典中所有键值对进行遍历
for key in scores:
    print(f'{key}: {scores[key]}')
# 更新字典中的元素
scores['白元芳'] = 65
scores['诸葛王朗'] = 71
scores.update(冷面=67, 方启鹤=85)
# 删除字典中的元素
print(scores.popitem())
print(scores.pop('骆昊', 100))
# 清空字典
scores.clear()
```

#### 练习题

##### 在屏幕上显示跑马灯文字

```python
import os
import time

def main():
    content = '北京欢迎你为你开天辟地…………'
    while True:
        # 清理屏幕上的输出
        os.system('clear')
        print(content)
        # 休眠200毫秒
        time.sleep(0.2)
        content = content[1:] + content[0]

if __name__ == '__main__':
    main()
```

##### 设计一个函数产生指定长度的验证码，验证码由大小写字母和数字构成

```python
import random

def generate_code(code_len=4):
    """
    生成指定长度的验证码

    :param code_len: 验证码的长度(默认4个字符)

    :return: 由大小写英文字母和数字构成的随机验证码
    """
    all_chars = '0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ'
    last_pos = len(all_chars) - 1
    code = ''
    for _ in range(code_len):
        index = random.randint(0, last_pos)
        code += all_chars[index]
    return code
```

##### 设计一个函数返回给定文件名的后缀名

```python
def get_suffix(filename, has_dot=False):
    """
    获取文件名的后缀名

    :param filename: 文件名
    :param has_dot: 返回的后缀名是否需要带点
    :return: 文件的后缀名
    """
    pos = filename.rfind('.')
    if 0 < pos < len(filename) - 1:
        index = pos if has_dot else pos + 1
        return filename[index:]
    else:
        return ''
```

##### 设计一个函数返回传入的列表中最大和第二大的元素的值

```python
def max2(x):
    m1, m2 = (x[0], x[1]) if x[0] > x[1] else (x[1], x[0])
    for index in range(2, len(x)):
        if x[index] > m1:
            m2 = m1
            m1 = x[index]
        elif x[index] > m2:
            m2 = x[index]
    return m1, m2
```

##### 双色球选号

```python
from random import randrange, randint, sample
"""
使用random模块的sample函数来实现从列表中选择不重复的n个元素。
"""
def display(balls):
    """
    输出列表中的双色球号码。
    """
    for index, ball in enumerate(balls):
        if index == len(balls) - 1:
            print('|', end=' ')
        print('%02d' % ball, end=' ')
    print()

def random_select():
    """
    随机选择一组号码
    """
    red_balls = [x for x in range(1, 34)]
    selected_balls = []
    selected_balls = sample(red_balls, 6)
    selected_balls.sort()
    selected_balls.append(randint(1, 16))
    return selected_balls

def main():
    n = int(input('机选几注: '))
    for _ in range(n):
        display(random_select())

if __name__ == '__main__':
    main()
"""
输出结果：
机选几注: 2
06 07 09 25 26 32 | 09 
06 11 18 25 30 32 | 08 
"""
```



### 面向对象编程

#### 基本概念

- 面向对象

  把一组数据结构和处理它们的方法组成对象`object`，把相同行为的对象归纳为类`class`，通过类的封装`encapsulation`隐藏内部细节，通过继承`inheritance`实现类的特化`specialization`和泛化`generalization`，通过多态`polymorphism`实现基于对象类型的动态分派。

  - 封装：
  - 继承：
  - 多态：

- 类和对象

  简单的说，类是对象的蓝图和模板，而对象是类的实例。类是抽象的概念，而对象是具体的东西。在面向对象编程的世界中，一切皆为对象，对象都有属性和行为，每个对象都是独一无二的，而且对象一定属于某个类（型）。当我们把一大堆拥有共同特征的对象的静态特征（属性）和动态特征（行为）都抽取出来后，就可以定义出一个叫做“类”的东西。

#### 定义类

在Python中可以使用`class`关键字定义类，然后在类中通过之前学习过的函数来定义方法，这样就可以将对象的动态特征描述出来。

```python
class Student(object):

    # __init__是一个特殊方法用于在创建对象时进行初始化操作
    # 通过这个方法我们可以为学生对象绑定name和age两个属性
    def __init__(self, name, age):
        self.name = name
        self.age = age

    def study(self, course_name):
        print('%s正在学习%s.' % (self.name, course_name))

    # PEP 8要求标识符的名字用全小写多个单词用下划线连接
    # 但是部分程序员和公司更倾向于使用驼峰命名法(驼峰标识)
    def watch_movie(self):
        if self.age < 18:
            print('%s只能观看《熊出没》.' % self.name)
        else:
            print('%s正在观看岛国爱情大电影.' % self.name)
```

说明：写在类中的函数，我们通常称之为（对象的）方法，这些方法就是对象可以接收的消息。

#### 创建和使用对象

当我们定义好一个类之后，可以通过下面的方式来创建对象并给对象发消息。

```python
def main():
    # 创建学生对象并指定姓名和年龄
    stu1 = Student('骆昊', 38)
    # 给对象发study消息
    stu1.study('Python程序设计')
    # 给对象发watch_movie消息
    stu1.watch_movie()
    stu2 = Student('王大锤', 15)
    stu2.study('思想品德')
    stu2.watch_movie()

if __name__ == '__main__':
    main()
```

#### 访问可见性问题

在很多面向对象编程语言中，我们通常会将对象的属性设置为私有的（private）或受保护的（protected），简单的说就是不允许外界访问，而对象的方法通常都是公开的（public），因为公开的方法就是对象能够接受的消息。在Python中，属性和方法的访问权限只有两种，也就是公开的和私有的，如果希望属性是私有的，在给属性命名时可以用两个下划线作为开头。

```python
class Test:
    def __init__(self, foo):
        self.__foo = foo

    def __bar(self):
        print(self.__foo)
        print('__bar')

def main():
    test = Test('hello')
    # AttributeError: 'Test' object has no attribute '__bar'
    test.__bar()
    # AttributeError: 'Test' object has no attribute '__foo'
    print(test.__foo)

if __name__ == "__main__":
    main()
"""
Python并没有从语法上严格保证私有属性或方法的私密性，它只是给私有的属性和方法换了一个名字来妨碍对它们的访问，事实上如果你知道更换名字的规则仍然可以访问到它们，下面的代码就可以验证这一点。之所以这样设定，可以用这样一句名言加以解释，就是&quot;**We are all consenting adults here**&quot;。因为绝大多数程序员都认为开放比封闭要好，而且程序员要自己为自己的行为负责。
"""
def main():
    test = Test('hello')
    test._Test__bar()
    print(test._Test__foo)
```

注：在实际开发中，我们并不建议将属性设置为私有的，因为这会导致子类无法访问。所以大多数Python程序员会遵循一种命名惯例就是让属性名以单下划线开头来表示属性是受保护的，本类之外的代码在访问这样的属性时应该要保持慎重。这种做法并不是语法上的规则，单下划线开头的属性和方法外界仍然是可以访问的，所以更多的时候它是一种暗示或隐喻。

#### @property装饰器

通过单下划线开头来暗示属性是受保护的，不建议外界直接访问，那么如果想访问属性可以通过属性的getter（访问器）和setter（修改器）方法进行对应的操作。我们可以考虑使用@property包装器来包装getter和setter方法，使得对属性的访问既安全又方便。

```python
class Person(object):

    def __init__(self, name, age):
        self._name = name
        self._age = age

    # 访问器 - getter方法
    @property
    def name(self):
        return self._name

    # 访问器 - getter方法
    @property
    def age(self):
        return self._age

    # 修改器 - setter方法
    @age.setter
    def age(self, age):
        self._age = age

    def play(self):
        if self._age <= 16:
            print('%s正在玩飞行棋.' % self._name)
        else:
            print('%s正在玩斗地主.' % self._name)

def main():
    person = Person('王大锤', 12)
    person.play()
    person.age = 22
    person.play()
    # person.name = '白元芳'  # AttributeError: can't set attribute

if __name__ == '__main__':
    main()
```

#### \_\_slots\_\_魔法

Python是一门[动态语言](https://zh.wikipedia.org/wiki/%E5%8A%A8%E6%80%81%E8%AF%AD%E8%A8%80)，通常，动态语言允许我们在程序运行时给对象绑定新的属性或方法，当然也可以对已经绑定的属性和方法进行解绑定。但是如果我们需要限定自定义类型的对象只能绑定某些属性，可以通过在类中定义\_\_slots\_\_变量来进行限定。需要注意的是\_\_slots\_\_的限定只对当前类的对象生效，对子类并不起任何作用。

```python
class Person(object):

    # 限定Person对象只能绑定_name, _age和_gender属性
    __slots__ = ('_name', '_age', '_gender')

    def __init__(self, name, age):
        self._name = name
        self._age = age

    @property
    def name(self):
        return self._name

    @property
    def age(self):
        return self._age

    @age.setter
    def age(self, age):
        self._age = age

    def play(self):
        if self._age <= 16:
            print('%s正在玩飞行棋.' % self._name)
        else:
            print('%s正在玩斗地主.' % self._name)

def main():
    person = Person('王大锤', 22)
    person.play()
    person._gender = '男'
    # AttributeError: 'Person' object has no attribute '_is_gay'
    # person._is_gay = True
```

#### 静态方法和类方法

我们在类中定义的方法都是对象方法，也就是说这些方法都是发送给对象的消息。实际上，我们写在类中的方法并不需要都是对象方法，例如我们定义一个“三角形”类，通过传入三条边长来构造三角形，并提供计算周长和面积的方法，但是传入的三条边长未必能构造出三角形对象，因此我们可以先写一个方法来验证三条边长是否可以构成三角形，这个方法很显然就不是对象方法，因为在调用这个方法时三角形对象尚未创建出来，所以这个方法是属于三角形类而并不属于三角形对象的。我们可以使用静态方法来解决这类问题。

```python
from math import sqrt

class Triangle(object):

    def __init__(self, a, b, c):
        self._a = a
        self._b = b
        self._c = c

    @staticmethod
    def is_valid(a, b, c):
        return a + b > c and b + c > a and a + c > b

    def perimeter(self):
        return self._a + self._b + self._c

    def area(self):
        half = self.perimeter() / 2
        return sqrt(half * (half - self._a) *
                    (half - self._b) * (half - self._c))

def main():
    a, b, c = 3, 4, 5
    # 静态方法和类方法都是通过给类发消息来调用的
    if Triangle.is_valid(a, b, c):
        t = Triangle(a, b, c)
        print(t.perimeter())
        # 也可以通过给类发消息来调用对象方法但是要传入接收消息的对象作为参数
        # print(Triangle.perimeter(t))
        print(t.area())
        # print(Triangle.area(t))
    else:
        print('无法构成三角形.')

if __name__ == '__main__':
    main()
```

补充：和静态方法比较类似，Python还可以在类中定义类方法，类方法的第一个参数约定名为cls，它代表的是当前类相关的信息的对象（类本身也是一个对象，有的地方也称之为类的元数据对象），通过这个参数我们可以获取和类相关的信息并且可以创建出类的对象。

```python
from time import time, localtime, sleep

class Clock(object):
    """数字时钟"""
    def __init__(self, hour=0, minute=0, second=0):
        self._hour = hour
        self._minute = minute
        self._second = second

    @classmethod
    def now(cls):
        ctime = localtime(time())
        return cls(ctime.tm_hour, ctime.tm_min, ctime.tm_sec)

    def run(self):
        """走字"""
        self._second += 1
        if self._second == 60:
            self._second = 0
            self._minute += 1
            if self._minute == 60:
                self._minute = 0
                self._hour += 1
                if self._hour == 24:
                    self._hour = 0

    def show(self):
        """显示时间"""
        return '%02d:%02d:%02d' % \
               (self._hour, self._minute, self._second)

def main():
    # 通过类方法创建对象并获取系统时间
    clock = Clock.now()
    while True:
        print(clock.show())
        sleep(1)
        clock.run()

if __name__ == '__main__':
    main()
```

#### 类之间的关系

简单的说，类和类之间的关系有三种：is-a、has-a和use-a关系，即继承、关联和依赖。

- is-a关系也叫继承或泛化，比如学生和人的关系、手机和电子产品的关系都属于继承关系。
- has-a关系通常称之为关联，比如部门和员工的关系，汽车和引擎的关系都属于关联关系；关联关系如果是整体和部分的关联，那么我们称之为聚合关系；如果整体进一步负责了部分的生命周期（整体和部分是不可分割的，同时同在也同时消亡），那么这种就是最强的关联关系，我们称之为合成关系。
- use-a关系通常称之为依赖，比如司机有一个驾驶的行为（方法），其中（的参数）使用到了汽车，那么司机和汽车的关系就是依赖关系。

利用类之间的这些关系，我们可以在已有类的基础上来完成某些操作，也可以在已有类的基础上创建新的类，这些都是实现代码复用的重要手段。复用现有的代码不仅可以减少开发的工作量，也有利于代码的管理和维护，这是我们在日常工作中都会使用到的技术手段。

#### 面向对象的三大支柱

面向对象有三大支柱：封装、继承和多态。

##### 封装

隐藏一切可以隐藏的实现细节，只向外界暴露/提供简单的编程接口。我们只需要知道方法的名字和传入的参数，而不需要知道方法内部的实现细节。

示例：定义一个类描述数字时钟

```python
from time import sleep


class Clock(object):
    """数字时钟"""
    def __init__(self, hour=0, minute=0, second=0):
        """初始化方法
        :param hour: 时
        :param minute: 分
        :param second: 秒
        """
        self._hour = hour
        self._minute = minute
        self._second = second
    def run(self):
        """走字"""
        self._second += 1
        if self._second == 60:
            self._second = 0
            self._minute += 1
            if self._minute == 60:
                self._minute = 0
                self._hour += 1
                if self._hour == 24:
                    self._hour = 0
    def show(self):
        """显示时间"""
        return '%02d:%02d:%02d' % \
               (self._hour, self._minute, self._second)

def main():
    clock = Clock(23, 59, 58)
    while True:
        print(clock.show())
        sleep(1)
        clock.run()

if __name__ == '__main__':
    main()
```

##### 继承

在已有类的基础上创建新类，这其中的一种做法就是让一个类从另一个类那里将属性和方法直接继承下来，从而减少重复代码的编写。提供继承信息的我们称之为父类，也叫超类或基类；得到继承信息的我们称之为子类，也叫派生类或衍生类。子类除了继承父类提供的属性和方法，还可以定义自己特有的属性和方法，所以子类比父类拥有的更多的能力，在实际开发中，我们经常会用子类对象去替换掉一个父类对象，这是面向对象编程中一个常见的行为，对应的原则称之为[里氏替换原则](https://zh.wikipedia.org/wiki/%E9%87%8C%E6%B0%8F%E6%9B%BF%E6%8D%A2%E5%8E%9F%E5%88%99)。

```python
class Person(object):
    """人"""

    def __init__(self, name, age):
        self._name = name
        self._age = age

    @property
    def name(self):
        return self._name

    @property
    def age(self):
        return self._age

    @age.setter
    def age(self, age):
        self._age = age

    def play(self):
        print('%s正在愉快的玩耍.' % self._name)

    def watch_av(self):
        if self._age >= 18:
            print('%s正在观看爱情动作片.' % self._name)
        else:
            print('%s只能观看《熊出没》.' % self._name)

class Student(Person):
    """学生"""

    def __init__(self, name, age, grade):
        super().__init__(name, age)
        self._grade = grade

    @property
    def grade(self):
        return self._grade

    @grade.setter
    def grade(self, grade):
        self._grade = grade

    def study(self, course):
        print('%s的%s正在学习%s.' % (self._grade, self._name, course))

class Teacher(Person):
    """老师"""

    def __init__(self, name, age, title):
        super().__init__(name, age)
        self._title = title

    @property
    def title(self):
        return self._title

    @title.setter
    def title(self, title):
        self._title = title

    def teach(self, course):
        print('%s%s正在讲%s.' % (self._name, self._title, course))

def main():
    stu = Student('王大锤', 15, '初三')
    stu.study('数学')
    stu.watch_av()
    t = Teacher('骆昊', 38, '砖家')
    t.teach('Python程序设计')
    t.watch_av()

if __name__ == '__main__':
    main()
```

##### 多态

子类在继承了父类的方法后，可以对父类已有的方法给出新的实现版本，这个动作称之为方法重写（override）。通过方法重写我们可以让父类的同一个行为在子类中拥有不同的实现版本，当我们调用这个经过子类重写的方法时，不同的子类对象会表现出不同的行为，这个就是多态（poly-morphism）。

```python
from abc import ABCMeta, abstractmethod

class Pet(object, metaclass=ABCMeta):
    """宠物"""

    def __init__(self, nickname):
        self._nickname = nickname

    @abstractmethod
    def make_voice(self):
        """发出声音"""
        pass

class Dog(Pet):
    """狗"""

    def make_voice(self):
        print('%s: 汪汪汪...' % self._nickname)

class Cat(Pet):
    """猫"""

    def make_voice(self):
        print('%s: 喵...喵...' % self._nickname)

def main():
    pets = [Dog('旺财'), Cat('凯蒂'), Dog('大黄')]
    for pet in pets:
        pet.make_voice()

if __name__ == '__main__':
    main()
```

说明：在上面的代码中，我们将`Pet`类处理成了一个抽象类，所谓抽象类就是不能够创建对象的类，这种类的存在就是专门为了让其他类去继承它。Python从语法层面并没有像Java或C#那样提供对抽象类的支持，但是我们可以通过`abc`模块的`ABCMeta`元类和`abstractmethod`包装器来达到抽象类的效果，如果一个类中存在抽象方法那么这个类就不能够实例化（创建对象）。上面的代码中，`Dog`和`Cat`两个子类分别对`Pet`类中的`make_voice`抽象方法进行了重写并给出了不同的实现版本，当我们在`main`函数中调用该方法时，这个方法就表现出了多态行为（同样的方法做了不同的事情）。

#### 综合案例

##### 扑克游戏

```python
import random

class Card(object):
    """一张牌"""

    def __init__(self, suite, face):
        self._suite = suite
        self._face = face

    @property
    def face(self):
        return self._face

    @property
    def suite(self):
        return self._suite

    def __str__(self):
        if self._face == 1:
            face_str = 'A'
        elif self._face == 11:
            face_str = 'J'
        elif self._face == 12:
            face_str = 'Q'
        elif self._face == 13:
            face_str = 'K'
        else:
            face_str = str(self._face)
        return '%s%s' % (self._suite, face_str)
    
    def __repr__(self):
        return self.__str__()

class Poker(object):
    """一副牌"""

    def __init__(self):
        self._cards = [Card(suite, face) 
                       for suite in '♠♥♣♦'
                       for face in range(1, 14)]
        self._current = 0

    @property
    def cards(self):
        return self._cards

    def shuffle(self):
        """洗牌(随机乱序)"""
        self._current = 0
        random.shuffle(self._cards)

    @property
    def next(self):
        """发牌"""
        card = self._cards[self._current]
        self._current += 1
        return card

    @property
    def has_next(self):
        """还有没有牌"""
        return self._current < len(self._cards)

class Player(object):
    """玩家"""

    def __init__(self, name):
        self._name = name
        self._cards_on_hand = []

    @property
    def name(self):
        return self._name

    @property
    def cards_on_hand(self):
        return self._cards_on_hand

    def get(self, card):
        """摸牌"""
        self._cards_on_hand.append(card)

    def arrange(self, card_key):
        """玩家整理手上的牌"""
        self._cards_on_hand.sort(key=card_key)

# 排序规则-先根据花色再根据点数排序
def get_key(card):
    return (card.suite, card.face)

def main():
    p = Poker()
    p.shuffle()
    players = [Player('东邪'), Player('西毒'), Player('南帝'), Player('北丐')]
    for _ in range(13):
        for player in players:
            player.get(p.next)
    for player in players:
        print(player.name + ':', end=' ')
        player.arrange(get_key)
        print(player.cards_on_hand)

if __name__ == '__main__':
    main()
```

##### 工资结算系统

```python
"""
某公司有三种类型的员工 分别是部门经理、程序员和销售员
需要设计一个工资结算系统 根据提供的员工信息来计算月薪
部门经理的月薪是每月固定15000元
程序员的月薪按本月工作时间计算 每小时150元
销售员的月薪是1200元的底薪加上销售额5%的提成
"""
from abc import ABCMeta, abstractmethod
"""
通过`abc`模块的`ABCMeta`元类和`abstractmethod`包装器来达到抽象类的效果
"""

class Employee(object, metaclass=ABCMeta):
    """员工"""

    def __init__(self, name):
        """
        初始化方法
        :param name: 姓名
        """
        self._name = name

    @property
    def name(self):
        return self._name

    @abstractmethod
    def get_salary(self):
        """
        获得月薪
        :return: 月薪
        """
        pass

class Manager(Employee):
    """部门经理"""
    def get_salary(self):
        return 15000.0

class Programmer(Employee):
    """程序员"""
    def __init__(self, name, working_hour=0):
        super().__init__(name)
        self._working_hour = working_hour

    @property
    def working_hour(self):
        return self._working_hour

    @working_hour.setter
    def working_hour(self, working_hour):
        self._working_hour = working_hour if working_hour > 0 else 0

    def get_salary(self):
        return 150.0 * self._working_hour

class Salesman(Employee):
    """销售员"""
    def __init__(self, name, sales=0):
        super().__init__(name)
        self._sales = sales

    @property
    def sales(self):
        return self._sales

    @sales.setter
    def sales(self, sales):
        self._sales = sales if sales > 0 else 0

    def get_salary(self):
        return 1200.0 + self._sales * 0.05

def main():
    emps = [
        Manager('刘备'), Programmer('诸葛亮'),
        Manager('曹操'), Salesman('荀彧'),
        Salesman('吕布'), Programmer('张辽'),
        Programmer('赵云')
    ]
    for emp in emps:
        if isinstance(emp, Programmer):
            emp.working_hour = int(input('请输入%s本月工作时间: ' % emp.name))
        elif isinstance(emp, Salesman):
            emp.sales = float(input('请输入%s本月销售额: ' % emp.name))
        # 同样是接收get_salary这个消息但是不同的员工表现出了不同的行为(多态)
        print('%s本月工资为: ￥%s元' %
              (emp.name, emp.get_salary()))

if __name__ == '__main__':
    main()
```



