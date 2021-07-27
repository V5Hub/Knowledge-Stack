# ROS基础

## 架构

### 节点

+ 执行运算任务的进程，一个系统一般又多个节点组成

### 消息

+ 节点之间最重要的一种通信机制就是基于发布/订阅模型的消息通信

  消息支持标准数据类型，也支持嵌套结构和数组

### 话题

+ 消息以一种发布/订阅的方式传递。一个节点可以针对一个给定的话题发布消息，也可以关注某个话题来订阅特定的消息

### 服务

+ 一种同步传输模式，基于客户端/服务器模型，包含两个部分的通信数据类型：一个用于请求，一个用于应答。ROS中只允许有一个节点提供指定命名的服务（与话题不同）

### 文件系统

+ `config`（放置配置文件）
+ `include`（防止功能包中要用到的头文件）
+ `scripts`（放python）
+ `src`（放需要编译的cpp）
+ launch（放launch）
+ `msg`（放置功能包自定义的消息类型）
+ `srv`（放置功能包自定义的服务类型）
+ action（放置功能包自定义的动作指令）

+ `package.xml`（功能包清单)
+ `CMakeList.txt` (编译规则)

#### package.xml

1. `<name>` ：包名
2. `<version>` ：软件包的版本号（必须为3个点分割的证书）
3. `<description>` ：包内容的描述
4. `<maintainer>` ：负责维护包的开发人员信息
5. `<license>` ：发布代码的软件许可证 (e.g. GPL, BSD, ASL)
6. `<depend>`：指定依赖关系为构建、导出和执行依赖关系，最常用，只需要提及每个ROS包依赖项就行了，比较简单。对于博主这种懒人，就适合使用它！！！
7. `<build_depend>`：指定构建此功能包所需要的包，仅使用某些特定的依赖关系来构建程序包，而不是在执行时使用，可编译来自依赖包的头文件（尤其CMake中使用find_package()时）
8. `<build_export_depend>`：指定针对该包构建所需要的功能包（尤其CMake中catkin_package()声明为(CATKIN_DEPENDS时）
9. _`<exec_depend>`：声明运行程序包时所需的共享库、可执行文件、Python模块、启动脚本和其他文件的依赖关系，指定运行该功能包中代码所需要的功能包（尤其`CMake`中catkin_package()声明为(CATKIN_DEPENDS时）
10. `<test_depend>`：指定单元测试的其他依赖项，他们不应该复制已经提到的任何以来
11. `<buildtool_depend>`：指定软件包自行构建所需要的构建系统工具，一般是catkin，这也是必须的标签
12. `<doc_depend>`：指定生成文档所需要的文档工具

#### launch文件

##### group标签

```xml
<group ns="turtlesim1">
      <node pkg="turtlesim" name="sim" type="turtlesim_node"/>
        </group>
<group ns="turtlesim2">
      <node pkg="turtlesim" name="sim" type="turtlesim_node"/>
        </group>
```

在这里我们创建了两个节点分组并以’命名空间（namespace)’标签来区分，其中一个名为turtulesim1，另一个名为turtlesim2，两个组里面都使用相同的turtlesim节点并命名为’sim’。 这样可以让我们同时启动两个turtlesim模拟器而不会产生命名冲突。(当然如果直接用不同的命名也是可以的,比如一个叫sim_1,一个叫sim_2,这样也可以同时多次启动同一节点).

+ 组可以通过判别条件来启用或者禁用节点

  如：

  ```xml
  <group if ="0-or-1">
      ........
  </group>
  或
  <group unless="1-or-0">
      ........
  </group>
  
  ```

  但往往可以结合$arg和\$()$来起到更大的作用

##### node标签

用于定义节点

```xml
<node pkg=""type=""name=""></node>
```

其中pkg与type分别是程序包名与可执行文件名

ros::init() 函数 提供的 name 信息将会全面的覆盖命名信息（launch文件中node标签里面的name 属性）/这句话没懂啥意思？

###### 拓展属性：

+ `output=“screen”`

  将标准输出信息显示在终端上

+ `respawn=“true“`

  `roslaunch`会检测每一个已经启动的节点，使用该属性可以在某个节点终止时，要求 `roslaunch` 重新启动它.

+ `required=“true“`

  当一个必须的节点终止时，`roslaunch`会做出响应，终止其他节点并退出

  （不要同时设置`respawn`和required）

+ `launch-prefix = “command-prefix”`

  对于使用`roslaunch`文件直接启动一系列节点的情形，如果节点并不需要依靠终端的输入则当然没有关系，但对于那些需要依赖终端操作而进行正常操作的节点，直接使用一个终端启动则十分让人难以接受，为了避免认为新开终端进行额外的`rosrun操作`，我们可以使用属性：`launch-prefix="xterm -e",`，其中xterm命令会新开一个终端窗口，而-e参数告诉xterm执行其命令行剩余部分命令

+ `ns = “namespace“`

  有关launch文件当中命名空间的操作

+ `args`

  将launch当中的参数arg传给节点

  

##### remap标签

+ 用于重映射：

  ```xml
  <remap from="original-name" to="new-name"/>
  ```

  如果这个remap是launch元素的一个子类，与node元素处于同一层级，并出现在launch元素内的u最顶层，那么这个remapping操作将作用于后续所有节点

##### include标签

###### file属性

```xml
<include file="path-to-launch-file"/>
可以选择使用完整路径
<include file="$(find package-name)/launch-file-name"/>
也可以使用find命令来搜索某个程序包，并依照程序名找到目标文件
```

###### ns属性

```xml
<include file="..."ns="namespace" />
```

将文件中的内容推送到一个命名空间当中

##### arg标签

+ 创建变量并赋值

  ```xml
  <arg name="arg_name" default="arg_value" />
  <arg name="arg_name" value="arg_value" />
  ```

  两种语法唯一的不同是，使用命令行可以覆盖default的值，但不能覆盖value

+ 获取argument的值

  `$(arg arg_name)`

+ 由于arg不能作用于include中包含的子launch文件使用，因此我们可以在include元素中插入arg元素作为其子类

  eg

  ```xaml
  <include file="path-to-launch-file">
  <arg name="arg-name" value="arg_value"/>
      ....
  </include>
  ```

  注意，作为include元素子类的arg不同于本launch当中已经声明过的arg

### ros命令

+ rospack 查找某个功能包的地址：

  `$ rospack find package_name`

+ roscd 跳转到某个pkg目录下

+ rosls列举某个pkg下的文件信息

+ rosed 编辑文件

+ catkin_create_pkg 创建一个pkg

+ rosdep 安装某个pkg所需依赖：`rosdep install [pkg_name]`

+ rosservice 有关service的操作

+ rossrv 有关srv的操作

+ rosparam 有关参数信息的相关操作



## 通信架构（写重复了）

### master

+ 每个node启动时都要向master注册。
+ master来管理node之间的通信。
+ 启动方法：roscore
  + 同时启动rosout用于日志输出
  + parameter server作为参数服务器

### node

+ 按照功能划分
+ `rosrun [pkg_name][node_name]`即可启动一个节点
+ `rosnode list` 列出当前节点信息
+ `rosnode info [node_name]`显示一个节点的信息
+ `rosnode kill [node_name]`结束某个node

### roslaunch 

+ 启动时会自动检测roscore是否启动

### service

+ 同步通信方式，Node间可以通过request-reply方式通信
+ 多对一
+ 适用于偶尔调用的功能

### Topic 

+ 异步通信方式
+ 多对多
+ 适用于连续高频的数据发布

### srv文件

+ 需要在package.xml和CMakeList.txt中添加相关依赖

+ 问答消息之间用`---`隔开

  + eg：

    ```xml
    bool start_detect
    ---
    pkg/mymsg_ return
    ```

    srv当中可以使用msg中自定义的消息类型

### action

+ 升级版的service，带有状态反馈的通信方式
+ 通常使用在上时间可抢占的任务当中，如导航（actionlib）

+ .action文件



## roscpp

### 主要功能

+ ros::init 解析传入的ROS参数（必备）
+ ros::NodeHandle 和topic，service，param等交互的公共接口
+ ros::master 包含从master查询信息的函数
+ ros::this_node包含查询这个进程的函数
+ ros::servic包含查询服务的函数
+ ros::param 包含查询参数服务器的函数，（不需要使用NodeHande）
+ ros::names包含处理ros图资源名称的函数
