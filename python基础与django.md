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

### 版本号

1. **系统版本号：**系统版本号SYS_ID:是一个递增的数字，没开始一个新事务，就自动递增
2. **事务版本号：**TRX_ID:事务开始时的系统版本号

























