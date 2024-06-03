### 4.2.2 执行指令

launch 中需要执行的命令被封装为了 launch.actions.ExecuteProcess 对象。

**需求：**在 launch 文件中执行 ROS2 命令，以简化部分功能的调用。

**示例：**

在 cpp01\_launch/launch/py 目录下新建 py02\_cmd.launch.py 文件，输入如下内容：

```py
from launch import LaunchDescription
from launch_ros.actions import Node
from launch.actions import ExecuteProcess
from launch.substitutions import FindExecutable

def generate_launch_description():
    turtle = Node(package="turtlesim", executable="turtlesim_node")
    spawn = ExecuteProcess(
        # cmd=["ros2 service call /spawn turtlesim/srv/Spawn \"{x: 8.0, y: 9.0,theta: 0.0, name: 'turtle2'}\""],
        # 或
        cmd = [
            FindExecutable(name = "ros2"), # 不可以有空格
            " service call",
            " /spawn turtlesim/srv/Spawn",
            " \"{x: 8.0, y: 9.0,theta: 1.0, name: 'turtle2'}\""
        ],
        output="both",
        shell=True)
    return LaunchDescription([turtle,spawn])
```

**代码解释：**

```py
spawn = ExecuteProcess(
        # cmd=["ros2 service call /spawn turtlesim/srv/Spawn \"{x: 8.0, y: 9.0,theta: 0.0, name: 'turtle2'}\""],
        # 或
        cmd = [
            FindExecutable(name = "ros2"), # 不可以有空格
            " service call",
            " /spawn turtlesim/srv/Spawn",
            " \"{x: 8.0, y: 9.0,theta: 1.0, name: 'turtle2'}\""
        ],
        output="both",
        shell=True)
```

上述代码用于执行 cmd 参数中的命令，该命令会在 turtlesim\_node 中生成一只新的小乌龟。

* cmd：被执行的命令；
* output：设置为 both 时，日志会被输出到日志文件和终端，默认为 log，日志只输出到日志文件。
* shell：如果为 True，则以 shell 的方式执行命令。



