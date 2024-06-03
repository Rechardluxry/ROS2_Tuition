### 2.4.1 案例以及案例分析

#### 1.案例需求

**需求：**编写动作通信，动作客户端提交一个整型数据N，动作服务端接收请求数据并累加1-N之间的所有整数，将最终结果返回给动作客户端，且每累加一次都需要计算当前运算进度并反馈给动作客户端。![](/assets/2.4.1案例演示.gif)

#### 2.案例分析

在上述案例中，需要关注的要素有三个：

1. 动作客户端；
2. 动作服务端；
3. 消息载体。

#### 3.流程简介

案例实现前需要先自定义动作接口，接口准备完毕后，动作通信实现主要步骤如下：

1. 编写动作服务端实现；
2. 编写动作客户端实现；
3. 编辑配置文件；
4. 编译；
5. 执行。

案例我们会采用C++和Python分别实现，二者都遵循上述实现流程。

#### 4.准备工作

终端下进入工作空间的src目录，调用如下两条命令分别创建C++功能包和Python功能包。

```
ros2 pkg create cpp03_action --build-type ament_cmake --dependencies rclcpp rclcpp_action base_interfaces_demo
ros2 pkg create py03_action --build-type ament_python --dependencies rclpy base_interfaces_demo
```



