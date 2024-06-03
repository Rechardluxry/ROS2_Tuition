### 1.5.1 ROS2文件系统

立足系统架构，如下图所示，ROS2可以划分为三层：

* **操作系统层（OS Layer）**

  如前所述，ROS虽然称之为机器人操作系统，但实质只是构建机器人应用程序的软件开发工具包，ROS必须依赖于传统意义的操作系统，目前ROS2可以运行在Linux、Windows、Mac或RTOS上。

* **中间层（Middleware Layer）**

  主要由数据分发服务DDS与ROS2封装的关于机器人开发的中间件组成。DDS是一种去中心化的数据通讯方式，ROS2还引入了服务质量管理 （Quality of Service）机制，借助该机制可以保证在某些较差网络环境下也可以具备良好的通讯效果。ROS2中间件则主要由客户端库、DDS抽象层与进程内通讯API构成。

* **应用层（Application Layer）**

  是指开发者构建的应用程序，在应用程序中是以功能包为核心的，在功能包中可以包含源码、数据定义、接口等内容。

![](/assets/1.5.1文件系统.png)

对于一般开发者而言，工作内容主要集中在应用层，开发者一般通过实现具有某一特定功能的功能包来构建机器人应用程序。对应的我们所介绍的ROS2文件系统主要是指在硬盘上以功能包为核心的目录与文件的组织形式。

#### 1.概览

功能包是ROS2应用程序的核心，但是功能包不能直接构建，必须依赖于工作空间，一个ROS2工作空间的目录结构如下：

```
WorkSpace --- 自定义的工作空间。
    |--- build：存储中间文件的目录，该目录下会为每一个功能包创建一个单独子目录。
    |--- install：安装目录，该目录下会为每一个功能包创建一个单独子目录。
    |--- log：日志目录，用于存储日志文件。
    |--- src：用于存储功能包源码的目录。
        |-- C++功能包
            |-- package.xml：包信息，比如:包名、版本、作者、依赖项。
            |-- CMakeLists.txt：配置编译规则，比如源文件、依赖项、目标文件。
            |-- src：C++源文件目录。
            |-- include：头文件目录。
            |-- msg：消息接口文件目录。
            |-- srv：服务接口文件目录。
            |-- action：动作接口文件目录。
        |-- Python功能包
            |-- package.xml：包信息，比如:包名、版本、作者、依赖项。
            |-- setup.py：与C++功能包的CMakeLists.txt类似。
            |-- setup.cfg：功能包基本配置文件。
            |-- resource：资源目录。
            |-- test：存储测试相关文件。
            |-- 功能包同名目录：Python源文件目录。
```

另外，无论是Python功能包还是C++功能包，都可以自定义一些配置文件相关的目录。

```
|-- C++或Python功能包
    |-- launch：存储launch文件。
    |-- rviz：存储rviz2配置相关文件。
    |-- urdf：存储机器人建模文件。
    |-- params：存储参数文件。
    |-- world：存储仿真环境相关文件。
    |-- map：存储导航所需地图文件。
    |-- ......
```

上述这些目录也可以定义为其他名称，或者根据需要创建其他一些目录。

#### 2.源文件说明

在**1.3 ROS2快速体验**中，实现第一个ROS2程序时，都需要创建节点，无论是C++实现还是Python实现，都是直接实例化的Node对象。

C++实例化Node示例如下：

```cpp
#include "rclcpp/rclcpp.hpp"

int main(int argc, char ** argv)
{
  rclcpp::init(argc,argv);
  auto node = rclcpp::Node::make_shared("helloworld_node");
  RCLCPP_INFO(node->get_logger(),"hello world!");
  rclcpp::shutdown();
  return 0;
}
```

Python实例化Node示例如下：

```py
import rclpy

def main():
    rclpy.init()
    node = rclpy.create_node("helloworld_py_node")
    node.get_logger().info("hello world!")
    rclpy.shutdown()

if __name__ == '__main__':
    main()
```

但是在ROS2中，上述编码风格是不被推荐的，更推荐以继承Node的方式来创建节点对象。

C++继承Node实现示例如下：

```cpp
#include "rclcpp/rclcpp.hpp"

class MyNode: public rclcpp::Node{
public:
    MyNode():Node("node_name"){
        RCLCPP_INFO(this->get_logger(),"hello world!");
    }

};

int main(int argc, char *argv[])
{
    rclcpp::init(argc,argv);
    auto node = std::make_shared<MyNode>();
    rclcpp::shutdown();
    return 0;
}
```

Python继承Node实现示例如下：

```py
import rclpy
from rclpy.node import Node

class MyNode(Node):
    def __init__(self):
        super().__init__("node_name_py")
        self.get_logger().info("hello world!")
def main():

    rclpy.init()
    node = MyNode() 
    rclpy.shutdown()
```

之所以继承比直接实例化Node更被推荐，是因为继承方式可以在一个进程内组织多个节点，这对于提高节点间的通信效率是很有帮助的，但是直接实例化则与该功能不兼容。

#### 3.配置文件说明

在ROS2功能包中，经常需要开发者编辑一些配置文件以设置功能包的构建信息，功能包类型不同，所需修改的配置文件也有所不同。C++功能包的构建信息主要包含在package.xml与CMakeLists.txt中，Python功能包的构建信息则主要包含在package.xml和setup.py中，接下来我们就简单了解一下这些配置文件。

##### 1.package.xml

不管是何种类型的功能包，package.xml的格式都是类似的，在该文件中包含了包名、版本、作者、依赖项的信息，package.xml可以为colcon构建工具确定功能包的编译顺序。一个简单的package.xml示例如下：

```xml
<?xml version="1.0"?>
<?xml-model href="http://download.ros.org/schema/package_format3.xsd" schematypens="http://www.w3.org/2001/XMLSchema"?>
<package format="3">
  <name>pkg01_helloworld_cpp</name>
  <version>0.0.0</version>
  <description>TODO: Package description</description>
  <maintainer email="ros2@todo.todo">ros2</maintainer>
  <license>TODO: License declaration</license>

  <buildtool_depend>ament_cmake</buildtool_depend>
  <depend>rclcpp</depend>

  <test_depend>ament_lint_auto</test_depend>
  <test_depend>ament_lint_common</test_depend>

  <export>
    <build_type>ament_cmake</build_type>
  </export>
</package>
```

**1.根标签**

* &lt;package&gt;：该标签为整个xml文件的根标签，format属性用来声明文件的格式版本。

**2.元信息标签**

* &lt;name&gt;：包名；
* &lt;version&gt;：包的版本号；
* &lt;description&gt;：包的描述信息；
* &lt;maintainer&gt;：维护者信息；
* &lt;license&gt;：软件协议；
* &lt;url&gt;：包的介绍网址；
* &lt;author&gt;：包的作者信息。

**3.依赖项**

* &lt;buildtool\_depend&gt;：声明编译工具依赖；
* &lt;build\_depend&gt;：声明编译依赖；
* &lt;build\_export\_depend&gt;：声明根据此包构建库所需依赖；
* &lt;exec\_depend&gt;：声明执行时依赖；
* &lt;depend&gt;：相当于&lt;build\_depend&gt;、&lt;build\_export\_depend&gt;、&lt;exec\_depend&gt;三者的集成；
* &lt;test\_depend&gt;：声明测试依赖；
* &lt;doc\_depend&gt;：声明构建文档依赖。

##### 2.CMakeLists.txt

C++功能包中需要配置CMakeLists.txt文件，该文件描述了如何构建C++功能包，一个简单的CMakeLists.txt示例如下：

```
# 声明cmake的最低版本
cmake_minimum_required(VERSION 3.8)
# 包名，需要与package.xml中的包名一致
project(pkg01_helloworld_cpp)

if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic)
endif()

# find dependencies
find_package(ament_cmake REQUIRED)
# 引入外部依赖包
find_package(rclcpp REQUIRED)

# 映射源文件与可执行文件
add_executable(helloworld src/helloworld.cpp)
# 设置目标依赖库
ament_target_dependencies(
  helloworld
  "rclcpp"
)
# 定义安装规则
install(TARGETS helloworld
  DESTINATION lib/${PROJECT_NAME})

if(BUILD_TESTING)
  find_package(ament_lint_auto REQUIRED)
  # the following line skips the linter which checks for copyrights
  # comment the line when a copyright and license is added to all source files
  set(ament_cmake_copyright_FOUND TRUE)
  # the following line skips cpplint (only works in a git repo)
  # comment the line when this package is in a git repo and when
  # a copyright and license is added to all source files
  set(ament_cmake_cpplint_FOUND TRUE)
  ament_lint_auto_find_test_dependencies()
endif()

ament_package()
```

在示例中关于文件的使用已经通过注释给出了简短说明，其实关于CMakeLists.txt的配置是比较复杂的，后续随着学习的深入，还会给出更多的补充说明。

##### 3.setup.py

Python功能包中需要配置setup.py文件，该文件描述了如何构建Python功能包，一个简单的setup.py示例如下：

```py
from setuptools import setup

package_name = 'pkg02_helloworld_py'

setup(
    name=package_name, # 包名
    version='0.0.0',   # 版本
    packages=[package_name], # 功能包列表
    data_files=[ #需要被安装的文件以及安装路径
        ('share/ament_index/resource_index/packages',
            ['resource/' + package_name]),
        ('share/' + package_name, ['package.xml']),
    ],
    install_requires=['setuptools'], # 安装依赖
    zip_safe=True,
    maintainer='ros2', # 维护者
    maintainer_email='ros2@todo.todo', # 维护者 email
    description='TODO: Package description', # 包描述
    license='TODO: License declaration', # 软件协议
    tests_require=['pytest'], # 测试依赖
    entry_points={
        'console_scripts': [
            # 映射源文件与可执行文件
            'helloworld = pkg02_helloworld_py.helloworld:main'
        ],
    },
)
```

使用语法可参考上述示例中的注释。

#### 4.操作命令

ROS2的文件系统核心是功能包，我们可以通过编译指令`colcon`和ROS2内置的工具指令`ros2`来实现功能包的创建、编译、查找与执行等相关操作。

##### 1.创建

新建功能包语法如下：

```
ros2 pkg create 包名 --build-type 构建类型 --dependencies 依赖列表 --node-name 可执行程序名称
```

格式解释：

* --build-type：是指功能包的构建类型，有cmake、ament\_cmake、ament\_python三种类型可选；
* --dependencies：所依赖的功能包列表；
* --node-name：可执行程序的名称，会自动生成对应的源文件并生成配置文件。

##### 2.编译

编译功能包语法如下：

```
colcon build
```

或

```
colcon build --packages-select 功能包列表
```

前者会构建工作空间下的所有功能包，后者可以构建指定功能包。

##### 3.查找

在`ros2 pkg`命令下包含了多个查询功能包相关信息的参数。

```
ros2 pkg executables [包名] # 输出所有功能包或指定功能包下的可执行程序。
ros2 pkg list # 列出所有功能包
ros2 pkg prefix 包名 # 列出功能包路径
ros2 pkg xml # 输出功能包的package.xml内容
```

##### 4.执行

执行命令语法如下：

```
ros2 run 功能包 可执行程序 参数
```

> _**小提示：**_
>
> 可以通过`命令 -h`或`命令 --help`来获取命令的帮助文档。



