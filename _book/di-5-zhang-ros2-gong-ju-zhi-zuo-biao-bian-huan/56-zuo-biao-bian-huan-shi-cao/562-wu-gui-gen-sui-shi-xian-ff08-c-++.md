### 5.6.2 乌龟跟随实现（C++）

#### 1.编写生成新乌龟实现

功能包 cpp05\_exercise 的 src 目录下，新建 C++ 文件 exer01\_tf\_spawn.cpp，并编辑文件，输入如下内容：

```cpp
/*  
  需求：编写客户端，发送请求生成一只新的乌龟。
  步骤：
    1.包含头文件；
    2.初始化 ROS2 客户端；
    3.定义节点类；
      3-1.声明并获取参数；
      3-2.创建客户端；
      3-3.等待服务连接；
      3-4.组织请求数据并发送；
    4.创建对象指针调用其功能,并处理响应；
    5.释放资源。

*/
// 1.包含头文件；
#include "rclcpp/rclcpp.hpp"
#include "turtlesim/srv/spawn.hpp"

using namespace std::chrono_literals;

// 3.定义节点类；
class TurtleSpawnClient: public rclcpp::Node{
  public:
    TurtleSpawnClient():Node("turtle_spawn_client"){
      // 3-1.声明并获取参数；
      this->declare_parameter("x",2.0);
      this->declare_parameter("y",8.0);
      this->declare_parameter("theta",0.0);
      this->declare_parameter("turtle_name","turtle2");
      x = this->get_parameter("x").as_double();
      y = this->get_parameter("y").as_double();
      theta = this->get_parameter("theta").as_double();
      turtle_name = this->get_parameter("turtle_name").as_string();
      // 3-2.创建客户端；
      client = this->create_client<turtlesim::srv::Spawn>("/spawn");
    }
    // 3-3.等待服务连接；
    bool connect_server(){
      while (!client->wait_for_service(1s))
      {
        if (!rclcpp::ok())
        {
          RCLCPP_INFO(this->get_logger(),"客户端退出！");
          return false;
        }

        RCLCPP_INFO(this->get_logger(),"服务连接中，请稍候...");
      }
      return true;
    }
    // 3-4.组织请求数据并发送；
    rclcpp::Client<turtlesim::srv::Spawn>::FutureAndRequestId send_request(){
      auto request = std::make_shared<turtlesim::srv::Spawn::Request>();
      request->x = x;
      request->y = y;
      request->theta = theta;
      request->name = turtle_name;
      return client->async_send_request(request);
    }


  private:
    rclcpp::Client<turtlesim::srv::Spawn>::SharedPtr client;
    float_t x,y,theta;
    std::string turtle_name;
};

int main(int argc, char ** argv)
{
  // 2.初始化 ROS2 客户端；
  rclcpp::init(argc,argv);

  // 4.创建对象指针并调用其功能；
  auto client = std::make_shared<TurtleSpawnClient>();
  bool flag = client->connect_server();
  if (!flag)
  {
    RCLCPP_INFO(client->get_logger(),"服务连接失败!");
    return 0;
  }

  auto response = client->send_request();

  // 处理响应
  if (rclcpp::spin_until_future_complete(client,response) == rclcpp::FutureReturnCode::SUCCESS)
  {
    RCLCPP_INFO(client->get_logger(),"请求正常处理");
    std::string name = response.get()->name;
    if (name.empty())
    {
        RCLCPP_INFO(client->get_logger(),"乌龟重名导致生成失败！");
    } else {
        RCLCPP_INFO(client->get_logger(),"乌龟%s生成成功！", name.c_str());
    }

  } else {
    RCLCPP_INFO(client->get_logger(),"请求异常");
  }

  // 5.释放资源。
  rclcpp::shutdown();
  return 0;
}
```

#### 2.编写坐标变换广播实现

功能包 cpp05\_exercise 的 src 目录下，新建 C++ 文件 exer02\_tf\_broadcaster.cpp，并编辑文件，输入如下内容：

```cpp
/*   
  需求：发布乌龟坐标系到窗口坐标系的坐标变换。
  步骤：
    1.包含头文件；
    2.初始化 ROS 客户端；
    3.定义节点类；
      3-1.声明并解析乌龟名称参数；
      3-2.创建动态坐标变换发布方；
      3-3.创建乌龟位姿订阅方；
      3-4.根据订阅到的乌龟位姿生成坐标帧并广播。
    4.调用 spin 函数，并传入对象指针；
    5.释放资源。

*/
// 1.包含头文件；
#include <geometry_msgs/msg/transform_stamped.hpp>

#include <rclcpp/rclcpp.hpp>
#include <tf2/LinearMath/Quaternion.h>
#include <tf2_ros/transform_broadcaster.h>
#include <turtlesim/msg/pose.hpp>

using std::placeholders::_1;

// 3.定义节点类；
class TurtleFrameBroadcaster : public rclcpp::Node
{
public:
  TurtleFrameBroadcaster(): Node("turtle_frame_broadcaster")
  {
    // 3-1.声明并解析乌龟名称参数；
    this->declare_parameter("turtle_name","turtle1");
    turtle_name = this->get_parameter("turtle_name").as_string();
    // 3-2.创建动态坐标变换发布方；
    tf_broadcaster_ = std::make_unique<tf2_ros::TransformBroadcaster>(*this);
    std::string topic_name = turtle_name + "/pose";

    // 3-3.创建乌龟位姿订阅方；
    subscription_ = this->create_subscription<turtlesim::msg::Pose>(
      topic_name, 10,
      std::bind(&TurtleFrameBroadcaster::handle_turtle_pose, this, _1));
  }

private:
  // 3-4.根据订阅到的乌龟位姿生成坐标帧并广播。   
  void handle_turtle_pose(const turtlesim::msg::Pose & msg)
  {
    // 组织消息
    geometry_msgs::msg::TransformStamped t;
    rclcpp::Time now = this->get_clock()->now();

    t.header.stamp = now;
    t.header.frame_id = "world";
    t.child_frame_id = turtle_name;

    t.transform.translation.x = msg.x;
    t.transform.translation.y = msg.y;
    t.transform.translation.z = 0.0;

    tf2::Quaternion q;
    q.setRPY(0, 0, msg.theta);
    t.transform.rotation.x = q.x();
    t.transform.rotation.y = q.y();
    t.transform.rotation.z = q.z();
    t.transform.rotation.w = q.w();
    // 发布消息
    tf_broadcaster_->sendTransform(t);
  }
  rclcpp::Subscription<turtlesim::msg::Pose>::SharedPtr subscription_;
  std::unique_ptr<tf2_ros::TransformBroadcaster> tf_broadcaster_;
  std::string turtle_name;
};

int main(int argc, char * argv[])
{
  // 2.初始化 ROS 客户端；
  rclcpp::init(argc, argv);
  // 4.调用 spin 函数，并传入对象指针；
  rclcpp::spin(std::make_shared<TurtleFrameBroadcaster>());
  // 5.释放资源。
  rclcpp::shutdown();
  return 0;
}
```

#### 3.编写坐标变换监听实现

功能包 cpp05\_exercise 的 src 目录下，新建 C++ 文件 exer03\_tf\_listener.cpp，并编辑文件，输入如下内容：

```cpp
/*  
  需求：广播的坐标系消息，并生成 turtle2 相对于 turtle1 的坐标系数据，
       并进一步生成控制 turtle2 运动的速度指令。
  步骤：
    1.包含头文件；
    2.初始化 ROS 客户端；
    3.定义节点类；
      3-1.声明并解析参数；
      3-2.创建tf缓存对象指针；
      3-3.创建tf监听器；
      3-4.按照条件查找符合条件的坐标系并生成变换后的坐标帧。
      3-5.生成 turtle2 的速度指令，并发布。
    4.调用 spin 函数，并传入对象指针；
    5.释放资源。

*/
#include "rclcpp/rclcpp.hpp"
#include "tf2_ros/transform_listener.h"
#include "tf2_ros/buffer.h"
#include "tf2/LinearMath/Quaternion.h"
#include "geometry_msgs/msg/twist.hpp"

using namespace std::chrono_literals;

// 3.定义节点类；
class TurtleFrameListener : public rclcpp::Node {
public:
  TurtleFrameListener():Node("turtle_frame_listener"){
    // 3-1.声明并解析参数；
    this->declare_parameter("target_frame","turtle2");
    this->declare_parameter("source_frame","turtle1");
    target_frame = this->get_parameter("target_frame").as_string();
    source_frame = this->get_parameter("source_frame").as_string();

    // 3-2.创建tf缓存对象指针；
    tf_buffer_ = std::make_unique<tf2_ros::Buffer>(this->get_clock());
    // 3-3.创建tf监听器；
    transform_listener_ = std::make_shared<tf2_ros::TransformListener>(*tf_buffer_);
    twist_pub_ = this->create_publisher<geometry_msgs::msg::Twist>(target_frame + "/cmd_vel",10);
    timer_ = this->create_wall_timer(1s, std::bind(&TurtleFrameListener::on_timer,this));
  }

private:
  void on_timer(){
    // 3-4.按照条件查找符合条件的坐标系并生成变换后的坐标帧。
    geometry_msgs::msg::TransformStamped transformStamped;
    try
    {
      transformStamped = tf_buffer_->lookupTransform(target_frame,source_frame,tf2::TimePointZero);
    }
    catch(const tf2::LookupException& e)
    {
      RCLCPP_INFO(this->get_logger(),"坐标变换异常：%s",e.what());
      return;
    }
    // 3-5.生成 turtle2 的速度指令，并发布。
    geometry_msgs::msg::Twist msg;
    static const double scaleRotationRate = 1.0;
    msg.angular.z = scaleRotationRate * atan2(
        transformStamped.transform.translation.y,
        transformStamped.transform.translation.x);

    static const double scaleForwardSpeed = 0.5;
    msg.linear.x = scaleForwardSpeed * sqrt(
        pow(transformStamped.transform.translation.x, 2) +
        pow(transformStamped.transform.translation.y, 2));

    twist_pub_->publish(msg);

  }
  rclcpp::Publisher<geometry_msgs::msg::Twist>::SharedPtr twist_pub_;
  rclcpp::TimerBase::SharedPtr timer_;
  std::shared_ptr<tf2_ros::TransformListener> transform_listener_;
  std::unique_ptr<tf2_ros::Buffer> tf_buffer_;
  std::string target_frame;
  std::string source_frame;
};

int main(int argc, char const *argv[])
{
  // 2.初始化 ROS 客户端；
  rclcpp::init(argc,argv);
  // 4.调用 spin 函数，并传入对象指针；
  rclcpp::spin(std::make_shared<TurtleFrameListener>());
  // 5.释放资源。
  rclcpp::shutdown();
  return 0;
}
```

#### 4.编写 launch 文件

launch 目录下新建文件exer01\_turtle\_follow.launch.py，并编辑文件，输入如下内容：

```py
from launch import LaunchDescription
from launch_ros.actions import Node
from launch.actions import DeclareLaunchArgument
from launch.substitutions import LaunchConfiguration

def generate_launch_description():
    # 声明参数
    turtle1 = DeclareLaunchArgument(name="turtle1",default_value="turtle1")
    turtle2 = DeclareLaunchArgument(name="turtle2",default_value="turtle2")
    # 启动 turtlesim_node 节点
    turtlesim_node = Node(package="turtlesim", executable="turtlesim_node", name="t1")
    # 生成一只新乌龟
    spawn = Node(package="cpp05_exercise", executable="exer01_tf_spawn",
                name="spawn1",
                parameters=[{"turtle_name":LaunchConfiguration("turtle2")}]
    )
    # tf 广播
    tf_broadcaster1 = Node(package="cpp05_exercise",executable="exer02_tf_broadcaster",
                            name="tf_broadcaster1")
    tf_broadcaster2 = Node(package="cpp05_exercise",executable="exer02_tf_broadcaster",
                            name="tf_broadcaster1",
                            parameters=[{"turtle_name":LaunchConfiguration("turtle2")}])
    # tf 监听
    tf_listener = Node(package="cpp05_exercise",executable="exer03_tf_listener",
                            name="tf_listener",
                            parameters=[{"target_frame":LaunchConfiguration("turtle2"),"source_frame":LaunchConfiguration("turtle1")}]
                            )
    return LaunchDescription([turtle1,turtle2,turtlesim_node,spawn,tf_broadcaster1,tf_broadcaster2,tf_listener])
```

#### 5.编辑配置文件

##### 1.package.xml

在创建功能包时，所依赖的功能包已经自动配置了，配置内容如下：

```XML
<depend>rclcpp</depend>
<depend>tf2</depend>
<depend>tf2_ros</depend>
<depend>geometry_msgs</depend>
<depend>turtlesim</depend>
```

##### 2.CMakeLists.txt

CMakeLists.txt 中发布和订阅程序核心配置如下：

```
# find dependencies
find_package(ament_cmake REQUIRED)
find_package(rclcpp REQUIRED)
find_package(tf2 REQUIRED)
find_package(tf2_ros REQUIRED)
find_package(geometry_msgs REQUIRED)
find_package(turtlesim REQUIRED)

add_executable(exer01_tf_spawn src/exer01_tf_spawn.cpp)
ament_target_dependencies(
  exer01_tf_spawn
  "rclcpp"
  "tf2"
  "tf2_ros"
  "geometry_msgs"
  "turtlesim"
)

add_executable(exer02_tf_broadcaster src/exer02_tf_broadcaster.cpp)
ament_target_dependencies(
  exer02_tf_broadcaster
  "rclcpp"
  "tf2"
  "tf2_ros"
  "geometry_msgs"
  "turtlesim"
)

add_executable(exer03_tf_listener src/exer03_tf_listener.cpp)
ament_target_dependencies(
  exer03_tf_listener
  "rclcpp"
  "tf2"
  "tf2_ros"
  "geometry_msgs"
  "turtlesim"
)

install(TARGETS 
  exer01_tf_spawn
  exer02_tf_broadcaster
  exer03_tf_listener
  DESTINATION lib/${PROJECT_NAME})

install(DIRECTORY launch
  DESTINATION share/${PROJECT_NAME}
)
```

#### 6.编译

终端中进入当前工作空间，编译功能包：

```
colcon build --packages-select cpp05_exercise
```

#### 7.执行

当前工作空间下启动终端，输入如下命令运行launch文件：

```
. install/setup.bash 
ros2 launch cpp05_exercise exer01_turtle_follow.launch.py
```

再新建终端，启动 turtlesim 键盘控制节点：

```
ros2 run turtlesim turtle_teleop_key
```

该终端下可以通过键盘控制 turtle1 运动，并且 turtle2 会跟随 turtle1 运动。最终的运行结果与演示案例类似。

