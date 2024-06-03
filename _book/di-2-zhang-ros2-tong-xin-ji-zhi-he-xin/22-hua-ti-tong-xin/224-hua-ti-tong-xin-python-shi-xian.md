### 2.2.6 话题通信之自定义消息（Python）

**准备**

> Python文件中导入自定义消息相关的包时，为了方便使用，可以配置VSCode中settings.json文件，在文件中的python.autoComplete.extraPaths和python.analysis.extraPaths属性下添加一行："${workspaceFolder}/install/base\_interfaces\_demo/local/lib/python3.10/dist-packages"
>
> 添加完毕后，代码可以高亮显示且可以自动补齐，其他接口文件或接口包的使用也与此同理。

#### 1.发布方实现

功能包py01\_topic的py01\_topic目录下，新建Python文件demo03\_talker\_stu\_py.py，并编辑文件，输入如下内容：

```py
"""  
    需求：以某个固定频率发送文本学生信息，包含学生的姓名、年龄、身高等数据。
"""
# 1.导包；
import rclpy
from rclpy.node import Node
from base_interfaces_demo.msg import Student

# 3.定义节点类；
class MinimalPublisher(Node):

    def __init__(self):
        super().__init__('stu_publisher_py')
        # 3-1.创建发布方；
        self.publisher_ = self.create_publisher(Student, 'topic_stu', 10)
        # 3-2.创建定时器；
        timer_period = 0.5 
        self.timer = self.create_timer(timer_period, self.timer_callback)
        self.i = 0

    # 3-3.组织消息并发布。
    def timer_callback(self):
        stu = Student()
        stu.name = "李四"
        stu.age = self.i
        stu.height = 1.70
        self.publisher_.publish(stu)
        self.get_logger().info('发布的学生消息(py): name=%s,age=%d,height=%.2f' % (stu.name, stu.age, stu.height))
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

功能包py01\_topic的py01\_topic目录下，新建Python文件demo04\_listener\_stu\_py.py，并编辑文件，输入如下内容：

```py
"""  
    需求：订阅发布方发布的学生消息，并输出到终端。
"""

# 1.导包；
import rclpy
from rclpy.node import Node
from base_interfaces_demo.msg import Student

# 3.定义节点类；
class MinimalSubscriber(Node):

    def __init__(self):
        super().__init__('stu_subscriber_py')
        # 3-1.创建订阅方；
        self.subscription = self.create_subscription(
            Student,
            'topic_stu',
            self.listener_callback,
            10)
        self.subscription  

    # 3-2.处理订阅到的消息。
    def listener_callback(self, stu):
        self.get_logger().info('订阅的消息(py): name=%s,age=%d,height=%.2f' % (stu.name, stu.age, stu.height))


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

package.xml无需修改，需要修改setup.py文件，`entry_points`字段的`console_scripts`中修改为如下内容：

```
entry_points={
    'console_scripts': [
        'demo01_talker_str_py = py01_topic.demo01_talker_str_py:main',
        'demo02_listener_str_py = py01_topic.demo02_listener_str_py:main',
        'demo03_talker_stu_py = py01_topic.demo03_talker_stu_py:main',
        'demo04_listener_stu_py = py01_topic.demo04_listener_stu_py:main'
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
ros2 run py01_topic demo03_talker_stu_py
```

终端2输入如下指令：

```
. install/setup.bash 
ros2 run py01_topic demo04_listener_stu_py
```

最终运行结果与案例2类似。

---



