### 4.3.4 文件包含

**需求：**新建 launch 文件，包含 4.2.3 中的 launch 文件并为之传入设置背景色相关的参数。

**示例：**

在 cpp01\_launch/launch/xml 目录下新建 xml04\_include.launch.xml 文件，输入如下内容：

```xml
<launch>
    <let name="bg_r" value="0" />
    <include file="$(find-pkg-share cpp01_launch)/launch/xml/xml03_args.launch.xml"/>

</launch>
```

在 cpp01\_launch/launch/yaml 目录下新建 yaml04\_include.launch.yaml 文件，输入如下内容：

```
launch:
- let:
    name: "bgr"
    value: "255"
- include:
    file: "$(find-pkg-share cpp01_launch)/launch/yaml/yaml03_arg.launch.yaml"
```

**代码解释：**

在 XML 实现中，include 标签用于实现文件包含，其属性如下：

* file：被包含的launch文件的路径。

let 标签用于向被包含的 launch 文件中导入参数，其属性如下：

* name：参数名称；
* value：参数值。

YAML 实现规则与之类似。

