### 4.2.1 节点设置

launch 中需要执行的节点被封装为了 launch\_ros.actions.Node 对象。

**需求：**launch 文件中配置节点的相关属性。

**示例：**

在 cpp01\_launch/launch/py 目录下新建 py01\_node.launch.py 文件，输入如下内容：

```py
from launch import LaunchDescription
from launch_ros.actions import Node
import os
from ament_index_python.packages import get_package_share_directory

def generate_launch_description():

    turtle1 = Node(package="turtlesim", 
                executable="turtlesim_node", 
                namespace="ns_1",
                name="t1", 
                exec_name="turtle_label", # 表示流程的标签
                respawn=True)
    turtle2 = Node(package="turtlesim", 
                executable="turtlesim_node", 
                name="t2",
                # 参数设置方式1
                # parameters=[{"background_r": 0,"background_g": 0,"background_b": 0}],
                # 参数设置方式2: 从 yaml 文件加载参数，yaml 文件所属目录需要在配置文件中安装。
                parameters=[os.path.join(get_package_share_directory("cpp01_launch"),"config","t2.yaml")],
                )
    turtle3 = Node(package="turtlesim", 
                executable="turtlesim_node", 
                name="t3", 
                remappings=[("/turtle1/cmd_vel","/cmd_vel")] #话题重映射
                )
    rviz = Node(package="rviz2",
                executable="rviz2",
                # 节点启动时传参
                arguments=["-d", os.path.join(get_package_share_directory("cpp01_launch"),"config","my.rviz")]
    )

    turtle4 = Node(package="turtlesim", 
                executable="turtlesim_node",
                # 节点启动时传参，相当于 arguments 传参时添加前缀 --ros-args 
                ros_arguments=["--remap", "__ns:=/t4_ns", "--remap", "__node:=t4"]
    )
    return LaunchDescription([turtle1, turtle2, turtle3, rviz, turtle4])
```

**代码解释：**

**1.Node使用语法1**

```py
turtle1 = Node(package="turtlesim", 
                executable="turtlesim_node", 
                namespace="group_1", 
                name="t1", 
                exec_name="turtle_label", # 表示流程的标签
                respawn=True)
```

上述代码会创建一个 turtlesim\_node 节点，设置了若干节点属性，并且节点关闭后会自动重启。

* package：功能包；
* executable：可执行文件；
* namespace：命名空间；
* name：节点名称；
* exe\_name：流程标签；
* respawn：设置为True时，关闭节点后，可以自动重启。

**2.Node使用语法2**

```py
turtle2 = Node(package="turtlesim", 
                executable="turtlesim_node", 
                name="t2",
                # 参数设置方式1
                # parameters=[{"background_r": 0,"background_g": 0,"background_b": 0}],
                # 参数设置方式2: 从 yaml 文件加载参数，yaml 文件所属目录需要在配置文件中安装。
                parameters=[os.path.join(get_package_share_directory("cpp01_launch"),"config","t2.yaml")],
                )
```

上述代码会创建一个 turtlesim\_node 节点，并导入背景色相关参数。

* parameters：导入参数。

parameter 用于设置被导入的参数，如果是从 yaml 文件加载参数，那么需要先准备 yaml 文件，在功能包下新建 config 目录，config目录下新建 t2.yaml 文件，并输入如下内容：

```
/t2:
  ros__parameters:
    background_b: 0
    background_g: 0
    background_r: 50
    qos_overrides:
      /parameter_events:
        publisher:
          depth: 1000
          durability: volatile
          history: keep_last
          reliability: reliable
    use_sim_time: false
```

注意，还需要在 CMakeLists.txt 中安装 config：

```
install(DIRECTORY 
  launch
  config
  DESTINATION share/${PROJECT_NAME})
```

**3.Node使用语法3**

```
turtle3 = Node(package="turtlesim", 
                executable="turtlesim_node", 
                name="t3", 
                remappings=[("/turtle1/cmd_vel","/cmd_vel")] #话题重映射
                )
```

上述代码会创建一个 turtlesim\_node 节点，并将话题名称从 /turtle1/cmd\_vel 重映射到 /cmd\_vel。

* remappings：话题重映射。

**4.Node使用语法4**

```
rviz = Node(package="rviz2",
                executable="rviz2",
                # 节点启动时传参
                arguments=["-d", os.path.join(get_package_share_directory("cpp01_launch"),"config","my.rviz")]
    )
```

上述代码会创建一个 rviz2 节点，并加载了 rviz2 相关的配置文件。

该配置文件可以先启动 rviz2 ，配置完毕后，保存到 config 目录并命名为 my.rviz。

* arguments：调用指令时的参数列表。

**5.Node使用语法5**

```
turtle4 = Node(package="turtlesim", 
                executable="turtlesim_node",
                # 节点启动时传参，相当于 arguments 传参时添加前缀 --ros-args 
                ros_arguments=["--remap", "__ns:=/t4_ns", "--remap", "__node:=t4"]
    )
```

上述代码会创建一个 turtlesim\_node 节点，并在指令调用时传入参数列表。

* ros\_arguments：相当于 arguments 前缀 --ros-args。



