### 2.2.1 案例以及案例分析

#### 1.案例需求

**需求1：**编写话题通信实现，发布方以某个频率发布一段文本，订阅方订阅消息，并输出在终端。

![](/assets/2.2.1案例1演示.gif)

**需求2：**编写话题通信实现，发布方以某个频率发布自定义接口消息，订阅方订阅消息，并输出在终端。

![](/assets/2.2.1案例2演示.gif)

#### 2.案例分析

在上述案例中，需要关注的要素有三个：

1. 发布方；
2. 订阅方；
3. 消息载体。

案例1和案例2的主要区别在于消息载体，前者可以使用原生的数据类型，后者需要自定义接口消息。

#### 3.流程简介

案例2需要先自定义接口消息，除此之外的实现流程与案例1一致，主要步骤如下：

1. 编写发布方实现；
2. 编写订阅方实现；
3. 编辑配置文件；
4. 编译；
5. 执行。

案例我们会采用C++和Python分别实现，二者都遵循上述实现流程。

#### 4.准备工作

终端下进入工作空间的src目录，调用如下两条命令分别创建C++功能包和Python功能包。

```
ros2 pkg create cpp01_topic --build-type ament_cmake --dependencies rclcpp std_msgs base_interfaces_demo
ros2 pkg create py01_topic --build-type ament_python --dependencies rclpy std_msgs base_interfaces_demo
```



