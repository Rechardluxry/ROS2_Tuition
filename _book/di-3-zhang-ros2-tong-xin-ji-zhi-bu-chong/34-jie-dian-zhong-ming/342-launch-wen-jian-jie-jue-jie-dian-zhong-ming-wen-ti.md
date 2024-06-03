### 3.4.2 launch设置节点名称

在ROS2中launch文件可以由Python、XML或YAML三种语言编写（关于launch文件的基本使用可以参考**4.1 启动文件launch简介**），每种实现方式都可以设置节点的命名空间或为节点起别名。

#### 1.Python方式实现的launch文件设置命名空间与名称重映射

在 Python 方式实现的 launch 文件中，可以通过类 `launch_ros.actions.Node`来创建被启动的节点对象，在对象的构造函数中提供了 name 和 namespace 参数来设置节点的名称与命名空间，使用示例如下：

```py
from launch import LaunchDescription
from launch_ros.actions import Node

def generate_launch_description():

    return LaunchDescription([
        Node(package="turtlesim",executable="turtlesim_node",name="turtle1"),
        Node(package="turtlesim",executable="turtlesim_node",namespace="t1"),
        Node(package="turtlesim",executable="turtlesim_node",namespace="t1", name="turtle1")
    ])
```

#### 2.XML方式实现的launch文件设置命名空间与名称重映射

在 XML 方式实现的 launch 文件中，可以通过 node 标签中 name 和 namespace 属性来设置节点的名称与命名空间，使用示例如下：

```XML
<launch>
    <node pkg="turtlesim" exec="turtlesim_node" name="turtle1" />
    <node pkg="turtlesim" exec="turtlesim_node" namespace="t1" />
    <node pkg="turtlesim" exec="turtlesim_node" namespace="t1" name="turtle1" />
</launch>
```

#### 3.YAML方式实现的launch文件设置命名空间与名称重映射

在 YAML 方式实现的 launch 文件中，可以通过 node 属性中 name 和 namespace 属性来设置节点的名称与命名空间，使用示例如下：

```yaml
launch:
- node:
    pkg: turtlesim
    exec: turtlesim_node
    name: turtle1
- node:
    pkg: turtlesim
    exec: turtlesim_node
    namespace: t1
- node:
    pkg: turtlesim
    exec: turtlesim_node
    namespace: t1
    name: turtle1
```

#### 4.测试

上述三种方式在设置命名空间与名称重映射时虽然语法不同，但是实现功能类似，都是启动了三个 turtlesim\_node 节点，第一个节点设置了节点名称，第二个节点设置了命名空间，第三个节点既设置了命名空间又设置了节点名称，分别执行三个launch文件，然后使用`ros2 node list`查看节点信息，显示结果都如下所示：

```
/t1/turtl1
/t1/turtlesim
/turtle1
```



