### 5.3.6 动态广播器（Python）

#### 1.广播实现

功能包 py03\_tf\_broadcaster 的 py03\_tf\_broadcaster 目录下，新建 Python 文件 demo02\_dynamic\_tf\_broadcaster\_py.py，并编辑文件，输入如下内容：

```py
"""  
  需求：编写动态坐标变换程序，启动 turtlesim_node 以及 turtle_teleop_key 后，该程序可以发布
       乌龟坐标系到窗口坐标系的坐标变换，并且键盘控制乌龟运动时，乌龟坐标系与窗口坐标系的相对关系
       也会实时更新。

  步骤：
    1.导包；
    2.初始化 ROS 客户端；
    3.定义节点类；
      3-1.创建动态坐标变换发布方；
      3-2.创建乌龟位姿订阅方；
      3-3.根据订阅到的乌龟位姿生成坐标帧并广播。
    4.调用 spin 函数，并传入对象；
    5.释放资源。
"""
# 1.导包；
from geometry_msgs.msg import TransformStamped
import rclpy
from rclpy.node import Node
from tf2_ros import TransformBroadcaster
import tf_transformations
from turtlesim.msg import Pose

# 3.定义节点类；
class MinimalDynamicFrameBroadcasterPy(Node):

    def __init__(self):
        super().__init__('minimal_dynamic_frame_broadcaster_py')

        # 3-1.创建动态坐标变换发布方；
        self.br = TransformBroadcaster(self)
        # 3-2.创建乌龟位姿订阅方；
        self.subscription = self.create_subscription(
            Pose,
            '/turtle1/pose',
            self.handle_turtle_pose,
            1)
        self.subscription
    # 3-3.根据订阅到的乌龟位姿生成坐标帧并广播。
    def handle_turtle_pose(self, msg):
        # 组织消息
        t = TransformStamped()

        t.header.stamp = self.get_clock().now().to_msg()
        t.header.frame_id = 'world'
        t.child_frame_id = 'turtle1'

        t.transform.translation.x = msg.x
        t.transform.translation.y = msg.y
        t.transform.translation.z = 0.0

        q = tf_transformations.quaternion_from_euler(0, 0, msg.theta)
        t.transform.rotation.x = q[0]
        t.transform.rotation.y = q[1]
        t.transform.rotation.z = q[2]
        t.transform.rotation.w = q[3]

        # 发布消息
        self.br.sendTransform(t)


def main():
    # 2.初始化 ROS 客户端；
    rclpy.init()
    # 4.调用 spin 函数，并传入对象；
    node = MinimalDynamicFrameBroadcasterPy()
    try:
        rclpy.spin(node)
    except KeyboardInterrupt:
        pass
    # 5.释放资源。
    rclpy.shutdown()
```

#### 2.编辑配置文件

package.xml 无需修改，需要修改 setup.py 文件，`entry_points`字段的`console_scripts`中修改为如下内容：

```
entry_points={
    'console_scripts': [
        'demo01_static_tf_broadcaster_py = py03_tf_broadcaster.demo01_static_tf_broadcaster_py:main',
        'demo02_dynamic_tf_broadcaster_py = py03_tf_broadcaster.demo02_dynamic_tf_broadcaster_py:main'
    ],
},
```

#### 3.编译

终端中进入当前工作空间，编译功能包：

```
colcon build --packages-select py03_tf_broadcaster
```

#### 4.执行

启动两个终端，终端1输入如下命令：

```
ros2 run turtlesim turtlesim_node
```

终端2输入如下命令：

```
ros2 run turtlesim turtle_teleop_key
```

再在当前工作空间下，启动终端，输入如下命令：

```
. install/setup.bash
ros2 run py03_tf_broadcaster demo02_dynamic_tf_broadcaster_py
```

#### 5.rviz2 查看坐标系关系

参考 **5.3.2 静态广播器（命令）**内容启动并配置 rviz2（Global Options 中的 Fixed Frame 设置为 world），通过键盘控制乌龟运动，最终执行结果与案例2类似。

---

**思考题：**

> 我们可以在 turtlesim\_node 的窗口中，生成多只乌龟，如何为每只乌龟都广播自身到“world”的动态坐标变换呢？



