### 3.8.4 服务通信实现

#### 1.服务接口文件

功能包base\_interfaces\_demo的srv目录下，新建srv文件Distance.srv，并编辑文件，输入如下内容：

```
float32 x
float32 y
float32 theta
---
float32 distance
```

#### 2.服务端实现

功能包cpp07\_exercise的src目录下，新建C++文件exe02\_server.cpp，并编辑文件，输入如下内容：

```cpp
/*
   需求：处理请求发送的目标点，计算乌龟与目标点之间的直线距离。
   步骤：
       1.包含头文件；
       2.初始化 ROS2 客户端；
       3.定义节点类；
            3-1.创建乌龟姿态订阅方，回调函数中获取x坐标与y坐标；
            3-2.创建服务端；
            3-3.解析目标值，计算距离并反馈结果。
       4.调用spin函数，并传入节点对象指针；
       5.释放资源。
*/
// 1.包含头文件；
#include "rclcpp/rclcpp.hpp"
#include "base_interfaces_demo/srv/distance.hpp"
#include "turtlesim/msg/pose.hpp"
#include "turtlesim/srv/spawn.hpp"

using namespace std::chrono_literals;
// 3.定义节点类；
class ExeDistanceServer: public rclcpp::Node {
public:
    ExeDistanceServer():Node("exe_distance_server"),turtle1_x(0.0),turtle1_y(0.0){
        // 3-1.创建乌龟姿态订阅方，回调函数中获取x坐标与y坐标；
        pose_sub = this->create_subscription<turtlesim::msg::Pose>("/turtle1/pose",10,std::bind(&ExeDistanceServer::poseCallBack, this, std::placeholders::_1));
        // 3-2.创建服务端；
        distance_server = this->create_service<base_interfaces_demo::srv::Distance>("distance",std::bind(&ExeDistanceServer::distanceCallBack, this, std::placeholders::_1, std::placeholders::_2));
    }
private:

    void poseCallBack(const turtlesim::msg::Pose::SharedPtr pose){
        turtle1_x = pose->x;
        turtle1_y = pose->y;
    }
    // 3-3.解析目标值，计算距离并反馈结果。
    void distanceCallBack(const base_interfaces_demo::srv::Distance_Request::SharedPtr request,
                    base_interfaces_demo::srv::Distance_Response::SharedPtr response
    ){
        // 解析目标值
        float goal_x = request->x;
        float goal_y = request->y;

        // 距离计算
        float x = goal_x - turtle1_x;
        float y = goal_y - turtle1_y;
        // 将结果设置到响应
        response->distance = std::sqrt(x * x + y * y);
        RCLCPP_INFO(this->get_logger(),"目标坐标:(%.2f,%.2f),距离:%.2f",goal_x,goal_y,response->distance);

    }
    rclcpp::Subscription<turtlesim::msg::Pose>::SharedPtr pose_sub;
    rclcpp::Service<base_interfaces_demo::srv::Distance>::SharedPtr distance_server;
    float turtle1_x;
    float turtle1_y;


};

int main(int argc, char const *argv[])
{
    // 2.初始化 ROS2 客户端；
    rclcpp::init(argc,argv);
    // 4.调用spin函数，并传入节点对象指针；
    rclcpp::spin(std::make_shared<ExeDistanceServer>());
    // 5.释放资源。
    rclcpp::shutdown();
    return 0;
}
```

#### 3.客户端实现

功能包cpp07\_exercise的src目录下，新建C++文件exe03\_client.cpp，并编辑文件，输入如下内容：

```cpp
/*
   需求：发布目标点的坐标，接收并处理服务端反馈的结果。
   步骤：
       1.包含头文件；
       2.初始化 ROS2 客户端；
       3.定义节点类；
            3-1.创建客户端；
            3-2.连接服务；
            3-3.发送请求。
       4.调用对象服务连接、发送请求、处理响应相关函数；
       5.释放资源。
*/
// 1.包含头文件；
#include "rclcpp/rclcpp.hpp"
#include "base_interfaces_demo/srv/distance.hpp"
#include "turtlesim/srv/spawn.hpp"

using namespace std::chrono_literals;

// 3.定义节点类；
class ExeDistanceClient: public rclcpp::Node {
public:
    ExeDistanceClient():Node("exe_distance_client"){
        // 3-1.创建客户端；
        distance_client = this->create_client<base_interfaces_demo::srv::Distance>("distance");
    }   
    // 3-2.连接服务；
    bool connect_server(){
      while (!distance_client->wait_for_service(1s))
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
    // 3-3.发送请求。
    rclcpp::Client<base_interfaces_demo::srv::Distance>::FutureAndRequestId send_distance(float x,float y,float theta){
        auto distance_request = std::make_shared<base_interfaces_demo::srv::Distance::Request>();
        distance_request->x = x;
        distance_request->y = y;
        distance_request->theta = theta;
        return distance_client->async_send_request(distance_request);
    }
private:
    rclcpp::Client<base_interfaces_demo::srv::Distance>::SharedPtr distance_client;
};
int main(int argc, char const *argv[])
{
    // 2.初始化 ROS2 客户端；
    rclcpp::init(argc,argv);
    // 4.调用对象服务连接、发送请求、处理响应相关函数；
    auto client = std::make_shared<ExeDistanceClient>();
    // 处理传入的参数
    if (argc != 5)
    {
        RCLCPP_INFO(client->get_logger(),"请传入目标的位姿参数:(x,y,theta)");
        return 1;
    }

    float x = atof(argv[1]);
    float y = atof(argv[2]);
    float theta = atof(argv[3]);
    // 服务连接
    bool flag = client->connect_server();
    if (!flag)
    {
        RCLCPP_INFO(client->get_logger(),"服务连接失败!");
        return 1;
    }
    // 发送请求
    auto distance_future = client->send_distance(x, y, theta);
    // 处理响应
    if(rclcpp::spin_until_future_complete(client,distance_future) == rclcpp::FutureReturnCode::SUCCESS){
        RCLCPP_INFO(client->get_logger(),"两只乌龟相距%.2f米。",distance_future.get()->distance);
    } else {
        RCLCPP_INFO(client->get_logger(),"获取距离服务失败!");
    }
    // 5.释放资源。
    rclcpp::shutdown();
    return 0;
}
```

#### 4.launch文件

该案例需要分别为服务端和客户端创建launch文件。

功能包cpp07\_exercise的launch目录下，首先新建服务端launch文件exe02\_server.launch.py，编辑文件，输入如下内容：

```py
from launch import LaunchDescription
from launch_ros.actions import Node

def generate_launch_description():
    # 创建turtlesim_node节点
    turtle = Node(package="turtlesim",executable="turtlesim_node")
    # 创建测距服务端节点
    server = Node(package="cpp07_exercise",executable="exe02_server")

    return LaunchDescription([turtle,server])
```

然后新建客户端launch文件exe03\_client.launch.py，编辑文件，输入如下内容：

```py
from launch import LaunchDescription
from launch_ros.actions import Node
from launch.actions import ExecuteProcess

def generate_launch_description():
    # 设置目标点的坐标，以及目标点乌龟的名称
    x = 8.54
    y = 9.54
    theta = 0.0
    name = "t2"
    # 生成新的乌龟
    spawn = ExecuteProcess(
        cmd=["ros2 service call /spawn turtlesim/srv/Spawn \"{'x': "
                + str(x) + ",'y': " + str(y) + ",'theta': " + str(theta) + ",'name': '" + name + "'}\""],
        # output="both",
        shell=True
    )
    # 创建客户端节点
    client = Node(package="cpp07_exercise",
                executable="exe03_client",
                arguments=[str(x),str(y),str(theta)])
    return LaunchDescription([spawn,client])
```

#### 5.编辑配置文件

此处需要编辑base\_interfaces\_demo和cpp07\_exercise两个功能包下的配置文件。

##### 1.base\_interfaces\_demo下的CMakeLists.txt

鉴于功能包base\_interfaces\_demo的基础配置以及设置过了，所以只需要修改CMakeLists.txt中的rosidl\_generate\_interfaces 函数即可，修改后的内容如下：

```
rosidl_generate_interfaces(${PROJECT_NAME}
  "msg/Student.msg"
  "srv/AddInts.srv"
  "srv/Distance.srv"
  "action/Progress.action"
)
```

##### 2.cpp07\_exercise下的CMakeLists.txt

CMakeLists.txt 文件需要添加如下内容：

```
add_executable(exe02_server src/exe02_server.cpp)
ament_target_dependencies(
  exe02_server
  "rclcpp"
  "turtlesim"
  "base_interfaces_demo"
  "geometry_msgs"
)
add_executable(exe03_client src/exe03_client.cpp)
ament_target_dependencies(
  exe03_client
  "rclcpp"
  "turtlesim"
  "base_interfaces_demo"
  "geometry_msgs"
)
```

文件中 install 修改为如下内容：

```
install(TARGETS 
  exe01_pub_sub
  exe02_server
  exe03_client
  DESTINATION lib/${PROJECT_NAME})
```

#### 6.编译

终端中进入当前工作空间，编译功能包：

```
colcon build --packages-select base_interfaces_demo cpp07_exercise
```

#### 7.执行

当前工作空间下，启动两个终端。

终端1输入如下指令：

```
. install/setup.bash
ros2 launch cpp07_exercise exe02_server.launch.py
```

指令执行后，将生成turtlesim\_node节点对应的窗口，并且会启动自定义的测距服务端。

终端2输入如下指令：

```
. install/setup.bash
ros2 launch cpp07_exercise exe03_client.launch.py
```

指令执行后，会生成一只新的乌龟，并且输出两只乌龟的直线距离，最终运行结果与演示案例类似。

