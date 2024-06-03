### 5.3.2 静态广播器（命令）

#### 1.静态广播器工具

在 `tf2_ros`功能包中提供了一个名为`static_transform_publisher`的可执行文件，通过该文件可以直接广播静态坐标系关系，其使用语法如下。

**格式1：**

使用以米为单位的 x/y/z 偏移量和以弧度为单位的roll/pitch/yaw（可直译为滚动/俯仰/偏航，分别指的是围绕 x/y/z 轴的旋转）向 tf2 发布静态坐标变换。

```
ros2 run tf2_ros static_transform_publisher --x x --y y --z z --yaw yaw --pitch pitch --roll roll --frame-id frame_id --child-frame-id child_frame_id
```

**格式2：**

使用以米为单位的 x/y/z 偏移量和 qx/qy/qz/qw 四元数向 tf2 发布静态坐标变换。

```
ros2 run tf2_ros static_transform_publisher --x x --y y --z z --qx qx --qy qy --qz qz --qw qw --frame-id frame_id --child-frame-id child_frame_id
```

注意：在上述两种格式中除了用于表示父级坐标系的`--frame-id`和用于表示子级坐标系的`--child-frame-id`之外，其他参数都是可选的，如果未指定特定选项，那么将直接使用默认值。

#### 2.静态广播器工具使用

打开两个终端，终端1输入如下命令发布雷达（laser）相对于底盘（base\_link）的静态坐标变换：

```
ros2 run tf2_ros static_transform_publisher --x 0.4 --y 0 --z 0.2 --yaw 0 --roll 0 --pitch 0 --frame-id base_link --child-frame-id laser
```

终端2输入如下命令发布摄像头（camera）相对于底盘（base\_link）的静态坐标变换：

```
ros2 run tf2_ros static_transform_publisher --x -0.5 --y 0 --z 0.4 --yaw 0 --roll 0 --pitch 0 --frame-id base_link --child-frame-id camera
```

#### 3.rviz2 查看坐标系关系

新建终端，通过命令`rviz2`打开 rviz2 并配置相关插件查看坐标变换消息：

1. 将 Global Options 中的 Fixed Frame 设置为 base\_link；
2. 点击 add 按钮添加 TF 插件；
3. 勾选 TF 插件中的 show names。

右侧 Grid 中将以图形化的方式显示坐标变换关系。

![](/assets/5.3.2rviz2查看坐标关系.gif)



