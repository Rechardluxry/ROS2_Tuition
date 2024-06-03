## 4.1 启动文件 launch 简介

#### 场景

如前所述，在机器人操作系统中，节点是程序的基本构成单元，一个完整的、系统性的功能模块是由若干节点组成的，启动某个功能模块时可能需要依次启动这些节点，比如：

> 以机器人的导航功能为例，涉及的节点主要有：
>
> * 底盘驱动；
> * 雷达驱动；
> * 摄像头驱动；
> * imu驱动；
> * 地图服务；
> * 路径规划；
> * 运动控制；
> * 环境感知；
> * 定位；
> * ......
>
> 并且不同节点启动时，可以还会涉及到各种参数的导入、节点间执行逻辑的处理等。

如果使用 `ros2 run`指令逐一执行节点的话，显然效率低下，基于此，ROS2中提供了launch模块用于实现节点的批量启动。

#### 概念

launch 字面意为“启动”、“发射”，在ROS2中主要用于启动程序。launch模块由launch文件与 `ros2 launch`命令组成，前者用于打包并配置节点，后者用于执行launch文件。

#### 作用

简化节点的配置与启动，提高程序的启动效率。

#### 准备工作

1.新建工作空间`ws02_tools`，本章后续编码实现都基于此工作空间；

2.终端下进入工作空间的src目录，调用如下两条命令分别创建C++功能包和Python功能包。

```
ros2 pkg create cpp01_launch --build-type ament_cmake --dependencies rclcpp
ros2 pkg create py01_launch --build-type ament_python --dependencies rclpy
```

3.在使用Python版的launch文件时，涉及的API众多，为了提高编码效率，可以在VScode中设置launch文件的代码模板，将VScode的配置文件python.json修改为为如下内容：

```json
{
    // Place your snippets for python here. Each snippet is defined under a snippet name and has a prefix, body and 
    // description. The prefix is what is used to trigger the snippet and the body will be expanded and inserted. Possible variables are:
    // $1, $2 for tab stops, $0 for the final cursor position, and ${1:label}, ${2:another} for placeholders. Placeholders with the 
    // same ids are connected.
    // Example:
    // "Print to console": {
    //     "prefix": "log",
    //     "body": [
    //         "console.log('$1');",
    //         "$2"
    //     ],
    //     "description": "Log output to console"
    // }

    "ros2 node": {
        "prefix": "ros2_node_py",
        "body": [
            "\"\"\"  ",
            "    需求：",
            "    流程：",
            "        1.导包；",
            "        2.初始化ROS2客户端；",
            "        3.自定义节点类；",
            "                        ",          
            "        4.调用spain函数，并传入节点对象；",
            "        5.资源释放。 ",
            "",
            "",
            "\"\"\"",
            "# 1.导包；",
            "import rclpy",
            "from rclpy.node import Node",
            "",
            "# 3.自定义节点类；",
            "class MyNode(Node):",
            "    def __init__(self):",
            "        super().__init__(\"mynode_node_py\")",
            "",
            "def main():",
            "    # 2.初始化ROS2客户端；",
            "    rclpy.init()",
            "    # 4.调用spain函数，并传入节点对象；",
            "    rclpy.spin(MyNode())",
            "    # 5.资源释放。 ",
            "    rclpy.shutdown()",
            "",
            "if __name__ == '__main__':",
            "    main()",
        ],
        "description": "ros2 node"
    },
    "ros2 launch py": {
        "prefix": "ros2_launch_py",
        "body": [
            "from launch import LaunchDescription",
            "from launch_ros.actions import Node",
            "# 封装终端指令相关类--------------",
            "# from launch.actions import ExecuteProcess",
            "# from launch.substitutions import FindExecutable",
            "# 参数声明与获取-----------------",
            "# from launch.actions import DeclareLaunchArgument",
            "# from launch.substitutions import LaunchConfiguration",
            "# 文件包含相关-------------------",
            "# from launch.actions import IncludeLaunchDescription",
            "# from launch.launch_description_sources import PythonLaunchDescriptionSource",
            "# 分组相关----------------------",
            "# from launch_ros.actions import PushRosNamespace",
            "# from launch.actions import GroupAction",
            "# 事件相关----------------------",
            "# from launch.event_handlers import OnProcessStart, OnProcessExit",
            "# from launch.actions import ExecuteProcess, RegisterEventHandler,LogInfo",
            "# 获取功能包下share目录路径-------",
            "# from ament_index_python.packages import get_package_share_directory",
            "",
            "def generate_launch_description():",
            "    ",    
            "    return LaunchDescription([])"
        ],
        "description": "ros2 launch"
    }
}
```



