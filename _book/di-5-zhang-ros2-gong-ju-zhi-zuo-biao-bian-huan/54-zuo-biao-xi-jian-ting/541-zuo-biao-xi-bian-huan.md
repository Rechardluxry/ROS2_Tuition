### 5.4.1 坐标系变换案例以及分析

#### 1.案例需求

**案例1：**在**5.3 坐标变换广播**中发布了laser相对于base\_link和camra相对于base\_link的坐标系关系，请求解laser相对于camera的坐标系关系。

![](/assets/5.4.1案例演示1.gif)

**案例2：**在**5.3 坐标变换广播**中发布了laser相对于base\_link的坐标系关系且发布了laser坐标系下的障碍物的坐标点数据，请求解base\_link坐标系下该障碍物的坐标。

![](/assets/5.4.1案例演示2.gif)

#### 2.案例分析

在上述案例中，案例1是多坐标系的场景下实现不同坐标系之间的变换，案例2则是要实现同一坐标点在不同坐标系下的变换，虽然需求不同，但是相关算法都被封装好了，我们只需要调用相关 API 即可。

#### 3.流程简介

两个案例的实现流程类似，主要步骤如下：

1. 编写坐标变换程序实现；
2. 编辑配置文件；
3. 编译；
4. 执行。

案例我们会采用 C++ 和 Python 分别实现，二者都遵循上述实现流程。

#### 4.准备工作

终端下进入工作空间的src目录，调用如下两条命令分别创建C++功能包和Python功能包。

```
ros2 pkg create cpp04_tf_listener --build-type ament_cmake --dependencies rclcpp tf2 tf2_ros geometry_msgs
ros2 pkg create py04_tf_listener --build-type ament_python --dependencies rclpy tf_transformations tf2_ros geometry_msgs
```



