### 6.4.3 URDF语法03\_joint

#### 1.简介 {#joint}

urdf 中的 joint 标签用于描述机器人关节的运动学和动力学属性，还可以指定关节运动的安全极限，机器人的两个部件\(分别称之为 parent link 与 child link\)以”关节“的形式相连接，不同的关节有不同的运动形式: 旋转、滑动、固定、旋转速度、旋转角度限制....,比如:安装在底座上的轮子可以360度旋转，而摄像头则可能是完全固定在底座上。

![](/assets/6.4.3urdf_joint.png)

#### 2.属性

* **name**（必填）：为关节命名，名称需要唯一。

* **type**（必填）：设置关节类型，可用类型如下：

  * continuous：旋转关节，可以绕单轴无限旋转。

  * revolute：旋转关节，类似于 continues，但是有旋转角度限制。

  * prismatic：滑动关节，沿某一轴线移动的关节，有位置极限。

  * planer：平面关节，允许在平面正交方向上平移或旋转。

  * floating：浮动关节，允许进行平移、旋转运动。

  * fixed：固定关节，不允许运动的特殊关节。

#### 3.子标签 {#2子标签}

* **&lt;parent&gt;**（必填）：指定父级link。

  * **link**（必填）：父级link的名字，是这个link在机器人结构树中的名字。

* **&lt;child&gt;**（必填）：指定子级link。

  * **link**（必填）：子级link的名字，是这个link在机器人结构树中的名字。

* **&lt;origin&gt;**（可选）：这是从父link到子link的转换，关节位于子link的原点。

  * **xyz**：各轴线上的偏移量。
  * **rpy**：各轴线上的偏移弧度。

* **&lt;axis&gt;**（可选）：如不设置，默认值为（1，0，0）。

  * **xyz**：用于设置围绕哪个关节轴运动。

* **&lt;calibration&gt;**（可选）：关节的参考位置，用于校准关节的绝对位置。

  * **rising**（可选）：当关节向正方向移动时，该参考位置将触发上升沿。

  * **falling**（可选）：当关节向正方向移动时，该参考位置将触发下降沿。

* **&lt;dynamics&gt;**（可选）：指定接头物理特性的元素。这些值用于指定关节的建模属性，对仿真较为有用。

  * **damping**（可选）：关节的物理阻尼值，默认为0。

  * **friction**（可选）：关节的物理静摩擦值，默认为0。

* **&lt;limit&gt;**（关节类型是revolute或prismatic时为必须的）：

  * **lower**（可选）：指定关节下限的属性（旋转关节以弧度为单位，棱柱关节以米为单位）。如果关节是连续的，则省略。

  * **upper**（可选）：指定关节上限的属性（旋转关节以弧度为单位，棱柱关节以米为单位）。如果关节是连续的，则省略。

  * **effort**（必填）：指定关节可受力的最大值。

  * **velocity**（必填）：用于设置最大关节速度（旋转关节以弧度每秒 \[rad/s\] 为单位，棱柱关节以米每秒 \[m/s\] 为单位）。

* **&lt;mimic&gt;**（可选）：此标签用于指定定义的关节模仿另一个现有关节。该关节的值可以计算为_value = multiplier \* other\_joint\_value + offset_。

  * **joint**（必填）：指定要模拟的关节的名称。

  * **multiplier**（可选）：指定上述公式中的乘法因子。

  * **offset**（可选）：指定要在上述公式中添加的偏移量，默认为 0（旋转关节的单位是弧度，棱柱关节的单位是米）。

* **&lt;safety\_controller&gt;**（可选）：安全控制器。

  * **soft\_lower\_limit**（可选）：指定安全控制器开始限制关节位置的下关节边界，此限制需要大于joint下限。

  * **soft\_upper\_limit**（可选）：指定安全控制器开始限制关节位置的关节上边界的属性，此限制需要小于joint上限。

  * **k\_position**（可选）：指定位置和速度限制之间的关系。

  * **k\_velocity**（必填）：指定力和速度限制之间的关系。

#### 4.示例 {#3案例}

##### 1.需求

创建机器人模型，底盘为长方体，在长方体的前面添加一摄像头，摄像头可以沿着 Z 轴 360 度旋转。

![](/assets/6.4.3urdf_joint.gif)

##### 2.实现

功能包 cpp06\_urdf 的 urdf/urdf 目录下，新建 urdf 文件 demo03\_joint.urdf，并编辑文件，输入如下内容：

```xml
<!-- 
    需求：创建机器人模型，底盘为长方体，
         在长方体的前面添加一摄像头，
         摄像头可以沿着 Z 轴 360 度旋转

 -->
 <robot name="joint_demo">
  <!-- 定义颜色 -->
  <material name="yellow">
    <color rgba="0.7 0.7 0 0.8" />
  </material>
  <material name="red">
    <color rgba="0.8 0.1 0.1 0.8" />
  </material>
  <link name="base_link">
    <visual>
        <!-- 形状 -->
        <geometry>
            <box size="0.5 0.3 0.1" />
        </geometry>
        <origin xyz="0 0 0" rpy="0 0 0" />
        <material name="yellow"/>
    </visual>
  </link>

  <!-- 摄像头 -->
  <link name="camera">
      <visual>
          <geometry>
              <box size="0.02 0.05 0.05" />
          </geometry>
          <origin xyz="0 0 0" rpy="0 0 0" />
          <material name="red" />
      </visual>
  </link>

  <!-- 关节 -->
  <joint name="camera2baselink" type="continuous">
      <parent link="base_link"/>
      <child link="camera" />
      <!-- 需要计算两个 link 的物理中心之间的偏移量 -->
      <origin xyz="0.2 0 0.075" rpy="0 0 0" />
      <axis xyz="0 0 1" />
  </joint>

</robot>
```

编译后，工作空间终端下调用如下命令执行：

    ros2 launch cpp06_urdf display.launch.py model:=`ros2 pkg prefix --share cpp06_urdf`/urdf/urdf/demo03_joint.urdf

执行指令后，在 rviz2 中会显示机器人模型。

然后再新建终端，执行如下命令：

```
ros2 run joint_state_publisher_gui joint_state_publisher_gui
```

执行指令后，会弹出一个新的窗口，在该窗口中有一个”进度条“，通过拖拽进度条可以控制相机旋转。

#### 5.使用base\_footprint优化urdf {#4basefootprint优化urdf}

##### 1.需求

前面实现的机器人模型是半沉到地下的，因为默认情况下: 底盘的中心点位于地图原点上，所以会导致这种情况产生，可以使用的优化策略，将初始 link 设置为一个尺寸极小的 link\(比如半径为 0.001m 的球体，或边长为 0.001m 的立方体\)，然后再在初始 link 上添加底盘等刚体，这样实现，虽然仍然存在初始link半沉的现象，但是基本可以忽略了，这个初始 link 一般称之为 base\_footprint。

![](/assets/6.4.3urdf_basefootprint.gif)

##### 2.实现

功能包 cpp06\_urdf 的 urdf/urdf 目录下，新建 urdf 文件 demo04\_basefootprint.urdf，并编辑文件，输入如下内容：

```xml
<!-- 
    需求：为机器人模型添加 base_footprint

 -->
 <robot name="base_footprint_demo">
  <!-- 定义颜色 -->
  <material name="yellow">
    <color rgba="0.7 0.7 0 0.8" />
  </material>
  <material name="red">
    <color rgba="0.8 0.1 0.1 0.8" />
  </material>

  <link name="base_footprint">
    <visual>
      <geometry>
          <sphere radius="0.001"/>
      </geometry>
    </visual>
  </link>

  <link name="base_link">
    <visual>
        <!-- 形状 -->
        <geometry>
            <box size="0.5 0.3 0.1" />
        </geometry>
        <origin xyz="0 0 0" rpy="0 0 0" />
        <material name="yellow"/>
    </visual>
  </link>

  <joint name="baselink2basefootprint" type="fixed">
    <parent link="base_footprint"/>
    <child link="base_link"/>
    <origin xyz="0.0 0.0 0.05"/>
  </joint>

  <!-- 摄像头 -->
  <link name="camera">
      <visual>
          <geometry>
              <box size="0.02 0.05 0.05" />
          </geometry>
          <origin xyz="0 0 0" rpy="0 0 0" />
          <material name="red" />
      </visual>
  </link>

  <!-- 关节 -->
  <joint name="camera2baselink" type="fixed">
      <parent link="base_link"/>
      <child link="camera" />
      <!-- 需要计算两个 link 的物理中心之间的偏移量 -->
      <origin xyz="0.2 0 0.075" rpy="0 0 0" />
      <axis xyz="0 0 1" />
  </joint>

</robot>
```

编译后，工作空间终端下调用如下命令执行：

    ros2 launch cpp06_urdf display.launch.py model:=`ros2 pkg prefix --share cpp06_urdf`/urdf/urdf/demo04_basefootprint.urdf

执行指令后，在 rviz2 将Fixed Frame设置为base\_footprint，机器人模型将正常显示在”地面“上。

