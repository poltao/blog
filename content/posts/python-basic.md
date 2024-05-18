+++
title = "Python基础学习"
description = "Linux 环境下的 Python 开发基础教程。"
date = 2020-09-21
[taxonomies]
tags= ["Linux", "Python3"]
+++

## 一、开发环境

### 1.安装依赖库

```bash
yum -y install zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel gdbm-devel db4-devel libpcap-devel xz-devel libffi-devel
```

### 2.下载安装包

```bash
# wget下载安装包并解压至 /usr/local/python3
wget https://www.python.org/ftp/python/3.7.0/Python-3.7.0.tgz
tar -C /usr/local -xzvf Python-3.7.0.tgz
cd /usr/local/Python-3.7.0
```

### 3.配置与安装

```bash
./configure --prefix=/usr/local/python3
make && make install
```

```bash
# 上面的参数，--prefix 是预期安装目录，--enable-optimizations 是优化选项（LTO，PGO 等）加上这个 flag 编译后，性能有 10% 左右的优化，但是这会明显的增加编译时间。
# 安装成功会提示如下消息:
Collecting setuptools
Collecting pip
Installing collected packages: setuptools, pip
Successfully installed pip-10.0.1 setuptools-39.0.1
```

### 4.创建软链接

```bash
ln -s /usr/local/python3/bin/python3.7 /usr/local/bin/python3
ln -s /usr/local/python3/bin/pip3.7 /usr/local/bin/pip3
```

### 5.正确使用 vim 编写第一个 python 程序

首先在 ~/.vimrc 配置 python 缩进：`autocmd FileType python set tabstop=4 | set expandtab | set autoindent`  
示例程序：

```python
#!/usr/local/bin/python3
# -*- coding:utf-8 -*-
for x in range(20):
    if x % 2 == 0:
        print(x)
```

### 6.可变对象与不可变对象

- 可变对象：list dict set
- 不可变对象：tuple string int float bool
- [可变对象与不可变对象比较](https://zhuanlan.zhihu.com/p/34395671)

## 二、运算符

### 1. 格式化输出——%s 与 f'{exp}'

格式化字符串除了%s，还可以写为 f'{表达式}'

```py
name = 'Tom'
years = 1998
print('name = %s, Year = %d' % (name, years))
print(f'name = {name}, Year = {years}')
```

### 2. 数据类型转换

#### 2.1 input

- [x] 当程序执⾏到 input ，等待⽤户输⼊，输⼊完成之后才继续向下执⾏。
- [x] 在 Python 中， input 接收⽤户输⼊后，⼀般存储到变量，⽅便使⽤。
- [x] 在 Python 中， input 会把接收到的任意⽤户输⼊的数据都当做字符串处理。

#### 2.2 转换数据类型的函数

| 函数                                 | 说明                                                         |
| ------------------------------------ | ------------------------------------------------------------ |
| <font color=yelloerw>int(x [,base ]) | 将 x 转换为⼀个整数</font>                                   |
| <font color=yelloerw>float(x )       | 将 x 转换为⼀个浮点数</font>                                 |
| <font color=yelloerw>str(x )         | 将对象 x 转换为字符串</font>                                 |
| <font color=yelloerw>eval(str )      | ⽤来计算在字符串中的有效 Python 表达式,并返回⼀个对象</font> |
| <font color=yelloerw>tuple(s )       | 将序列 s 转换为⼀个元组</font>                               |
| <font color=yelloerw>list(s )        | 将序列 s 转换为⼀个列表 </font>                              |
| chr(x )                              | 将⼀个整数转换为⼀个 Unicode 字符                            |
| ord(x )                              | 将⼀个字符转换为它的 ASCII 整数值                            |
| hex(x )                              | 将⼀个整数转换为⼀个⼗六进制字符串                           |
| oct(x )                              | 将⼀个整数转换为⼀个⼋进制字符串                             |
| bin(x )                              | 将⼀个整数转换为⼀个⼆进制字符串                             |
| complex(real [,imag ])               | 创建⼀个复数，real 为实部，imag 为虚部                       |
| repr(x )                             | 将对象 x 转换为表达式字符串                                  |

#### 2.3 示例

```py
# 1. float() -- 转换成浮点型
num1 = 1
print(float(num1))
print(type(float(num1)))
# 2. str() -- 转换成字符串类型
num2 = 10
print(type(str(num2)))
# 3. tuple() -- 将⼀个序列转换成元组
list1 = [10, 20, 30]
print(tuple(list1))
print(type(tuple(list1)))
# 4. eval() -- 将字符串中的数据转换成Python表达式原本类型
str1 = '10'
str2 = '[1, 2, 3]'
str3 = '(1000, 2000, 3000)'
print(type(eval(str1)))
print(type(eval(str2)))
print(type(eval(str3)))
```

### 3. 运算符

#### 3.1 分类

- 算数运算符
- 赋值运算符
- 复合赋值运算符
- ⽐较运算符
- 逻辑运算符

#### 3.2 py 风格的运算符

**算数运算符**

| 运算符 | 描述 | 实例                                |
| ------ | ---- | ----------------------------------- |
| //     | 整除 | 9 // 4 输出结果为 2                 |
| \*\*   | 指数 | 2 \** 4 输出结果为 16，即 `2*2*2*2` |

**逻辑运算符**

| 运算符 | 描述   | 实例                        |
| ------ | ------ | --------------------------- |
| and    | 逻辑与 | True and False， 返回 False |
| or     | 逻辑或 | False or True， 返回 True   |
| not    | 逻辑非 | not True 返回 False         |

#### 3.3 拓展

**数字之间的逻辑运算**  
<font color=red>and 运算符，只要有⼀个值为 0，则结果为 0，否则结果为最后⼀个⾮ 0 数字  
or 运算符，只有所有值为 0 结果才为 0，否则结果为第⼀个⾮ 0 数字</font>

```py
a = 0
b = 1
c = 2
# and运算符，只要有⼀个值为0，则结果为0，否则结果为最后⼀个⾮0数字
print(a and b)  # 0
print(b and c)  # 2
print(c and b)  # 1
# or运算符，只有所有值为0结果才为0，否则结果为第⼀个⾮0数字
print(a or b)  # 1
print(a or c)  # 2
print(b or c)  # 1
```

## 三、字符串

### 1. 修改字符串大小写

```py
firstName = 'ada'
lastName = 'LOVER'
print(firstName.title() + ' ' + lastName.title())  # Ada Lover
print(firstName.upper())  # ADA
print(lastName.lower())  # lover
```

### 2. 删除空白

```py
favoriteLan = ' python3 '
print(favoriteLan.lstrip())  # 'python3 '
print(favoriteLan.rstrip())  # ' python3'
print(favoriteLan.strip())  # 'python3'
```

### 3. 字符串长度

```py
favoriteLan = ' python3 '
print(favoriteLan.__len__())  # 9
print(len(favoriteLan))  # 9
```

### 4. if 示例程序

```py
cars = ['audi', 'bmw', 'subaru', 'toyota']

for car in cars:
    if car == 'bmw':
        print(car.upper())
    else:
        print(car.title())
```

### 5. 条件测试

**and, or, in, not in**

```py
cars = ['audi', 'bmw', 'subaru', 'toyota']
# and
if cars[0].lower() == 'audi' and cars[1] == 'bo':
    print(True)  # True
elif cars[1] == 'bmw':
    print(cars[1].title())  # Bmw
# in
print('audi' in cars)  # True
print('bo' in cars)  # False
# not in
print('bo' not in cars)  # True

```

## 四、列表

### 1. 列表

```py
bicycles = ['trek', 'cannondale', 'redline', 'speci']
print(bicycles)  # ['trek', 'cannondale', 'redline', 'speci']
```

### 2. 增加元素

```py
# add
bicycles.append('hongqi')
print(bicycles)  # ['trek', 'cannondale', 'redline', 'speci', 'hongqi']
# insert
bicycles.insert(0,'beijing')
print(bicycles)  # ['beijing', 'trek', 'cannondale', 'redline', 'speci', 'hongqi']
```

### 3. 删除元素

```py
bicycles = ['trek', 'redline', 'speci']
print(bicycles)  # ['trek', 'redline', 'speci']
```

#### 3.1 使用 del 语句删除元素

```py
del bicycles[0]
print(bicycles)  # ['redline', 'speci']
```

#### 3.2 使用方法 pop 删除元素

```py
bics = bicycles.pop(1)
print(bics)  # redline
print(bicycles)  # ['trek', 'speci']
bicycles.pop()
print(bicycles)  # ['trek']
```

#### 3.3 根据值删除元素

```py
bicycles.remove('speci')
print(bicycles)  # ['trek', 'redline']
```

<font color = red> remove 方法只删除第一个指定的值</font>

### 4. 组织列表

#### 4.1 sort 永久性排序

```py
bicycles.sort()
print(bicycles)  # ['redline', 'speci', 'trek']
bicycles.sort(reverse=True)
print(bicycles)  # ['trek', 'speci', 'redline']
```

#### 4.2 sorted 函数临时对列表排序

```py
print(sorted(bicycles))
print(sorted(bicycles, reverse=True))
```

#### 4.3 reverse 方法反转列表

```py
bicycles.reverse()
print(bicycles)  # ['speci', 'redline', 'trek']
```

### 5. 操作列表

#### 5.1 列表解析

<font color = red>列表解析将 for 循环和创建新元素的代码合并为一行，并自动附加新元素</font>

```py
numbers = [value ** 2 for value in range(1, 11)]
print(numbers)  # [1, 4, 9, 16, 25, 36, 49, 64, 81, 100]
```

#### 5.2 列表切片

**指定范围切片：**

```py
numbers = [value ** 2 for value in range(1, 11)]
print(numbers)  # [1, 4, 9, 16, 25, 36, 49, 64, 81, 100]
print(numbers[0:2])  # [1, 4]
print(numbers[:4])  # [1, 4, 9, 16]
print(numbers[5:])  # [36, 49, 64, 81, 100]
print(numbers[-3:])  # [64, 81, 100]
```

**遍历切片：**

```py
numbers = [value ** 2 for value in range(1, 11)]
print(numbers)  # [1, 4, 9, 16, 25, 36, 49, 64, 81, 100]
for val in numbers[:4]:
    print(val, end=' ')  # 1 4 9 16
```

#### 5.3 复制列表

<font color = red>必须使用整个列表切片的方法来确保复制的是列表的副本，而不是使用 '='进行复制</font>

```py
numbers = [value ** 2 for value in range(1, 6)]
myNumbers = numbers[:]
numbers.append(36)
myNumbers.append(49)
print(numbers)  # [1, 4, 9, 16, 25, 36]
print(myNumbers)  # [1, 4, 9, 16, 25, 49]
```

### 6. 元组

<font color = red>Python 将不能修改的值称为不可变的， 而不可变的列表被称为元组。
元组中元素虽然不可以被改变，但可以通过赋值的方法改变元组变量的值
</font>

```py
numbers = tuple(value ** 2 for value in range(1, 6))
print(numbers)  # (1, 4, 9, 16, 25)
numbers = tuple(value ** 2 for value in range(1, 4))
print(numbers)  # (1, 4, 9)
```

## 五、字典

### 1. 示例程序

```py
alien = {'color': 'green', 'points': 5}
# 访问字典键-值对
print(alien['color'], end=' ')
print(alien['points'])
# 添加键-值对
alien['x_position'] = 0
alien['y_position'] = 25
print(alien)  # {'color': 'green', 'points': 5, 'x_position': 0, 'y_position': 25}
# 删除键-值对
del alien['points']
print(alien)  # {'color': 'green', 'x_position': 0, 'y_position': 25}
```

### 2. 遍历字典

#### 2.1 遍历所有键值对

```py
alien = {'x_position': 0, 'y_position': 25}
for key, value in alien.items():
    print(f'Key = {key}, Value = {value}')
# 输出：
# Key = x_position, Value = 0
# Key = y_position, Value = 25
```

#### 2.2 遍历字典所有的键

```py
alien = {'x_position': 0, 'y_position': 25}
names = list(alien.keys())
print(names)  # ['x_position', 'y_position']
```

#### 2.3 按顺序遍历字典中的所有键

```py
alien = {'x_position': 0, 'y_position': 25, 'a': 'test1', 'b': 'test2'}
for key in sorted(alien.keys()):
    print(key, end=' ')
```

#### 2.4 遍历字典所有的值

```py
alien = {'x_position': 0, 'y_position': 25, 'a': 'test1', 'b': 'test2'}
for value in alien.values():
    print(value)
```

### 3. 嵌套

#### 3.1 字典列表

<font color=red>即在列表中嵌套字典</font>

```py
alien = {'x_position': 0, 'y_position': 25}
aliens = []
for index in range(30):
    aliens.append(alien)
for app in aliens[:5]:
    print(app)
```

#### 3.2 字典中存储列表

```py
favoriteLan = {
    'jen': ['python', 'rust'],
    'sarah': ['c'],
    'edward': ['ruby', 'go'],
    'phil': ['shell', 'php'],
}

for name, languages in favoriteLan.items():
    print("\n" + name.title() + "'s favorite languages are: ")
    for language in languages:
        print("\t" + language.title())
```

#### 3.3 字典中存储字典

```py
users = {
    'torvalds': {
        'first': 'linus',
        'last': 'torvalds',
        'location': 'finland'
    },
    'buterin': {
        'first': 'vitalik',
        'last': 'buterin',
        'location': 'russia'
    }
}

for name, user in users.items():
    print(f"\nUsername: {name.title()}")
    print(f"first: {user['first'].title()}, last: {user['last'].title()}, location: {user['location'].title()}")
```

输出：

```bash
Username: Torvalds
first: Linus, last: Torvalds, location: Finland

Username: Buterin
first: Vitalik, last: Buterin, location: Russia
```

## 六、函数

### 1. 函数示例

```py
def greet_user(username):
    """greet example"""
    print(f"Hello, {username.title()}")


greet_user('siya')
print(greet_user.__doc__)
```

### 2. 传递实参

#### 2.1 位置实参

<font color=red>实参和形参完全对应的函数参数关联方式</font>

```py
def describe_pet(animal_type, pet_name):
    print(f"I have a {animal_type}.")
    print(f"My {animal_type}'s name is {pet_name.title()}.")


describe_pet('hamster', 'harry')
```

#### 2.2 关键字实参

```py
describe_pet(animal_type='hamster', pet_name='harry')
describe_pet(pet_name='harry', animal_type='hamster')
```

#### 2.3 默认值

```py
def describe_pet(animal_type, pet_name='harry'):
    print(f"I have a {animal_type}.")
    print(f"My {animal_type}'s name is {pet_name.title()}.")


describe_pet('hamster', 'vitalik')
describe_pet('dog')
```

### 3. 返回值

```py
def get_formatted_name(first_name, last_name):
    full_name = first_name + ' ' + last_name
    return full_name.title()


full_name = get_formatted_name('vitalik', 'buterin')
print(full_name)  # Vitalik Buterin
```

### 4. 传递列表

```py
def aTob(nums1, nums2):
    while nums1:
        nums2.append(nums1.pop())


nums1 = [val * 2 for val in range(1, 10)]
nums2 = []
aTob(nums1[:], nums2)
```

### 5. 传递任意数量的实参

#### 5.1 示例程序

```py
def aTob(*nums):
    print(nums)
aTob(1, 2, 4)  # (1, 2, 4)
```

#### 5.2 结合使用位置实参与任意数量实参

```py
def aTob(nums1, *nums2):
    print(nums1)
    print(nums2)
aTob(1, 2, 4)
# 1
# (2, 4)
```

#### 5.3 使用任意数量的关键字实参

```py
def aTob(first, last, **nums):
    print(f"{first.title()} {last.title()}.")
    print(nums)
aTob('vitalik', 'buterin', age=31, sex='male')
# Vitalik Buterin.
# {'age': 31, 'sex': 'male'}
```

## 七、类

### 1. 类示例

```py
class Dog():
    def __init__(self, name, age):
        self.name = name
        self.age = age

    def sit(self):
        print(f"{self.name.title()} is now sitting.")

    def roll_over(self):
        print(f"{self.name.title()} rolled over.")


dog = Dog('vitalik', 20)
dog.sit()
dog.roll_over()
```

### 2. 类的继承

#### 2.1 继承示例

**基类为 Car , 派生类为 ElectricCar**

```py
class Car:
    def __init__(self, make, model, year):
        self.make = make
        self.model = model
        self.year = year
        self.odometer_reading = 0

    def get_descriptive_name(self):
        long_name = f"{self.year} {self.make} {self.model}"
        return long_name

    def read_odometer(self):
        print(f"This car has {self.odometer_reading} miles on it.")

    def update_odometer(self, mileage):
        if mileage >= self.odometer_reading:
            self.odometer_reading = mileage
        else:
            print("You can't roll back an odometer!")

    def increment_odometer(self, miles):
        self.odometer_reading += miles


class ElectricCar(Car):
    def __init__(self, make, model, year):
        super().__init__(make, model, year)
        self.battery_size = 70

    def describe_battery(self):
        print(f"This car has a {self.battery_size}-kWh battery")


car = ElectricCar('tesla', 'model s', 2016)
longName = car.get_descriptive_name()
print(longName)  # 2016 tesla model s
car.describe_battery()  # This car has a 70-kWh battery
```

#### 2.2 重写父类方法

```py
class Car:
    def fill_gas_tank(self):
        pass
class ElectricCar(Car):
    def fill_gas_tank(self):
        print("This car doesn't need a gas tank!")
car = ElectricCar()
car.fill_gas_tank()
```

### 3. 使用 OrderedDict 类

```py
from collections import OrderedDict
favorite_languages = OrderedDict()
favorite_languages['jen'] = ['python', 'rust']
favorite_languages['sarah'] = ['c']
favorite_languages['edward'] = ['ruby', 'go']
favorite_languages['phil'] = ['shell', 'php']

for name, languages in favorite_languages.items():
    print(f"\n{name.title()}'s languages: ", end=' ')
    for language in languages:
        print(f"{language.title()}", end=' ')
```

## 八、GUI 编程

### 1. GUI 编程基础(Tkinter)

**常用 Python GUI 库如下：**

- Tkinter： Tkinter 模块(Tk 接口)是 Python 的标准 Tk GUI 工具包的接口 .Tk 和 Tkinter 可以在大多数的 Unix 平台下使用,同样可以应用在 Windows 和 Macintosh 系统里。Tk8.0 的后续版本可以实现本地窗口风格,并良好地运行在绝大多数平台中。
- wxPython：wxPython 是一款开源软件，是 Python 语言的一套优秀的 GUI 图形库，允许 Python 程序员很方便的创建完整的、功能健全的 GUI 用户界面。
- Jython：Jython 程序可以和 Java 无缝集成。

#### 1.1 Tkinter 编程

创建一个 GUI 程序:

- 1、导入 Tkinter 模块
- 2、创建控件
- 3、指定这个控件的 master， 即这个控件属于哪一个
- 4、告诉 GM(geometry manager) 有一个控件产生了。

**示例：**

```py
# This is a sample Python script.
# Press Shift+F10 to execute it or replace it with your code.
# Press Double Shift to search everywhere for classes, files, tool windows, actions, and settings.
import tkinter

top = tkinter.Tk()
# 进入消息循环
top.mainloop()
```

执行之后显示：  
![BUxYqA.png](https://s1.ax1x.com/2020/10/31/BUxYqA.png)

#### 1.2 Tkinter 组件

目前有 15 种 Tkinter 的部件。我们提出这些部件以及一个简短的介绍，在下面的表:

|                                  控件                                   |                                描述                                |
| :---------------------------------------------------------------------: | :----------------------------------------------------------------: |
|      [Button](https://www.runoob.com/python/python-tk-button.html)      |                    按钮控件；在程序中显示按钮。                    |
|      [Canvas](https://www.runoob.com/python/python-tk-canvas.html)      |                 画布控件；显示图形元素如线条或文本                 |
| [Checkbutton](https://www.runoob.com/python/python-tk-checkbutton.html) |               多选框控件；用于在程序中提供多项选择框               |
|    [Entry](https://www.runoob.com/python/python-tkinter-entry.html)     |                  输入控件；用于显示简单的文本内容                  |
|       [Frame](https://www.runoob.com/python/python-tk-frame.html)       |         框架控件；在屏幕上显示一个矩形区域，多用来作为容器         |
|       [Label](https://www.runoob.com/python/python-tk-label.html)       |                    标签控件；可以显示文本和位图                    |
|                                 Listbox                                 |  列表框控件；在 Listbox 窗口小部件是用来显示一个字符串列表给用户   |
|                               Menubutton                                |                   菜单按钮控件，用于显示菜单项。                   |
|                                  Menu                                   |              菜单控件；显示菜单栏,下拉菜单和弹出菜单               |
|                                 Message                                 |           消息控件；用来显示多行文本，与 label 比较类似            |
|                               Radiobutton                               |                单选按钮控件；显示一个单选的按钮状态                |
|                                  Scale                                  |        范围控件；显示一个数值刻度，为输出限定范围的数字区间        |
|                                Scrollbar                                |        滚动条控件，当内容超过可视化区域时使用，如列表框。.         |
|                                  Text                                   |                     文本控件；用于显示多行文本                     |
|                                Toplevel                                 |       容器控件；用来提供一个单独的对话框，和 Frame 比较类似        |
|                                 Spinbox                                 |          输入控件；与 Entry 类似，但是可以指定输入范围值           |
|                               PanedWindow                               | PanedWindow 是一个窗口布局管理的插件，可以包含一个或者多个子控件。 |
|                               LabelFrame                                |      labelframe 是一个简单的容器控件。常用与复杂的窗口布局。       |
|                              tkMessageBox                               |                    用于显示你应用程序的消息框。                    |

#### 1.3 标准属性

标准属性也就是所有控件的共同属性，如大小，字体和颜色等等。

|   属性    |    描述    |
| :-------: | :--------: |
| Dimension | 控件大小； |
|   Color   | 控件颜色； |
|   Font    | 控件字体； |
|  Anchor   |   锚点；   |
|  Relief   | 控件样式； |
|  Bitmap   |   位图；   |
|  Cursor   |   光标；   |

#### 1.4 几何管理

Tkinter 控件有特定的几何状态管理方法，管理整个控件区域组织，以下是 Tkinter 公开的几何管理类：包、网格、位置

| 几何方法 |  描述  |
| :------: | :----: |
|  pack()  | 包装； |
|  grid()  | 网格； |
| place()  | 位置； |

### 2. Tkinter 编程实战

#### 2.1 Frame 与 pack 的初体验

```python
from tkinter import *
def say_hi():
    print("hello ~ !")
root = Tk()
frame1 = Frame(root, bd=128, cursor="plus", bg="white")
frame2 = Frame(root, bd=128, cursor="circle", bg="pink")
root.title("tkinter frame")
label = Label(frame1, text="Label", bg="red")
label.pack(fill=X)
hi_there = Button(frame2, text="say hi~", command=say_hi)
hi_there.pack()
frame1.pack(padx=10, pady=10, side=RIGHT)
frame2.pack(padx=10, pady=10, side=RIGHT)
root.mainloop()
```

执行之后显示：  
![BaVaoF.md.png](https://s1.ax1x.com/2020/10/31/BaVaoF.md.png)

#### 2.2 Python Tkinter 文本框（Entry）

- **你如果需要输入多行文本，可以使用 Text 组件。**
- **你如果需要显示一行或多行文本且不允许用户修改，你可以使用 Label 组件。**

```python
import tkinter

top = tkinter.Tk()
top.title("emergency system")

frame = tkinter.Frame(top, bd=256, bg="white")
l1 = tkinter.Label(frame, text="name")
w = tkinter.Entry(frame, bd=10, highlightcolor="red", fg="black")
l1.pack(side=tkinter.LEFT)
w.pack(side=tkinter.LEFT)
frame.pack(padx=10, pady=10)

top.mainloop()
```

### 3. 廖雪峰图形编程入门

加入一个文本框，让用户可以输入文本，然后点按钮后，弹出消息对话框。

```python
from tkinter import *
import tkinter.messagebox as messagebox


class Application(Frame):
    def __init__(self, master=None):
        Frame.__init__(self, master, bd=128, bg="green")
        self.pack()
        self.createWidgets()

    def createWidgets(self):
        self.nameInput = Entry(self, bd=12, bg="white")
        self.nameInput.pack()
        self.quitButton = Button(self, text='Quit', command=self.quit)
        self.helloButton = Button(self, text='hello', command=self.hello)
        self.helloButton.pack(side=LEFT, padx=10, pady=10)
        self.quitButton.pack(side=LEFT, padx=10, pady=10)

    def hello(self):
        name = self.nameInput.get()
        messagebox.showinfo("Message", "Hello %s" % name)


app = Application()
app.master.title("Hello World!")
app.mainloop()
```

![32](https://s1.ax1x.com/2020/10/31/BaYUw4.png)

### 4. 参考资料

[[Tkinter 教程 12] 布局管理 (Pack Place Grid)](https://blog.csdn.net/liuxu0703/article/details/54428405)  
[Python GUI 之 tkinter 窗口视窗教程大集合（看这篇就够了）](https://www.cnblogs.com/shwee/p/9427975.html)

## 九、元类

[廖雪峰使用元类](https://www.liaoxuefeng.com/wiki/1016959663602400/1017592449371072)

[谈谈 Python 中元类 Metaclass(一)：什么是元类](https://www.cnblogs.com/ArsenalfanInECNU/p/9036407.html)

[谈谈 Python 中元类 Metaclass(二)：ORM 实践](https://www.cnblogs.com/ArsenalfanInECNU/p/9100874.html)

[深入理解 Python 中的元类(metaclass)](https://www.cnblogs.com/JetpropelledSnake/p/9094103.html)
