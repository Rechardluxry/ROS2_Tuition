### 6.4.2 URDF语法02\_link

#### 1.简介 {#link}

urdf 中的 link 标签用于描述机器人某个部件\(也即刚体部分\)的外观和物理属性，比如: 机器人底座、轮子、激光雷达、摄像头...每一个部件都对应一个 link, 在 link 标签内，可以设计该部件的形状、尺寸、颜色、惯性矩阵、碰撞参数等一系列属性。

![](/assets/6.4.2urdf_link.png)

#### 2.属性 {#1属性}

* name（必填）：为连杆命名。

#### 3.子标签 {#2子标签}

**&lt;visual&gt;**（可选）：用于描述link的可视化属性，可以设置link的形状（立方体、球体、圆柱等）。

* **name**（可选）：指定link名称，此名称会映射为同名坐标系，还可以通过引用该值定位定位link。
* **&lt;geometry&gt;**（必填）：用于设置link的形状，比如：立方体、球体或圆柱。
  * **&lt;box&gt;**：立方体标签，通过size属性设置立方体的边长，原点为其几何中心。
  * **&lt;cylinder&gt;**：圆柱标签，通过radius属性设置圆柱半径，通过length属性设置圆柱高度，原点为其几何中心。
  * **&lt;sphere&gt;**：球体标签，通过radius属性设置球体半径，原点为其几何中心。
  * **&lt;mesh&gt;**：通过属性filename引用“皮肤”文件，为link设置外观，该文件必须是本地文件。使用 package://&lt;packagename&gt;/&lt;path&gt;为文件名添加前缀。
* **&lt;origin&gt;**（可选）：用于设置link的相对偏移量以及旋转角度，如未指定则使用默认值（无偏移且无旋转）。
  * **xyz**：表示x、y、z三个维度上的偏移量（以米为单位），不同数值之间使用空格分隔，如未指定则使用默认值（三个维度无偏移）。
  * **rpy**：表示翻滚、俯仰与偏航的角度（以弧度为单位），不同数值之间使用空格分隔，如未指定则使用默认值（三个维度无旋转）。
* **&lt;material&gt;**（可选）：视觉元素的材质。也可以在根标签robot中定义material标签，然后，可以在link中按名称进行引用。 
  * **name**（可选）：为material指定名称，可以通过该值进行引用。
  * **&lt;color&gt;**（可选）：rgba 材质的颜色，由代表red/green/blue/alpha 的四个数字组成，每个数字的范围为 \[0,1\]。
  * **&lt;texture&gt;**（可选）：材质的纹理，可以由属性filename设置。

**&lt;collision&gt;**（可选）：link的碰撞属性。可以与link的视觉属性一致，也可以不同，比如：我们会通常使用更简单的碰撞模型来减少计算时间，或者设置的值大于link的视觉属性，以尽量避免碰撞。另外，同一链接可以存在多个 &lt;collision&gt;标签实例，多个几何图形组合表示link的碰撞属性。

* **name**（可选）：为collision设置名称。
* **&lt;geometry&gt;**（必须）：请参考visual标签的geometry使用规则。
* **&lt;origin&gt;**（可选）：请参考visual标签的origin使用规则。

**&lt;inertial&gt;**（可选）：用于设置link的质量、质心位置和中心惯性特性，如果未指定，则默认为质量为0、惯性为0。

* **&lt;origin&gt;**（可选）：该位姿（平移、旋转）描述了链接的质心框架 C 相对于链接框架 L 的位置和方向。
  * **xyz**：表示从 Lo（链接框架原点）到 Co（链接的质心）的位置向量为 x L̂x + y L̂y + z L̂z，其中 L̂x、L̂y、L̂z 是链接框架 L 的正交单位向量。
  * **rpy**：将 C 的单位向量 Ĉx、Ĉy、Ĉz 相对于链接框架 L 的方向表示为以弧度为单位的欧拉旋转序列 \(r p y\)。注意：Ĉx、Ĉy、Ĉz 不需要与连杆的惯性主轴对齐。
* **&lt;mass&gt;**（必填）：通过其value属性设置link的质量。
* **&lt;inertia&gt;**（必填）：对于固定在质心坐标系 C 中的单位向量 Ĉx、Ĉy、Ĉz，该连杆的惯性矩 ixx、iyy、izz 以及关于 Co（连杆的质心）的惯性 ixy、ixz、iyz 的乘积。

**注意：**&lt;collision&gt; 和 &lt;inertial&gt; 在仿真环境下才需要使用到，如果只是在 rviz2 中集成 urdf，那么不必须为 link 定义这两个标签。

#### 4.示例 {#3案例}

##### 1.需求

分别生成长方体、圆柱与球体的机器人部件。

##### 2.实现

功能包 cpp06\_urdf 的 urdf/urdf 目录下，新建 urdf 文件 demo02\_link.urdf，并编辑文件，输入如下内容：

```xml
<robot name="link_demo">
  <!-- 定义颜色 -->
  <material name="yellow">
    <color rgba="0.7 0.7 0 0.8" />
  </material>
  <link name="base_link">
    <visual>
        <!-- 形状 -->
        <geometry>
            <!-- 长方体的长宽高 -->
            <box size="0.5 0.3 0.1" />
            <!-- 圆柱，半径和长度 -->
            <!-- <cylinder radius="0.5" length="1.0" /> -->
            <!-- 球体，半径-->
            <!-- <sphere radius="0.3" /> -->

        </geometry>
        <!-- xyz坐标 rpy翻滚俯仰与偏航角度(3.14=180度) -->
        <origin xyz="0 0 0" rpy="0 0 0" />
        <!-- 调用已定义的颜色 -->
        <material name="yellow"/>
    </visual>
  </link>
</robot>
```

编译后，工作空间终端下调用如下命令执行：

    ros2 launch cpp06_urdf display.launch.py model:=`ros2 pkg prefix --share cpp06_urdf`/urdf/urdf/demo02_link.urdf

rviz2 中可以根据 geometry 标签中的设置显示对应形状的机器人。

