### 6.3.2 案例实现

#### 1.URDF文件实现

功能包 cpp06\_urdf 的 urdf/urdf 目录下，新建 urdf 文件 demo01\_helloworld.urdf，并编辑文件，输入如下内容：

```xml
<!-- 
    需求：显示一盒状机器人
 -->
 <robot name="hello_world">
   <link name="base_link">
     <visual>
       <geometry>
         <box size="0.5 0.2 0.1"/>
       </geometry>
     </visual>
   </link>
 </robot>
```

#### 2.launch文件实现

功能包 cpp06\_urdf 的 launch 目录下，新建 launch 文件 display.launch.py，并编辑文件，输入如下内容：

```py
from launch import LaunchDescription
from launch_ros.actions import Node
import os
from ament_index_python.packages import get_package_share_directory
from launch_ros.parameter_descriptions import ParameterValue
from launch.substitutions import Command,LaunchConfiguration
from launch.actions import DeclareLaunchArgument

#示例： ros2 launch cpp06_urdf display.launch.py model:=`ros2 pkg prefix --share cpp06_urdf`/urdf/urdf/demo01_helloworld.urdf
def generate_launch_description():

    cpp06_urdf_dir = get_package_share_directory("cpp06_urdf")
    default_model_path = os.path.join(cpp06_urdf_dir,"urdf/urdf","demo01_helloworld.urdf")
    default_rviz_path = os.path.join(cpp06_urdf_dir,"rviz","display.rviz")
    model = DeclareLaunchArgument(name="model", default_value=default_model_path)

    # 加载机器人模型
    # 1.启动 robot_state_publisher 节点并以参数方式加载 urdf 文件
    robot_description = ParameterValue(Command(["xacro ",LaunchConfiguration("model")]))
    robot_state_publisher = Node(
        package="robot_state_publisher",
        executable="robot_state_publisher",
        parameters=[{"robot_description": robot_description}]
    )
    # 2.启动 joint_state_publisher 节点发布非固定关节状态
    joint_state_publisher = Node(
        package="joint_state_publisher",
        executable="joint_state_publisher"
    )
    # rviz2 节点
    rviz2 = Node(
        package="rviz2",
        executable="rviz2"
        # arguments=["-d", default_rviz_path]
    )
    return LaunchDescription([
        model,
        robot_state_publisher,
        joint_state_publisher,
        rviz2
    ])
```

launch 文件启动时，可以通过参数 model 动态传入被加载的 urdf 文件。

#### 3.编辑配置文件

##### 1.package.xml

在 package.xml 中需要手动添加一些执行时依赖，核心内容如下：

```XML
<exec_depend>rviz2</exec_depend>
<exec_depend>xacro</exec_depend>
<exec_depend>robot_state_publisher</exec_depend>
<exec_depend>joint_state_publisher</exec_depend>
<exec_depend>ros2launch</exec_depend>
```

##### 2.CMakeLists.txt

在功能包下，新建了若干目录，需要为这些目录配置安装路径，核心内容如下：

```
install(
  DIRECTORY launch urdf rviz meshes
  DESTINATION share/${PROJECT_NAME}  
)
```

#### 4.编译

终端中进入当前工作空间，编译功能包：

```
colcon build --packages-select cpp06_urdf
```

#### 5.执行

当前工作空间下，启动终端，输入如下指令：

```
. install/setup.bash
ros2 launch cpp06_urdf display.launch.py
```

然后 rviz2 会启动，启动后做如下配置：

1. Global Options 中的 Fixed Frame 设置为 base\_link（和  urdf 文件中 link 标签的 name 一致）；
2. 添加机器人模型插件，并将参数 Description Topic 的值设置为 /robot\_description，即可显示机器人模型。

![](/assets/6.3.2rviz2显示机器人模型.gif)

**小提示：**

在本章的后续案例中，所有实现都遵循上述步骤，在后续案例中我们只需要关注 urdf 实现即可，launch 文件和 配置文件无需修改。

