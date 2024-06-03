### 5.6.3 乌龟跟随实现（Python）

#### 1.编写生成新乌龟实现

功能包 py05\_exercise 的 py05\_exercise 目录下，新建 Python 文件 exer01\_tf\_spawn\_py.py，并编辑文件，输入如下内容：

```py
"""  
  需求：编写客户端，发送请求生成一只新的乌龟。

  步骤：
    1.导包；
    2.初始化 ROS2 客户端；
    3.定义节点类；
      3-1.声明并获取参数；
      3-2.创建客户端；
      3-3.等待服务连接；
      3-4.组织请求数据并发送；
    4.创建对象调用其功能,并处理响应；
    5.释放资源。  
"""
# 1.导包；
import rclpy
from rclpy.node import Node
from turtlesim.srv import Spawn

# 3.定义节点类；
class TurtleSpawnClient(Node):

    def __init__(self):
        super().__init__('turtle_spawn_client_py')

        # 3-1.声明并获取参数；
        self.x = self.declare_parameter("x",2.0)
        self.y = self.declare_parameter("y",2.0)
        self.theta = self.declare_parameter("theta",0.0)
        self.turtle_name = self.declare_parameter("turtle_name","turtle2")

        # 3-2.创建客户端；
        self.cli = self.create_client(Spawn, '/spawn')
        # 3-3.等待服务连接；
        while not self.cli.wait_for_service(timeout_sec=1.0):
            self.get_logger().info('服务连接中，请稍候...')
        self.req = Spawn.Request()

    # 3-4.组织请求数据并发送；
    def send_request(self):
        self.req.x = self.get_parameter("x").get_parameter_value().double_value
        self.req.y = self.get_parameter("y").get_parameter_value().double_value
        self.req.theta = self.get_parameter("theta").get_parameter_value().double_value
        self.req.name = self.get_parameter("turtle_name").get_parameter_value().string_value
        self.future = self.cli.call_async(self.req)


def main():
    # 2.初始化 ROS2 客户端；
    rclpy.init()

    # 4.创建对象并调用其功能；
    client = TurtleSpawnClient()
    client.send_request()

    # 处理响应
    rclpy.spin_until_future_complete(client,client.future)
    try:
        response = client.future.result()
    except Exception as e:
        client.get_logger().info(
            '服务请求失败： %r' % e)
    else:
        if len(response.name) == 0:
            client.get_logger().info(
                '乌龟重名了，创建失败！')
        else:
            client.get_logger().info(
                '乌龟%s被创建' % response.name)

    # 5.释放资源。
    rclpy.shutdown()


if __name__ == '__main__':
    main()
```

#### 2.编写坐标变换广播实现

功能包 py05\_exercise 的 py05\_exercise 目录下，新建 Python 文件 exer02\_tf\_broadcaster\_py.py，并编辑文件，输入如下内容：

```py
"""  
  需求：发布乌龟坐标系到窗口坐标系的坐标变换。
  步骤：
    1.导包；
    2.初始化 ROS2 客户端；
    3.定义节点类；
      3-1.声明并解析乌龟名称参数；
      3-2.创建动态坐标变换发布方；
      3-3.创建乌龟位姿订阅方；
      3-4.根据订阅到的乌龟位姿生成坐标帧并广播。
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
class TurtleFrameBroadcaster(Node):

    def __init__(self):
        super().__init__('turtle_frame_broadcaster_py')
        # 3-1.声明并解析乌龟名称参数；
        self.declare_parameter('turtle_name', 'turtle1')
        self.turtlename = self.get_parameter('turtle_name').get_parameter_value().string_value

        # 3-2.创建动态坐标变换发布方；
        self.br = TransformBroadcaster(self)

        # 3-3.创建乌龟位姿订阅方；
        self.subscription = self.create_subscription(
            Pose,
            self.turtlename+ '/pose',
            self.handle_turtle_pose,
            1)
        self.subscription

    def handle_turtle_pose(self, msg):
        # 3-4.根据订阅到的乌龟位姿生成坐标帧并广播。
        t = TransformStamped()

        t.header.stamp = self.get_clock().now().to_msg()
        t.header.frame_id = 'world'
        t.child_frame_id = self.turtlename

        t.transform.translation.x = msg.x
        t.transform.translation.y = msg.y
        t.transform.translation.z = 0.0

        q = tf_transformations.quaternion_from_euler(0, 0, msg.theta)
        t.transform.rotation.x = q[0]
        t.transform.rotation.y = q[1]
        t.transform.rotation.z = q[2]
        t.transform.rotation.w = q[3]

        self.br.sendTransform(t)


def main():
    # 2.初始化 ROS2 客户端；
    rclpy.init()
    # 4.调用 spin 函数，并传入对象；
    node = TurtleFrameBroadcaster()
    try:
        rclpy.spin(node)
    except KeyboardInterrupt:
        pass
    # 5.释放资源。
    rclpy.shutdown()
```

#### 3.编写坐标变换监听实现

功能包 py05\_exercise 的 py05\_exercise 目录下，新建 Python 文件 exer03\_tf\_listener\_py.py，并编辑文件，输入如下内容：

```py
"""  
  需求：广播的坐标系消息，并生成 turtle2 相对于 turtle1 的坐标系数据，
       并进一步生成控制 turtle2 运动的速度指令。
  步骤：
    1.导包；
    2.初始化 ROS2 客户端；
    3.定义节点类；
      3-1.声明并解析参数；
      3-2.创建tf缓存对象指针；
      3-3.创建tf监听器；
      3-4.按照条件查找符合条件的坐标系并生成变换后的坐标帧。
    4.调用 spin 函数，并传入对象；
    5.释放资源。

"""
# 1.导包；
import math
from geometry_msgs.msg import Twist
import rclpy
from rclpy.node import Node
from tf2_ros import TransformException
from tf2_ros.buffer import Buffer
from tf2_ros.transform_listener import TransformListener

# 3.定义节点类；
class TurtleFrameListener(Node):

    def __init__(self):
        super().__init__('turtle_frame_listener_py')
        # 3-1.声明并解析参数；
        self.declare_parameter('target_frame', 'turtle2')
        self.declare_parameter('source_frame', 'turtle1')
        self.target_frame = self.get_parameter('target_frame').get_parameter_value().string_value
        self.source_frame = self.get_parameter('source_frame').get_parameter_value().string_value

        # 3-2.创建tf缓存对象指针；
        self.tf_buffer = Buffer()
        # 3-3.创建tf监听器；
        self.tf_listener = TransformListener(self.tf_buffer, self)
        self.publisher = self.create_publisher(Twist, self.target_frame + '/cmd_vel', 1)

        self.timer = self.create_timer(1.0, self.on_timer)

    def on_timer(self):
        # 3-4.按照条件查找符合条件的坐标系并生成变换后的坐标帧。
        try:
            now = rclpy.time.Time()
            trans = self.tf_buffer.lookup_transform(
                self.target_frame,
                self.source_frame,
                now)
        except TransformException as ex:
            self.get_logger().info(
                '%s 到 %s 坐标变换异常！' % (self.source_frame,self.target_frame))
            return

        # 3-5.生成 turtle2 的速度指令，并发布。
        msg = Twist()
        scale_rotation_rate = 1.0
        msg.angular.z = scale_rotation_rate * math.atan2(
            trans.transform.translation.y,
            trans.transform.translation.x)

        scale_forward_speed = 0.5
        msg.linear.x = scale_forward_speed * math.sqrt(
            trans.transform.translation.x ** 2 +
            trans.transform.translation.y ** 2)

        self.publisher.publish(msg)

def main():
    # 2.初始化 ROS2 客户端；
    rclpy.init()
    # 4.调用 spin 函数，并传入对象；
    node = TurtleFrameListener()
    try:
        rclpy.spin(node)
    except KeyboardInterrupt:
        pass

    # 5.释放资源。
    rclpy.shutdown()
```

#### 4.编写 launch 文件

launch 目录下新建文件 exer01\_turtle\_follow.launch.xml，并编辑文件，输入如下内容：

```xml
<launch>
    <!-- 乌龟准备 -->
    <node pkg="turtlesim" exec="turtlesim_node" name="t1" />
    <node pkg="py05_exercise" exec="exer01_tf_spawn_py" name="t2" />
    <!-- 发布坐标变换 -->
    <node pkg="py05_exercise" exec="exer02_tf_broadcaster_py" name="tf_broadcaster1_py">
    </node>
    <node pkg="py05_exercise" exec="exer02_tf_broadcaster_py" name="tf_broadcaster2_py">
        <param name="turtle_name" value="turtle2" />
    </node>
    <!-- 监听坐标变换 -->
    <node pkg="py05_exercise" exec="exer03_tf_listener_py" name="tf_listener_py">
        <param name="target_frame" value="turtle2" />
        <param name="source_frame" value="turtle1" />
    </node>
</launch>
```

#### 5.编辑配置文件

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
        'exer01_tf_spawn_py = py05_exercise.exer01_tf_spawn_py:main',
        'exer02_tf_broadcaster_py = py05_exercise.exer02_tf_broadcaster_py:main',
        'exer03_tf_listener_py = py05_exercise.exer03_tf_listener_py:main'
    ],
},
```

#### 6.编译

终端中进入当前工作空间，编译功能包：

```
colcon build --packages-select py05_exercise
```

#### 7.执行

当前工作空间下启动终端，输入如下命令运行launch文件：

```
. install/setup.bash 
ros2 launch py05_exercise exer01_turtle_follow.launch.xml
```

再新建终端，启动 turtlesim 键盘控制节点：

```
ros2 run turtlesim turtle_teleop_key
```

该终端下可以通过键盘控制 turtle1 运动，并且 turtle2 会跟随 turtle1 运动。最终的运行结果与演示案例类似。

