### 3.8.6 动作通信实现

#### 1.动作接口文件

功能包base\_interfaces\_demo的action目录下，新建action文件Nav.action，并编辑文件，输入如下内容：

```
float32 goal_x
float32 goal_y
float32 goal_theta
---
float32 turtle_x
float32 turtle_y
float32 turtle_theta
---
float32 distance
```

#### 2.动作服务端实现

功能包cpp07\_exercise的src目录下，新建C++文件exe04\_action\_server.cpp，并编辑文件，输入如下内容：

```cpp
/*
   需求：处理请求发送的目标点，控制乌龟向该目标点运动，并连续反馈乌龟与目标点之间的剩余距离。
   步骤：
       1.包含头文件；
       2.初始化 ROS2 客户端；
       3.定义节点类；
            3-1.创建原生乌龟位姿订阅方，回调函数中获取乌龟位姿；
            3-2.创建原生乌龟速度发布方；
            3-3.创建动作服务端；
            3-4.解析动作客户端发送的请求；
            3-5.处理动作客户端发送的取消请求；
            3-6.创建新线程处理请求；
            3-7.新线程产生连续反馈并响应最终结果。
       4.调用spin函数，并传入节点对象指针；
       5.释放资源。
*/
// 1.包含头文件； 
#include "rclcpp/rclcpp.hpp"
#include "rclcpp_action/rclcpp_action.hpp"
#include "base_interfaces_demo/action/nav.hpp"
#include "turtlesim/msg/pose.hpp"
#include "geometry_msgs/msg/twist.hpp"

using base_interfaces_demo::action::Nav;
using namespace std::placeholders;

// 3.定义节点类；
class ExeNavActionServer: public rclcpp::Node {
public:
    ExeNavActionServer(const rclcpp::NodeOptions & options = rclcpp::NodeOptions()):
        Node("exe_nav_action_server",options){
        // 3-1.创建原生乌龟位姿订阅方，回调函数中获取乌龟位姿；
        pose_sub = this->create_subscription<turtlesim::msg::Pose>("/turtle1/pose",10,std::bind(&ExeNavActionServer::poseCallBack, this, std::placeholders::_1));
        // 3-2.创建原生乌龟速度发布方；
        cmd_vel_pub = this->create_publisher<geometry_msgs::msg::Twist>("/turtle1/cmd_vel",10);
        // 3-3.创建动作服务端；
        nav_action_server = rclcpp_action::create_server<Nav>(
            this,
            "nav",
            std::bind(&ExeNavActionServer::handle_goal,this,_1,_2),
            std::bind(&ExeNavActionServer::handle_cancel,this,_1),
            std::bind(&ExeNavActionServer::handle_accepted,this,_1)
            );

    }
private:
    turtlesim::msg::Pose::SharedPtr turtle1_pose = nullptr;
    rclcpp::Subscription<turtlesim::msg::Pose>::SharedPtr pose_sub;
    rclcpp_action::Server<Nav>::SharedPtr nav_action_server;
    rclcpp::Publisher<geometry_msgs::msg::Twist>::SharedPtr cmd_vel_pub;
    void poseCallBack(const turtlesim::msg::Pose::SharedPtr pose){
        turtle1_pose = pose;
    }
    // 3-4.解析动作客户端发送的请求；
    rclcpp_action::GoalResponse handle_goal(const rclcpp_action::GoalUUID & goal_uuid, std::shared_ptr<const Nav::Goal> goal){
        (void)goal_uuid;
        RCLCPP_INFO(this->get_logger(),"请求坐标:(%.2f,%.2f),航向:%.2f", goal->goal_x,goal->goal_y,goal->goal_theta);
        if (goal->goal_x < 0 || goal->goal_x > 11.1 || goal->goal_y < 0 || goal->goal_y > 11.1)
        {
            return rclcpp_action::GoalResponse::REJECT;
        }

        return rclcpp_action::GoalResponse::ACCEPT_AND_EXECUTE;
    }
    // 3-5.处理动作客户端发送的取消请求；
    rclcpp_action::CancelResponse handle_cancel(std::shared_ptr<rclcpp_action::ServerGoalHandle<Nav>> goal_handle){
        (void)goal_handle;
        RCLCPP_INFO(this->get_logger(),"任务取消!");
        return rclcpp_action::CancelResponse::ACCEPT;
    }
    // 3-7.新线程产生连续反馈并响应最终结果。
    void execute(std::shared_ptr<rclcpp_action::ServerGoalHandle<Nav>> goal_handle){
        RCLCPP_INFO(this->get_logger(),"开始执行任务......");
        // 解析目标值
        float goal_x = goal_handle->get_goal()->goal_x;
        float goal_y = goal_handle->get_goal()->goal_y;
        // 创建连续反馈对象指针；
        auto feedback = std::make_shared<Nav::Feedback>();
        // 创建最终结果对象指针；
        auto result = std::make_shared<Nav::Result>();

        rclcpp::Rate rate(1.0);
        while (true)
        {
            // 任务执行中，关于客户端发送取消请求的处理；
            if(goal_handle->is_canceling()){
                goal_handle->canceled(result);
                return;
            }
            // 解析原生乌龟位姿数据；
            float turtle1_x = turtle1_pose->x;
            float turtle1_y = turtle1_pose->y;
            float turtle1_theta = turtle1_pose->theta;
            // 计算原生乌龟与目标乌龟的x向以及y向距离；
            float x_distance = goal_x - turtle1_x;
            float y_distance = goal_y - turtle1_y;


            // 计算速度
            geometry_msgs::msg::Twist twist;
            double scale = 0.5;
            twist.linear.x = scale * x_distance;
            twist.linear.y = scale * y_distance;
            cmd_vel_pub->publish(twist);
            // 计算剩余距离
            float distance = sqrt(pow(x_distance,2) + pow(y_distance,2));

            // 当两龟距离小于0.15米时，将当前乌龟位姿设置进result并退出循环
            if (distance < 0.15)
            {   
                //将当前乌龟坐标赋值给 result
                result->turtle_x = turtle1_x;
                result->turtle_y = turtle1_y;
                result->turtle_theta = turtle1_theta;
                break;
            }
            // 为feedback设置数据并发布
            feedback->distance = distance;
            goal_handle->publish_feedback(feedback);

            rate.sleep();
        }
        // 设置最终响应结果
        if (rclcpp::ok())
        {
            goal_handle->succeed(result);
            RCLCPP_INFO(this->get_logger(),"任务结束!");
        }


    }
    // 3-6.创建新线程处理请求；
    void handle_accepted(std::shared_ptr<rclcpp_action::ServerGoalHandle<Nav>> goal_handle){
        std::thread{std::bind(&ExeNavActionServer::execute,this,_1),goal_handle}.detach();
    }


};

int main(int argc, char const *argv[])
{
    // 2.初始化 ROS2 客户端；
    rclcpp::init(argc,argv);
    // 4.调用spin函数，并传入节点对象指针；
    rclcpp::spin(std::make_shared<ExeNavActionServer>());
    // 5.释放资源。
    rclcpp::shutdown();
    return 0;
}
```

#### 3.动作客户端实现

功能包cpp07\_exercise的src目录下，新建C++文件exe05\_action\_client.cpp，并编辑文件，输入如下内容：

```cpp
/*
   需求：向动作服务端发送目标点数据，并处理服务端的响应数据。
   步骤：
       1.包含头文件；
       2.初始化 ROS2 客户端；
       3.定义节点类；
            3-1.创建动作客户端；
            3-2.发送请求数据，并处理服务端响应；
            3-3.处理目标响应；
            3-4.处理响应的连续反馈；
            3-5.处理最终响应。
       4.调用spin函数，并传入节点对象指针；
       5.释放资源。
*/
// 1.包含头文件；
#include "rclcpp/rclcpp.hpp"
#include "rclcpp_action/rclcpp_action.hpp"
#include "base_interfaces_demo/action/nav.hpp"
#include "turtlesim/srv/spawn.hpp"

using base_interfaces_demo::action::Nav;
using namespace std::chrono_literals;
using namespace std::placeholders;

// 3.定义节点类；
class ExeNavActionClient: public rclcpp::Node{
public:
    ExeNavActionClient(const rclcpp::NodeOptions & options = rclcpp::NodeOptions())
    :Node("exe_nav_action_client",options){
        // 3-1.创建动作客户端；
        nav_client = rclcpp_action::create_client<Nav>(this,"nav");
    }
    // 3-2.发送请求数据，并处理服务端响应；
    void send_goal(float x, float y, float theta){
        // 连接动作服务端，如果超时（5s），那么直接退出。
        if (!nav_client->wait_for_action_server(5s))
        {
            RCLCPP_ERROR(this->get_logger(),"服务连接失败!");
            return;
        }
        // 组织请求数据
        auto goal_msg = Nav::Goal();
        goal_msg.goal_x = x;
        goal_msg.goal_y = y;
        goal_msg.goal_theta = theta;
        //const rclcpp_action::Client<base_interfaces_demo::action::Nav>::SendGoalOptions &options
        rclcpp_action::Client<Nav>::SendGoalOptions options;
        options.goal_response_callback = std::bind(&ExeNavActionClient::goal_response_callback, this, _1);
        options.feedback_callback = std::bind(&ExeNavActionClient::feedback_callback, this, _1, _2);
        options.result_callback = std::bind(&ExeNavActionClient::result_callback, this, _1);
        // 发送
        nav_client->async_send_goal(goal_msg,options);

    }
private:
    rclcpp_action::Client<Nav>::SharedPtr nav_client;
    // 3-3.处理目标响应；
    void goal_response_callback(rclcpp_action::ClientGoalHandle<Nav>::SharedPtr goal_handle){
        if(!goal_handle){
            RCLCPP_ERROR(this->get_logger(),"目标请求被服务器拒绝");
        } else {
            RCLCPP_INFO(this->get_logger(),"目标请求被接收!");
        }
    }
    // 3-4.处理响应的连续反馈；
    void feedback_callback(rclcpp_action::ClientGoalHandle<Nav>::SharedPtr goal_handle, 
        const std::shared_ptr<const Nav::Feedback> feedback){
        (void)goal_handle;
        RCLCPP_INFO(this->get_logger(),"距离目标点还有 %.2f 米。",feedback->distance);
    }
    // 3-5.处理最终响应。
    void result_callback(const rclcpp_action::ClientGoalHandle<Nav>::WrappedResult & result){
        switch (result.code){
        case rclcpp_action::ResultCode::SUCCEEDED :
            RCLCPP_INFO(this->get_logger(),
                "乌龟最终坐标:(%.2f,%.2f),航向:%.2f",
                            result.result->turtle_x,
                            result.result->turtle_y,
                            result.result->turtle_theta
                            );
            break;
        case rclcpp_action::ResultCode::CANCELED:
            RCLCPP_ERROR(this->get_logger(),"任务被取消");
            break;      
        case rclcpp_action::ResultCode::ABORTED:
            RCLCPP_ERROR(this->get_logger(),"任务被中止");
            break;   
        default:
            RCLCPP_ERROR(this->get_logger(),"未知异常");
            break;
        }
        // rclcpp::shutdown();
    }
};

int main(int argc, char const *argv[])
{
    // 2.初始化 ROS2 客户端；
    rclcpp::init(argc,argv);
    // 4.调用spin函数，并传入节点对象指针；
    auto client = std::make_shared<ExeNavActionClient>();

    if (argc != 5)
    {
        RCLCPP_INFO(client->get_logger(),"请传入目标的位姿参数:(x,y,theta)");
        return 1;
    }
    // 发送目标点
    client->send_goal(atof(argv[1]), atof(argv[2]), atof(argv[3]));
    rclcpp::spin(client);
    // 5.释放资源。
    rclcpp::shutdown();
    return 0;
}
```

#### 4.launch文件

该案例需要分别为动作服务端和动作客户端创建launch文件。

功能包cpp07\_exercise的launch目录下，首先新建动作服务端launch文件exe04\_action\_server.launch.py，编辑文件，输入如下内容：

```py
from launch import LaunchDescription
from launch_ros.actions import Node

def generate_launch_description():
    # 创建turtlesim_node节点
    turtle = Node(package="turtlesim",executable="turtlesim_node")
    # 创建动作服务端节点
    server = Node(package="cpp07_exercise",executable="exe04_action_server")

    return LaunchDescription([turtle,server])
```

然后新建动作客户端launch文件exe05\_action\_client.launch.py，编辑文件，输入如下内容：

```py
from launch import LaunchDescription
from launch_ros.actions import Node
from launch.actions import ExecuteProcess

def generate_launch_description():
    # 设置目标点的坐标，以及目标点乌龟的名称
    x = 8.54
    y = 9.54
    theta = 0.0
    name = "t3"
    # 生成新的乌龟
    spawn = ExecuteProcess(
        cmd=["ros2 service call /spawn turtlesim/srv/Spawn \"{'x': "
                + str(x) + ",'y': " + str(y) + ",'theta': " + str(theta) + ",'name': '" + name + "'}\""],
        # output="both",
        shell=True
    )
    # 创建动作客户端节点
    client = Node(package="cpp07_exercise",
                executable="exe05_action_client",
                arguments=[str(x),str(y),str(theta)])
    return LaunchDescription([spawn,client])
```

#### 5.编辑配置文件

此处需要编辑base\_interfaces\_demo和cpp07\_exercise两个功能包下的配置文件。

##### 1.base\_interfaces\_demo下的CMakeLists.txt

和前面服务通信一样，只需要修改CMakeLists.txt中的rosidl\_generate\_interfaces 函数即可，修改后的内容如下：

```
rosidl_generate_interfaces(${PROJECT_NAME}
  "msg/Student.msg"
  "srv/AddInts.srv"
  "srv/Distance.srv"
  "action/Progress.action"
  "action/Nav.action"
)
```

##### 2.cpp07\_exercise下的CMakeLists.txt

CMakeLists.txt 文件需要添加如下内容：

```
add_executable(exe04_action_server src/exe04_action_server.cpp)
ament_target_dependencies(
  exe04_action_server
  "rclcpp"
  "turtlesim"
  "base_interfaces_demo"
  "geometry_msgs"
  "rclcpp_action"
)

add_executable(exe05_action_client src/exe05_action_client.cpp)
ament_target_dependencies(
  exe05_action_client
  "rclcpp"
  "turtlesim"
  "base_interfaces_demo"
  "geometry_msgs"
  "rclcpp_action"
)
```

文件中 install 修改为如下内容：

```
install(TARGETS 
  exe01_pub_sub
  exe02_server
  exe03_client
  exe04_action_server
  exe05_action_client  
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
ros2 launch cpp07_exercise exe04_action_server.launch.py
```

指令执行后，将生成turtlesim\_node节点对应的窗口，并且会启动乌龟导航的动作服务端。

终端2输入如下指令：

```
. install/setup.bash
ros2 launch cpp07_exercise exe05_action_client.launch.py
```

指令执行后，会生成一只新的乌龟，并且原生乌龟会以新乌龟为目标点向其运动，运动过程中，动作客户端会接收服务端连续反馈的剩余距离消息，最终运行结果与演示案例类似。

