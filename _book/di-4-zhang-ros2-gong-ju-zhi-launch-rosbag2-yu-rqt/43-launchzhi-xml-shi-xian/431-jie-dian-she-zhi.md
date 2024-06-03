### 4.3.1 节点设置

**需求：**launch 文件中配置节点的相关属性。

**示例：**

在 cpp01\_launch/launch/xml 目录下新建 xml01\_node.launch.xml 文件，输入如下内容：

```xml
<launch>
    <node pkg="turtlesim" exec="turtlesim_node" name="t1" namespace="t1_ns" exec_name="t1_label" respawn="true"/>
    <node pkg="turtlesim" exec="turtlesim_node" name="t2">
        <!-- <param name="background_r" value="255" />
            <param name="background_g" value="255" />
            <param name="background_b" value="255" /> -->
        <param from="$(find-pkg-share cpp01_launch)/config/t2.yaml" />
    </node>
    <node pkg="turtlesim" exec="turtlesim_node" name="t3">
        <remap from="/turtle1/cmd_vel" to="/cmd_vel" />
    </node>
    <node pkg="turtlesim" exec="turtlesim_node" ros_args="--remap __name:=t4 --remap __ns:=/group_2" />
    <node pkg="rviz2" exec="rviz2" args="-d $(find-pkg-share cpp01_launch)/config/my.rviz" />  

</launch>
```

在 cpp01\_launch/launch/yaml 目录下新建 yaml01\_node.launch.yaml 文件，输入如下内容：

```yaml
launch:
- node:
    pkg: "turtlesim"
    exec: "turtlesim_node"
    name: "t1"
    namespace: "t1_ns"
    exec_name: "t1_label"
    respawn: "false"
- node:
    pkg: "turtlesim"
    exec: "turtlesim_node"
    name: "t2"
    param:
    # - 
    #   name: "background_r"
    #   value: 255
    # - 
    #   name: "background_b"
    #   value: 255
    -
      from: "$(find-pkg-share cpp01_launch)/config/t2.yaml"
- node:
    pkg: "turtlesim"
    exec: "turtlesim_node"
    remap:
    -
      from: "/turtle1/cmd_vel"
      to: "/cmd_vel"

- node:
    pkg: "turtlesim"
    exec: "turtlesim_node"
    ros_args: "--ros-args --remap __name:=t4 --remap __ns:=/t4"
- node:
    pkg: "rviz2"
    exec: "rviz2"
    args: "-d $(find-pkg-share cpp01_launch)/config/my.rviz"
```

**代码解释：**

在XML 实现中 node 标签用于表示节点，其属性包含：

* pkg：功能包；
* exec：可执行文件；
* name：节点名称；
* namespace：命名空间；
* exec\_name：流程标签；
* respawn：节点关闭后是否重启；
* args：调用指令时的参数列表；
* ros\_args：相当于 args 前缀 --ros-args。 

node 标签的子级标签包含：

* param，设置参数的标签，其属性包含：

  * name：参数名称；

  * value：参数值；

  * from：参数文件路径；

* remap，话题重映射标签，其属性包含：

  * from：原话题名称；

  * to：新话题名称。

YAML 实现规则与之类似。

