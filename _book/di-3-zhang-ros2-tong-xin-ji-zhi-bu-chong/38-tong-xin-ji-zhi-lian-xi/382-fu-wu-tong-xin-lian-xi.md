### 3.8.2 话题通信实现

#### 1.速度订阅与发布

功能包cpp07\_exercise的src目录下，新建C++文件exe01\_pub\_sub.cpp，并编辑文件，输入如下内容：

```cpp
/*
   需求：订阅窗口1中的乌龟速度，然后生成控制窗口2乌龟运动的指令并发布。
   步骤：
       1.包含头文件；
       2.初始化 ROS2 客户端；
       3.定义节点类；
          3-1.创建控制第二个窗体乌龟运动的发布方；
          3-2.创建订阅第一个窗体乌龟pose的订阅方；
          3-3.根据订阅的乌龟的速度生成控制窗口2乌龟运动的速度消息并发布。
       4.调用spin函数，并传入节点对象指针；
       5.释放资源。
*/
// 1.包含头文件；
#include <rclcpp/rclcpp.hpp>
#include <turtlesim/msg/pose.hpp>
#include <geometry_msgs/msg/twist.hpp>
// 3.定义节点类；
class ExePubSub : public rclcpp::Node
{
public:
  ExePubSub() : rclcpp::Node("demo01_pub_sub")
  {
    // 3-1.创建控制第二个窗体乌龟运动的发布方；
    twist_pub_ = this->create_publisher<geometry_msgs::msg::Twist>("/t2/turtle1/cmd_vel", 1);
    // 3-2.创建订阅第一个窗体乌龟pose的订阅方；
    pose_sub_ = this->create_subscription<turtlesim::msg::Pose>(
      "/turtle1/pose", 1, std::bind(&ExePubSub::poseCallback, this, std::placeholders::_1));
  }

private:
  // 3-3.根据订阅的乌龟的速度生成控制窗口2乌龟运动的速度消息并发布。
  void poseCallback(const turtlesim::msg::Pose::ConstSharedPtr pose)
  {
    geometry_msgs::msg::Twist twist;
    twist.angular.z = -(pose->angular_velocity); //角速度取反
    twist.linear.x = pose->linear_velocity; //线速度不变
    twist_pub_->publish(twist);
  }

  rclcpp::Publisher<geometry_msgs::msg::Twist>::SharedPtr twist_pub_;
  rclcpp::Subscription<turtlesim::msg::Pose>::SharedPtr pose_sub_;
};

int main(int argc, char** argv)
{
  // 2.初始化 ROS2 客户端；
  rclcpp::init(argc, argv);
  // 4.调用spin函数，并传入节点对象指针；
  rclcpp::spin(std::make_shared<ExePubSub>());
  // 5.释放资源。
  rclcpp::shutdown();
}
```

#### 2.launch文件

功能包cpp07\_exercise的launch目录下，新建launch文件exe01\_pub\_sub.launch.py，并编辑文件，输入如下内容：

```py
from launch import LaunchDescription
from launch_ros.actions import Node
from launch.actions import ExecuteProcess,RegisterEventHandler
from launch.event_handlers import OnProcessExit

def generate_launch_description():
    # 1.创建两个 turtlesim_node 节点
    t1 = Node(package="turtlesim",executable="turtlesim_node")
    t2 = Node(package="turtlesim",executable="turtlesim_node",namespace="t2")
    # 2.让第二只乌龟掉头
    rotate = ExecuteProcess(
        cmd=["ros2 action send_goal /t2/turtle1/rotate_absolute turtlesim/action/RotateAbsolute \"{'theta': 3.14}\""],
        output="both",
        shell=True
    )
    # 3.自实现的订阅发布实现
    pub_sub = Node(package="cpp07_exercise",executable="exe01_pub_sub")
    # 4.乌龟掉头完毕后，开始执行步骤3
    rotate_exit_event = RegisterEventHandler(
        event_handler=OnProcessExit(
            target_action=rotate,
            on_exit=pub_sub
        )
    )
    return LaunchDescription([t1,t2,rotate,rotate_exit_event])
```

#### 3.编辑配置文件

##### 1.package.xml

在创建功能包时，所依赖的功能包已经自动配置了，配置内容如下：

```XML
<depend>rclcpp</depend>
<depend>turtlesim</depend>
<depend>base_interfaces_demo</depend>
<depend>geometry_msgs</depend>
<depend>rclcpp_action</depend>
```

##### 2.CMakeLists.txt

CMakeLists.txt 中发布和订阅程序核心配置如下：

```
# find dependencies
find_package(ament_cmake REQUIRED)
find_package(rclcpp REQUIRED)
find_package(turtlesim REQUIRED)
find_package(base_interfaces_demo REQUIRED)
find_package(geometry_msgs REQUIRED)
find_package(rclcpp_action REQUIRED)

add_executable(exe01_pub_sub src/exe01_pub_sub.cpp)
ament_target_dependencies(
  exe01_pub_sub
  "rclcpp"
  "turtlesim"
  "geometry_msgs"
)
install(TARGETS 
  exe01_pub_sub
  DESTINATION lib/${PROJECT_NAME})

install(DIRECTORY launch DESTINATION share/${PROJECT_NAME})
```

#### 4.编译

终端中进入当前工作空间，编译功能包：

```
colcon build --packages-select cpp07_exercise
```

#### 5.执行

当前工作空间下，启动终端输入如下指令：

```
. install/setup.bash
ros2 launch cpp07_exercise exe01_pub_sub.launch.py
```

指令执行后，将生成两个turtlesim\_node节点对应的窗口，并且其中一个窗口的乌龟开始调头。

再启动一个终端，输入如下指令：

```
ros2 run turtlesim turtle_teleop_key
```

待乌龟调头完毕，就可以通过键盘控制乌龟运动了，最终运行结果与演示案例类似。

