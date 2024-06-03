### 5.3.9 坐标点发布（Python）

#### 1.话题发布实现

功能包 py03\_tf\_broadcaster 的 py03\_tf\_broadcaster 目录下，新建 Python 文件 demo03\_point\_publisher\_py.py，并编辑文件，输入如下内容：

```py
"""  
    需求：发布雷达坐标系中某个坐标点相对于雷达（laser）坐标系的位姿。
    步骤：
        1.导包；
        2.初始化 ROS 客户端；
        3.定义节点类；
            3-1.创建坐标点发布方；
            3-2.创建定时器；
            3-3.组织并发布坐标点消息。
        4.调用 spin 函数，并传入对象；
        5.释放资源。
"""
# 1.导包；
from geometry_msgs.msg import PointStamped
import rclpy
from rclpy.node import Node

# 3.定义节点类；
class MinimalPointPublisher(Node):

    def __init__(self):
        super().__init__('minimal_point_publisher_py')
        # 3-1.创建坐标点发布方；
        self.pub = self.create_publisher(PointStamped, 'point', 10)
        # 3-2.创建定时器；
        self.timer = self.create_timer(1.0, self.on_timer)
        self.x = 0.1
    def on_timer(self):
        # 3-3.组织并发布坐标点消息。
        ps = PointStamped()
        ps.header.stamp = self.get_clock().now().to_msg()
        ps.header.frame_id = 'laser'
        self.x += 0.02
        ps.point.x = self.x
        ps.point.y = 0.0
        ps.point.z = 0.2
        self.pub.publish(ps)


def main():
    # 2.初始化 ROS 客户端；
    rclpy.init()
    # 4.调用 spin 函数，并传入对象；
    node = MinimalPointPublisher()
    rclpy.spin(node)
    # 5.释放资源。3-3.组织并发布坐标点消息。
```

#### 2.编辑配置文件

package.xml 无需修改，需要修改 setup.py 文件，`entry_points`字段的`console_scripts`中修改为如下内容：

```py
entry_points={
    'console_scripts': [
        'demo01_static_tf_broadcaster_py = py03_tf_broadcaster.demo01_static_tf_broadcaster_py:main',
        'demo02_dynamic_tf_broadcaster_py = py03_tf_broadcaster.demo02_dynamic_tf_broadcaster_py:main',
        'demo03_point_publisher_py = py03_tf_broadcaster.demo03_point_publisher_py:main'
    ],
},
```

#### 3.编译

终端中进入当前工作空间，编译功能包：

```
colcon build --packages-select py03_tf_broadcaster
```

#### 4.执行

当前工作空间下，启动两个终端，终端1输入如下命令发布雷达（laser）相对于底盘（base\_link）的静态坐标变换：

```
. install/setup.bash 
ros2 run py03_tf_broadcaster demo01_static_tf_broadcaster_py 0.4 0 0.2 0 0 0 base_link laser
```

终端2输入如下命令发布障碍物相对于雷达（laser）的坐标点：

```
. install/setup.bash 
ros2 run py03_tf_broadcaster demo03_point_publisher_py
```

#### 5.rviz2 查看坐标系关系

参考** 5.3.8 坐标点发布（C++） **操作。

