### 2.3.4 服务通信（Python）

#### 1.服务端实现

功能包py02\_service的py02\_service目录下，新建Python文件demo01\_server\_py.py，并编辑文件，输入如下内容：

```py
"""  
    需求：编写服务端，接收客户端发送请求，提取其中两个整型数据，相加后将结果响应回客户端。
    步骤：
        1.导包；
        2.初始化 ROS2 客户端；
        3.定义节点类；
            3-1.创建服务端；
            3-2.处理请求数据并响应结果。
        4.调用spin函数，并传入节点对象；
        5.释放资源。

"""

# 1.导包；
import rclpy
from rclpy.node import Node
from base_interfaces_demo.srv import AddInts

# 3.定义节点类；
class MinimalService(Node):

    def __init__(self):
        super().__init__('minimal_service_py')
        # 3-1.创建服务端；
        self.srv = self.create_service(AddInts, 'add_ints', self.add_two_ints_callback)
        self.get_logger().info("服务端启动！")

    # 3-2.处理请求数据并响应结果。
    def add_two_ints_callback(self, request, response):
        response.sum = request.num1 + request.num2
        self.get_logger().info('请求数据:(%d,%d),响应结果:%d' % (request.num1, request.num2, response.sum))
        return response


def main():
    # 2.初始化 ROS2 客户端；
    rclpy.init()
    # 4.调用spin函数，并传入节点对象；
    minimal_service = MinimalService()
    rclpy.spin(minimal_service)
    # 5.释放资源。
    rclpy.shutdown()


if __name__ == '__main__':
    main()
```

#### 2.客户端实现

功能包py02\_service的py02\_service目录下，新建Python文件demo02\_client\_py.py，并编辑文件，输入如下内容：

```py
"""  
    需求：编写客户端，发送两个整型变量作为请求数据，并处理响应结果。
    步骤：
        1.导包；
        2.初始化 ROS2 客户端；
        3.定义节点类；
            3-1.创建客户端；
            3-2.等待服务连接；
            3-3.组织请求数据并发送；
        4.创建对象调用其功能，处理响应结果；
        5.释放资源。

"""
# 1.导包；
import sys
import rclpy
from rclpy.node import Node
from base_interfaces_demo.srv import AddInts

# 3.定义节点类；
class MinimalClient(Node):

    def __init__(self):
        super().__init__('minimal_client_py')
        # 3-1.创建客户端；
        self.cli = self.create_client(AddInts, 'add_ints')
        # 3-2.等待服务连接；
        while not self.cli.wait_for_service(timeout_sec=1.0):
            self.get_logger().info('服务连接中，请稍候...')
        self.req = AddInts.Request()

    # 3-3.组织请求数据并发送；
    def send_request(self):
        self.req.num1 = int(sys.argv[1])
        self.req.num2 = int(sys.argv[2])
        self.future = self.cli.call_async(self.req)


def main():
    # 2.初始化 ROS2 客户端；
    rclpy.init()

    # 4.创建对象并调用其功能；
    minimal_client = MinimalClient()
    minimal_client.send_request()

    # 处理响应
    rclpy.spin_until_future_complete(minimal_client,minimal_client.future)
    try:
        response = minimal_client.future.result()
    except Exception as e:
        minimal_client.get_logger().info(
            '服务请求失败： %r' % (e,))
    else:
        minimal_client.get_logger().info(
            '响应结果： %d + %d = %d' %
            (minimal_client.req.num1, minimal_client.req.num2, response.sum))

    # 5.释放资源。
    rclpy.shutdown()


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
        'demo01_server_py = py02_service.demo01_server_py:main',
        'demo02_client_py = py02_service.demo02_client_py:main'
    ],
},
```

#### 4.编译

终端中进入当前工作空间，编译功能包：

```
colcon build --packages-select py02_service
```

#### 5.执行

当前工作空间下，启动两个终端，终端1执行服务端程序，终端2执行客户端程序。

终端1输入如下指令：

```
. install/setup.bash
ros2 run py02_service demo01_server_py
```

终端2输入如下指令：

```
. install/setup.bash
ros2 run py02_service demo02_client_py 100 200
```

最终运行结果与案例类似。

