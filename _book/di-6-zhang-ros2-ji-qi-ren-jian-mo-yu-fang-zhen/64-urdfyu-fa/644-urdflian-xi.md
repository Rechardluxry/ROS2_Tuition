### 6.4.4 URDF练习

##### 1.需求

创建一个四轮机器人模型，机器人参数如下：底盘为长方体状，长20cm宽12cm高7cm，车轮半径为2.5cm，车轮厚度为2cm，底盘离地间距为 1.5cm。

![](/assets/6.4.4urdf_练习.gif)

##### 2.实现

功能包 cpp06\_urdf 的 urdf/urdf 目录下，新建 urdf 文件 demo05\_exercise.urdf，并编辑文件，输入如下内容：

```xml
<!-- 
    练习：编写四轮差速机器人的底盘模型

    参数：
        长 0.2m
        宽 0.12m
        高 0.07m
        离地 0.015m
        车轮半径：0.025m
        车轮厚度：0.02m

 -->
 <robot name="exercise_demo">
  <!-- 定义颜色 -->
  <material name="yellow">
    <color rgba="0.7 0.7 0 0.8" />
  </material>
  <material name="red">
    <color rgba="0.8 0.1 0.1 0.8" />
  </material>
  <material name="gray">
    <color rgba="0.2 0.2 0.2 0.8" />
  </material>

  <link name="base_footprint">
    <visual>
      <geometry>
          <sphere radius="0.001"/>
      </geometry>
    </visual>
  </link>
  <!-- 车体 -->
  <link name="base_link">
    <visual>
        <!-- 形状 -->
        <geometry>
            <box size="0.2 0.12 0.07" />
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

  <!-- ==============车轮================= -->
  <!-- 左前轮 -->
  <link name="front_left_wheel">
      <visual>
          <geometry>
              <cylinder radius="0.025" length="0.02"/>
          </geometry>
          <origin xyz="0 0 0" rpy="1.57 0 0" />
          <material name="gray" />
      </visual>
  </link>

  <!-- 左前轮关节 -->
  <joint name="frontleftwheel2baselink" type="continuous">
      <parent link="base_link"/>
      <child link="front_left_wheel" />
      <!-- 需要计算两个 link 的物理中心之间的偏移量 -->
      <origin xyz="0.075 0.06 -0.025" rpy="0 0 0" />
      <axis xyz="0 1 0" />
  </joint>

  <!-- 右前轮 -->
  <link name="front_right_wheel">
    <visual>
        <geometry>
            <cylinder radius="0.025" length="0.02"/>
        </geometry>
        <origin xyz="0 0 0" rpy="1.57 0 0" />
        <material name="gray" />
    </visual>
  </link>

  <!-- 右前轮关节 -->
  <joint name="frontrightwheel2baselink" type="continuous">
      <parent link="base_link"/>
      <child link="front_right_wheel" />
      <!-- 需要计算两个 link 的物理中心之间的偏移量 -->
      <origin xyz="0.075 -0.06 -0.025" rpy="0 0 0" />
      <axis xyz="0 1 0" />
  </joint>

  <!-- 左后轮 -->
  <link name="back_left_wheel">
    <visual>
        <geometry>
            <cylinder radius="0.025" length="0.02"/>
        </geometry>
        <origin xyz="0 0 0" rpy="1.57 0 0" />
        <material name="gray" />
    </visual>
  </link>

  <!-- 左后轮关节 -->
  <joint name="backleftwheel2baselink" type="continuous">
    <parent link="base_link"/>
    <child link="back_left_wheel" />
    <!-- 需要计算两个 link 的物理中心之间的偏移量 -->
    <origin xyz="-0.075 0.06 -0.025" rpy="0 0 0" />
    <axis xyz="0 1 0" />
  </joint>

  <!-- 右后轮 -->
  <link name="back_right_wheel">
    <visual>
        <geometry>
            <cylinder radius="0.025" length="0.02"/>
        </geometry>
        <origin xyz="0 0 0" rpy="1.57 0 0" />
        <material name="gray" />
    </visual>
  </link>

  <!-- 右后轮关节 -->
  <joint name="backrightwheel2baselink" type="continuous">
    <parent link="base_link"/>
    <child link="back_right_wheel" />
    <!-- 需要计算两个 link 的物理中心之间的偏移量 -->
    <origin xyz="-0.075 -0.06 -0.025" rpy="0 0 0" />
    <axis xyz="0 1 0" />
  </joint>


</robot>
```

编译后，工作空间终端下调用如下命令执行：

    ros2 launch cpp06_urdf display.launch.py model:=`ros2 pkg prefix --share cpp06_urdf`/urdf/urdf/demo05_exercise.urdf

命令执行后，rviz2 中可以显示与需求类似的机器人模型。

