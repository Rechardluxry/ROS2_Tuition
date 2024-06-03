### 2.3.1 案例以及案例分析

#### 1.案例需求

**需求：**编写服务通信，客户端可以提交两个整数到服务端，服务端接收请求并解析两个整数求和，然后将结果响应回客户端。

![](/assets/2.3案例演示.gif)

#### 2.案例分析

在上述案例中，需要关注的要素有三个：

1. 客户端；
2. 服务端；
3. 消息载体。

#### 3.流程简介

案例实现前需要先自定义服务接口，接口准备完毕后，服务实现主要步骤如下：

1. 编写服务端实现；
2. 编写客户端实现；
3. 编辑配置文件；
4. 编译；
5. 执行。

案例我们会采用C++和Python分别实现，二者都遵循上述实现流程。

#### 4.准备工作

终端下进入工作空间的src目录，调用如下两条命令分别创建C++功能包和Python功能包。

```
ros2 pkg create cpp02_service --build-type ament_cmake --dependencies rclcpp base_interfaces_demo
ros2 pkg create py02_service --build-type ament_python --dependencies rclpy base_interfaces_demo
```



