### 2.5.4 参数服务（Python）

#### 1.参数服务端

功能包py04\_param的py04\_param目录下，新建Python文件demo01\_param\_server\_py.py，并编辑文件，输入如下内容：

```py
"""  
    需求：编写参数服务端，设置并操作参数。
    步骤：
        1.导包；
        2.初始化 ROS2 客户端；
        3.定义节点类；
            3-1.声明参数；
            3-2.查询参数；
            3-3.修改参数；
            3-4.删除参数。
        4.创建节点对象，调用参数操作函数，并传递给spin函数；
        5.释放资源。

"""
# 1.导包；
import rclpy
from rclpy.node import Node

# 3.定义节点类；
class MinimalParamServer(Node):
    def __init__(self):
        super().__init__("minimal_param_server",allow_undeclared_parameters=True)
        self.get_logger().info("参数演示")

    # 3-1.声明参数；
    def declare_param(self):
        self.declare_parameter("car_type","Tiger")
        self.declare_parameter("height",1.50)
        self.declare_parameter("wheels",4)
        self.p1 = rclpy.Parameter("car_type",value = "Mouse")
        self.p2 = rclpy.Parameter("undcl_test",value = 100)
        self.set_parameters([self.p1,self.p2])

    # 3-2.查询参数；
    def get_param(self):
        self.get_logger().info("--------------查-------------")
        # 判断包含
        self.get_logger().info("包含 car_type 吗？%d" % self.has_parameter("car_type"))
        self.get_logger().info("包含 width 吗？%d" % self.has_parameter("width"))
        # 获取指定
        car_type = self.get_parameter("car_type")
        self.get_logger().info("%s = %s " % (car_type.name, car_type.value))
        # 获取所有
        params = self.get_parameters(["car_type","height","wheels"])
        self.get_logger().info("解析所有参数:")
        for param in params:
            self.get_logger().info("%s ---> %s" % (param.name, param.value))

    # 3-3.修改参数；
    def update_param(self):
        self.get_logger().info("--------------改-------------")
        self.set_parameters([rclpy.Parameter("car_type",value = "horse")])
        param = self.get_parameter("car_type")
        self.get_logger().info("修改后: car_type = %s" %param.value)

    # 3-4.删除参数。
    def del_param(self):
        self.get_logger().info("--------------删-------------")
        self.get_logger().info("删除操作前包含 car_type 吗？%d" % self.has_parameter("car_type"))
        self.undeclare_parameter("car_type")
        self.get_logger().info("删除操作后包含 car_type 吗？%d" % self.has_parameter("car_type"))


def main():
    # 2.初始化 ROS2 客户端；
    rclpy.init()

    # 4.创建节点对象，调用参数操作函数，并传递给spin函数；
    param_server = MinimalParamServer()
    param_server.declare_param()
    param_server.get_param()
    param_server.update_param()
    param_server.del_param()

    rclpy.spin(param_server)

    # 5.释放资源。
    rclpy.shutdown()


if __name__ == "__main__":
    main()
```

#### 2.参数客户端

ROS2的Python客户端暂时没有提供参数客户端专用的API，但是参数服务的底层是基于服务通信的，所以可以通过服务通信操作参数服务端的参数。

#### 3.编辑配置文件

##### 1.package.xml

在创建功能包时，所依赖的功能包已经自动配置了，配置内容如下：

```
<depend>rclpy</depend>
```

##### 2.setup.py

`entry_points`字段的`console_scripts`中添加如下内容：

```
entry_points={
    'console_scripts': [
        'demo01_param_server_py = py04_param.demo01_param_server_py:main'
    ],
},
```

#### 4.编译

终端中进入当前工作空间，编译功能包：

```
colcon build --packages-select py04_param
```

#### 5.执行

当前工作空间下，启动两个终端，终端1执行参数服务端程序，终端2执行参数客户端程序\(使用2.5.3中的C++实现\)。

终端1输入如下指令：

```
. install/setup.bash
ros2 run py04_param demo01_param_server_py
```

终端2输入如下指令：

```
. install/setup.bash
ros2 run cpp04_param demo02_param_client
```

最终运行结果与案例类似。

---

**资料：**

以服务通信方式操作参数服务端示例代码：

```py
# 1.导包
import rclpy
from rclpy.node import Node
from rcl_interfaces.srv import ListParameters
from rcl_interfaces.srv import GetParameters
from rcl_interfaces.srv import SetParameters
from rcl_interfaces.msg import ParameterType
from rcl_interfaces.msg import Parameter
from rcl_interfaces.msg import ParameterValue
from ros2param.api import get_parameter_value

class MinimalParamClient(Node):

    def __init__(self):
        super().__init__('minimal_param_client_py')

    def list_params(self):
        # 3-1.创建客户端；
        cli_list = self.create_client(ListParameters, '/minimal_param_server/list_parameters')
        # 3-2.等待服务连接；
        while not cli_list.wait_for_service(timeout_sec=1.0):
            self.get_logger().info('列出参数服务连接中，请稍候...')
        req = ListParameters.Request()
        future = cli_list.call_async(req)
        rclpy.spin_until_future_complete(self,future)
        return future.result()

    def get_params(self,names):
        # 3-1.创建客户端；
        cli_get = self.create_client(GetParameters, '/minimal_param_server/get_parameters')
        # 3-2.等待服务连接；
        while not cli_get.wait_for_service(timeout_sec=1.0):
            self.get_logger().info('列出参数服务连接中，请稍候...')
        req = GetParameters.Request()
        req.names = names
        future = cli_get.call_async(req)
        rclpy.spin_until_future_complete(self,future)
        return future.result()

    def set_params(self):
        # 3-1.创建客户端；
        cli_set = self.create_client(SetParameters, '/minimal_param_server/set_parameters')
        # 3-2.等待服务连接；
        while not cli_set.wait_for_service(timeout_sec=1.0):
            self.get_logger().info('列出参数服务连接中，请稍候...')

        req = SetParameters.Request()

        p1 = Parameter()
        p1.name = "car_type"

        # v1 = ParameterValue()
        # v1.type = ParameterType.PARAMETER_STRING
        # v1.string_value = "Pig"
        # p1.value = v1
        p1.value = get_parameter_value(string_value="Pig")

        p2 = Parameter()
        p2.name = "height"

        v2 = ParameterValue()
        v2.type = ParameterType.PARAMETER_DOUBLE
        v2.double_value = 0.3
        p2.value = v2
        # p2.value = get_parameter_value(string_value="0.3")

        req.parameters = [p1, p2]
        future = cli_set.call_async(req)
        rclpy.spin_until_future_complete(self,future)
        return future.result()

def main():
    # 2.初始化 ROS2 客户端；
    rclpy.init()
    # 4.创建对象并调用其功能；
    client = MinimalParamClient()

    # 获取参数列表
    client.get_logger().info("---------获取参数列表---------")
    response = client.list_params()
    for name in response.result.names:
        client.get_logger().info(name)

    client.get_logger().info("---------获取参数---------")
    names = ["height","car_type"]
    response = client.get_params(names)
    # print(response.values)
    for v in response.values:
        if v.type == ParameterType.PARAMETER_STRING:
            client.get_logger().info("字符串值:%s" % v.string_value)
        elif v.type == ParameterType.PARAMETER_DOUBLE:
            client.get_logger().info("浮点值:%.2f" % v.double_value)

    client.get_logger().info("---------设置参数---------")
    response = client.set_params()
    results = response.results
    client.get_logger().info("设置了%d个参数" % len(results))
    for result in results:
        if not result.successful:
            client.get_logger().info("参数设置失败")
    rclpy.shutdown()

if __name__ == "__main__":
    main()
```



