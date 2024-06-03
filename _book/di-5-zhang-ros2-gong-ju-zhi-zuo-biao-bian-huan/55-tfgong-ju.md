## 5.5 坐标变换工具

在 ROS2 的 TF 框架中除了封装了坐标广播与订阅功能外，还提供了一些工具，可以帮助我们提高开发、调试效率。本节主要介绍这些工具的使用，这些工具主要涉及到两个功能包：`tf2_ros`和`tf2_tools`。

`tf2_ros`包中提供的常用节点如下：

* static\_transform\_publisher：该节点用于广播静态坐标变换；

* tf2\_monitor：该节点用于打印所有或特定坐标系的发布频率与网络延迟；

* tf2\_echo：该节点用于打印特定坐标系的平移旋转关系。

`tf2_tools`包中提供的节点如下：

* view\_frames：该节点可以生成显示坐标系关系的 pdf 文件，该文件包含树形结构的坐标系图谱。

上述诸多工具中，功能包`tf2_ros`中的`static_transform_publisher`节点在 **5.3.2 静态广播器（命令）**一节中已有详细说明，本节不再介绍。

**准备工作：**

为了更好的演示工具的使用，请先启动若干坐标系广播节点，比如：可以按照**5.3.2 静态广播器（命令）**和 **5.3.5 动态广播器（C++）**广播一些坐标系消息。

#### 1.tf2\_monitor

##### 1.1打印所有坐标系的发布频率与网络延迟

终端执行命令：

```
ros2 run tf2_ros tf2_monitor
```

运行结果：

![](/assets/5.5tf2_monitor.PNG)

##### 1.2打印指定坐标系的发布频率与网络延迟

终端执行命令：

```
ros2 run tf2_ros tf2_monitor camera laser
```

运行结果：

![](/assets/5.5tf2_monitor2.PNG)

#### 2.tf2\_echo

打印两个坐标系的平移旋转关系。

终端执行命令：

```
ros2 run tf2_ros tf2_echo world turtle1
```

运行结果：

![](/assets/5.5tf2_echo.PNG)

#### 3.view\_frames

以图形化的方式显示坐标系关系。

终端执行命令：

```
ros2 run tf2_tools view_frames
```

运行结果：将会生成 frames\_xxxx.gv 与 frames\_xxxx.pdf 文件，其中 xxxx 为时间戳。打开 pdf 文件显示如下内容：

![](/assets/5.5view_frames.PNG)

