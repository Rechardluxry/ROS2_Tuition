### 2.2.3 话题通信之原生消息（Python）

#### 1.发布方实现

功能包py01\_topic的py01\_topic目录下，新建Python文件demo01\_talker\_str\_py.py，并编辑文件，输入如下内容：

```py
"""  
    需求：以某个固定频率发送文本“hello world!”，文本后缀编号，每发送一条消息，编号递增1。
    步骤：
        1.导包；
        2.初始化 ROS2 客户端；
        3.定义节点类；
            3-1.创建发布方；
            3-2.创建定时器；
            3-3.组织消息并发布。
        4.调用spin函数，并传入节点对象；
        5.释放资源。
"""
# 1.导包；
import rclpy
from rclpy.node import Node
from std_msgs.msg import String


# 3.定义节点类；
class MinimalPublisher(Node):

    def __init__(self):
        super().__init__('minimal_publisher_py')
        # 3-1.创建发布方；
        self.publisher_ = self.create_publisher(String, 'topic', 10)
        # 3-2.创建定时器；
        timer_period = 0.5 
        self.timer = self.create_timer(timer_period, self.timer_callback)
        self.i = 0

    # 3-3.组织消息并发布。
    def timer_callback(self):
        msg = String()
        msg.data = 'Hello World(py): %d' % self.i
        self.publisher_.publish(msg)
        self.get_logger().info('发布的消息: "%s"' % msg.data)
        self.i += 1


def main(args=None):
    # 2.初始化 ROS2 客户端；
    rclpy.init(args=args)
    # 4.调用spin函数，并传入节点对象；
    minimal_publisher = MinimalPublisher()
    rclpy.spin(minimal_publisher)
    # 5.释放资源。
    rclpy.shutdown()


if __name__ == '__main__':
    main()
```

#### 2.订阅方实现

功能包py01\_topic的py01\_topic目录下，新建Python文件demo02\_listener\_str\_py.py，并编辑文件，输入如下内容：

```py
"""  
    需求：订阅发布方发布的消息，并输出到终端。
    步骤：
        1.导包；
        2.初始化 ROS2 客户端；
        3.定义节点类；
            3-1.创建订阅方；
            3-2.处理订阅到的消息。
        4.调用spin函数，并传入节点对象；
        5.释放资源。
"""

# 1.导包；
import rclpy
from rclpy.node import Node
from std_msgs.msg import String


# 3.定义节点类；
class MinimalSubscriber(Node):

    def __init__(self):
        super().__init__('minimal_subscriber_py')
        # 3-1.创建订阅方；
        self.subscription = self.create_subscription(
            String,
            'topic',
            self.listener_callback,
            10)
        self.subscription  

    # 3-2.处理订阅到的消息。
    def listener_callback(self, msg):
        self.get_logger().info('订阅的消息: "%s"' % msg.data)


def main(args=None):
    # 2.初始化 ROS2 客户端；
    rclpy.init(args=args)

    # 4.调用spin函数，并传入节点对象；
    minimal_subscriber = MinimalSubscriber()
    rclpy.spin(minimal_subscriber)

    # 5.释放资源。
    rclpy.shutdown()


if __name__ == '__main__':
    main()
```

#### 3.编辑配置文件

在Python功能包中，配置文件主要关注package.xml与setup.py。

##### 1.package.xml

在创建功能包时，所依赖的功能包已经自动配置了，配置内容如下：

```XML
<depend>rclcpp</depend>
<depend>std_msgs</depend>
<depend>base_interfaces_demo</depend>
```

需要说明的是和上一节C++实现一样`<depend>base_interfaces_demo</depend>`在本案例中不是必须的。

##### 2.setup.py

`entry_points`字段的`console_scripts`中添加如下内容：

```py
entry_points={
    'console_scripts': [
        'demo01_talker_str_py = py01_topic.demo01_talker_str_py:main',
        'demo02_listener_str_py = py01_topic.demo02_listener_str_py:main'
    ],
},
```

#### 4.编译

终端中进入当前工作空间，编译功能包：

```
colcon build --packages-select py01_topic
```

#### 5.执行

当前工作空间下，启动两个终端，终端1执行发布程序，终端2执行订阅程序。

终端1输入如下指令：

```
. install/setup.bash
ros2 run py01_topic demo01_talker_str_py
```

终端2输入如下指令：

```
. install/setup.bash 
ros2 run py01_topic demo02_listener_str_py
```

最终运行结果与案例1类似。

