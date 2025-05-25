# python基础语法

## 错误与异常

异常有的不会被程序捕捉

异常类型做为错误信息的一部分显示出来：示例中的异常分别为 零除错误（ ZeroDivisionError ） ，命名错误（ NameError） 和 类型错误（ TypeError ）。打印错误信息时，异常的类型作为异常的内置名显示。

### 用户自定义异常

在程序中可以创建新的异常类型来命名自己的异常，异常类应该直接或间接从Exception类派生

```python
class MyError(Exception):
    def __init__(self,value):
        self.value = value
       	
```

如果一个新创建的模块中**需要抛出几种不同的错误时**，一个**通常的作法是为该模块定义一个异常基类**，然后**针对不同的错误类型派生出对应的异常子类:**

#### step1:定义异常基类和子类

```python
# image_processing.py

class ImageProcessingError(Exception):
    """图像处理模块的基类异常"""
    pass

class ImageNotFoundError(ImageProcessingError):
    """图像文件未找到"""
    pass

class UnsupportedFormatError(ImageProcessingError):
    """图像格式不受支持"""
    pass

class CorruptedImageError(ImageProcessingError):
    """图像内容为空或损坏"""
    pass

class InvalidImageSizeError(ImageProcessingError):
    """图像尺寸不符合要求"""
    pass

```

#### step2:模块中抛出异常（使用raise语句）

```python
import cv2
import os

def load_image(path):
    if not os.path.exists(path):
        raise ImageNotFoundError(f"找不到图像文件：{path}")
    	#raise语句抛出异常
    image = cv2.imread(path)
    if image is None:
        raise CorruptedImageError(f"图像可能已损坏或格式不受支持：{path}")
    
    h, w = image.shape[:2]
    if h < 100 or w < 100:
        raise InvalidImageSizeError(f"图像尺寸太小：{w}x{h}")

    return image

```

#### Step 3：调用方使用异常处理

```python
from image_processing import load_image, ImageProcessingError, InvalidImageSizeError

try:
    img = load_image("test.jpg")
except InvalidImageSizeError as e:
    print("图像太小，不能用于处理。")
except ImageProcessingError as e:
    print(f"图像处理错误：{e}")

```

#### 好处总结：

- 所有错误统一继承自 `ImageProcessingError`，便于调用方捕获和统一处理；
- 调用者可以**根据具体错误子类做更细致处理**；
- 日后**新增错误类型（比如：图像过曝）只需添加新的子类即可**，不影响现有结构



#### else语句：

try … except 语句可以带有一个 *else子句*，该子句只能出现在所有 except 子句之后。当 try 语句没有抛出异常时，需要执行一些代码，可以使用这个子句。例如:

### finally子句

 finally放在最后执行，在任何情况下不论是否有异常抛出都会被执行。

通常用于释放外部资源（文件或者网络链接之类）





## 类



### 作用域

如果一个命名声明为**全局的**，那么对它的**所有引用和赋值会直接搜索包含这个模块全局命名的作用域**。如果要**重新绑定最里层作用域之外的变量，可以使用 nonlocal 语句**；如果不声明为 nonlocal，这些变量将是只读的（对这样的变量赋值会在最里面的作用域创建一个新的局部变量，外部具有相同命名的那个变量不会改变）

重要的是作用域决定于源程序的意义：一个定义于某模块中的函数的全局作用域是该模块的命名空间，而不是该函数的别名被定义或调用的位置，了解这一点非常重要。

```python
def scope_test():
    def do_local():
        spam = "local spam"  # 局部变量，修改的是 do_local 作用域中的 spam

    def do_nonlocal():
        nonlocal spam         # 使用外层作用域（即 scope_test 中的 spam）
        spam = "nonlocal spam"

    def do_global():
        global spam           # 使用全局变量 spam
        spam = "global spam"

    spam = "test spam"       # scope_test 局部变量 spam
   
    do_local()
    print("After local assignment:", spam)#只在函数内有作用，输出还是"test spam" 
    do_nonlocal()
    print("After nonlocal assignment:", spam)#会修改外层作用域的spam
    do_global()
    print("After global assignment:", spam)#会创建或者修改全局变量spam但是局部不会受到影响

scope_test()
print("In global scope:", spam)#这里是全局的

```



### 初识类

#### 继承

两种

| 目标                  | 是否需要调用父类构造函数          |
| --------------------- | --------------------------------- |
| 子类不定义 `__init__` | 自动调用父类的                    |
| 子类定义了 `__init__` | 需要手动调用 `super().__init__()` |









类继承机制允许多重继承，派生类可以覆盖（override）基类中的任何方法或类，可以使用相同的方法名称调用基类的方法。对象可以包含任意数量的私有数据。



## 迭代器和生成器



### 迭代器

在后台， for 语句在容器对象中调用 iter() 。该函数返回一个定义了 __next__() 方法的迭代器对象，它在容器中逐一访问元素。没有后续的元素时， __next__() 抛出一个 StopIteration 异常通知 for 语句循环结束



### 生成器

生成器是创建迭代器的简单强大工具

写起来就像是正规的函数，需要返回数据的时候使用 yield 语句。每次 next() 被调用时，生成器回复它脱离的位置（它记忆语句最后一次执行的位置和所有的数据值）。

生成器因为**自动创建**了 __iter__() 和 __next__() 方法，生成器显得如此简洁。



### 常见标准库

操作系统接口：OS

日常的文件和目录管理系统：shutil

文件通配符：glob模块 函数用于从**目录通配符搜索中**生成文件列表

```python
>>> import glob
>>> glob.glob('*.py')
['primes.py', 'random.py', 'quote.py']
```

命令行参数：sys

这些命令行参数以链表形式存储于 sys 模块的 *argv* 变量例如在命令行中执行 `python demo.py one two three` 后可以得到以下输出结果:

```python
>>> import sys
>>> print(sys.argv)
['demo.py', 'one', 'two', 'three']
```

数据压缩：

datetime





















# SQL语法

### 插入检索出来的数据

```sql
INSERT INTO mytable(col1,col2)
SELECT col1，col2
FROM mytable2;
```

将一个表的内容插入到一个新表

```sql
CREATE TABLE newtable AS
SELECT * FROM mytable
```

### 更新

```sql
UPDATE MYTABLE
SET col = val
WHERE id = 1；
```

### 删除

```sql
DELETE FROM mytable
WHERE id = 1;
```

使用更新和删除操作时一定要用 WHERE 子句，不然会把整张表的数据都破坏。可以先用 SELECT 语句进行测试，防止错误删除。

### 查询

DISTINCT(相同的值只会出现一次)

```sql
SELECT DISTINCT col1,col2
FROM mytable;
```

LIMIT  限制返回的行数，可以有两个参数，第一个为起始行第二个为返回总行数

```sql
##返回三到五行
SELECT *
FROM mytable
LIMIT 2, 3;
```

### 排序

asc升序

desc降序

```sql
SELECT *
FROM mytable
ORDER BY col1 DESC, col2 ASC;
```

### 过滤

不进行过滤的数据非常大，导致通过网络传输了多余的数据，从而浪费了网络带宽。因此尽量使用 SQL 语句来过滤不必要的数据，而不是传输所有的数据到客户端中然后由客户端进行过滤。

```sql
SELECT *
FROM mytable
WHERE col IS NULL;
```

应该注意到，NULL 与 0、空字符串都不同。

**AND 和 OR** 用于连接多个过滤条件。优先处理 AND，当一个过滤表达式涉及到多个 AND 和 OR 时，可以使用 () 来决定优先级，使得优先级关系更清晰。

**IN** 操作符用于匹配一组值，其后也可以接一个 SELECT 子句，从而匹配子查询得到的一组值。

**NOT** 操作符用于否定一个条件



### 通配符

通配符可以用在过滤语句中，但他**只能用于文本字段**

- **%** 匹配 >=0 个任意字符；
- **_** 匹配 ==1 个任意字符；
- **[ ]** 可以匹配集合内的字符，例如 [ab] 将匹配字符 a 或者 b。用脱字符 ^ 可以对其进行否定，也就是不匹配集合内的字符。

```sql
SELECT *
FROM mytable
WHERE col LIKE '[^AB]%';  --不以A和B开头的任意文本
```

不要滥用通配符  通配符位于开头匹配会非常慢

### 计算字段

在**数据库服务器上完成数据的转换和格式化的工作往往比客户端上快得多**，并且转换和格式化后的数据量更少的话可以减少网络通信量。

计算字段通常需要使用 **AS** 来取别名，否则输出的时候字段名为计算表达式

```sql
SELECT col1 * col2 AS alais  ##挑选两个乘数
FROM mytable；
```

**CONCAT()** 用于连接两个字段。许多数据库会使用空格把一个值填充为列宽，因此连接的结果会出现一些不必要的空格，使用 **TRIM()** 可以去除首尾空格。

```sql
SELECT CONCAT(TRIM(col1), '(', TRIM(col2), ')') AS concat_col
FROM mytable;
```



### 函数

以mysql为例

#### 汇总

|  函 数  |      说 明       |
| :-----: | :--------------: |
|  AVG()  | 返回某列的平均值 |
| COUNT() |  返回某列的行数  |
|  MAX()  | 返回某列的最大值 |
|  MIN()  | 返回某列的最小值 |
|  SUM()  |  返回某列值之和  |

文本处理

日期和时间处理

数值处理



### 分组

把具有相同的数据值的行放在同一组中

可以对**同一分组数据使用汇总函数**进行处理

指定的分组字段除了能按该字段进行分组 也会自动按该字段进行排序

```sql
SELECT col，COUNT（*） AS num
FROM mytable
GROUP BY col;   ##查询col列的内容，对每个分组统计行数，最后进行分组
```

GROUP BY 自动按分组字段进行排序，ORDER BY 也可以按汇总字段来进行排序

```sql
SELECT  col ,COUNT(*) AS num
FROM mytable
GROUP BY col
ORDER BY num;
```

WHERE 过滤行，Having过滤分组

```sql
SELECT col,COUNT(*) AS num
FROM mytable
WHERE col > 2
GROUP BY col
HAVING num >= 2;
```



# MySQL

## 一、事务

### 概念：

事务指的是满足 ACID 特性的一组操作，可以通过 Commit 提交一个事务，也可以使用 Rollback 进行回滚。

### ACID：

1. 原子性
2. 一致性：数据库在事务执行前后都保持一致性状态。在一致性状态下，所有事务对同一个数据的读取结果都是相同的
3. 隔离性：一个事务所做的修改在最终提交以前，对其它事务是不可见的
4. 持久性：一旦事务提交，则其所做的修改将会永远保存到数据库中。即使系统发生崩溃，事务执行的结果也不能丢失

只有满足一致性，事物的执行结果才是正确的

在无并发的情况下，事务串行执行，隔离性一定能够满足，此时只要能满足原子性就能满足一致性

在并发情况下，多个事务并行执行，事务不仅要满足原子型，还需要满足隔离性，才能满足一致性

事务满足持久化是为了应对系统崩溃现象

![img](https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/image-20191207210437023.png)



## 二、并发一致性问题

丢失修改

脏读

不可重复读

幻影读



## 三、封锁

### 封锁粒度

表级锁和行级锁

### 封锁类型

#### 1.读写锁

互斥锁又称写锁

共享锁又称读锁

#### 2.意向锁  

可以更容易**支持多粒度封锁**

在存在行级锁和表级锁的情况下，事务 T 想要对表 A 加 X 锁，就需要先检测是否有其它事务对表 A 或者表 A 中的任意一行加了锁，那么就需要对表 A 的每一行都检测一次，这是非常耗时的

- 一个事务在获得某个数据行对象的 S 锁之前，必须先获得表的 IS 锁或者更强的锁；
- 一个事务在获得某个数据行对象的 X 锁之前，必须先获得表的 IX 锁



### 封锁协议

#### 三级封锁协议

##### 一级封锁协议

事务T要修改数据A必须加X锁，直到T结束才释放

##### 二级封锁协议

在一级的基础上要求读取数据A时候必须加S锁，读完马上释放

#### 三级封锁协议

在二级的基础上，要求读数据a时必须加s锁 知道事务结束才能释放

#### 两段锁协议

加锁和解锁分为两个阶段进行

**可串行调度：** 通过并发控制，使得并行执行的事务结果与某个串行执行的事务结果相同。串行执行的事务互补干扰，不会出现并发一致性问题

事务遵循两端锁协议是保证可串行调度的**充分条件**





### 隔离级别

**未递交读**: 事务的修改，即使没有递交对其他事务也是可见的

**提交读：** 一个事务只能读取以及递交的事务所作的修改

**可重复读** 保证同一个事务中多次读取的同一数据结果是一样的

**可串行化**： 强制事务串行执行

![img](https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/image-20191207223400787.png)

### 多版本并发控制（MVCC）

这个是Mysql的InnoDB存储引擎实现隔离级别的一种具体方式

用于**实现提交读和可重复读**两种隔离级别

而 MVCC 利用了多版本的思想，**写操作更新最新的版本快照**，**而读操作去读旧版本快照**，**没有互斥关系**，这一点和 CopyOnWrite 类似。

#### 版本号

1. **系统版本号：**系统版本号SYS_ID:是一个递增的数字，没开始一个新事务，就自动递增
2. **事务版本号：**TRX_ID:事务开始时的系统版本号



#### Undo日志：

MVCC多版本指的是多个版本的快照，快照存储在Undo日志中，该日志通过回滚指针ROLL_PTR把一个数据行的所有快照链接起来

假如在MYSql中创建了一个表t，包含主键id和一个字段x。我们先插入一个数据行然后对该数据行执行两次更新操作

```sql
INSERT INTO t(id, x) VALUES(1, "a");
UPDATE t SET x="b" WHERE id=1;
UPDATE t SET x="c" WHERE id=1;
```

因为没有使用 `START TRANSACTION` 将上面的操作当成一个事务来执行，根据 MySQL 的 AUTOCOMMIT 机制，每个操作都会被当成一个事务来执行，所以上面的操作总共涉及到三个事务。快照中除了记录事务版本号 TRX_ID 和操作之外，还记录了一个 bit 的 DEL 字段，用于标记是否被删除。

![img](https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/image-20191208164808217.png)

INSERT、UPDATE、DELETE 操作会创建一个日志，并将事务版本号 TRX_ID 写入。DELETE 可以看成是一个特殊的 UPDATE，还会额外将 DEL 字段设置为 1。



#### ReadView：

MVCC维护了一个ReadView结构，主要包含了当前系统未递交的事务列表

TRX_IDs







# Python面试

## python中有哪些可变类型与不可变类型

可变类型：列表 字典

不可变类型：整形 字符串  元组



### 类方法和实例方法以及静态方法

| 特征         | 实例方法           | 类方法               | 静态方法                 |
| ------------ | ------------------ | -------------------- | ------------------------ |
| 装饰器       | 无（默认）         | `@classmethod`       | `@staticmethod`          |
| 第一个参数   | `self`（实例）     | `cls`（类）          | 无                       |
| 访问实例属性 | ✅可以              | ❌不行                | ❌不行                    |
| 访问类属性   | ✅可以              | ✅可以                | ✅可以（但需手动传类）    |
| 通常用途     | 操作对象属性和行为 | 工厂方法、类属性操作 | 与类相关但独立的工具函数 |
| 调用方式     | 实例调用（推荐）   | 类或实例都可以       | 类或实例都可以           |



### 什么是迭代器为什么要使用它

可以被next（）函数调用并且不断返回下一个值得对象称为迭代器：

**迭代器**是一个实现了**迭代协议**的对象，即同时实现了：

- `__iter__()` 方法，返回迭代器对象本身
- `__next__()` 方法，每次调用返回下一个元素，直到抛出 `StopIteration` 异常

迭代是访问集合元素的一种方式》

迭代器保存的是获取数据得方式而不是结果，所以想用得时候就可以生成，**节省大量内存空间**，他是一个可以记住遍历得位置的对象，迭代器从集合的第一个元素开始访问，知道所有的元素被访问完结束。迭代器只能往前不会后退。

**3. 特点**

- 每次迭代调用 `__next__()`
- 内存效率高（按需加载）
- 一次性消费：迭代器只能遍历一次





### 什么是生成器为何要使用它

生成器是**简化版的迭代器，**是python提供的一种写迭代器的快捷方式，使用yield来返回每一个值

**形式：**

1. 生成器函数（带有yield的函数）
2. 生成器表达式（类似列表推导式）



### 特点

- 语法简单，易于实现复杂逻辑
- 自动实现了 `__iter__()` 和 `__next__()`
- 同样支持 `for` 遍历或手动使用 `next()`
- **延迟执行**（lazy evaluation），节省内存

生成器定义在Python中,**一边循环一边计算**的机制，使用了 yield 的函数被称为生成器（generator）。
跟普通函数不同的是，**生成器是一个返回迭代器的函数**，**只能用于迭代操作**，更简单点理解生成器就是一个特殊的迭代器。
在调用生成器运行的过程中，每次遇到 yield 时函数会暂停并保存当前所有的运行信息，
返回 yield 的值, 并在下一次执行 next() 方法时从当前位置继续运行。
调用一个生成器函数，返回的是一个迭代器对象。

| 特性       | 迭代器                                  | 生成器                      |
| ---------- | --------------------------------------- | --------------------------- |
| 定义方式   | 自定义类并实现 `__iter__` 和 `__next__` | 使用 `yield` 编写生成器函数 |
| 写法复杂度 | 相对繁琐                                | 简洁、直观                  |
| 状态维护   | 需手动维护                              | 自动保存状态                |
| 使用内存   | 按需加载，内存效率高                    | 同样按需加载，效率高        |
| 应用场景   | 自定义复杂遍历逻辑                      | 简单、快速实现惰性数据处理  |

### 应用场景：

1. 读取大文件：避免一次性加载进内存
2. 流式数据处理：例如网络流、日志流
3. 复杂数据管道：如数据清洗、过滤、转换等步骤串联

### 举例子：

在数据管道中定义多个步骤处理

```python
# 1. 读取文件的生成器
def read_lines(filepath):
    with open(filepath, 'r') as file:
        for line in file:
            yield line.strip()

# 2. 去除空行
def remove_empty(lines):
    for line in lines:
        if line:  # 非空字符串
            yield line

# 3. 转换成大写
def to_upper(lines):
    for line in lines:
        yield line.upper()

# 4. 过滤包含关键字的行
def filter_keyword(lines, keyword):
    for line in lines:
        if keyword in line:
            yield line

# 构建数据处理管道
def data_pipeline(filepath, keyword):
    lines = read_lines(filepath)
    lines = remove_empty(lines)
    lines = to_upper(lines)
    lines = filter_keyword(lines, keyword)
    return lines

# 运行处理流程
for line in data_pipeline('log.txt', 'ERROR'):
    print(line)

```



**生成器表达式：**

语法：expression for item in iterable if condition

```python
expression for item in iterable if condition
```

举例子：平方数生成器

```python
squares = (x * x for x in range(5))
print(next(squares))  #输出0
print(next(squares))  #输出1
print(list(squares))  #输出剩下的[4 9 16]
```

注意：生成器只能遍历一次，用next取之后 再list（）只会剩余未取的部分

实例二：从字符串中提取大写字母

```python
text = "Hello World I Am Gpt"
caps = (ch for ch in text if ch.isupper())
print(''.join(caps))  #输出HWIAG
```

这里为何不能直接打印caps呢，因为生成器不能直接打印，其本身是一个迭代器对象，本身并不存储数据，只有你取它的元素才会执行计算逻辑

**触发生成器执行的方法有**：

1. 用for循环取出每个值：
2. 用join（）拼接成字符串：
3. 把他抓换成列表或其他可打印的类型：



生成器表达式和列表推导式的区别：

| 特点     | 列表推导式 `[]`              | 生成器表达式 `()`          |
| -------- | ---------------------------- | -------------------------- |
| 返回类型 | 列表（一次性计算所有结果）   | 生成器（懒加载，按需生成） |
| 内存占用 | 高（所有元素同时存在内存中） | 低（一次只生成一个元素）   |
| 使用场景 | 小数据量或需要多次访问       | 大数据量、一次性遍历或处理 |



## 装饰器：

接受一个函数作为参数，返回一个增强后的函数

- 装饰器本质是一个函数，这个函数主要作用是包装另外一个函数或者类包装的目的是不改变原函数名字的情况下改变被包装对象的行为
- 接受一个函数，内部对其包装，然后返回一个新函数，这样子**动态增强函数的功能**
- 通过高阶函数**传递函数参数，新参数添加旧函数的需求，然后执行旧函数**

django的middkeware中间件就是高级的装饰器用法

**装饰器目的：**

- 代码复用：多个函数可以共享一套额外功能，比如打印日志认证等
- 解耦逻辑：把额外逻辑从业务逻辑中分离开来
- 保持简洁：原函数保持纯净，增强逻辑用装饰器集中管理
- 动态增强：装饰器可以灵活插拔，实现函数行为动态修改



首先理解：函数在python中是一等对象，可以被当作函数参数也可以被用来返回

例子1：函数作为参数

```python
def square(x):
    return x*x
def print_running(f,x):
    print(f'{f.__name__} is runing ' )
    return f(x)
result = print_running(square,2)
print(result)
```

例子2：装饰器

```python
import time
def squar(x):
    return x*x
#定义一个装饰器
def decorter(func):
	def wrapper(*args,**kwargs):
        start_time = time.time()   #在装饰器中加入新功能
		result = func()     #实现原来函数的功能
        end_time = time.time()
        print(f'{func__name__} exection time:{end_time-start_time} seconds')       #测量函数运行的时间
        return result
	return wrapper
```

使用装饰器

方法1：直接使用装饰器，将原来函数作为参数传给装饰器

```python
decorated_square = decorator(square)  #用一个变量接受装饰后的函数
decorated_squuare(10)       #这样就会使用装饰后的函数
```

方法2：因为装饰后的函数可以代替原来的函数，直接用原来的变量名字

```python
square = decortor(square)
square(10)      #直接代替了原来的函数
```

方法3：给函数带个帽子  和方法二是等价的  这样就代替了原来的函数了

```python
@decorator
def square(x):
	return x*x
square(10)
```





## 什么是线程安全：





## is和==的区别

”==“ 仅仅判断A和B的值是否相等

is不仅是会判断A和B的值是否相等，还会判断A和B的id是否一致



## new和init区别

```python
__new__() 和 __init()__区别
```

前者作用于后者之前，前者可以决定是否调用后者，可以决定调用哪个类的init方法



## range和xrange的区别

xrange在python3中被剔除   range本身就是惰性生成器



## yield和return的相同点和区别

- 相同点：功能都是返回程序执行结果
- 区别：yield返回执行结果并不中断程序的运行，return在返回执行结果的同时中断程序的运行

## 列举5个python常用标准库并说明其作用

- os：提供不少于操作系统相关联的函数
- sys：通常用于命令行参数
- datetime：日期时间
- re：正则匹配
- math：数学运算



列表和元组的区别：

列表是动态数组，它们可变且可以重设长度（改变其内部元素个数）

元组是静态数组，他们不可变

二者设计哲学上有不同：

列表可以被用于保存多个互相独立对象的数据集合

元组用于描述一个不会改的事务的多个属性







如果要让内部属性不被外部访问，可以把属性的名称前加上两个下划线`__`，在Python中，实例的变量名如果以`__`开头，就变成了一个私有变量（private），只有内部可以访问，外部不能访问，所以，我们把Student类改一改



这样就确保了外部代码不能随意修改对象内部的状态，这样通过访问限制的保护，代码更加健壮。

但是如果外部代码要获取name和score怎么办？可以给Student类增加`get_name`和`get_score`这样的方法：

如果又要允许外部代码修改score怎么办？可以再给Student类增加`set_score`方法：



当子类和父类都存在相同的`run()`方法时，我们说，子类的`run()`覆盖了父类的`run()`，在代码运行的时候，总是会调用子类的`run()`。这样，我们就获得了继承的另一个好处：多态

















# Django开发：

### 学习

MTV架构

django的响应模式如下：

![image-20250513164905982](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20250513164905982.png)

有人来请求通过中间后经过url进行路由分发，由相应的视图函数来找到对用的模型和模板来渲染HTML界面，然后回馈给查看的人



## 会话控制

Session存储在服，务器端口，访问网站时候所有HTTP请求都经过中间件处理，中间件SessionMiddleware会判断当前请求的用户身份是否存在，并根据判断结果执行相应的程序处理



## 缓存机制

djagno提供了五种缓存机制



●Memcached：一个高性能的分布式内存对象缓存系统，用于动态网站，以减轻数据库负载。通过在内存中缓存数据和对象来减少读取数据库的次数，从而提高网站的响应速度。使用Memcached需要安装Memcached系统服务器，Django通过python-memcached或pylibmc模块调用Memcached系统服务器，实现缓存读写操作，适合超大型网站使用。

●数据库缓存：缓存信息存储在网站数据库的缓存表中，缓存表可以在项目的配置文件中配置，适合大中型网站使用。

●文件系统缓存：缓存信息以文本文件格式保存，适合中小型网站使用。

●本地内存缓存：Django默认的缓存保存方式，将缓存存放

有视图缓存 模板缓存 路由缓存

## 面试题目：

### 什么是WSGI：

python定义的web服务器和web应用程序或者框架之间的一种简单通用的接口。让python写的应用程序可以和web服务器对接起来



### uwsgi、uwsgi和wsgi的区别：

![image-20250514153158830](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20250514153158830.png)

![image-20250514153212539](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20250514153212539.png)



### 什么是中间件：

中间件是用来处理Django请求和相应的框架级别钩子，他是一个轻量低级别的插件系统，用于全局范围内改变Django的输入和输出，

### 什么是信号量

Django包含一个"信号调度程序"，它有助于在框架中的其他位置发生操作时通知分离的应用程序。简而言之，信号允许某些发送者通知一组接收器已经发生了某些动作。当许多代码可能对同一事件感兴趣时，它们特别有用.



### Django的session的运行机制是什么：

session存储可以利用中间件实现，需要在setting.py文件中注册APP设置中间件用于启动。设置存储模式（数据库/缓存/混合存储）和配置数据库缓存用于存储，生成django_session表单用于读写。















# 爬虫



## 多线程

### 使用多线程的两种方法：

1. 函数式：调用_thread模块中的start_new_thread()函数产生新线程。
2. 类包装式：调用Threading库创建线程，从threading.Thread继承

_thread提供了低级别、原始的线程，它相比于threading模块，功能还是比较有限的。threading模块则提供了Thread类来处理线程，包括以下方法。

1. ·run()：用以表示线程活动的方法。
2. ·start()：启动线程活动。
3. ·join([time])：等待至线程中止。阻塞调用线程直至线程的join()方法被调用为止。
4. ·isAlive()：返回线程是否是活动的。
5. getName()：返回线程名。
6. ·setName()：设置线程名

## 使用Queue的多线程爬虫

模拟了同步的线程安全的队列类，使得在爬取的时候几个线程都是全速的

这就好比银行排队，单线程表示该支行只有一个窗口，要处理1000个人需要花费很长时间；简单的多线程是开5个窗口，然后将1000个人平均分到5个窗口中，因为有些窗口可能处理得比较快，所以先处理完了，但是其他窗口的人不

能去那个已经处理完的窗口排队，这样就造成了资源的闲置。Queue方法相对于前两种方法而言，是将1000人排一个长队，5个窗口中哪个窗口有了空位，便叫队列中的第一个过去（FIFO先入先出方法



# 多进程爬虫：

Python的多线程爬虫只能运行在单核上，各个线程以并发的方法异步运行，由于GIL锁的存在，多线程爬虫并不能充分的发挥多核CPU的资源

多进程爬虫则可以利用CPU的多核，进程数取决于计算机CPU的处理器个数。由于运行在不同的核上，各个进程的运行是并行的。

使用multiprocess库有两种方法，

1. 一种是使用Process+Queue的方法，
2. 另一种是使用Pool+Queue的方法。



# 多协程爬虫



















# 简历项目面试





## 爬虫项目：

重构后的代码架构

![image-20250513170509225](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20250513170509225.png)



加入flask后的项目结构图形

![image-20250513170948529](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20250513170948529.png)