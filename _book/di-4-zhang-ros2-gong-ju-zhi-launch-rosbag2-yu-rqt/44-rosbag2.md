## 4.4 录制回放工具——rosbag2

#### 场景

机器人传感器获取到的信息，有时我们可能需要时时处理，有时可能只是采集数据，事后分析，比如:

> 机器人导航实现中，可能需要绘制导航所需的全局地图，地图绘制实现，有两种方式，方式1：可以控制机器人运动，将机器人传感器感知到的数据时时处理，生成地图信息。方式2：同样是控制机器人运动，将机器人传感器感知到的数据留存，事后，再重新读取数据，生成地图信息。两种方式比较，显然方式2使用上更为灵活方便。

在ROS2 中关于数据的留存以及读取实现，提供了专门的工具: rosbag2。

#### 概念

是用于录制和回放话题的一个工具集。

#### **作用** {#作用}

实现了数据的复用，方便调试、测试。

#### 案例

录制并读取数据。

实现步骤：

1. 序列化；
2. 反序列化；
3. 编译执行。

准备：

终端下进入工作空间的src目录，调用如下两条命令分别创建C++功能包和Python功能包。

```
ros2 pkg create cpp02_rosbag --build-type ament_cmake --dependencies rclcpp rosbag2_cpp geometry_msgs
ros2 pkg create py02_rosbag --build-type ament_python --dependencies rclpy rosbag2_py geometry_msgs
```



