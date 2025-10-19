# 1. c++基础语法

#### 变量类型

##### 字符型

char



####  函数

##### 1.1 函数分文件编写

是为了将大量的代码分成各个模块分给不同文件中

![image-20250102102913687](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20250102102913687.png)

函数的声明是为了告诉编译器，系统有这个函数，但是现在还没有定义，声明可以有多个 但是函数的定义只能有一个

做法就是将声明的函数一个在头文件中声明用到的函数   

再在源文件中写函数的定义  但是要加上#include" "  双引号表明是自己定义的头文件

####  指针

```c++
int *p //定义一个指针

p = &a   //p 指向a的地址
cout << p;   //输出内存地址
cout << *p; //输出内存中的内容   解引用
```

空指针用来给指针进行初始化变量 但是不能进行访问

int *p = NULL; 



- 常量的指针
  **const修饰指针，指针的指向可以改 但是指针的值不可以改**

  ```c++
  const int *p = &a;  
  p = &b;//正确
  //错误示范  
  // *p = 100;  //就会报错 
  ```

- 指针的常量    

- **const修饰常量，指针的指向不可以改 指针指向的值可以改**

```c++
int *const p2 = &a; //p2这个值就定下来了 ，即地址就定下来了 所以指针的指向肯定是没法改
*p2 = 100;
p2 = &b; //会报错
```





利用指针作为函数的参数 可以修改实参的值



#### 结构体

结构体数组，在这之中用  .  来访问结构体数组中的东西

```c++
//结构体定义
struct student
{
	//成员列表
	string name;  //姓名
	int age;      //年龄
	int score;    //分数
}

int main() {
	
	//结构体数组
	struct student arr[3]=
	{
		{"张三",18,80 },
		{"李四",19,60 },
		{"王五",20,70 }
	};
```

结构体指针

通过 -> 可以访问结构体指针中的结构体属性

```c++
//结构体定义
struct student
{
	//成员列表
	string name;  //姓名
	int age;      //年龄
	int score;    //分数
};


int main() {
	
	struct student stu = { "张三",18,100, };
	
	struct student * p = &stu;
	
	p->score = 80; //指针通过 -> 操作符可以访问成员
```

结构体作为函数的形参

```c++
//学生结构体定义
struct student
{
	//成员列表
	string name;  //姓名
	int age;      //年龄
	int score;    //分数
};

//值传递
void printStudent(student stu )
{
	stu.age = 28;
	cout << "子函数中 姓名：" << stu.name << " 年龄： " << stu.age  << " 分数：" << stu.score << endl;
}

//地址传递
void printStudent2(student *stu)
{
	stu->age = 28;
	cout << "子函数中 姓名：" << stu->name << " 年龄： " << stu->age  << " 分数：" << stu->score << endl;
}
```













# 2.面向对象编程基础

## 内存分区

C++程序在执行时，将内存大方向划分为**4个区域**

- 代码区：存放函数体的二进制代码，由操作系统进行管理的
- 全局区：存放全局变量和静态变量以及常量
- 栈区：由编译器自动分配释放, 存放函数的参数值,局部变量等
- 堆区：由程序员分配和释放,若程序员不释放,程序结束时由操作系统回收  由new开辟，由delete释放

```c++
int* func()
{
	int* a = new int(10);  //new 关键字返回的是一个地址  所以要用指针去接受
	return a;
}
int main() {

	int *p = func();

	cout << *p << endl;
	cout << *p << endl;

	//利用delete释放堆区数据
	delete p;

	//cout << *p << endl; //报错，释放的空间不可访问

	system("pause");

	return 0;
}
```



**内存四区意义：**

不同区域存放的数据，赋予不同的生命周期, 给我们更大的灵活编程





#### 引用

引用的本质在c++内部实现是一个**指针的常量.**





## 类和对象

**struct和class区别：**唯一区别在于类默认为私有权限  结构体默认为公共权限

**成员属性设置为私有优点**



**优点1：**将所有成员属性设置为私有，可以自己控制读写权限

**优点2：**对于写权限，我们可以检测数据的有效性





### 对象的初始化和清理

-  **初始化**：构造函数（主要作用在于创建对象时为对象的成员属性赋值，构造函数由编译器自动调用，无须手动调用。）

​	**构造函数语法：**`类名(){}`

1. 构造函数，没有返回值也不写void
2. 函数名称与类名相同
3. 构造函数可以有参数，因此可以发生重载
4. 程序在调用对象时候会**自动调用构造**，无须手动调用,而且**只会调用一次**



- **清理**：析构函数（主要作用在于对象**销毁前**系统自动调用，执行一些清理工作）

**	析构函数语法：** `~类名(){}`

1. 析构函数，没有返回值也不写void
2. **函数名称与类名相同,在名称前加上符号  ~**
3. 析构函数**不可以有参数，因此不可以发生重载**
4. 程序在对象销毁前会自动调用析构，无须手动调用,而且只会调用一次





**构造函数分类：**

- 有参构造



- 无参构造

在调用是不需要加上括号

Person p; //调用无参构造函数

- 拷贝构造

注意写法:  

Person(const Person& p){ //用const保证p不会被修改

​	age = p.age; 

}

```c++
class Person {
public:
	//无参（默认）构造函数
	Person() {
		cout << "无参构造函数!" << endl;
	}
	//有参构造函数
	Person(int a) {
		age = a;
		cout << "有参构造函数!" << endl;
	}
	//拷贝构造函数
	Person(const Person& p) {   //const修饰形参 防止形参改变实参
		age = p.age;
		cout << "拷贝构造函数!" << endl;
	}
	//析构函数
	~Person() {
		cout << "析构函数!" << endl;
	}
public:
	int age;
};
```





### 深拷贝与浅拷贝

浅拷贝：简单的赋值拷贝操作



深拷贝：在堆区重新申请空间，进行拷贝操作

```c++
class Person {
public:
	//无参（默认）构造函数
	Person() {
		cout << "无参构造函数!" << endl;
	}
	//有参构造函数
	Person(int age ,int height) {
		
		cout << "有参构造函数!" << endl;

		m_age = age;
		m_height = new int(height);   //因为new出来的数据会返回该类型数据所对应的指针，这里mheight就是一个指针
		
	}
	//拷贝构造函数  
	Person(const Person& p) {
		cout << "拷贝构造函数!" << endl;
		//如果不利用深拷贝在堆区创建新内存，会导致浅拷贝带来的重复释放堆区问题
		m_age = p.m_age;
		m_height = new int(*p.m_height);
		
	}

	//析构函数
	~Person() {
		cout << "析构函数!" << endl;
		if (m_height != NULL)
		{
			delete m_height;
		}
	}
public:
	int m_age;
	int* m_height;
};
```





**总结：如果属性有在堆区开辟的，一定要自己提供拷贝构造函数，防止浅拷贝带来的问题**

c++中常用new 在堆区来开辟空间



### 列表初始化



```c++
//初始化列表方式初始化
	Person(int a, int b, int c) :m_A(a), m_B(b), m_C(c) {}
```



### 类对象作为类成员





### 静态成员

静态成员就是在成员变量和成员函数前加上关键字static，称为静态成员

静态成员分为：



*  静态成员变量
   *  所有对象共享同一份数据
   *  在编译阶段分配内存
   *  **类内声明，类外初始化**
*  静态成员函数
   *  所有对象共享同一个函数
   *  静态成员函数只能访问静态成员变量



### this指针

用来







### 继承

继承的语法：`class 子类 : 继承方式  父类`

**继承方式一共有三种：**

* 公共继承
* 保护继承
* 私有继承

![image-20250116123123449](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20250116123123449.png)



















# c primer plus

形参 函数定义的列表内的参数叫做形参

局部变量 函数内部的变量叫做局部变量，每次函数被调用时到其定义语句生成 块调用结束时候销毁

局部静态变量 在程序第一次经过该定义语句时候生成 直到程序结束才被销毁  即使函数调用结束对其也无影响





为什么使用引用  为了避免拷贝









# QT

- 为什么要使用命名空间：

​	答：可以在另外一个文件中引入，从而轻松的使用函数



- 标准错误流 ( cerr  ) 和标准日志流 ( clog  )

 		cerr 用于输出**错误**消息。与  cout 不同，cerr **不是缓冲**的，这意味着它会立即输出。
 	
 		**clog** 类似于   cerr ，但**它是缓冲**的。它**通常用于记录错误和日志信息**。



- 内联函数：

  - 其定义直接在调用点的地方展开，**速度较快**	

  - ：适合于性能要求高的代码中频繁使用的函数

  - 缺点：若使用较多可能会造成代码体积肿胀

  - ```c++
    inline int max(int x, int y) {  //inline指示出内联函数
     return x > y ? x : y;
     }
    ```

- Lambda表达式

它允许你在需要函数的地方内联的定义它，而无需单独命名函数



**二者区别**  ： Lambda函数核心优势在于他们的匿名性质，和对外部变量的捕获能力，		而内联则主要关注于提高小型函数的性能。







- 普通变量访问成员函数和成员变量时候，使用“ . ”运算符
- 指针变量访问时候，使用 ->  来访问

关于指针什么时候使用最合适：

1. 动态分配内存（new出来的对象）

2. 创建大型对象或者复杂对象的时候

3. 需要共享一个对象（多个变量指向同一个对象）   比如多个模块函数都要操作同一个对象不希望复制

4. 实现多态（继承＋虚函数时候**必须**要用），

5. ```c++
   class Animal {
   public:
       virtual void speak() { cout << "Animal sound" << endl; }
   };
   
   class Dog : public Animal {
   public:
       void speak() override { cout << "汪汪汪" << endl; }
   };
   
   Animal* a = new Dog(); // 父类指针指向子类对象（多态）
   a->speak();            // 输出：汪汪汪
   ```

   

6. 在函数中修改外部变量（直接通过指针传回来的地址直接改动）

7. 管理可能为“空”的对象

c++ 11之后，推荐用智能指针代替裸指针  可以自动回收内存

```c++
#include <memory>

std::shared_ptr<Car> car1 = std::make_shared<Car>();    //shared_ptr: 多人共享一辆车（多个指针指向）
std::unique_ptr<Car> car2 = std::make_unique<Car>();    //你是这辆车唯一的主人（一个指针独占）

```





## 引用

引用：引用是一个别名，是给一个变量起的另外的一个名字，但是**是直达地址的**

引用很容易与指针混淆，它们之间有三个主要的不同：

1.  **不存在空引用**。引用必须连接到一块合法的内存。
2.  一旦引用被初始化为一个对象，**就不能被指向到另一个对象**。指针可以在任何时候指向到另一个对 象。 
3. 引用**必须在创建时**被初始化。指针可以在任何时间被初始化。

官方没有明确说明，但是引用确实不是传统意义上的独立变量，它不能“变”嘛 试想变量名称是变量附属在内存位置中的标签，您可以把引用当成是变量附属在内存位置中的第二 个标签。因此，您可以通过原始变量名称或引用来访问变量的内容。



## 重载

函数重载：在同一个作用域中，可以声明几个功能类似的同名函数

运算符重载：

```c++
class Point {
public:
 int x, y;
 // 重载 + 运算符
Point operator+(const Point& other) const {
 return Point(x + other.x, y + other.y);
 }
 };
```



## 构造函数

无参构造函数

有参构造函数

拷贝构造函数：用于创建一个新对象，作为现有对象的副本  他在以下几种情况被调用

1. 当一个新对象被创建为另一个同类型的现有对象的副本时候
2. 当对象作为参数传递给函数时候，并且参数不是引用
3. 从函数返回对象的时候
4. 初始化数组或者容器中的元素时候   

```c++
class MyClass {
 public:
 MyClass(const MyClass& other);
 };
```







避免不必要的拷贝：，尤其是在处理大型对象或资源密集型对象时

1. 使用引用（包括常量引用来传递对象），**可以避免在函数调用时创建对象的副本**
2. 使用移动语义





拷贝构造函数会一些情况下被隐式调用：

1. . 作为函数参数传递（按值传递）
2. 从函数返回对象时候（按值返回）
3. 初始化另外一个对象时候，用这一个初始另外一个

如何禁用拷贝构造函数（尤其是设计那些不应该被复制的类的时候）：使用delete关键字

```c++
class NonCopyable {
 public:
    NonCopyable() = default;  // 使用默认构造函数
    // 禁用拷贝构造函数
    NonCopyable(const NonCopyable&) = delete;
    // 禁用拷贝赋值运算符
    NonCopyable& operator=(const NonCopyable&) = delete;
 };
 int main() {
     NonCopyable obj1;
     // NonCopyable obj2 = obj1;  // 编译错误，拷贝构造函数被禁用
     return 0;
 }
```

![image-20250412163820763](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20250412163820763.png)





























# 记事本项目

弹簧：使得按钮保持位置

加入frame（widget） 使得加入背景色，加入颜色有选项可选





## 自定义信号与槽



QFile打开文件  创建文件 写入文件

QTextStream读写文件  有些优点 

**bug**  ：写入文件时候碰到乱码问题

1. 解决： 解决办法一：写入 UTF-8 带 BOM（推荐）

   添加 BOM，可以让 Windows 记事本正确识别 UTF-8 编码。

**文件选择对话框** QFileDialog     可设置模式 设置过滤器 































# 智能家居项目

![image-20250413095717470](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20250413095717470.png)



## 界面开发

放图片，放文本，放gif图的组建就是qlabel

文本输入 line edit  一行文本



如何添加图片

- 添加图片资源文件   给工程添加资源文件夹
- 引用这个图片

布局：保证页面不会乱

1. 水平
2. 垂直
3. 栅格    使用弹簧

界面切换

1. 创建新的界面   qt设计师界面类





## **qt的三驾马车**

1. qt下的串口编程
2. qt下的网络编程
3. qt下的操作GPIO



## qt下串口编程



接受 框框组件：   PlainTextEdit

属性选择：下拉组件： Combo BOX

发送框  ：line edit

**写代码之前给控件改名字**



### 打包与部署操作

![image-20250415203526467](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20250415203526467.png)

然后创建一个新的文件夹，将exe文件放进去

![image-20250415204040756](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20250415204040756.png)

最后一步我们使用windeployqt工具加到我们的

![image-20250415204235029](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20250415204235029.png)





## 网络编程

TCP篇

需要Qtcpsocket和Qtcpsocket

客户端只需要Qtcpsocket

服务端都需要

### 服务端开发

tcpserver - > listen(QHostAddress::Any，ui->portEdit->text().toUnint )  监听来自所有人的连接，后面是端口号的记录

槽函数中实例化socket





# MP3音乐播放器搜索引擎

布置界面   linedit   plaintextedit

groupbox  边框

horizion slider   //拖拉进度条

**bug**：： 在一个资源文件qrc中，一个文件夹下面加了另外一个qrc文件   编译不通过  删除之后还是不行

解决：直接把resource文件删除了，重新加入图标

![image-20250414110702797](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20250414110702797.png)



resource文件中多了一个  qrc文件 ，就没法过去 了，删除之后没有选他就可以过了











# 串口通信

什么是串口通信：串口通信（Serial Communication）是一种**数据按位（bit）顺序一位一位依次传输**的通信方式，广泛用于**嵌入式系统、传感器、模块、计算机与外部设备之间的数据传输**。





























# linux下编程：

由于linux下

\Linux 上常用的编译器是 gcc 和 clang。



























# 高阶语法

## 移动构造函数

```c++
// 移动构造函数
LogRecord::LogRecord(LogRecord&& other) noexcept 
    : level(other.level), message(std::move(other.message)), 
      timestamp(std::move(other.timestamp)), thread_id(other.thread_id) {
}
```

两个基本参数level和thread_ID就是直接复制的，因为复制和移动的成本**相同**但是对于复杂的类型**就是只转移指针**（移动）更方便

noexcept表示函数不会抛出异常，**编译器可以进行更激烈的优化**所以有些容易会优先使用

```c++
message(std::move(other.message))
```

std::move 的本质：
std::move 实际上是一个类型转换函数，不是移动操作本身
它**将左值引用转换为右值引用**
真正的移动操作由目标类型的**移动构造函数**完成



### 重要：为什么需要移动构造函数为何不用指针

1.指针传递的问题：

- 内存管理复杂：需要**手动管理内存**，容易造成内存泄漏
- 所有权不明确：不清楚谁负责释放内存
- 异常不安全：如果发生异常，可能导致内存泄漏
- 性能开销：需要动态分配内存

2.移动构造函数的优势

- 自动内存管理：**使用智能指针**，自动管理内存
- 所有权转移：**明确资源所有权转移**
- 异常安全：RAII机制保证异常安全
- 性能优化：保证不必要的拷贝

同时**智能指针**也可以做到自动管理内存的作用，智能指针解释见下文2.3



## 左值引用与右值引用

### 左值引用

```c++
Type& ref = lvalue
const Type& ref = lvalue
```

**特点**：

- 绑定到左值（有名字的变量）
- 可以取地址
- 生命周期长
- 可以修改（除非是const引用）

当函数的参数声明为常量引用的时候，同时可以接受左值和右值

左值是有着某种存储支持的变量，右值是临时的值

**使用右值引用作为参数的时候，我们不需要担心是否完整，是否被拷贝，我们可以简单的偷它的资源，给到特定的对象或者在其他地方使用它们**



## 移动语义



## 类外声明函数

类外声明函数的情况：

1. 全局工具函数：不属于任何类，提供通用功能

1. 友元函数：需要**访问私有成员****但不是成员函数**

1. 运算符重载：某些运算符必须在类外定义

1. 工厂函数：创建对象的函数

1. 处理函数：处理特定类型对象的函数

声明方式：

- 在类外直接声明和定义

- 在头文件中声明，源文件中定义

- 使用 friend 关键字声明友元函数

- 使用 inline 关键字声明内联函数



## 为什么需要友元函数

### 1. 1访问控制需求

- 某些外部函数需要访问类的私有成员

- 提供比公共接口更直接、更高效的访问方式

### 1.2. 运算符重载

- 流运算符 << 和 >> 必须在类外定义

- 需要访问私有成员来正确实现运算符功能

### 1.3. 设计模式支持

- 工厂模式、访问者模式等需要友元函数

- 支持更复杂的类间协作

### 1.4. 性能考虑

- 避免通过公共接口的间接访问

- 直接访问私有成员，提高性能

### 1.5. 功能完整性

- 提供完整的类功能接口

- 支持全局函数与类的深度集成



### 2. 再问：为什么工厂模式和访问者模式需要友元函数

#### 2.1工厂模式定义

工厂模式是一种创建型设计模式，他提供一种创建对象的最佳方式，其**不使用new操作**符操作队形而是**通过工厂方法**来创建对象

#### 2.2工厂模式结构

```c++
//抽象产品类
class Logger{
public:
	virtual ~Logger() = default;
    virtual void log(const std::string& message) = 0;
};
//具体产品类
class Filelogger : public Logger{
private:
    std::string filename_;
public:	
    FileLogger(const std::string& filename) : filename_(filename){}
    void log(const std::string&) override{
        std::cout << "文件日志："<< message << std::endl;
    }
};
class ConsoleLogger : public Logger{
public:
    void log(const std::string& message) override{
        std::cout<<"控制台日志" << message <<std::endl;
    }
};
//工厂类
class LoggerFactory{
public:
    static std::unique_ptr<Logger> creatLogger(const std::string& type,const std::string& filename = ""){
        if(type == "file"){
            return std::make_unique<FileLogger>(filename);
        }else if(type == "console"){
            return std::make_unique<ConsoleLogger>();
        }
        return nullptr;
    }
};
	
```

加入友元函数之后可以访问类的内部变量（私有成员），进行额外配置

### 2.3 工厂模式中的友元函数使用示例

```c++
class Filelogger : public Logger{
private:
    std::string filename_;
    std::ofstream file_stream_;  //文件输出流类 用于向文件中写入数据
    
public:
    //友元函数声明
    friend std::unique_ptr<FileLogger> createFileLogger(const std::string& filename);
    FileLogger(const std::string& filename):filename_(filename){
        file_stream_.open(filename_);
    }
    void log (const std:: string&message)override{
        file_stream_<<message << std::endl; //使用流操作符写入文件
    }
};
//友元函数定义
std::unique_ptr<FileLogger> createFileLogger(const std::string& filename){
    auto logger = std::make_unique<FileLogger>(filename);
    //可以访问私有成员进行额外配置
    return logger;
}
```

在友元函数中的声明中：std::unique_ptr<FileLogger>  的含义**是返回类型是智能指针指向FileLogger对象**

##### （智能指针解释）什么是 std::unique_ptr

```c++
std::unique_ptr<FileLogger> createFileLogger(const std::string& filename)
```

引入的智能指针

- 作用：自动管理动态分配的内存
- 特点：独占所有权，不能拷贝 只能移动
- 优势：自动释放内存，防止内存泄漏

**使用：**

```c++
//创建unique_ptr
auto logger = std::make_unique<FileLogger>(filename);
//移动语义
std::unique_ptr<FileLogger> logger2 = std::move(logger);//logger变为空
//自动释放
//当logger离开作用域的时候，自动调用析构函数释放内存
```

#### 再再问：为什么抽象类中需要定义虚析构函数和虚函数

**先解释一下虚函数**：虚函数是C++中实现多态的核心机制，它允许运行的时候决定运行哪个函数。

```c++
class Animal {
public:
    virtual void makeSound() {  // 虚函数
        std::cout << "动物发出声音" << std::endl;
    }
    
    virtual ~Animal() = default;  // 虚析构函数
};

class Dog : public Animal {
public:
    void makeSound() override {  // 重写虚函数
        std::cout << "汪汪汪" << std::endl;
    }
};

class Cat : public Animal {
public:
    void makeSound() override {  // 重写虚函数
        std::cout << "喵喵喵" << std::endl;
    }
};
```

虚函数表，编译器为每个包含虚函数的类**创建了一个虚函数表**：

虚函数的调用过程

```c++
Animal* animal = new Dog()
animal->makeSound()
```

调用过程：

1.通过对象指针找到虚函数表指针

2.在虚函数表中查找对应的函数

3.调用找到的函数



**再解释一下为何需要虚析构函数：**

因为定义了虚析构函数，才能在派生类中**正常释放资源**，会先**调用***DerivedWithVirtual的析构函数，再调用BaseWithVirtual的析构函数*，从而保证内存不会泄露。

**设计原则：**

1. 基类有虚函数时，必须有虚析构函数

1. 抽象类必须提供虚析构函数

1. 使用 override 关键字确保正确重写



### 3.再问：为什么访问者模式需要使用友元函数

#### 3.1访问者模式的定义：

访问者模式是一种行为型设计模式，它允许你在不改变对象结构的情况下，**定义作用于这些对象的新操作**

#### 3.2访问者模式的结构







## 拷贝构造函数与拷贝赋值运算符

### 1. 基本概念对比

| 操作     | 拷贝构造函数                      | 拷贝赋值运算符                               |
| :------- | :-------------------------------- | :------------------------------------------- |
| 调用时机 | 创建新对象时                      | 已存在对象赋值时                             |
| 语法     | ClassName(const ClassName& other) | ClassName& operator=(const ClassName& other) |
| 对象状态 | 目标对象不存在                    | 目标对象已存在                               |

### 2.调用场景

#### 拷贝构造函数：

```c++
LogRecord record1(Loglevel::INFO,"消息1");
LogRecord record2 = record1;    //拷贝构造函数：创建新对象record2
LogRecord record3(record1);      //拷贝构造函数：创建新对象record3
    
```

#### 拷贝赋值运算符：

```c++
LogRecord record1(LogLevel::INFO, "消息1");
LogRecord record2(LogLevel::DEBUG, "消息2");
record2 = record1;  // 拷贝赋值运算符：将record1的值赋给已存在的record2
```

```c++
// 拷贝赋值运算符
LogRecord& LogRecord::operator=(const LogRecord& other) {
    if (this != &other) {
        level = other.level;
        message = other.message;
        timestamp = other.timestamp;
        thread_id = other.thread_id;
    }
    return *this;
}
```

关键点：

1. 自赋值检查：if(this != &other) 防止自己赋值给自己
2. 逐成员赋值：将源对象的每个成员赋值到目标对象
3. 返回引用：返回*this 以支持链式赋值

注意：在logger类中就删除了拷贝赋值运算符，因为在其中包含文件输入流，处理起来较为麻烦

#### 代码示例：

```c++
class AssignmentDemo{
private:
    std::string data_;
    int id_;
public:
    //省略其他
    AssigmentDemo& operator=(const AssignmentDemo& other){
        std::cout << "拷贝赋值运算符被调用"<<std::endl;
        if(this != &other){//自赋检查
            data = other.data_;
            id = other.id;
            std::cout<<"赋值："<<data_<<"(id:"<< id_ <<")"<<std::endl;}else{
            std::cout<<"自赋值 跳过"<<std::endl;
}
            
        }
    }
};
```



## 移动赋值运算符

### 1.基本概念：

移动赋值运算符用于将右值对象的资源转移给左值对象，**避免不必要的拷贝**操作

### 2.语法形式：

```c++
class MyClass{
public:
    //移动赋值运算符声明
    MyClass& operator =(MyClass&& other) noexcept;
};
//移动赋值运算符实现
MyClass& MyClass::operator=(MyClass&& other)noexcept{
    if(this != &other){//自赋值检查
        //释放当前资源
        //移动other的资源
        //将other置于有效但未指定状态
    }
    return *this;    //返回当前对象的引用，支持链式赋值
}
```

### 3. LogRecord 类的移动赋值运算符

```c++
LogRecord& LogRecord::operator=(LogRecord&& other) noexcept {
    if (this != &other) {  // 自赋值检查
        level = other.level;                    // 基本类型，直接赋值
        message = std::move(other.message);     // string，移动赋值
        timestamp = std::move(other.timestamp); // string，移动赋值
        thread_id = other.thread_id;            // 基本类型，直接赋值
    }
    return *this;  // 返回当前对象的引用
}
```

### 4.使用场景

#### 4.1直接移动赋值

```c++
return *this; //返回当前对象的引用，支持链式赋值
```

#### 4.2从临时对象赋值

```c++
LogRecord obj1("消息1",LogLevel::INFO);
LogRecord obj2("消息2",Loglevel::WARNING);
obj1 =  std::move(obj2);  //调用移动赋值运算符
//obj2处于移动后状态
```

#### 4.3在容器中使用

```c++
LogRecord obj("临时",LogLevel::DEBUG);
obj = LogRecord("新消息",LogLevel::ERROR); //临时对象被移动
```

### 移动赋值 vs 拷贝赋值

| 特性       | 移动赋值       | 拷贝赋值         |
| :--------- | :------------- | :--------------- |
| 参数类型   | ClassName&&    | const ClassName& |
| 资源处理   | 转移资源所有权 | 复制资源         |
| 性能       | 高效（O(1)）   | 较慢（O(n)）     |
| 原对象状态 | 有效但未指定   | 保持不变         |
| 适用场景   | 右值对象       | 左值对象         |



## 枚举类型

### 1 定义：

枚举类型是一种用户定义的数据类型，它包含一组命名的常量值。

### 2 基本语法

```c++
//传统枚举
enum Color{
    RED, //0
    GREEN, //1
    BLUE  //2
}
//强类型枚举
enum class LogLevel{
    DEBUG = 0，
    INFO = 1，
    WARNING = 2，
    ERROR = 3
}
```

### 3 为何需要使用枚举类型

#### 3.1替代魔法数字

```c++
// 使用魔法数字 - 不推荐
void logMessage(int level, const std::string& message) {
    if (level == 0) {  // 0 代表什么？不清楚
        std::cout << "DEBUG: " << message << std::endl;
    } else if (level == 1) {  // 1 代表什么？不清楚
        std::cout << "INFO: " << message << std::endl;
    }
}

// 调用时容易出错
logMessage(5, "test");  // 5 是什么级别？错误！
```

但是使用枚举类型就可以轻松解决

```c++
enum class LogLevel {
    DEBUG = 0,
    INFO = 1,
    WARNING = 2,
    ERROR = 3
};

void logMessage(LogLevel level, const std::string& message) {
    if (level == LogLevel::DEBUG) {  // 清晰明确
        std::cout << "DEBUG: " << message << std::endl;
    } else if (level == LogLevel::INFO) {  // 清晰明确
        std::cout << "INFO: " << message << std::endl;
    }
}

// 调用时类型安全
logMessage(LogLevel::INFO, "test");  // 正确
// logMessage(5, "test");            // 编译错误！
```

- 使用枚举类型还可以提高代码可读性

```c++
// 使用枚举 - 可读性好
if (currentLevel == LogLevel::ERROR) {
    // 处理错误级别
}

// 使用数字 - 可读性差
if (currentLevel == 3) {  // 3 是什么意思？
    // 处理错误级别
}
```



- 也可以直接使用枚举值进行比较

  ```c++
  void Logger::log(LogLevel level, const std::string& message) {
      if (level < min_level_) {  // 枚举值可以比较
          return;  // 日志级别过低，不记录
      }
      // 记录日志
  }
  ```

  ### 3.2总结

  枚举类型的作用和优势：

  1. 替代魔法数字：用有意义的名称替代数字

  1. 类型安全：防止类型错误

  1. 提高可读性：代码更易理解

  1. 自文档化：代码即文档

  1. 易于维护：集中管理常量值

  1. 防止错误：编译时检查

  在日志系统中，LogLevel 枚举类型确保了：

  - 日志级别的类型安全

  - 代码的可读性和维护性

  - 防止使用无效的日志级别

  - 提供清晰的级别比较和过滤功能

  #### 番外：什么是类型安全，为何枚举类型可以保证类型安全

  如果使用常量

  ```c++
  // 使用常量定义
  const int DEBUG = 0;
  const int INFO = 1;
  const int WARNING = 2;
  const int ERROR = 3;
  
  // 函数参数
  void logMessage(int level, const std::string& message) {
      if (level == DEBUG) {
          std::cout << "DEBUG: " << message << std::endl;
      } else if (level == INFO) {
          std::cout << "INFO: " << message << std::endl;
      }
  }
  ```

  问题一：类型不匹配

  ```c++
  // 这些调用都是合法的，但可能不是我们想要的
  logMessage(5, "test");        // 5 不是有效的日志级别
  logMessage(-1, "test");       // -1 不是有效的日志级别
  logMessage(999, "test");      // 999 不是有效的日志级别
  ```

  问题二：意外赋值

  ```c++
  int level = DEBUG;  // 正确
  level = 5;          // 错误：5 不是有效的日志级别，但编译器不会报错
  level = -1;         // 错误：-1 不是有效的日志级别，但编译器不会报错
  ```

  问题三：函数参数混乱

  ```c++
  void processUser(int userId, int level) {
      // 容易搞混参数
  }
  
  // 调用时可能搞混参数
  processUser(DEBUG, 123);  // 错误：参数顺序搞反了
  processUser(123, DEBUG);  // 正确
  ```

  **使用枚举的解决方案**

  ```c++
  // 使用强类型枚举
  enum class LogLevel {
      DEBUG = 0,
      INFO = 1,
      WARNING = 2,
      ERROR = 3
  };
  
  // 函数参数
  void logMessage(LogLevel level, const std::string& message) {
      if (level == LogLevel::DEBUG) {
          std::cout << "DEBUG: " << message << std::endl;
      } else if (level == LogLevel::INFO) {
          std::cout << "INFO: " << message << std::endl;
      }
  }
  ```

  优势一：类型检查

  ```c++
  // 这些调用会导致编译错误
  // logMessage(5, "test");           // 错误：类型不匹配
  // logMessage(-1, "test");          // 错误：类型不匹配
  // logMessage(999, "test");         // 错误：类型不匹配
  
  // 只有正确的调用才合法
  logMessage(LogLevel::INFO, "test");  // 正确
  ```

  优势二：防止意外赋值

  ```c++
  LogLevel level = LogLevel::DEBUG;  // 正确
  // level = 5;                      // 错误：类型不匹配
  // level = -1;                     // 错误：类型不匹配
  ```

  优势三：参数类型明确

  ```c++
  void processUser(int userId, LogLevel level) {
      // 参数类型明确，不会搞混
  }
  
  // 调用时类型明确
  processUser(123, LogLevel::DEBUG);  // 正确
  // processUser(LogLevel::DEBUG, 123);  // 错误：类型不匹配
  ```


## RAII

Resource Acquisition Is Initialization

**资源获取即初始化**

当对象被创建时候（构造函数执行时）就获取资源

当对象被销毁的时候（析构函数执行时）就自动释放资源

### 常见的 RAII 类

| 资源类型     | RAII 类                                          |
| ------------ | ------------------------------------------------ |
| 内存         | `std::unique_ptr`, `std::shared_ptr`             |
| 文件         | `std::ifstream`, `std::ofstream`, `std::fstream` |
| 互斥锁       | `std::lock_guard`, `std::unique_lock`            |
| 动态数组     | `std::vector`                                    |
| 临时状态管理 | `std::scoped_lock`, `std::jthread`（C++20）      |















# GRPC

使用protobuffer协议，序列化后的二进制比http更短

与HTTP都是使用TCP传输，但是GRPC封装了HTTP2

**二者区别如下**

![image-20251018103851886](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20251018103851886.png)

![image-20251018104348743](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20251018104348743.png)

![image-20251018104524521](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20251018104524521.png)

GRPC只需要想好如何构造response结构体就好

protoc会帮助生产两个结构体

## 为什么在分布式系统中使用GRPC

![image-20251018105959240](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20251018105959240.png)

### 分布式系统内存在的痛点

1. 性能低：json是文本格式，占空间大，解析慢
2. 接口不稳定/缺乏约束：每个服务自己定义接口字段，容易出错
3. 无原生语言支持：不同语言要手写HTTP客户端，请求/相应容易不一致
4. 没有流式通信：hTTP/1.1只能一次请求对应一次响应，无法实时推送或双向通信

### GRPC的优势解析：

1. 高性能（二进制传输+HTTP/2）

   1. grpc使用Protocol Buffers（protobuf）编码，比json小10倍以上
   2. 解析速度快，cpu消耗低
   3. HTTP/2支持多路复用（一个链接同时跑多个请求）减少延迟
   4. **同样的带宽，grpc可以传更多数据，减少延迟**

2. 强类型，自动生成代码（IDL驱动）

   1. grpc的**接口通过.proto文件定义**，然后通过proroc一键生成多语言代码

3. 原生支持流式通信

   ![image-20251018112012706](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20251018112012706.png)

4. 内置认证、超时、负载均衡、拦截器

   ![image-20251018112142091](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20251018112142091.png)

![image-20251018112852095](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20251018112852095.png)

