### 6.5.3 xacro\_练习

##### 1.需求

使用xacro创建一个四轮机器人模型，该模型底盘可以参考**6.4.4 URDF练习**中的实现，并且在底盘之上添加了相机与激光雷达。相机与激光雷达的尺寸参数、安装位置可自定义。

![](/assets/6.5.3xacro_练习.gif)

##### 2.实现分析

需求中的机器人模型是由底盘、摄像头和雷达三部分组成的，那么可以将每一部分都封装进一个xacro文件，最后再通过xacro文件包含组织成一个完整的机器人模型。

##### 3.实现

功能包cpp06\_urdf的urdf/xacro目录下，新建多个xacro文件，分别为：

* car.urdf.xacro：用于包含不同机器人部件对应的xacro文件；
* car\_base.urdf.xacro：描述机器人底盘的xacro文件；
* car\_camera.urdf.xacro：描述摄像头的xacro文件；
* car\_laser.urdf.xacro：描述雷达的xacro文件。

编辑car.urdf.xacro文件，输入如下内容：

```xml
<robot name="car" xmlns:xacro="http://wiki.ros.org/xacro">
    <xacro:include filename="car_base.urdf.xacro"/>
    <xacro:include filename="car_camera.urdf.xacro"/>
    <xacro:include filename="car_laser.urdf.xacro"/>
</robot>
```

编辑car\_base.urdf.xacro文件，输入如下内容：

```xml
<robot xmlns:xacro="http://wiki.ros.org/xacro">
    <!-- PI 值 -->
    <xacro:property name="PI" value="3.1416"/>
    <!-- 定义车辆参数 -->
    <!-- 车体长宽高 -->
    <xacro:property name="base_link_x" value="0.2"/>
    <xacro:property name="base_link_y" value="0.12"/>
    <xacro:property name="base_link_z" value="0.07"/>
    <!-- 离地间距 -->
    <xacro:property name="distance" value="0.015"/>
    <!-- 车轮半径 宽度 -->
    <xacro:property name="wheel_radius" value="0.025"/>
    <xacro:property name="wheel_length" value="0.02"/>

    <!-- 定义颜色 -->
    <material name="yellow">
        <color rgba="0.7 0.7 0 0.8" />
    </material>
    <material name="red">
        <color rgba="0.8 0.1 0.1 0.8" />
    </material>
    <material name="gray">
        <color rgba="0.2 0.2 0.2 0.95" />
      </material>
    <!-- 定义 base_footprint -->
    <link name="base_footprint">
        <visual>
            <geometry>
                <sphere radius="0.001"/>
            </geometry>
        </visual>
    </link>

    <!-- 定义 base_link -->
    <link name="base_link">
        <visual>
            <!-- 形状 -->
            <geometry>
                <box size="${base_link_x} ${base_link_y} ${base_link_z}" />
            </geometry>
            <origin xyz="0 0 0" rpy="0 0 0" />
            <material name="yellow"/>
        </visual>
    </link>
    <joint name="baselink2basefootprint" type="fixed">
        <parent link="base_footprint"/>
        <child link="base_link"/>
        <origin xyz="0.0 0.0 ${distance + base_link_z / 2}"/>
    </joint>
    <!-- 车轮宏定义 -->
    <xacro:macro name="wheel_func" params="wheel_name is_front is_left" >
        <link name="${wheel_name}_wheel">
            <visual>
                <geometry>
                    <cylinder radius="${wheel_radius}" length="${wheel_length}" />
                </geometry>
                <origin xyz="0 0 0" rpy="${PI / 2} 0 0" />
                <material name="gray"/>
            </visual>
        </link>
        <joint name="${wheel_name}2baselink" type="continuous">
            <parent link="base_link"  />
            <child link="${wheel_name}_wheel" />
            <origin xyz="${(base_link_x / 2 - wheel_radius) * is_front} ${base_link_y / 2 * is_left} ${(base_link_z / 2 + distance - wheel_radius) * -1}" rpy="0 0 0" />
            <axis xyz="0 1 0" />
        </joint>
    </xacro:macro>
    <!-- 车轮宏调用 -->
    <xacro:wheel_func wheel_name="left_front" is_front="1" is_left="1" />
    <xacro:wheel_func wheel_name="left_back" is_front="-1" is_left="1" />
    <xacro:wheel_func wheel_name="right_front" is_front="1" is_left="-1" />
    <xacro:wheel_func wheel_name="right_back" is_front="-1" is_left="-1" />
</robot>
```

编辑car\_camera.urdf.xacro文件，输入如下内容：

```xml
<!-- 摄像头相关的 xacro 文件 -->
<robot xmlns:xacro="http://wiki.ros.org/xacro">
    <!-- 摄像头属性 -->
    <xacro:property name="camera_x" value="0.012" /> <!-- 摄像头长度(x) -->
    <xacro:property name="camera_y" value="0.05" /> <!-- 摄像头宽度(y) -->
    <xacro:property name="camera_z" value="0.01" /> <!-- 摄像头高度(z) -->
    <xacro:property name="camera_joint_x" value="${base_link_x / 2 - camera_x / 2}" /> <!-- 摄像头安装的x坐标 -->
    <xacro:property name="camera_joint_y" value="0.0" /> <!-- 摄像头安装的y坐标 -->
    <xacro:property name="camera_joint_z" value="${base_link_z / 2 + camera_z / 2}" /> <!-- 摄像头安装的z坐标:底盘高度 / 2 + 摄像头高度 / 2  -->

    <!-- 摄像头关节以及link -->
    <link name="camera">
        <visual>
            <geometry>
                <box size="${camera_x} ${camera_y} ${camera_z}" />
            </geometry>
            <origin xyz="0.0 0.0 0.0" rpy="0.0 0.0 0.0" />
            <material name="red" />
        </visual>
    </link>

    <joint name="camera2baselink" type="fixed">
        <parent link="base_link" />
        <child link="camera" />
        <origin xyz="${camera_joint_x} ${camera_joint_y} ${camera_joint_z}" />
    </joint>
</robot>
```

编辑car\_laser.urdf.xacro文件，输入如下内容：

```xml
<!--
    小车底盘添加雷达
-->
<robot xmlns:xacro="http://wiki.ros.org/xacro">

    <material name="blue">
        <color rgba="0.0 0.0 0.4 0.95" />
    </material>

    <!-- 雷达属性 -->
    <xacro:property name="laser_length" value="0.03" /> <!-- 雷达长度 -->
    <xacro:property name="laser_radius" value="0.03" /> <!-- 雷达半径 -->
    <xacro:property name="laser_joint_x" value="0.0" /> <!-- 雷达安装的x坐标 -->
    <xacro:property name="laser_joint_y" value="0.0" /> <!-- 雷达安装的y坐标 -->
    <xacro:property name="laser_joint_z" value="${base_link_z / 2 + laser_length / 2}" /> <!-- 雷达安装的z坐标:车体高度 / 2 + 雷达高度 / 2  -->

    <!-- 雷达关节以及link -->
    <link name="laser">
        <visual>
            <geometry>
                <cylinder radius="${laser_radius}" length="${laser_length}" />
            </geometry>
            <origin xyz="0.0 0.0 0.0" rpy="0.0 0.0 0.0" />
            <material name="blue" />
        </visual>
    </link>

    <joint name="laser2baselink" type="fixed">
        <parent link="base_link" />
        <child link="laser" />
        <origin xyz="${laser_joint_x} ${laser_joint_y} ${laser_joint_z}" />
    </joint>
</robot>
```

编译后，工作空间终端下调用如下命令执行：

    ros2 launch cpp06_urdf display.launch.py model:=`ros2 pkg prefix --share cpp06_urdf`/urdf/xacro/car.urdf.xacro

命令执行后，rviz2 中可以显示与需求类似的机器人模型。

