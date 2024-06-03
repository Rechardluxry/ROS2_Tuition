### 5.6.5 乌龟护航实现（C++）

#### 1.编写 launch 文件

功能包 cpp05\_exercise 的 launch 目录下新建文件exer02\_turtle\_escort.launch.py，并编辑文件，输入如下内容：

```py
from launch import LaunchDescription
from launch_ros.actions import Node
from launch.actions import ExecuteProcess, DeclareLaunchArgument
from launch.substitutions import LaunchConfiguration

def spawn(node_name,x=2.0,y=2.0,theta=0.0,turtle_name="turtle2"):
  return Node(package="cpp05_exercise", executable="exer01_tf_spawn",
              name=node_name,
              parameters=[{"x":x,"y":y,"theta":theta,"turtle_name":turtle_name}])

def tf_broadcaster(node_name,turtle_name="turtle1"):
  return Node(package="cpp05_exercise",executable="exer02_tf_broadcaster",
              name=node_name,
              parameters=[{"turtle_name":turtle_name}])

def escort(node_name,x="0.0",y="0.0",frame_id="turtle1",child_frame_id="escort1"):
  return Node(package="tf2_ros",executable="static_transform_publisher",
              name=node_name,
              arguments=["--x",x,"--y",y,"--frame-id",frame_id,"--child-frame-id",child_frame_id]
            )
def tf_listener(node_name,target_frame,source_frame):
  return Node(package="cpp05_exercise",executable="exer03_tf_listener",
              name=node_name,
              parameters=[{"target_frame":target_frame,"source_frame":source_frame}]
              )
def generate_launch_description():

    # 启动 turtlesim_node 节点
    turtlesim_node = Node(package="turtlesim", executable="turtlesim_node", name="t1")
    # 生成一只新乌龟
    spawn1 = spawn(node_name="spawn1")
    spawn2 = spawn(node_name="spawn2",x=2.0,y=7.0,turtle_name="turtle3")
    spawn3 = spawn(node_name="spawn3",x=7.0,y=2.0,theta=1.57,turtle_name="turtle4")
    # tf 广播
    tf_broadcaster1 = tf_broadcaster(node_name="tf_broadcaster1")
    tf_broadcaster2 = tf_broadcaster(node_name="tf_broadcaster2",turtle_name="turtle2")
    tf_broadcaster3 = tf_broadcaster(node_name="tf_broadcaster3",turtle_name="turtle3")
    tf_broadcaster4 = tf_broadcaster(node_name="tf_broadcaster4",turtle_name="turtle4")

    escort1 = escort(node_name="static_transform_publisher1",x="-1")
    escort2 = escort(node_name="static_transform_publisher2",y="1",child_frame_id="escort2")
    escort3 = escort(node_name="static_transform_publisher3",y="-1",child_frame_id="escort3")

    # tf 监听
    tf_listener1 = tf_listener(node_name="tf_listener1",target_frame="turtle2",source_frame="escort1")
    tf_listener2 = tf_listener(node_name="tf_listener2",target_frame="turtle3",source_frame="escort2")
    tf_listener3 = tf_listener(node_name="tf_listener3",target_frame="turtle4",source_frame="escort3")

    return LaunchDescription([turtlesim_node,spawn1,spawn2,spawn3,
            tf_broadcaster1,tf_broadcaster2,tf_broadcaster3,tf_broadcaster4,
            escort1,escort2,escort3,
            tf_listener1,tf_listener2,tf_listener3
    ])
```

#### 2.编译

终端中进入当前工作空间，编译功能包：

```
colcon build --packages-select cpp05_exercise
```

#### 3.执行

当前工作空间下启动终端，输入如下命令运行launch文件：

```
. install/setup.bash 
ros2 launch cpp05_exercise exer02_turtle_escort.launch.py
```

此时窗口中会新生成三只乌龟，向原生乌龟 turtle1 运动，并会组成固定队形。

再新建终端，启动 turtlesim 键盘控制节点：

```
ros2 run turtlesim turtle_teleop_key
```

该终端下可以通过键盘控制 turtle1 运动，并且护航的乌龟会按照特定队形跟随 turtle1 运动。最终的运行结果与演示案例类似。

