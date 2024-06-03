### 4.3.2 执行指令

**需求：**在 launch 文件中执行 ROS2 命令，以简化部分功能的调用。

**示例：**

在 cpp01\_launch/launch/xml 目录下新建 xml02\_cmd.launch.xml 文件，输入如下内容：

```py
<launch>
    <node pkg="turtlesim" exec="turtlesim_node" />
    <executable cmd="ros2 run turtlesim turtlesim_node" output="both" />
</launch>
```

在 cpp01\_launch/launch/yaml 目录下新建 yaml02\_cmd.launch.yaml 文件，输入如下内容：

```yaml
launch:
- executable:
    cmd: "ros2 run turtlesim turtlesim_node"
    output: "both"
```

**代码解释：**

在 XML 实现中 executable 标签用于表示可执行指令，其属性包含：

* cmd：被执行的命令；
* output：日志输出目的地设置。

YAML 实现规则与之类似。

