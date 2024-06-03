### 4.2.3 参数设置

参数设置主要涉及到参数的声明与调用两部分，其中声明被封装为 launch.actions.DeclareLaunchArgument，调用则被封装为 launch.substitutions import LaunchConfiguration。

**需求：**启动turtlesim\_node节点时，可以动态设置背景色。

**示例：**

在 cpp01\_launch/launch/py 目录下新建 py03\_args.launch.py 文件，输入如下内容：

```py
from pkg_resources import declare_namespace
from launch import LaunchDescription
from launch_ros.actions import Node
from launch.actions import DeclareLaunchArgument
from launch.substitutions import LaunchConfiguration

def generate_launch_description():

    decl_bg_r = DeclareLaunchArgument(name="background_r",default_value="255")
    decl_bg_g = DeclareLaunchArgument(name="background_g",default_value="255")
    decl_bg_b = DeclareLaunchArgument(name="background_b",default_value="255")

    turtle = Node(package="turtlesim", 
            executable="turtlesim_node",
            parameters=[{"background_r": LaunchConfiguration("background_r"), "background_g": LaunchConfiguration("background_g"), "background_b": LaunchConfiguration("background_b")}]
            )
    return LaunchDescription([decl_bg_r,decl_bg_g,decl_bg_b,turtle])
```

**代码解释：**

```py
decl_bg_r = DeclareLaunchArgument(name="background_r",default_value="255")
decl_bg_g = DeclareLaunchArgument(name="background_g",default_value="255")
decl_bg_b = DeclareLaunchArgument(name="background_b",default_value="255")
```

上述代码会使用DeclareLaunchArgument对象声明三个参数，且每个参数都有参数名称以及默认值。

* name：参数名称；
* default\_value：默认值。

```
parameters=[{"background_r": LaunchConfiguration(variable_name="background_r"), "background_g": LaunchConfiguration("background_g"), "background_b": LaunchConfiguration("background_b")}]
```

上述代码会使用LaunchConfiguration对象获取参数值。

* variable\_name：被解析的参数名称。

launch文件执行时，可以动态传入参数，示例如下：

```
ros2 launch cpp01_launch py03_args.launch.py background_r:=200 background_g:=80 background_b:=30
```

如果执行launch文件时不手动传入参数，那么解析到的参数值是声明时设置的默认值。

