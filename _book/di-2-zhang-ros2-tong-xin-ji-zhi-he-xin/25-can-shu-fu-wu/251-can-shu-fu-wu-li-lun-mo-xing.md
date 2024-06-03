### 2.5.1 案例以及案例分析

#### 1.案例需求

**需求：**在参数服务端设置一些参数，参数客户端访问服务端并操作这些参数。

![](/assets/2.5.1案例演示.gif)

#### 2.案例分析

在上述案例中，需要关注的要素有三个：

1. 参数客户端；
2. 参数服务端；
3. 参数。

#### 3.流程简介

案例实现前需要先了解ROS2中参数的相关API，无论是客户端还是服务端都会使用到参数，而参数服务案例实现主要步骤如下：

1. 编写参数服务端实现；
2. 编写参数客户端实现；
3. 编辑配置文件；
4. 编译；
5. 执行。

案例我们会采用C++和Python分别实现，二者都遵循上述实现流程。

#### 4.准备工作

终端下进入工作空间的src目录，调用如下两条命令分别创建C++功能包和Python功能包。

```
ros2 pkg create cpp04_param --build-type ament_cmake --dependencies rclcpp
ros2 pkg create py04_param --build-type ament_python --dependencies rclpy
```



