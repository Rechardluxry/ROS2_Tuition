### 4.4.3 rosbag2 编程（Python）

#### 1.序列化

功能包 py02\_rosbag 的 py02\_rosbag 目录下，新建 Python 文件 demo01\_writer\_py.py，并编辑文件，输入如下内容：

```py
"""  
  需求：录制 turtle_teleop_key 节点发布的速度指令。
  步骤：
    1.导包；
    2.初始化 ROS 客户端；
    3.定义节点类；
      3-1.创建写出对象；
      3-2.设置写出的目标文件、话题等参数；
      3-3.写出消息。
    4.调用 spin 函数，并传入对象；
    5.释放资源。

"""
# 1.导包；
import rclpy
from rclpy.node import Node
from rclpy.serialization import serialize_message
from geometry_msgs.msg import Twist
import rosbag2_py
# 3.定义节点类；
class SimpleBagRecorder(Node):
    def __init__(self):
        super().__init__('simple_bag_recorder_py')
        # 3-1.创建写出对象；
        self.writer = rosbag2_py.SequentialWriter()
        # 3-2.设置写出的目标文件、话题等参数；
        storage_options = rosbag2_py._storage.StorageOptions(
            uri='my_bag_py',
            storage_id='sqlite3')
        converter_options = rosbag2_py._storage.ConverterOptions('', '')
        self.writer.open(storage_options, converter_options)

        topic_info = rosbag2_py._storage.TopicMetadata(
            name='/turtle1/cmd_vel',
            type='geometry_msgs/msg/Twist',
            serialization_format='cdr')
        self.writer.create_topic(topic_info)

        self.subscription = self.create_subscription(
            Twist,
            '/turtle1/cmd_vel',
            self.topic_callback,
            10)
        self.subscription

    def topic_callback(self, msg):
        # 3-3.写出消息。
        self.writer.write(
            '/turtle1/cmd_vel',
            serialize_message(msg),
            self.get_clock().now().nanoseconds)


def main(args=None):
    # 2.初始化 ROS 客户端；
    rclpy.init(args=args)
    # 4.调用 spin 函数，并传入对象；
    sbr = SimpleBagRecorder()
    rclpy.spin(sbr)
    # 5.释放资源。
    rclpy.shutdown()


if __name__ == '__main__':
    main()
```

#### 2.反序列化

功能包 py02\_rosbag 的 py02\_rosbag 目录下，新建 Python 文件 demo02\_reader\_py.py，并编辑文件，输入如下内容：

```py
"""  
    需求：读取 bag 文件数据。
    步骤：
        1.导包；
        2.初始化 ROS 客户端；
        3.定义节点类；
            3-1.创建读取对象；
            3-2.设置读取的目标文件、话题等参数；
            3-3.读消息；
            3-4.关闭文件。
        4.调用 spin 函数，并传入对象；
        5.释放资源。

"""
# 1.导包；
import rclpy
from rclpy.node import Node
import rosbag2_py
from rclpy.logging import get_logger
# 3.定义节点类；
class SimpleBagPlayer(Node):
    def __init__(self):
        super().__init__('simple_bag_player_py')
        # 3-1.创建读取对象；
        self.reader = rosbag2_py.SequentialReader()
        # 3-2.设置读取的目标文件、话题等参数；
        storage_options = rosbag2_py._storage.StorageOptions(
                uri="my_bag_py",
                storage_id='sqlite3')
        converter_options = rosbag2_py._storage.ConverterOptions('', '')
        self.reader.open(storage_options,converter_options)

    def read(self):    
        # 3-3.读消息；
        while self.reader.has_next():
            msg = self.reader.read_next()
            get_logger("rclpy").info("topic = %s, time = %d, value=%s" % (msg[0], msg[2], msg[1]))

def main(args=None):
    # 2.初始化 ROS 客户端；
    rclpy.init(args=args)

    # 4.调用 spin 函数，并传入对象；
    reader = SimpleBagPlayer()
    reader.read()
    rclpy.spin(reader)
    # 5.释放资源。
    rclpy.shutdown()


if __name__ == '__main__':
    main()
```

#### 3.编辑配置文件

##### 1.package.xml

在创建功能包时，所依赖的功能包已经自动配置了，配置内容如下：

```XML
<depend>rclpy</depend>
<depend>rosbag2_py</depend>
<depend>geometry_msgs</depend>
```

##### 2.setup.py

`entry_points`字段的`console_scripts`中添加如下内容：

```
entry_points={
    'console_scripts': [
        'demo01_writer_py = py02_rosbag.demo01_writer_py:main',
        'demo02_reader_py = py02_rosbag.demo02_reader_py:main'
    ],
},
```

#### 4.编译

终端中进入当前工作空间，编译功能包：

```
colcon build --packages-select py02_rosbag
```

#### 5.执行

当前工作空间下，启动两个终端，终端1执行录制程序，终端2执行回放程序。

终端1输入如下指令：

```
. install/setup.bash
ros2 run py02_rosbag demo01_writer_py
```

执行完毕后，会在当前工作空间下生成一个名为 my\_bag\_py 的目录。

终端2输入如下指令：

```
. install/setup.bash 
ros2 run py02_rosbag demo02_reader_py
```

该程序运行会读取 my\_bag 中记录的数据，其结果是在终端打印录制的速度指令的话题、时间戳与其内容（二进制格式）。

