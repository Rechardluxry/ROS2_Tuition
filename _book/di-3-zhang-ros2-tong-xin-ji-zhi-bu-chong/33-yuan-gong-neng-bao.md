## 3.3 元功能包 {#41-ros元功能包}

#### 场景

> 完成一个系统性的功能，可能涉及到多个功能包，比如实现了机器人导航模块，该模块下有地图、定位、路径规划...等不同的子级功能包。那么调用者安装该模块时，需要逐一的安装每一个功能包吗？

显而易见的，逐一安装功能包的效率低下，在ROS2中，提供了一种方式可以将不同的功能包打包成一个功能包，当安装某个功能模块时，直接调用打包后的功能包即可，该包又称之为元功能包\(metapackage\)。

#### 概念

MetaPackage是Linux的一个文件管理系统的概念。是 ROS2 中的一个虚包，里面没有实质性的内容，但是它依赖了其他的软件包，通过这种方法可以把其他包组合起来，我们可以认为它是一本书的目录索引，告诉我们这个包集合中有哪些子包，并且该去哪里下载。

例如：

* sudo apt install ros-&lt;ros2-distro&gt;-desktop 命令安装 ros2 时就使用了元功能包，该元功能包依赖于 ROS2 中的其他一些功能包，安装该包时会一并安装依赖。

#### 作用 {#作用}

方便用户的安装，我们只需要这一个包就可以把其他相关的软件包组织到一起安装了。

#### 实现 {#实现}

1.新建一个功能包

```
ros2 pkg create tutorails_plumbing
```

2.修改 package.xml 文件，添加执行时所依赖的包：

```xml
<?xml version="1.0"?>
<?xml-model href="http://download.ros.org/schema/package_format3.xsd" schematypens="http://www.w3.org/2001/XMLSchema"?>
<package format="3">
  <name>tutorails_plumbing</name>
  <version>0.0.0</version>
  <description>TODO: Package description</description>
  <maintainer email="ros2@todo.todo">ros2</maintainer>
  <license>TODO: License declaration</license>

  <buildtool_depend>ament_cmake</buildtool_depend>

  <exec_depend>base_interfaces_demo</exec_depend>
  <exec_depend>cpp01_topic</exec_depend>
  <exec_depend>cpp02_service</exec_depend>
  <exec_depend>cpp03_action</exec_depend>
  <exec_depend>cpp04_param</exec_depend>
  <exec_depend>py01_topic</exec_depend>
  <exec_depend>py02_service</exec_depend>
  <exec_depend>py03_action</exec_depend>
  <exec_depend>py04_param</exec_depend>


  <export>
    <build_type>ament_cmake</build_type>
  </export>
</package>
```

3.文件CMakeLists.txt内容如下:

```
cmake_minimum_required(VERSION 3.8)
project(tutorails_plumbing)

if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic)
endif()

find_package(ament_cmake REQUIRED)

ament_package()
```



