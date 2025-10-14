# ROS2



## 1、Linux基础

### 1.1环境变量

linux终端执行命令，默认把命令当作可执行的东西

![image-20250924214534753](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20250924214534753.png)

run是运行   turtlesim是功能包    turtlesim_node是可执行文件

原理：利用**环境变量 找到对应的功能包下面的可执行文件**







## 2、节点

### 2.1编写第一个节点

![image-20250927131523736](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20250927131523736.png)



手动编写的camkelist 是为了可以让程序倒找对应的头文件和依赖

先添加可执行文件  然后查找库文件和头文件  然后再添加

![image-20250927131702684](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20250927131702684.png)

- ros2 node list  
- 

- 修复路径 让插件可以知道去哪找头文件，这样才能进行代码提示。



![image-20250927132426006](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20250927132426006.png)



### 2.2使用功能包组织c++节点

什么是功能包：

#### 首先创建功能包

![image-20250927133603503](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20250927133603503.png)

<u>**ldb** cpp_node  查看依赖和动态链接库</u>

使用 colcon build命令在/ros2/chapt2$ colcon build目录下对代码文件进行编译了之后，可以在~/ros2/chapt2/build/demo_cpp_pkg这个目录下找到可执行文件，cpp_node .

如果想在/ros2/chapt2这个目录下使用功能包加节点名字进行运行的话，是不可以的，**因为没有添加到环境变量**，系统找不到。

![image-20250927184132569](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20250927184132569.png)

<u>**ros2 pkg prefix** demo_cpp_pkg   可以查看功能包的路径 </u>

为了可以使用，需要添加环境变量

- ![image-20250927184843191](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20250927184843191.png)

  首先是记得在cmakelists中添加拷贝，将可执行文件**拷贝到功能包的lib文件夹下面**这里的${PROJECT_NAME}是环境变量 代表当前工程名字配置的值![image-20250927185128489](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20250927185128489.png)

- 其次在chapt2目录下重新构建（~/ros2/chapt2$ colcon build）一次（**注意不要在功能包里面进行构建**）
- 然后可以在这个~/ros2/chapt2$ source install/setup.bash ，**将环境变量添加**

<u> printenv | grep AME   可以打印环境变量，然后过滤显示</u>





#### **功能包结构分析：**

- include 是用来存放头文件的
- src是代码资源目录，可以放一些节点的代码或其他相关代码
- Cmakelist  是完成对可执行文件的依赖的查找和添加
- package。xml 是功能包的
- 可以根据实际功能添加 例如放置地图的map 放置配置的config

![image-20250927182754625](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20250927182754625.png)



### 2.3多功能包的最佳实现WorkSpace（工作空间）

建立功能包：mkdir -p chapt2_ws/src -p是循环创建（工作空间是个概念，本质是文件夹）

构建功能包下的单独功能包：

```sh
colcon build --packages-select demo_cpp_pkg
```

构建的时候如果有依赖的话，可以在xml中声明依赖，那样就可以先构建依赖的包。





### 2.4常用的C++新特性

#### **共享指针**，

避免重复拷贝，且可以自动释放内存占用（当引用计数为零的时候）

```c++
int  main(){
    auto p1 = std::make_shared<std::string>("This is a str");
    std::cout<<"p1的饮用计数"<<p1.use_count()<<std::endl;

    auto p2 =p1;
    std::cout<<"p1的引用计数："<<p1.use_count()<<"p1的内存地址"<<p1.get()<<std::endl;
    std::cout<<"p2的引用计数："<<p2.use_count()<<"p2 memory address"<<p2.get()<<std::endl;

    p1.reset();
    std::cout<<"p1的引用计数："<<p1.use_count()<<"p1 memory address:"<<p1.get()<<std::endl;
    std::cout<<"p2的引用计数："<<p2.use_count()<<"p2 memory address:"<<p2.get()<<std::endl;

    std::cout<<"p2的值："<<p2->c_str()<<std::endl;

    return 0;
}
```



#### **lambda表达式**(换个语法写的函数)

格式

为了写匿名函数，

【capture list】（parameters）-> return_type{function body}

```c++
int main(){
    auto add = [](int a,int b)->int{return a+b;};
    int sum = add(10,20);  
    auto print_sum = [sum]()->void
    {
        std::cout<<"sum = "<<sum<<std::endl;
    };
    print_sum();
    return 0;
}
```

在上面的例子上就把【sum】直接给捕获了，所以里面就不用传入sum，如果里面使用【&】就捕获**了上下文的所有参数**





#### 函数包装器

统一三种函数的实现方式

1. 自由函数

2. 成员函数

3. lambda函数

4. ```c++
   int main(){
       FileSaver file_saver;
       auto save_with_lambad_fun = [](const std::string &file_name) -> void
       {
           std::cout << "save file with lambad function:" << file_name << std::endl;
       };
       //将自由函数放进function对象
       std::function<void(const std::string &)> func1 = save_with_free_func;
       //将Lambda函数放进function对象
       std::function<void(const std::string &)> func2 = save_with_lambad_fun;
       //将成员函数放进function对象
       std::function<void(const std::string &)> func3 = std::bind(&FileSaver::save_wihs_member_func,&file_saver,std::placeholders::_1);
   
       func1("file1.txt");
       func2("file2.txt");
       func3("file3.txt");
   }
   ```

   注意在为成员函数绑定进入function对象的时候的语法很复杂

**函数**： `&FileSaver::save_wihs_member_func`
 表示 `FileSaver` 类中的一个成员函数，比如定义可能是：

**对象**： `&file_saver`
 这是一个 `FileSaver` 类的实例地址，告诉 `bind` 调用这个对象的成员函数。

**占位符**： `std::placeholders::_1`
 代表以后调用这个函数对象时传入的**第一个参数**，会被替换到这里。



#### 多线程和回调函数





# 3、话题

ros2相关命令解释

- ros2 topic info  加话题的名字    就可以查看到话题的订阅者和消息接口和接受者
- ros2 interface show （查看消息接口的详细信息）
- ros topic pub +话题的名字 +消息的接口+参数  表示**发布话题**
- ros2 topic list  查看当前的话题
- ros2 node list 查看节点列表
- ros2 node info +节点的名字  查看节点的信息
- ros2 topic echo+话题的名字   可以输出**话题数据**



## C++中的话题订阅与服务
