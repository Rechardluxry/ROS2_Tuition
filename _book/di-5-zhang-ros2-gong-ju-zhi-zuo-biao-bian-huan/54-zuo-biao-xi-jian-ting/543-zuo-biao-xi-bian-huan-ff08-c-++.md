### 5.4.3 坐标系变换（Python）

#### 1.坐标系变换实现

功能包 py04\_tf\_listener 的 py04\_tf\_listener 目录下，新建 Python 文件 demo01\_tf\_listener\_py.py，并编辑文件，输入如下内容：

```py
"""  
  需求：订阅 laser 到 base_link 以及 camera 到 base_link 的坐标系关系，
       并生成 laser 到 camera 的坐标变换。
  步骤：
    1.导包；
    2.初始化 ROS 客户端；
    3.定义节点类；
      3-1.创建tf缓存对象指针；
      3-2.创建tf监听器；
      3-3.按照条件查找符合条件的坐标系并生成变换后的坐标帧。
    4.调用 spin 函数，并传入对象指针；
    5.释放资源。
"""

# 1.导包；
import rclpy
from rclpy.node import Node
from tf2_ros.buffer import Buffer
from tf2_ros.transform_listener import TransformListener
from rclpy.time import Time
 
# 3.自定义节点类；
class TFListenerPy(Node):
    def __init__(self):
        super().__init__("tf_listener_py_node_py")
        # 3-1.创建一个缓存对象，融合多个坐标系相对关系为一棵坐标树；
        self.buffer = Buffer()
        # 3-2.创建一个监听器，绑定缓存对象，会将所有广播器广播的数据写入缓存；
        self.listener = TransformListener(self.buffer,self)
        # 3-3.编写一个定时器，循环实现转换。
        self.timer = self.create_timer(1.0,self.on_timer)
 
    def on_timer(self):
        # 判断是否可以实现转换
        if self.buffer.can_transform("camera","laser",Time()):
            ts = self.buffer.lookup_transform("camera","laser",Time())
            self.get_logger().info("-------转换后的数据-------")
            self.get_logger().info(
                "转换的结果，父坐标系:%s,子坐标系:%s,偏移量:(%.2f,%.2f,%.2f)"
                % (ts.header.frame_id,ts.child_frame_id,
                ts.transform.translation.x,
                ts.transform.translation.y,
                ts.transform.translation.z)
            )
        else:
            self.get_logger().info("转换失败......")
 
 
def main():
    # 2.初始化ROS2客户端；
    rclpy.init()
    # 4.调用spain函数，并传入节点对象；
    rclpy.spin(TFListenerPy())
    # 5.资源释放。 
    rclpy.shutdown()
 
if __name__ == '__main__':
    main()
```

#### 2.编辑配置文件

##### 1.package.xml

在创建功能包时，所依赖的功能包已经自动配置了，配置内容如下：

```xml
<depend>rclpy</depend>
<depend>tf_transformations</depend>
<depend>tf2_ros</depend>
<depend>geometry_msgs</depend>
```

##### 2.setup.py

`entry_points`字段的`console_scripts`中添加如下内容：

```py
entry_points={
    'console_scripts': [
        'demo01_tf_listener_py = py04_tf_listener.demo01_tf_listener_py:main'
    ],
},
```

#### 3.编译

终端中进入当前工作空间，编译功能包：

```
colcon build --packages-select py04_tf_listener
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

终端3输入如下命令执行坐标系变换：

```
. install/setup.bash 
ros2 run py04_tf_listener demo01_tf_listener_py
```

终端3将输出 laser 相对于 camera 的坐标，具体结果请参考案例1。

