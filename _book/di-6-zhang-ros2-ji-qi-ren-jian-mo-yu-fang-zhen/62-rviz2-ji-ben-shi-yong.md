## 6.2 rviz2 基本使用

#### 1.安装

以`sudo apt install ros-[ROS_DISTRO]-desktop`格式安装ROS2时，RViz已经默认被安装了。

_如果rviz没有安装，请调用如下命令自行安装:_

```
sudo apt install ros-[ROS_DISTRO]-rviz2
```

**备注：**命令中的 \[ROS\_DISTRO\] 指代ROS2版本。

#### 2.启动

rviz2的启动方式比较简单，常用启动命令有两种。

方式1：`rviz2`；

方式2：`ros2 run rviz2 rviz2`。

#### 3.界面布局

**rviz2 启动之后，默认界面如下：**![](/assets/6.2rviz2界面简介.PNG)

1. 上部为工具栏：包括视角控制、预估位姿设置、目标设置等，还可以添加自定义插件；
2. 左侧为插件显示区：包括添加、删除、复制、重命名插件，显示插件，以及设置插件属性等功能；
3. 中间为3D试图显示区：以可视化的方式显示添加的插件信息；
4. 右侧为观测视角设置区：可以设置不同的观测视角；
5. 下侧为时间显示区：包括系统时间和ROS时间。

**左侧插件显示区默认有两个插件：**

![](/assets/6.2rviz2界面左侧.PNG)

* Global Options：该插件用于设置全局显示相关的参数，一般情况下，需要自行设置的是 Fixed Frame 选项，该选项是其他所有数据在可视化显示时所参考的全局坐标系；
* Global Status：该插件用于显示在 Global Options 设置完毕 Fixed Frame 之后，所有的坐标变换是否正常。

#### 4.rviz2 中的预定义插件

在 rviz2 中已经预定义了一些插件，这些插件名称、功能以及订阅的消息类型如下：

| **名称** | **功能** | **消息类型** |
| :--- | :--- | :--- |
| [Axes](https://wiki.ros.org/rviz/DisplayTypes/Axes) | 显示 rviz2 默认的坐标系。 |  |
| [Camera](https://wiki.ros.org/rviz/DisplayTypes/Camera) | 显示相机图像，必须需要使用消息：CameraInfo。 | sensor\_msgs/msg/Image，sensor\_msgs/msg/CameraInfo |
| [Grid](https://wiki.ros.org/rviz/DisplayTypes/Grid) | 显示以参考坐标系原点为中心的网格。 |  |
| [Grid Cells](https://wiki.ros.org/rviz/DisplayTypes/GridCells) | 从网格中绘制单元格，通常是导航堆栈中成本地图中的障碍物。 | nav\_msgs/msg/GridCells |
| [Image](https://wiki.ros.org/rviz/DisplayTypes/Image) | 显示相机图像，但是和Camera插件不同，它不需要使用 CameraInfo 消息。 | sensor\_msgs/msg/Image |
| [InteractiveMarker](https://wiki.ros.org/rviz/DisplayTypes/InteractiveMarker) | 显示来自一个或多个交互式标记服务器的 3D 对象，并允许与它们进行鼠标交互。 | visualization\_msgs/msg/InteractiveMarker |
| [Laser Scan](https://wiki.ros.org/rviz/DisplayTypes/LaserScan) | 显示激光雷达数据。 | sensor\_msgs/msg/LaserScan |
| [Map](https://wiki.ros.org/rviz/DisplayTypes/Map) | 显示地图数据。 | nav\_msgs/msg/OccupancyGrid |
| [Markers](https://wiki.ros.org/rviz/DisplayTypes/Marker) | 允许开发者通过主题显示任意原始形状的几何体。 | visualization\_msgs/msg/Marker，visualization\_msgs/msg/MarkerArray |
| [Path](https://wiki.ros.org/rviz/DisplayTypes/Path) | 显示机器人导航中的路径相关数据。 | nav\_msgs/msg/Path |
| [PointStamped](https://wiki.ros.org/rviz/DisplayTypes/Point) | 以小球的形式绘制一个点。 | geometry\_msgs/msg/PointStamped |
| [Pose](https://wiki.ros.org/rviz/DisplayTypes/Pose) | 以箭头或坐标轴的方式绘制位姿。 | geometry\_msgs/msg/PoseStamped |
| [Pose Array](https://wiki.ros.org/rviz/DisplayTypes/PoseArray) | 绘制一组 Pose。 | geometry\_msgs/msg/PoseArray |
| [Point Cloud2](https://wiki.ros.org/rviz/DisplayTypes/PointCloud) | 绘制点云数据。 | sensor\_msgs/msg/PointCloud，sensor\_msgs/msg/PointCloud2 |
| [Polygon](https://wiki.ros.org/rviz/DisplayTypes/Polygon) | 将多边形的轮廓绘制为线。 | geometry\_msgs/msg/Polygon |
| [Odometry](https://wiki.ros.org/rviz/DisplayTypes/Odometry) | 显示随着时间推移累积的里程计消息。 | nav\_msgs/msg/Odometry |
| [Range](https://wiki.ros.org/rviz/DisplayTypes/Range) | 显示表示来自声纳或红外距离传感器的距离测量值的圆锥。 | sensor\_msgs/msg/Range |
| [RobotModel](https://wiki.ros.org/rviz/DisplayTypes/RobotModel) | 显示机器人模型。 |  |
| [TF](https://wiki.ros.org/rviz/DisplayTypes/TF) | 显示 tf 变换层次结构。 |  |
| [Wrench](https://wiki.ros.org/rviz/DisplayTypes/Wrench) | 将geometry\_msgs /WrenchStamped消息显示为表示力的箭头和表示扭矩的箭头加圆圈。 | geometry\_msgs/msg/WrenchStamped |
| [Oculus](https://wiki.ros.org/oculus_rviz_plugins) | 将 RViz 场景渲染到 Oculus 头戴设备。 |  |

上述每一种插件又包含了诸多属性，可以通过设置插件属性来控制插件的最终显示效果。

#### 5.示例

下图是关于 rviz2 插件使用的示例，在该示例中 rviz2 集成了 TF、Laser Scan、Image 等插件，通过这些插件我们可以将一些肉眼不可见的数据以可视化的方式展现出来，以机器人的视角来看世界。![](/assets/6.2rviz2基本使用示例.png)

