### 4.2.6 添加事件

节点在运行过程中会触发不同的事件，当事件触发时可以为之注册一定的处理逻辑。事件使用相关的 API 为：launch.actions.RegisterEventHandler、launch.event\_handlers.OnProcessStart、launch.event\_handlers.OnProcessExit。

**需求：**为 turtlesim\_node 节点添加事件，事件1：节点启动时调用spawn服务生成新乌龟；事件2：节点关闭时，输出日志信息。

**示例：**

在 cpp01\_launch/launch/py 目录下新建 py06\_event.launch.py 文件，输入如下内容：

```py
from launch import LaunchDescription
from launch_ros.actions import Node
from launch.actions import ExecuteProcess, RegisterEventHandler,LogInfo
from launch.substitutions import FindExecutable
from launch.event_handlers import OnProcessStart, OnProcessExit
def generate_launch_description():
    turtle = Node(package="turtlesim", executable="turtlesim_node")
    spawn = ExecuteProcess(
        cmd = [
            FindExecutable(name = "ros2"), # 不可以有空格
            " service call",
            " /spawn turtlesim/srv/Spawn",
            " \"{x: 8.0, y: 1.0,theta: 1.0, name: 'turtle2'}\""
        ],
        output="both",
        shell=True)

    start_event = RegisterEventHandler(
        event_handler=OnProcessStart(
            target_action = turtle,
            on_start = spawn
        )
    )
    exit_event = RegisterEventHandler(
        event_handler=OnProcessExit(
            target_action = turtle,
            on_exit = [LogInfo(msg = "turtlesim_node退出!")]
        )
    )

    return LaunchDescription([turtle,start_event,exit_event])
```

**代码解释：**

```py
start_event = RegisterEventHandler(
        event_handler=OnProcessStart(
            target_action = turtle,
            on_start = spawn
        )
    )
exit_event = RegisterEventHandler(
        event_handler=OnProcessExit(
            target_action = turtle,
            on_exit = [LogInfo(msg = "turtlesim_node退出!")]
        )
    )
```

上述代码为 turtle 节点注册启动事件和退出事件，当 turtle 节点启动后会执行 spwn 节点，当 turtle 节点退出时，会输出日志文本：“turtlesim\_node退出!”。

对象 RegisterEventHandler 负责注册事件，其参数为：

* event\_handler：注册的事件对象。

OnProcessStart 是启动事件对象，其参数为：

* target\_action：被注册事件的目标对象； 
* on\_start：事件触发时的执行逻辑。 

OnProcessExit 是退出事件对象，其参数为：

* target\_action：被注册事件的目标对象； 
* on\_exit：事件触发时的执行逻辑。 

LogInfo 是日志输出对象，其参数为：

* msg：被输出的日志信息。



