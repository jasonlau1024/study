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



