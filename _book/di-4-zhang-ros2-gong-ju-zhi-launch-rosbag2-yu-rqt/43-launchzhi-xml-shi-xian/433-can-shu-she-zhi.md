### 4.3.3 参数设置

**需求：**启动turtlesim\_node节点时，可以动态设置背景色。

**示例：**

在 cpp01\_launch/launch/xml 目录下新建 xml03\_args.launch.xml 文件，输入如下内容：

```py
<launch>
    <arg name="bg_r" default="255"/>
    <arg name="bg_g" default="255"/>
    <arg name="bg_b" default="255"/>
    <node pkg="turtlesim" exec="turtlesim_node">
        <param name="background_r" value="$(var bg_r)" />
        <param name="background_g" value="$(var bg_g)" />
        <param name="background_b" value="$(var bg_b)" />
    </node>

</launch>
```

在 cpp01\_launch/launch/yaml 目录下新建 yaml03\_args.launch.yaml 文件，输入如下内容：

```yaml
launch:
- arg:
    name: "bgr"
    default: "255"
- node:
    pkg: "turtlesim"
    exec: "turtlesim_node"
    param:
    -
      name: "background_r"
      value: $(var bgr)
```

**代码解释：**

在 XML 实现中，arg 标签用于声明参数，其属性包含：

* name：参数名称；
* default：参数默认值。

参数的调用语法为：

* $\(var 参数名称\)。

可以在启动 launch 文件时动态传入参数，其语法与 Python 格式实现的 launch 文件一致。

YAML 实现规则与之类似。

