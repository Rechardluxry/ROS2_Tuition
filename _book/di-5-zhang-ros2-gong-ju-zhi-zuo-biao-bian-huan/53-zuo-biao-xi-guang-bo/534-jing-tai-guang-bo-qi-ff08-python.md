### 5.3.4 静态广播器（Python）

#### 1.广播实现

功能包 py03\_tf\_broadcaster 的 py03\_tf\_broadcaster 目录下，新建 Python 文件 demo01\_static\_tf\_broadcaster\_py.py，并编辑文件，输入如下内容：

```py
"""  
    需求：编写静态坐标变换程序，执行时传入两个坐标系的相对位姿关系以及父子级坐标系id，
         程序运行发布静态坐标变换。
    步骤：
        1.导包；
        2.判断终端传入的参数是否合法；
        3.初始化 ROS 客户端；
        4.定义节点类；
            4-1.创建静态坐标变换发布方；
            4-2.组织并发布消息。
        5.调用 spin 函数，并传入对象；
        6.释放资源。 

"""
# 1.导包；
import sys
from geometry_msgs.msg import TransformStamped
import rclpy
from rclpy.node import Node
from tf2_ros.static_transform_broadcaster import StaticTransformBroadcaster
import tf_transformations

# 4.定义节点类；
class MinimalStaticFrameBroadcasterPy(Node):

    def __init__(self, transformation):
        super().__init__('minimal_static_frame_broadcaster_py')
        # 4-1.创建静态坐标变换发布方；
        self._tf_publisher = StaticTransformBroadcaster(self)
        self.make_transforms(transformation)

    # 4-2.组织并发布消息。
    def make_transforms(self, transformation):
        # 组织消息
        static_transformStamped = TransformStamped()
        static_transformStamped.header.stamp = self.get_clock().now().to_msg()
        static_transformStamped.header.frame_id = sys.argv[7]
        static_transformStamped.child_frame_id = sys.argv[8]
        static_transformStamped.transform.translation.x = float(sys.argv[1])
        static_transformStamped.transform.translation.y = float(sys.argv[2])
        static_transformStamped.transform.translation.z = float(sys.argv[3])
        quat = tf_transformations.quaternion_from_euler(
            float(sys.argv[4]), float(sys.argv[5]), float(sys.argv[6]))
        static_transformStamped.transform.rotation.x = quat[0]
        static_transformStamped.transform.rotation.y = quat[1]
        static_transformStamped.transform.rotation.z = quat[2]
        static_transformStamped.transform.rotation.w = quat[3]
        # 发布消息
        self._tf_publisher.sendTransform(static_transformStamped)


def main():
    # 2.判断终端传入的参数是否合法；
    logger = rclpy.logging.get_logger('logger')
    if len(sys.argv) < 9:
        logger.info('运行程序时请按照：x y z roll pitch yaw frame_id child_frame_id 的格式传入参数')
        sys.exit(0)

    # 3.初始化 ROS 客户端；
    rclpy.init()
    # 5.调用 spin 函数，并传入对象；
    node = MinimalStaticFrameBroadcasterPy(sys.argv)
    try:
        rclpy.spin(node)
    except KeyboardInterrupt:
        pass

    # 6.释放资源。 
    rclpy.shutdown()
```

#### 2.编辑配置文件

##### 1.package.xml

在创建功能包时，所依赖的功能包已经自动配置了，配置内容如下：

```xml
<depend>rclpy</depend>
<depend>tf_transformations</depend>
<depend>tf2_ros</depend>
<depend>geometry_msgs</depend>
<depend>turtlesim</depend>
```

##### 2.setup.py

`entry_points`字段的`console_scripts`中添加如下内容：

```py
entry_points={
    'console_scripts': [
        'demo01_static_tf_broadcaster_py = py03_tf_broadcaster.demo01_static_tf_broadcaster_py:main'
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

终端2输入如下命令发布摄像头（camera）相对于底盘（base\_link）的静态坐标变换：

```
. install/setup.bash 
ros2 run py03_tf_broadcaster demo01_static_tf_broadcaster_py -0.5 0 0.4 0 0 0 base_link camera
```

#### 5.rviz2 查看坐标系关系

参考 **5.3.2 静态广播器（命令）**内容启动并配置 rviz2，最终执行结果与案例1类似。

