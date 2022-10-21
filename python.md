## 标识符

由字母、数字、下划线组成

**区分大小写**,不能以数字开头

**以下划线开头的标识符是有特殊意义的**

* 单下划线开头  不能直接访问的类属性，需通过类提供的接口进行访问，不能用 **from xxx import \*** 而导入

* 双下划线开头  类的私有成员
* 双下划线开头和结尾  Python 里特殊方法专用的标识，如 **__init__()** 代表类的构造函数。



## 多行语句

多行语句写在同一行时,要用 ; 隔开

单行语句需要拆成多行时, 行尾加 \ ,若语句中包含 [], {} 或 () 括号就不需要使用\



单行注释	#

多行注释	‘‘‘ 或 """







# 数据类型



## 数字

**不可变**，改变数字数据类型会分配一个新的对象



支持四种不同的数字类型：

- int（有符号整型）
- long（长整型，也可以代表八进制和十六进制）
- float（浮点型）
- complex（复数）



### math/cmath

数学运算常用的函数基本都在 math 模块、cmath 模块中

```python
import math
```



### 随机数



| 函数                                                         | 描述                                                         |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| [choice(seq)](https://www.runoob.com/python/func-number-choice.html) | 从序列的元素中随机挑选一个元素，比如random.choice(range(10))，从0到9中随机挑选一个整数。 |
| [randrange ([start,\] stop [,step])](https://www.runoob.com/python/func-number-randrange.html) | 从指定范围内，按指定基数递增的集合中获取一个随机数，基数默认值为 1 |
| [random()](https://www.runoob.com/python/func-number-random.html) | 随机生成下一个实数，它在[0,1)范围内。                        |
| [seed([x\])](https://www.runoob.com/python/func-number-seed.html) | 改变随机数生成器的种子seed。如果你不了解其原理，你不必特别去设定seed，Python会帮你选择seed。 |
| [shuffle(lst)](https://www.runoob.com/python/func-number-shuffle.html) | 将序列的所有元素随机排序                                     |
| [uniform(x, y)](https://www.runoob.com/python/func-number-uniform.html) | 随机生成下一个实数，它在[x,y]范围内。                        |













## 字符串

不支持单字符类型char,单字符也作为字符串存储



2种取值顺序:

- 从左到右索引默认0开始的，最大范围是字符串长度少1
- 从右到左索引默认-1开始的，最大范围是字符串开头



### 截取

**[头下标:尾下标]**	[)左闭右开, 头尾可以只填一个

```python
s = 'abcdef'
s[1:5]	#'bcde'
```



### 格式化

Python2.6 开始，新增了一种格式化字符串的函数 **str.format()**，它增强了字符串格式化的功能。

基本语法是通过 **{}** 和 **:** 来代替 %



```python
"{} {}".format("hello", "world")    # 不设置指定位置，按默认顺序
'hello world'
 
"{0} {1}".format("hello", "world")  # 设置指定位置
'hello world'

'{0.value}'.format(value)		#传入对象, 访问对象的属性
```



### 数字格式化

下表展示了 str.format() 格式化数字的多种方法：

```
>>> print("{:.2f}".format(3.1415926))
3.14
```

| 数字       | 格式    | 输出      | 描述                         |
| :--------- | :------ | :-------- | :--------------------------- |
| 3.1415926  | {:.2f}  | 3.14      | 保留小数点后两位             |
| 3.1415926  | {:+.2f} | +3.14     | 带符号保留小数点后两位       |
| -1         | {:-.2f} | -1.00     | 带符号保留小数点后两位       |
| 2.71828    | {:.0f}  | 3         | 不带小数                     |
| 5          | {:0>2d} | 05        | 数字补零 (填充左边, 宽度为2) |
| 5          | {:x<4d} | 5xxx      | 数字补x (填充右边, 宽度为4)  |
| 10         | {:x<4d} | 10xx      | 数字补x (填充右边, 宽度为4)  |
| 1000000    | {:,}    | 1,000,000 | 以逗号分隔的数字格式         |
| 0.25       | {:.2%}  | 25.00%    | 百分比格式                   |
| 1000000000 | {:.2e}  | 1.00e+09  | 指数记法                     |
| 13         | {:>10d} | 13        | 右对齐 (默认, 宽度为10)      |
| 13         | {:<10d} | 13        | 左对齐 (宽度为10)            |
| 13         | {:^10d} | 13        | 中间对齐 (宽度为10)          |

**^**, **<**, **>** 分别是居中、左对齐、右对齐，后面带宽度， **:** 号后面带填充的字符，只能是一个字符，不指定则默认是用空格填充。

**+** 表示在正数前显示 **+**，负数前显示 **-**； （空格）表示在正数前加空格

b、d、o、x 分别是二进制、十进制、八进制、十六进制。

此外我们可以使用大括号 **{}** 来转义大括号









## list



支持 + 操作符, 相当于java的addAll

支持 * 操作符, 重复列表中的元素



| [list.append(obj)](https://www.runoob.com/python/att-list-append.html) 在列表末尾添加新的对象 |
| :----------------------------------------------------------- |
| [list.count(obj)](https://www.runoob.com/python/att-list-count.html) 统计某个元素在列表中出现的次数 |
| [list.extend(seq)](https://www.runoob.com/python/att-list-extend.html) 在列表末尾一次性追加另一个序列中的多个值（用新列表扩展原来的列表） |
| [list.index(obj)](https://www.runoob.com/python/att-list-index.html) 从列表中找出某个值第一个匹配项的索引位置 |
| [list.insert(index, obj)](https://www.runoob.com/python/att-list-insert.html) 将对象插入列表 |
| [list.pop([index=-1\])](https://www.runoob.com/python/att-list-pop.html) 移除列表中的一个元素（默认最后一个元素），并且返回该元素的值 |
| [list.remove(obj)](https://www.runoob.com/python/att-list-remove.html) 移除列表中某个值的第一个匹配项 |
| [list.reverse()](https://www.runoob.com/python/att-list-reverse.html) 反向列表中元素 |
| [list.sort(cmp=None, key=None, reverse=False)](https://www.runoob.com/python/att-list-sort.html) 对原列表进行排序 |



## 截取

与字符串一致



## tuple 元组

只读的list





### 无关闭分隔符

==任意无符号对象，以逗号隔开，默认为元组==



```python
print 'abc', -4.24e93, 18+6.6j, 'xyz' 
# abc -4.24e+93 (18+6.6j) xyz

x, y = 1, 2 
print "Value of x , y : ", x,y
# Value of x , y : 1 2
```



## dict 字典

list是有序的元素集合, dict无序 -> dict通过key取值



**key唯一,不可变**, 在重复定义key时,会进行**替换**

**key可以取任意数据类型(除了list)**,支持不同的类型混用

```python
d = { 'abc': 123, 98.6: 37 }
```



| del dict['xxx'] 删除字典元素                                 |
| :----------------------------------------------------------- |
| [dict.clear()](https://www.runoob.com/python/att-dictionary-clear.html) 删除字典内所有元素 |
| [dict.copy()](https://www.runoob.com/python/att-dictionary-copy.html) 返回字典的浅复制 |
| dict.fromkeys(seq, val) 创建一个新字典，以序列 seq 中元素做字典的键，val 为字典所有键对应的初始值 |
| [dict.get(key, default=None)](https://www.runoob.com/python/att-dictionary-get.html) 返回指定键的值，如果值不在字典中返回default值 |
| [dict.has_key(key)](https://www.runoob.com/python/att-dictionary-has_key.html) 如果键在字典dict里返回true，否则返回false |
| [dict.items()](https://www.runoob.com/python/att-dictionary-items.html) 以列表返回可遍历的(键, 值) 元组数组 |
| [dict.keys()](https://www.runoob.com/python/att-dictionary-keys.html) 以列表返回一个字典所有的键 |
| [dict.setdefault(key, default=None)](https://www.runoob.com/python/att-dictionary-setdefault.html) 和get()类似, 但如果键不存在于字典中，将会添加键并将值设为default |
| [dict.update(dict2)](https://www.runoob.com/python/att-dictionary-update.html) 把字典dict2的键/值对更新到dict里 |
| [dict.values()](https://www.runoob.com/python/att-dictionary-values.html) 以列表返回字典中的所有值 |
| [pop(key[,default\])](https://www.runoob.com/python/python-att-dictionary-pop.html) 删除字典给定键 key 所对应的值，返回值为被删除的值。key值必须给出。 否则，返回default值。 |
| [popitem()](https://www.runoob.com/python/python-att-dictionary-popitem.html) 返回并删除字典中的最后一对键和值。 |







# IO





## open



```python
f = open(file_name [, mode][, buffering])
```



mode: 文件的操作权限

| 模式 | 描述                                                         |
| :--- | :----------------------------------------------------------- |
| t    | 文本模式 (默认)                                              |
| x    | 写模式，新建一个文件，如果该文件已存在则会报错。             |
| b    | 返回二进制的文本内容                                         |
| +    | 打开一个文件进行更新(可读可写)                               |
| r    | 以只读方式打开文件。文件的指针将会放在文件的开头。这是默认模式。 |
| w    | 打开一个文件只用于写入。如果该文件已存在则打开文件，并从开头开始编辑，即原有内容会被删除。如果该文件不存在，创建新文件。 |
| a    | 打开一个文件用于追加。如果该文件已存在，文件指针将会放在文件的结尾。也就是说，新的内容将会被写入到已有内容之后。如果该文件不存在，创建新文件进行写入。 |



|    模式    |  r   |  r+  |  w   |  w+  |  a   |  a+  |
| :--------: | :--: | :--: | :--: | :--: | :--: | :--: |
|     读     |  +   |  +   |      |  +   |      |  +   |
|     写     |      |  +   |  +   |  +   |  +   |  +   |
|    创建    |      |      |  +   |  +   |  +   |  +   |
|    覆盖    |      |      |  +   |  +   |      |      |
| 指针在开始 |  +   |  +   |  +   |  +   |      |      |
| 指针在结尾 |      |      |      |      |  +   |  +   |

















# 面向对象



````python
class Employee:
  empCount = 0

  def __init__(self, name, salary):	#self: 类的实例
    self.name = name
    self.salary = salary
    Employee.empCount += 1
````

类方法必须有额外的**第一个参数,代表类的实例**,通常叫self,可以用别的变量名



### 实例化

python没有new关键词,使得类的实例化类似函数调用方式

```python
e = Employee("Zara", 2000)
```





## 内置类属性

- \_\_dict__ : 类的属性（包含一个字典，由类的数据属性组成）
- \_\_doc__ : 类的文档字符串
- \_\_name__: 类名
- \_\_module__: 类定义所在的模块
- \_\_bases__ : 类的所有父类构成元素（包含了一个由所有父类组成的元组）



# GC

Python用了引用计数实现GC

在 Python 内部记录着所有使用中的对象各有多少引用。

通过一个内部跟踪变量，记录对象被引用的次数

回收不是"立即"的，由解释器在适当的时机，将垃圾对象占用的内存空间回收









































