### 4.1.1 launch 基本使用流程

在 ROS2 中，launch 文件可以使用Python、XML或YAML编写，不同格式的 launch 文件基本使用流程一致。接下来我们通过一个案例演示 launch 文件的基本编写编译流程，案例需求：编写并执行 launch 文件，可以启动两个`turtlesim_node`节点。

实现步骤如下：

1. 编写launch文件；
2. 编辑配置文件；
3. 编译；
4. 执行。

#### 1.C++实现

##### 1.编写launch文件

功能包cpp01\_launch下，创建launch目录。launch文件可以是python文件、xml文件或yaml文件，不同类型的launch文件可以直接存储在launch目录下，或者为了方便管理，我们也可以在launch目录下新建py、xml和yaml三个文件夹分别存储对应类型的launch文件，并且建议不同格式的launch文件命名时分别使用`_launch.py`、`_launch.xml`、`_launch.yaml`或`.launch.py`、`.launch.xml`、`.launch.yaml`作为后缀名。

不同类型的 launch 文件示例如下：

**Python文件：**py00\_base.launch.py

```py
from launch import LaunchDescription
from launch_ros.actions import Node

def generate_launch_description():

    turtle1 = Node(package="turtlesim", executable="turtlesim_node", name="t1")
    turtle2 = Node(package="turtlesim", executable="turtlesim_node", name="t2")
    return LaunchDescription([turtle1, turtle2])
```

**XML文件：**xml00\_base.launch.xml

```xml
<launch>
    <node pkg="turtlesim" exec="turtlesim_node" name="t1" />
    <node pkg="turtlesim" exec="turtlesim_node" name="t2" />    
</launch>
```

**YAML文件：**yaml00\_base.launch.yaml

```
launch:
- node:
    pkg: "turtlesim"
    exec: "turtlesim_node"
    name: "t1"
- node:
    pkg: "turtlesim"
    exec: "turtlesim_node"
    name: "t2"
```

launch 文件编写完毕后，其实已经可以直接使用 `ros2 launch 文件路径` 的方式执行了，终端中进入当前工作空间，输入如下指令：

```
ros2 launch src/cpp01_launch/launch/py/py00_base.launch.py
```

但是这种执行方式不建议。

##### 2.编辑配置文件

CMakeLists.txt 中添加语句：

```
install(DIRECTORY launch DESTINATION share/${PROJECT_NAME})
```

无论该功能包的launch目录下有多少个launch文件，launch相关配置只需设置一次即可。

##### 3.编译

终端中进入当前工作空间，编译功能包：

```
colcon build --packages-select cpp01_launch
```

##### 4.执行

当前工作空间下，启动终端，输入如下指令：

```
. install/setup.bash
ros2 run cpp01_launch py00_base.launch.py
```

该指令运行的是 python 格式的 launch 文件，其他两个launch文件与之同理。最终，会启动两个 turtlesim\_node 节点。

#### 2.Python实现

##### 1.编写launch文件

功能包py01\_launch下，创建launch目录，launch目录下新建py、xml和yaml三个文件夹分别存储对应类型的launch文件，launch 文件实现与 rclcpp 完全一致。并且也可以使用`ros2 launch 文件路径`的方式执行。

##### 2.编辑配置文件

编辑 setup.py 文件，需要在 data\_files 属性中，添加相关 launch 文件的路径，修改后的内容如下：

```py
from setuptools import setup
from glob import glob
package_name = 'py01_launch'

setup(
    name=package_name,
    version='0.0.0',
    packages=[package_name],
    data_files=[
        ('share/ament_index/resource_index/packages',
            ['resource/' + package_name]),
        ('share/' + package_name, ['package.xml']),
        # launch 文件相关配置
        ('share/' + package_name, glob("launch/py/*.launch.py")),
        ('share/' + package_name, glob("launch/xml/*.launch.xml")),
        ('share/' + package_name, glob("launch/yaml/*.launch.yaml"))
    ],
    install_requires=['setuptools'],
    zip_safe=True,
    maintainer='ros2',
    maintainer_email='ros2@todo.todo',
    description='TODO: Package description',
    license='TODO: License declaration',
    tests_require=['pytest'],
    entry_points={
        'console_scripts': [
        ],
    },
)
```

无论该功能包的launch目录下有多少个launch文件，launch相关配置只需设置一次即可。

##### 3.编译

终端中进入当前工作空间，编译功能包：

```
colcon build --packages-select py01_launch
```

##### 4.执行

当前工作空间下，启动终端，输入如下指令：

```
. install/setup.bash
ros2 run py01_launch py00_base.launch.py
```

该指令运行的是 python 格式的 launch 文件，其他两个launch文件与之同理。最终，会启动两个 turtlesim\_node 节点。

---

**建议：**对于带有启动文件的功能包，最好在功能包的`package.xml`中添加对包`ros2launch`的执行时依赖：

```
<exec_depend>ros2launch</exec_depend>
```

这有助于确保在构建功能包后`ros2 launch`命令可用。它还确保可以识别不同格式的 launch 文件。

