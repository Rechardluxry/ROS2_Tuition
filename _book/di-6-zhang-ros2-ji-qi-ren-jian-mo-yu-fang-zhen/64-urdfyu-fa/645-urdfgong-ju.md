### 6.4.5 URDF工具 {#635-urdf工具}

在 ROS2 中，提供了一些URDF文件相关的工具，比如:

* `check_urdf`命令可以检查复杂的 urdf 文件是否存在语法问题；

* `urdf_to_graphviz`命令可以查看 urdf 模型结构，显示不同 link 的层级关系。

当然，要使用工具之前，请先安装，安装命令：`sudo apt install liburdfdom-tools`。

#### 1.check\_urdf 语法检查 {#1checkurdf-语法检查}

进入urdf文件所属目录，调用：`check_urdf urdf文件`，如果不抛出异常，说明文件合法，否则非法。

示例，终端下进入功能包 cpp06\_urdf 的 urdf/urdf 目录，执行如下命令：

```
check_urdf demo05_exercise.urdf
```

urdf 文件如无异常，将显示urdf中link的层级关系，如下图所示：

![](/assets/6.4.5工具1check_urdf.PNG)

否则将会给出错误提示。

#### 2.urdf\_to\_graphviz 结构查看 {#2urdftographiz-结构查看}

进入urdf文件所属目录，调用:`urdf_to_graphviz urdf文件`，当前目录下会生成 pdf 文件。

示例，终端下进入功能包 cpp06\_urdf 的 urdf/urdf 目录，执行如下命令：

```
urdf_to_graphviz demo05_exercise.urdf
```

当前目录下，将生成以urdf中robot名称命名的.pdf和.gv文件，打开pdf文件会显示如下图内容：

![](/assets/6.4.5工具2urdf2pdf.PNG)

在上图中会以树形结构显示link与joint的关系。

**注意：**该工具以前名为`urdf_to_graphiz`现建议使用`urdf_to_graphviz`替代。

