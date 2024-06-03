## 3.8 通信机制实操（期中大作业）

本节主要介绍通信机制相关的一些练习，这些练习基于turtlesim功能包，练习类型覆盖了话题、服务、动作、参数这四种通信机制。

#### 准备

终端下进入工作空间的src目录，调用如下命令创建C++功能包。

```
ros2 pkg create cpp07_exercise --build-type ament_cmake --dependencies rclcpp turtlesim base_interfaces_demo geometry_msgs rclcpp_action
```

功能包下新建launch目录以备用。

