### 2.4.4 动作通信（Python）

#### 1.动作服务端实现

功能包py03\_action的py03\_action目录下，新建Python文件demo01\_action\_server\_py.py，并编辑文件，输入如下内容：

```py
"""  
    需求：编写动作服务端实习，可以提取客户端请求提交的整型数据，并累加从1到该数据之间的所有整数以求和，
       每累加一次都计算当前运算进度并连续反馈回客户端，最后，在将求和结果返回给客户端。
    步骤：
        1.导包；
        2.初始化 ROS2 客户端；
        3.定义节点类；
            3-1.创建动作服务端；
            3-2.生成连续反馈；
            3-3.生成最终响应。
        4.调用spin函数，并传入节点对象；
        5.释放资源。
"""

# 1.导包；
import time
import rclpy
from rclpy.action import ActionServer
from rclpy.node import Node

from base_interfaces_demo.action import Progress

# 3.定义节点类；
class ProgressActionServer(Node):

    def __init__(self):
        super().__init__('progress_action_server')
        # 3-1.创建动作服务端；
        self._action_server = ActionServer(
            self,
            Progress,
            'get_sum',
            self.execute_callback)
        self.get_logger().info('动作服务已经启动！')

    def execute_callback(self, goal_handle):
        self.get_logger().info('开始执行任务....')


        # 3-2.生成连续反馈；
        feedback_msg = Progress.Feedback()

        sum = 0
        for i in range(1, goal_handle.request.num + 1):
            sum += i
            feedback_msg.progress = i / goal_handle.request.num
            self.get_logger().info('连续反馈: %.2f' % feedback_msg.progress)
            goal_handle.publish_feedback(feedback_msg)
            time.sleep(1)

        # 3-3.生成最终响应。
        goal_handle.succeed()
        result = Progress.Result()
        result.sum = sum
        self.get_logger().info('任务完成！')

        return result


def main(args=None):

    # 2.初始化 ROS2 客户端；
    rclpy.init(args=args)

    # 4.调用spin函数，并传入节点对象；
    Progress_action_server = ProgressActionServer()
    rclpy.spin(Progress_action_server)

    # 5.释放资源。
    rclpy.shutdown()

if __name__ == '__main__':
    main()
```

#### 2.动作客户端实现

功能包py03\_action的py03\_action目录下，新建Python文件demo02\_action\_client\_py.py，并编辑文件，输入如下内容：

```py
"""  
    需求：编写动作客户端实现，可以提交一个整型数据到服务端，并处理服务端的连续反馈以及最终返回结果。
    步骤：
        1.导包；
        2.初始化 ROS2 客户端；
        3.定义节点类；
            3-1.创建动作客户端；
            3-2.发送请求；
            3-3.处理目标发送后的反馈；
            3-4.处理连续反馈；
            3-5.处理最终响应。
        4.调用spin函数，并传入节点对象；
        5.释放资源。

"""
# 1.导包；
import rclpy
from rclpy.action import ActionClient
from rclpy.node import Node
from base_interfaces_demo.action import Progress

# 3.定义节点类；
class ProgressActionClient(Node):

    def __init__(self):
        super().__init__('progress_action_client')
        # 3-1.创建动作客户端；
        self._action_client = ActionClient(self, Progress, 'get_sum')

    def send_goal(self, num):
        # 3-2.发送请求；
        goal_msg = Progress.Goal()
        goal_msg.num = num
        self._action_client.wait_for_server()
        self._send_goal_future = self._action_client.send_goal_async(goal_msg, feedback_callback=self.feedback_callback)
        self._send_goal_future.add_done_callback(self.goal_response_callback)

    def goal_response_callback(self, future):
        # 3-3.处理目标发送后的反馈；
        goal_handle = future.result()
        if not goal_handle.accepted:
            self.get_logger().info('请求被拒绝')
            return

        self.get_logger().info('请求被接收，开始执行任务！')

        self._get_result_future = goal_handle.get_result_async()
        self._get_result_future.add_done_callback(self.get_result_callback)

    # 3-5.处理最终响应。
    def get_result_callback(self, future):
        result = future.result().result
        self.get_logger().info('最终计算结果：sum = %d' % result.sum)
        # 5.释放资源。
        rclpy.shutdown()

    # 3-4.处理连续反馈；
    def feedback_callback(self, feedback_msg):
        feedback = (int)(feedback_msg.feedback.progress * 100)
        self.get_logger().info('当前进度: %d%%' % feedback)


def main(args=None):

    # 2.初始化 ROS2 客户端；
    rclpy.init(args=args)
    # 4.调用spin函数，并传入节点对象；

    action_client = ProgressActionClient()
    action_client.send_goal(10)
    rclpy.spin(action_client)

    # rclpy.shutdown()


if __name__ == '__main__':
    main()
```

#### 3.编辑配置文件

##### 1.package.xml

在创建功能包时，所依赖的功能包已经自动配置了，配置内容如下：

```
<depend>rclpy</depend>
<depend>base_interfaces_demo</depend>
```

##### 2.setup.py

`entry_points`字段的`console_scripts`中添加如下内容：

```
entry_points={
    'console_scripts': [
        'demo01_action_server_py = py03_action.demo01_action_server_py:main',
        'demo02_action_client_py = py03_action.demo02_action_client_py:main'
    ],
},
```

#### 4.编译

终端中进入当前工作空间，编译功能包：

```
colcon build --packages-select py03_action
```

#### 5.执行

当前工作空间下，启动两个终端，终端1执行动作服务端程序，终端2执行动作客户端程序。

终端1输入如下指令：

```
. install/setup.bash
ros2 run py03_action demo01_action_server_py
```

终端2输入如下指令：

```
. install/setup.bash
ros2 run py03_action demo02_action_client_py
```

最终运行结果与案例类似。

