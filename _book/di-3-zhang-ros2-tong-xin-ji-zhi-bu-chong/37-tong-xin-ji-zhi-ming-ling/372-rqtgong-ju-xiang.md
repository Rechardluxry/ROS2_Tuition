### 3.7.2 rqt工具箱

本节主要介绍ROS2中rqt工具箱的使用，比如：rqt的安装、启动与插件使用等。

#### 1.安装 {#1安装}

* 一般只要安装的是desktop版本就会默认安装rqt工具箱；

* 如果需要安装可以以如下方式安装

  ```
  $ sudo apt install ros-humble-rqt*
  ```

#### 2.启动 {#2启动}

常用的`rqt`启动命令有：

* 方式1：`rqt`

* 方式2：`ros2 run rqt_gui rqt_gui`

#### 3.插件使用 {#3基本使用}

启动rqt之后，可以通过plugins添加所需的插件：

![](/assets/3.7.2RQT工具箱.gif)

在plugins中包含了话题、服务、动作、参数、日志等等相关的插件，我们可以按需选用，方便的实现ROS2程序调试。使用示例如下。

**1.topic 插件**

添加topic插件并发送速度指令控制乌龟运动。

![](/assets/3.7.2RQT工具箱topic.gif)

**2.service插件**

添加 service 插件并发送请求，在制定位置生成一只乌龟。

![](/assets/3.7.2RQT工具箱service.gif)

**3.参数插件**

通过参数插件动态修改乌龟窗体背景颜色。

![](/assets/3.7.2RQT工具箱param.gif)

