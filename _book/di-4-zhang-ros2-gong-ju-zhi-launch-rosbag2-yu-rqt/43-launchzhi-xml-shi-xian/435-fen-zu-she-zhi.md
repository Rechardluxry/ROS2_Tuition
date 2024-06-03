### 4.3.5 分组设置

**需求：**对 launch 文件中的多个 Node 进行分组。

**示例：**

在 cpp01\_launch/launch/xml 目录下新建 xml05\_group.launch.xml 文件，输入如下内容：

```py
<launch>

    <group>
        <push_ros_namespace namespace="g1" />
        <node pkg="turtlesim" exec="turtlesim_node" name="t1"/>
        <node pkg="turtlesim" exec="turtlesim_node" name="t2"/>
    </group>
    <group>
        <push_ros_namespace namespace="g2" />
        <node pkg="turtlesim" exec="turtlesim_node" name="t3"/>
    </group>

</launch>
```

在 cpp01\_launch/launch/yaml 目录下新建 yaml05\_group.launch.yaml 文件，输入如下内容：

```yaml
launch:
- group:
   - push_ros_namespace:
       namespace: "g1"
   - node:
       pkg: "turtlesim"
       exec: "turtlesim_node"
       name: "t1"
   - node:
       pkg: "turtlesim"
       exec: "turtlesim_node"
       name: "t2"
- group:
   - push_ros_namespace:
       namespace: "g2"
   - node:
       pkg: "turtlesim"
       exec: "turtlesim_node"
       name: "t3"
```

**代码解释：**

在 XML 实现中，group 标签用于分组，其子标签如下：

* push\_ros\_namespace：可以通过该标签中的 namespace 属性设置组内节点使用的命名空间。
* node：节点标签。

YAML 实现规则与之类似。

