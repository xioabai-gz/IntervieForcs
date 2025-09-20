# Python 

## 面向对象编程：



### 面向对象编程三个步骤：

- 首先对系统进行分析，识别系统中有哪些对象，即思考系统中有哪些概念
- 除去识别这些概念以外，还要分析这些概念之间的关系
- 其次对于每个概念还要分析这个概念的所有对象具有哪些共同属性。属性包括描述对象状态的数据属性，和描述对象行为的功能属性





**实例方法**：针对不同类的实例（对象）产生不同计算结果的方法称为实例方法

**类属性**：其不是属于一个具体的类实例（对象）的，通常可以通过类名.类属性来查询或者修改类属性

也可以通过实例名.类属性（包括self.类属性）来**查询**实例属性，但是不能通过此方法修改类属性



del运算符可以用于删除一个变量，该变量的引用计数会减少1，当一个对象的引用计数减少为0时候，系统会自动回收这个对象占用的内存空间

### 访问控制和私有属性

python可以通过给类的属性名前面添加两个下划线来禁止外界访问这些属性，这样这些属性就变为了**私有属性**   私有变量也类似



#### 重载运算符

重写运算符对应的运算符方法：

如____add____():



### 派生类:

从一个基类（BASE）定义一个派生类的格式如下

```python
class Derived（Base）:
	派生类类体
```

例如定义经理类和职员类，经理是特殊的职员 有他自己的属性和方法

```python
class Employee:
    '这是一个描述公司普通雇员的类'
    def __init__(self,Name,Salary):
        self.__name = Name
        self.__salary = Salary
    def printInfo(self):
        print(self.__name,",",self.__salary)
    def get_name(self):
        return self.__name
```



定义经理类继承普通职员类   然后给其（派生类）添加特有的属性（数据属性和方法）

类Manager继承了类Employee的属性（数据和方法）还修改了其init（）的构造方法和printInfo（）方法，还定义了新的get_level()方法

```python
class Manager(Employee):
    '这是一个描述经理的类'
    def __init__(self,name,salary,level,employee):
        Employee.__init__(self,name,salary)  #基类构造函数对基类部分初始化
        self.__level = level
        self.__employes = employee
    def printInfo(self):
        Employee.printInfo(self)
        print("经理级别：",self.__level)
        print("管理的员工有：")
        for e in self.__employes:
            print(e.get_name())
    def get_level(self):
        return self.__level
```

```python
e = Employee("林立",3000)
e2 = Employee("z张伟",20000)
employees = []
employees.append(e)
employees.append(e2)
m = Manager("赵四",70000,2,employees) #调用Manager自身的构造函数
m.printInfo()
print()
print(m.get_name(),"的级别：",m.get_level())
```



#### super（）方法

调用父类的方法

```python
class Manager(Employee):
    '这是一个描述经理的类'
    def __init__(self,name,salary,level,employee):
        super.__init__(self,name,salary)
        self.__level = level
        self.__employes = employee
    def printInfo(self):
        Employee.printInfo(self)
        print("经理级别：",self.__level)
        print("管理的员工有：")
        for e in self.__employes:
            print(e.get_name())
    def get_level(self):
        return self.__level
```

使用super可以避免使用基类名字



#### 覆盖

子类通过**定义和父类同名的属性**可以覆盖父类的数据属性和方法

上面的例子就是重新定义了父类中的 printInfo 方法



#### 多继承

一个类可以继承多个类的特性，即可以定义一个从多个类派生出来的派生类



### 绑定属性

#### 动态绑定：给类和对象任意绑定属性

作为动态类型语言，Python可以随时给一个类及其对象绑定属性，如前面所示创建类对象后可以通过成员访问运算符 给对象添加属性或方法

例如上面的e.name = 'liping'  这就是动态绑定了属性

```python
class Employee:
    pass
def set_name(self,name):
    self.name = name
e = Employee()
e.name = 'Li Ping'

```

动态绑定方法可以通过types的MethodType（）方法

```python
import types
e.set_name = types.MethodType(set_name,e) #给e绑定一个set_name 方法
e.set_name(“Zhang Wei”)    #
print(e.name)
isinstance(e.set_name,types.MethodType)


#输出 Zhang Wei
#True

```







## 输入输出：



### 标准输入输出函数

#### print



#### 美观输入输出函数ppprint

其可以自定义输出的 **宽度、缩进和深度**

如果一个用户定义类的内部定义了

```python
__repr__()
```

则ppprint（)使用的类PrettyPrinter可以支持打印这个自定义类



##### 函数pformat

如果要**格式化数据结构，而不直接将其输出到流中**，则可以使用pformat（）来构建对象的字符串





### 文件读写

基本分为三个步骤

1. 打开（open）一个文件
2. 读/写一个文件
3. 关闭这个文件

#### open函数

可以设置打开方式   以及缓冲区



#### 文件对象的方法

##### **write方法**

读或写完文件记得调用函数close（）关闭它

##### read方法

从文件对象中读取给定大小的文本或者字节

##### readline（）和readlines（）方法

readlines返回一个列表

由于其返回一个列表  可以用对应方法  例如使用enumerate（）得到每一行的索引



##### writelines（）方法

可以一次写多个字符串到一个文件中，而且没有返回值





#### 二进制文件读写

如果想要读取二进制文件 就要在访问模式下如r后面增加一个二进制模式b 例如用‘rb’模式表示以二进制模式打开文件



有时候希望定位到文件的某个位置进行读/写这就需要用到tell（）和seek（）方法







## 错误和异常



### 1 错误



#### **语法错误（SyntaxError）**



#### 运行时错误：异常



#### 逻辑错误



### 2 异常处理



#### 捕获异常的基本形式

try...except 程序块

```python
try:
    
except:
    print('出现了异常')
print('hello')
```

将



#### 捕获特定类型的异常

为了根据不同类型的异常 需要在except关键字后面说明具体的异常类型

```python
try:
    x = int(input('Enter x: '))
    y = int(input('Enter y:'))
    print(x/y)
    
except ZeroDivisionError as e:
    print('x / y出现了除0的错误',e)
except ValueError as e:
    print('类型错误ValueError ！',e)
print('hello world')
    
```

在处理异常的时候应该注意，特殊的异常应该在前面  而更一般的异常放在后面 这是因为如果更一般的异常被先处理的话，后面的异常就不会被处理了



#### 捕获未知的内置异常

由于Python的所有异常类型都是由BaseException派生而来的，因此可以**用最后一个except子句捕获这个异常类**



#### else子句

可以在最后一个except子句后面添加一个else子句，当没有异常发生时候，会自动执行else子句  如果发生了异常则不会执行



#### finally子句

与else不同，不论出现不出现异常，其都会被执行

因此主要任务是做一些清理工作，**如释放外部资源等**无论他们在使用过程中是否会出现异常







## 生成器与迭代器

容器：由于在类里面实现了“\_\_contains\_\_”方法，可以用in  not in判断元素

**对象是可迭代的**：它所属的类定义了一个可以返回一个**迭代器**的\_\_iter\_\_()方法

**一个对象是一个迭代器：**是指它所属的类定义了\_\_next()\_\_方法

迭代器是用于记录可迭代对象的**当前元素位置的一个对象**，可通过next方法指向下一个迭代器

```python
class A(object):
    def__iter__(self):
        return B()
    
class B(object):
    def __next__(self):
        return "hello"
    
iterable = A()    #iterable是一个迭代器
iterator = iter(iterable)  #返回一个迭代器
next(iterator)      #输出‘hello'                           
```



# Python_pandas



## 1  pandas的对象



### 1.1  Series对象

是一个特殊的字典，是一个带索引数据结构的一维数组  

### 1.2  Dataframe对象

使用series加上以及字典可以创建一个新的Dataframe对象





### 1.3  Index对象

可以将它看作一个**不可变数组**，或者有序集合

例如 ind[1] = 0 就是不正确，不能更改来的

Index对象遵循集合的许多习惯用法，包括交并差集操作





## 2. 处理缺失值

一般分为两种，一种是通过一个覆盖全局的**掩码**来表示缺失值，一种是用一个**标签值**来表示缺失值

使用前要综合考量，

1. 掩码需要单独开一个掩码布尔数组来表示，
2. 而标签值也可能会缩小有效值的范围

### 2.1 Pandada处理缺失值：选取标签值

- 第一种None：Python**对象**类型的缺失值
  - 由于是对象来的，所以操作会比原生的要慢
- NaN：数值类型的缺失值
  - 其是一种**特殊的浮点数**
  - 对NaN进行任何操作，最终结果都会是NaN
  - numpy也提供了 一些特殊的函数忽略了缺失值的影响（np.nansum()等）

### 2.2 如何处理缺失值呢？

- 发现缺失值：	

  - isnull()  notnull() 每种方法都返回布尔类型的**掩码数组**

- 剔除缺失值：

  - dropna()  fillna() 
  - 无法从Dateframe剔除单独的数据，要么剔除整行整列，
    - 在这其中可以通过设置how或者thresh参数（非缺失值的最小数量）来满足一个阈值筛选的问题

- 填充缺失值：

  - fillna() 可以填充特定值  也可以用缺失值前面的或者后面的有效数据来填充（method = ‘ffill’，method=‘bfill’）

    

### 2.3 层级索引

在实践过程中，更直观的形式是通过**层级索引**，也被称为**多级索引**，**配合多个有不同等级的一级索引**一起使用，这样可以将高维数组转换一维series或者二维的形式



#### 2.3.1多级索引series





### 2.4 合并数据集



#### 1. 链接方式

- 
- 一对一 
- 多对一
- 多对多

#### 2. 设置数据合并的键

因为pd.merge（）默认会把两个输入的一个或多个共同列作为键，进行合并

- **参数on**（只有俩个dateframe有共同列名字的时候才可以使用）
  - 指定一个或多个列作为键进行合并
- **left_on与right_on**
  - 用来合并两个列名不同的数据集
- **left_index与right_index**
  - 合并索引，设置这两个参数将索引设置为键进行合并

也可以将二者混合使用。



#### 3. 设置数据链接的集合操作规则

当一个值出现在一列而没有出现在另外一列

合并两个数据集时，**指定的列只有一个或几个共同的值的时候**，默认情况下结果包含两个输入集合的交集（**内连接**）

如果想包含两个集合的**并集**，并且缺失值用nan填充可以选择 how=‘’outer‘

还有左连接右链接。返回的结果只包含左列或者右列

#### 4. 有重复列名 的：suffixes

可以为**重复**列名中自动增加后缀，可以指定参数的后缀名



#### 5. 案例











​	













# Python_leetcode



## defaultdict:

**`collections` 模块中的一个类**

核心功能：**当访问一个不存在的键时，会自动创建该键并赋予一个默认值**（由你指定的工厂函数决定，这里是 `list`）

###  	**为什么用 `list` 作为默认值？**

- `list` 是一个工厂函数，调用它会返回一个空列表 `[]`。
- 这样当你访问 `mp[key]` 时，如果 `key` 不存在，`defaultdict` 会自动创建这个键，并将其值设为 `[]`，而不是抛出 `KeyError`

### . **其他常见默认值**

- `int`: `defaultdict(int)` 默认值为 `0`，适合计数。
- `set`: `defaultdict(set)` 默认值为空集合 `set()`，适合去重。



## 三元组之和题目：

解法一：暴力，三层循环。

学到的地方：**排序**后转化为**元组**再进行去重。加入集合的时候就会自动去重。

因为元组是不可变的，才可加入列表中进行哈希。不可直接用list

```python
res = set()  # 用集合去重
        for i in range(length):
            for j in range(length):
                for k in range(length):
                    if i != j and i != k and j != k:
                        if nums[i] + nums[j] + nums[k] == 0:
                            triplet = tuple(sorted([nums[i], nums[j], nums[k]]))  # 排序后转为元组去重
                            res.add(triplet)
        return [list(triplet) for triplet in res]
```

























# LangChain



## 一、大语言模型与langchain



### 开发流程

langchain的开发流程围绕LLM的部署和集成展开，通过**定义任务链，提示词等组件**，实现多阶段的自动化处理和数据流转。核心是将LLM的能力嵌入业务逻辑中。

1. 需求分析和场景设计：
   - 需要分析**哪些**功能和操作可有大语言模型完成，并且**评估任务的输入和输出**特征
2. 任务链的设计和配置：
   - 决定LLm如何按照逻辑顺序执行多个任务，每个任务链由多个任务构成，通常包括
3. 
4. 



