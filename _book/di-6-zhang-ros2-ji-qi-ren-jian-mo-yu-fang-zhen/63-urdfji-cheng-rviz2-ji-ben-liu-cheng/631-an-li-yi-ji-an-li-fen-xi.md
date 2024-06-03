### 6.3.1 案例以及案例分析

#### 1.案例需求

**需求：**在 rviz2 中显示一个简单的盒状机器人。![](/assets/6.3.1盒状机器人.PNG)

#### 2.案例分析

在上述案例中，需要关注的要素有三个：

1. 如何编写 URDF 文件；
2. 如何加载 URDF 文件到系统；
3. 如何使用 rviz2 显示机器人模型；

#### 3.流程简介

主要步骤如下：

1. 编写 URDF 文件；
2. 编写 launch 文件；
3. 编辑配置文件；
4. 编译；
5. 执行 launch 文件并在 rviz2 加载机器人模型。

该案例中，C++ 与 Python 实现都遵顼上述流程，且大多数实现都基本一致，只是在配置文件上稍有差异，本节主要以 C++ 功能包为例演示该实现。

#### 4.准备工作

##### 1.安装所需功能包

请调用如下命令，安装案例所需的两个功能包：

```
sudo apt install ros-humble-joint-state-publisher
sudo apt install ros-humble-joint-state-publisher-gui
```

##### 2.新建功能包

终端下进入工作空间的src目录，调用如下命令创建C++功能包。

```
ros2 pkg create cpp06_urdf --build-type ament_cmake
```

功能包下新建 urdf、rviz、launch、meshes目录以备用，其中 urdf 目录下再新建子目录 urdf 与 xacro，分别用于存储 urdf 文件和 xacro 文件。

