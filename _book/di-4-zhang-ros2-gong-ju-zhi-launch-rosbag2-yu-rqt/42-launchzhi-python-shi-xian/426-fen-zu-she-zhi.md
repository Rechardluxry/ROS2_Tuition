### 4.2.5 分组设置

在 launch 文件中，为了方便管理可以对节点分组，分组相关API为：launch.actions.GroupAction和launch\_ros.actions.PushRosNamespace。

**需求：**对 launch 文件中的多个 Node 进行分组。

**示例：**

在 cpp01\_launch/launch/py 目录下新建 py05\_group.launch.py 文件，输入如下内容：

```py
from launch import LaunchDescription
from launch_ros.actions import Node
from launch_ros.actions import PushRosNamespace
from launch.actions import GroupAction

def generate_launch_description():
    turtle1 = Node(package="turtlesim",executable="turtlesim_node",name="t1")
    turtle2 = Node(package="turtlesim",executable="turtlesim_node",name="t2")
    turtle3 = Node(package="turtlesim",executable="turtlesim_node",name="t3")
    g1 = GroupAction(actions=[PushRosNamespace(namespace="g1"),turtle1, turtle2])
    g2 = GroupAction(actions=[PushRosNamespace(namespace="g2"),turtle3])
    return LaunchDescription([g1,g2])
```

**代码解释：**

```py
g1 = GroupAction(actions=[PushRosNamespace(namespace="g1"),turtle1, turtle2])
g2 = GroupAction(actions=[PushRosNamespace(namespace="g2"),turtle3])
```

上述代码将创建两个组，两个组使用了不同的命名空间，每个组下包含了不同的节点。

在 GroupAction 对象中，使用的参数为：

* actions：action列表，比如被包含到组内的命名空间、节点等。

在 PushRosNamespace 对象中，使用的参数为：

* namespace：当前组使用的命名空间。



