### 5.6.1 乌龟跟随案例以及分析

#### 1.案例需求

**需求：**编写程序实现，程序运行后会启动 turtlesim\_node 节点，该节点会生成一个窗口，窗口中有一只原生乌龟（turtle1），紧接着再生成一只新的乌龟（turtle2），无论是 turtle1 静止或是被键盘控制运动时，turtle2 都会以 turtle1 为目标并向其运动。

![](/assets/5.1.1乌龟跟随案例.gif)

#### 2.案例分析

“乌龟跟随”案例的核心是如何确定 turtle1 相对 turtle2 的位置，只要该位置确定就可以把其作为目标点并控制 turtle2 向其运动。相对位置的确定可以通过坐标变换实现，大致思路是先分别广播 turtle1 相对于 world 和 turtle2 相对于 world 的坐标系关系，然后再通过监听坐标系关系进一步获取 turtle1 相对于 turtle2 的坐标系关系。

#### 3.流程简介

案例实现主要步骤如下：

1. 编写程序调用 /spawn 服务生成一只新乌龟；
2. 编写坐标变换广播实现，通过该实现可以广播 turtle1 相对于 world 和 turtle2 相对于 world 的坐标系关系；
3. 编写坐标变换监听实现，获取 turtle1 相对于 turtle2 的坐标系关系并生成控制 turtle2 运动的速度指令；
4. 编写 launch 文件集成相关节点；
5. 编辑配置文件；
6. 编译；
7. 执行。

案例我们会采用 C++ 和 Python 分别实现，二者都遵循上述实现流程。

#### 4.准备工作

##### 1.新建功能包

终端下进入工作空间的src目录，调用如下两条命令分别创建C++功能包和Python功能包。

```
ros2 pkg create cpp05_exercise --build-type ament_cmake --dependencies rclcpp tf2 tf2_ros geometry_msgs turtlesim
ros2 pkg create py05_exercise --build-type ament_python --dependencies rclpy tf_transformations tf2_ros geometry_msgs turtlesim
```

##### 2.创建launch目录

功能包`cpp05_exercise`和`py05_exercise`下分别新建launch文件，并编辑配置文件。

功能包`cpp05_exercise`的 CMakeLists.txt 文件添加如下内容：

```
install(DIRECTORY launch
  DESTINATION share/${PROJECT_NAME}
)
```

功能包`py05_exercise`的 setup.py 文件中需要修改 data\_files 属性，修改后的内容如下：

```py
data_files=[
    ('share/ament_index/resource_index/packages',
        ['resource/' + package_name]),
    ('share/' + package_name, ['package.xml'],),
    ('share/' + package_name, glob("launch/*.launch.xml")),
    ('share/' + package_name, glob("launch/*.launch.py")),
    ('share/' + package_name, glob("launch/*.launch.yaml")),
],
```



