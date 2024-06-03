### 1.3.3 HelloWorld（Python）

#### 1.创建功能包

终端下，进入ws00\_helloworld/src目录，使用如下指令创建一个python功能包：

```
ros2 pkg create pkg02_helloworld_py --build-type ament_python --dependencies rclpy --node-name helloworld
```

执行完毕，在src目录下将生成一个名为pkg02\_helloworld\_py的目录，且目录中已经默认生成了一些子级文件与文件夹。

#### 2.编辑源文件

进入pkg02\_helloworld\_py/pkg02\_helloworld\_py目录，该目录下有一helloworld.py文件，修改文件内容如下：

```py
import rclpy

def main():
    # 初始化 ROS2
    rclpy.init()
    # 创建节点
    node = rclpy.create_node("helloworld_py_node")
    # 输出文本
    node.get_logger().info("hello world!")
    # 释放资源
    rclpy.shutdown()


if __name__ == '__main__':
    main()
```

#### 3.编辑配置文件

与C++类似的，在步骤1创建功能包时所使用的指令也已经默认生成且配置了配置文件，不过实际应用中经常需要自己编辑配置文件，所以在此对相关内容做简单介绍，所使用的配置文件主要有两个，分别是功能包下的package.xml与setup.py。

##### 1.package.xml

文件内容如下：

```xml
<?xml version="1.0"?>
<?xml-model href="http://download.ros.org/schema/package_format3.xsd" schematypens="http://www.w3.org/2001/XMLSchema"?>
<package format="3">
  <name>pkg02_helloworld_py</name>
  <version>0.0.0</version>
  <description>TODO: Package description</description>
  <maintainer email="ros2@todo.todo">ros2</maintainer>
  <license>TODO: License declaration</license>

  <!-- 所需要依赖 -->
  <depend>rclpy</depend>

  <test_depend>ament_copyright</test_depend>
  <test_depend>ament_flake8</test_depend>
  <test_depend>ament_pep257</test_depend>
  <test_depend>python3-pytest</test_depend>

  <export>
    <build_type>ament_python</build_type>
  </export>
</package>
```

注释部分以后需要根据实际的包依赖进行添加或修改。

##### 2.setup.py

文件内容如下：

```py
from setuptools import setup

package_name = 'pkg02_helloworld_py'

setup(
    name=package_name,
    version='0.0.0',
    packages=[package_name],
    data_files=[
        ('share/ament_index/resource_index/packages',
            ['resource/' + package_name]),
        ('share/' + package_name, ['package.xml']),
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
            # 映射源文件与可执行文件
            'helloworld = pkg02_helloworld_py.helloworld:main'
        ],
    },
)
```

注释部分以后可能需要根据实际情况修改。

#### 4.编译

终端下进入到工作空间，执行如下指令：

```
colcon build
```

#### 5.执行

终端下进入到工作空间，执行如下指令：

```
. install/setup.bash
ros2 run pkg02_helloworld_py helloworld
```

程序执行，在终端下将输出文本："hello world!"。

