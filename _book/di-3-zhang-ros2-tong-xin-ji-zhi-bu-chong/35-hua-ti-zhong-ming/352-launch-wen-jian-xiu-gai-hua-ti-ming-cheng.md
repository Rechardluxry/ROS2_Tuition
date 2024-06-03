### 3.5.2 launch 文件修改话题名称

#### 1.Python方式实现的launch文件修改话题名称

在 Python 方式实现的 launch 文件中，可以通过类 `launch_ros.actions.Node`的构造函数中的参数 remappings 修改话题名称，使用示例如下：

```py
from launch import LaunchDescription
from launch_ros.actions import Node

def generate_launch_description():

    return LaunchDescription([
        Node(package="turtlesim",executable="turtlesim_node",namespace="t1"),
        Node(package="turtlesim",
            executable="turtlesim_node",
            remappings=[("/turtle1/cmd_vel","/cmd_vel")]
        )

    ])
```

#### 2.XML方式实现的launch文件修改话题名称

在 XML 方式实现的 launch 文件中，可以通过 node 标签的子标签 remap（属性from取值为被修改的话题名称，属性to的取值为修改后的话题名称） 修改话题名称，使用示例如下：

```xml
<launch>
    <node pkg="turtlesim" exec="turtlesim_node" namespace="t1" />
    <node pkg="turtlesim" exec="turtlesim_node">
        <remap from="/turtle1/cmd_vel" to="/cmd_vel" />
    </node>
</launch>
```

#### 3.YAML方式实现的launch文件修改话题名称

在 YAML 方式实现的 launch 文件中，可以通过 node 属性中 remap（属性from取值为被修改的话题名称，属性to的取值为修改后的话题名称） 修改话题名称，使用示例如下：

```yaml
launch:
- node:
    pkg: turtlesim
    exec: turtlesim_node
    namespace: t1
- node:
    pkg: turtlesim
    exec: turtlesim_node
    remap:
    -
        from: "/turtle1/cmd_vel"
        to: "/cmd_vel"
```

#### 4.测试

上述三种方式在修改话题名称时虽然语法不同，但是实现功能类似，都是启动了两个`turtlesim_node`节点，一个节点添加了命名空间，另一个节点将话题从`/turtle1/cmd_vel`映射到了`/cmd_vel`。使用`ros2 topic list`查看节点信息，显示结果：

添加命名空间的节点对应的话题为：

```
/t1/turtle1/cmd_vel
/t1/turtle1/color_sensor
/t1/turtle1/pose
```

重映射的节点对应的话题为：

```
/cmd_vel
/turtle1/color_sensor
/turtle1/pose
```



