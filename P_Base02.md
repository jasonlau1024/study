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
    list01.remote(3)	==> [1, 400, 5, 7, 100, 200, 1000, 2000]
# 指定索引位置删除元素
list1.pop(0)	==> [400, 5, 7, 100, 200, 1000, 2000]

 ```

