### 2.4.3 动作通信（C++）

#### 1.动作服务端实现

功能包cpp03\_action的src目录下，新建C++文件demo01\_action\_server.cpp，并编辑文件，输入如下内容：

```cpp
/*  
  需求：编写动作服务端实习，可以提取客户端请求提交的整型数据，并累加从1到该数据之间的所有整数以求和，
       每累加一次都计算当前运算进度并连续反馈回客户端，最后，在将求和结果返回给客户端。
  步骤：
    1.包含头文件；
    2.初始化 ROS2 客户端；
    3.定义节点类；
      3-1.创建动作服务端；
      3-2.处理请求数据；
      3-3.处理取消任务请求；
      3-4.生成连续反馈。
    4.调用spin函数，并传入节点对象指针；
    5.释放资源。

*/
// 1.包含头文件；
#include "rclcpp/rclcpp.hpp"
#include "rclcpp_action/rclcpp_action.hpp"
#include "base_interfaces_demo/action/progress.hpp"

using namespace std::placeholders;
using base_interfaces_demo::action::Progress;
using GoalHandleProgress = rclcpp_action::ServerGoalHandle<Progress>;

// 3.定义节点类；
class MinimalActionServer : public rclcpp::Node
{
public:

  explicit MinimalActionServer(const rclcpp::NodeOptions & options = rclcpp::NodeOptions())
  : Node("minimal_action_server", options)
  {
    // 3-1.创建动作服务端；
    this->action_server_ = rclcpp_action::create_server<Progress>(
      this,
      "get_sum",
      std::bind(&MinimalActionServer::handle_goal, this, _1, _2),
      std::bind(&MinimalActionServer::handle_cancel, this, _1),
      std::bind(&MinimalActionServer::handle_accepted, this, _1));
    RCLCPP_INFO(this->get_logger(),"动作服务端创建，等待请求...");
  }

private:
  rclcpp_action::Server<Progress>::SharedPtr action_server_;

  // 3-2.处理请求数据；
  rclcpp_action::GoalResponse handle_goal(const rclcpp_action::GoalUUID & uuid,std::shared_ptr<const Progress::Goal> goal)
  {
    (void)uuid;
    RCLCPP_INFO(this->get_logger(), "接收到动作客户端请求，请求数字为 %ld", goal->num);
    if (goal->num < 1) {
      return rclcpp_action::GoalResponse::REJECT;
    }
    return rclcpp_action::GoalResponse::ACCEPT_AND_EXECUTE;
  }

  // 3-3.处理取消任务请求；
  rclcpp_action::CancelResponse handle_cancel(
    const std::shared_ptr<GoalHandleProgress> goal_handle)
  {
    (void)goal_handle;
    RCLCPP_INFO(this->get_logger(), "接收到任务取消请求");
    return rclcpp_action::CancelResponse::ACCEPT;
  }

  void execute(const std::shared_ptr<GoalHandleProgress> goal_handle)
  {
    RCLCPP_INFO(this->get_logger(), "开始执行任务");
    rclcpp::Rate loop_rate(10.0);
    const auto goal = goal_handle->get_goal();
    auto feedback = std::make_shared<Progress::Feedback>();
    auto result = std::make_shared<Progress::Result>();
    int64_t sum= 0;
    for (int i = 1; (i <= goal->num) && rclcpp::ok(); i++) {
      sum += i;
      // Check if there is a cancel request
      if (goal_handle->is_canceling()) {
        result->sum = sum;
        goal_handle->canceled(result);
        RCLCPP_INFO(this->get_logger(), "任务取消");
        return;
      }
      feedback->progress = (double_t)i / goal->num;
      goal_handle->publish_feedback(feedback);
      RCLCPP_INFO(this->get_logger(), "连续反馈中，进度：%.2f", feedback->progress);

      loop_rate.sleep();
    }

    if (rclcpp::ok()) {
      result->sum = sum;
      goal_handle->succeed(result);
      RCLCPP_INFO(this->get_logger(), "任务完成！");
    }
  }

  // 3-4.生成连续反馈。
  void handle_accepted(const std::shared_ptr<GoalHandleProgress> goal_handle)
  {
    std::thread{std::bind(&MinimalActionServer::execute, this, _1), goal_handle}.detach();
  }
}; 

int main(int argc, char ** argv)
{
  // 2.初始化 ROS2 客户端；
  rclcpp::init(argc, argv);
  // 4.调用spin函数，并传入节点对象指针；
  auto action_server = std::make_shared<MinimalActionServer>();
  rclcpp::spin(action_server);
  // 5.释放资源。
  rclcpp::shutdown();
  return 0;
}
```

#### 2.动作客户端实现

功能包cpp03\_action的src目录下，新建C++文件demo02\_action\_client.cpp，并编辑文件，输入如下内容：

```cpp
/*  
  需求：编写动作客户端实现，可以提交一个整型数据到服务端，并处理服务端的连续反馈以及最终返回结果。
  步骤：
    1.包含头文件；
    2.初始化 ROS2 客户端；
    3.定义节点类；
      3-1.创建动作客户端；
      3-2.发送请求；
      3-3.处理目标发送后的反馈；
      3-4.处理连续反馈；
      3-5.处理最终响应。
    4.调用spin函数，并传入节点对象指针；
    5.释放资源。
*/
// 1.包含头文件；
#include "rclcpp/rclcpp.hpp"
#include "rclcpp_action/rclcpp_action.hpp"
#include "base_interfaces_demo/action/progress.hpp"

using base_interfaces_demo::action::Progress;
using GoalHandleProgress = rclcpp_action::ClientGoalHandle<Progress>;
using namespace std::placeholders;

// 3.定义节点类；
class MinimalActionClient : public rclcpp::Node
{
public:

  explicit MinimalActionClient(const rclcpp::NodeOptions & node_options = rclcpp::NodeOptions())
  : Node("minimal_action_client", node_options)
  {
    // 3-1.创建动作客户端；
    this->client_ptr_ = rclcpp_action::create_client<Progress>(this,"get_sum");
  }

  // 3-2.发送请求；
  void send_goal(int64_t num)
  {

    if (!this->client_ptr_) {
      RCLCPP_ERROR(this->get_logger(), "动作客户端未被初始化。");
    }

    if (!this->client_ptr_->wait_for_action_server(std::chrono::seconds(10))) {
      RCLCPP_ERROR(this->get_logger(), "服务连接失败！");
      return;
    }

    auto goal_msg = Progress::Goal();
    goal_msg.num = num;
    RCLCPP_INFO(this->get_logger(), "发送请求数据！");

    auto send_goal_options = rclcpp_action::Client<Progress>::SendGoalOptions();
    send_goal_options.goal_response_callback =std::bind(&MinimalActionClient::goal_response_callback, this, _1);
    send_goal_options.feedback_callback =std::bind(&MinimalActionClient::feedback_callback, this, _1, _2);
    send_goal_options.result_callback =std::bind(&MinimalActionClient::result_callback, this, _1);
    auto goal_handle_future = this->client_ptr_->async_send_goal(goal_msg, send_goal_options);
  }

private:
  rclcpp_action::Client<Progress>::SharedPtr client_ptr_;

  // 3-3.处理目标发送后的反馈；
  void goal_response_callback(GoalHandleProgress::SharedPtr goal_handle)
  {
    if (!goal_handle) {
      RCLCPP_ERROR(this->get_logger(), "目标请求被服务器拒绝！");
    } else {
      RCLCPP_INFO(this->get_logger(), "目标被接收，等待结果中");
    }
  }

  // 3-4.处理连续反馈；
  void feedback_callback(GoalHandleProgress::SharedPtr,const std::shared_ptr<const Progress::Feedback> feedback)
  {
    int32_t progress = (int32_t)(feedback->progress * 100);
    RCLCPP_INFO(this->get_logger(), "当前进度: %d%%", progress);
  }

  // 3-5.处理最终响应。
  void result_callback(const GoalHandleProgress::WrappedResult & result)
  {
    switch (result.code) {
      case rclcpp_action::ResultCode::SUCCEEDED:
        break;
      case rclcpp_action::ResultCode::ABORTED:
        RCLCPP_ERROR(this->get_logger(), "任务被中止");
        return;
      case rclcpp_action::ResultCode::CANCELED:
        RCLCPP_ERROR(this->get_logger(), "任务被取消");
        return;
      default:
        RCLCPP_ERROR(this->get_logger(), "未知异常");
        return;
    }

    RCLCPP_INFO(this->get_logger(), "任务执行完毕，最终结果: %ld", result.result->sum);
  }
}; 

int main(int argc, char ** argv)
{
  // 2.初始化 ROS2 客户端；
  rclcpp::init(argc, argv);

  // 4.调用spin函数，并传入节点对象指针；
  auto action_client = std::make_shared<MinimalActionClient>();
  action_client->send_goal(10);
  rclcpp::spin(action_client);
  // 5.释放资源。
  rclcpp::shutdown();
  return 0;
}
```

#### 3.编辑配置文件

##### 1.packages.xml

在创建功能包时，所依赖的功能包已经自动配置了，配置内容如下：

```xml
<depend>rclcpp</depend>
<depend>rclcpp_action</depend>
<depend>base_interfaces_demo</depend>
```

##### 2.CMakeLists.txt

CMakeLists.txt中服务端和客户端程序核心配置如下：

```
find_package(rclcpp REQUIRED)
find_package(rclcpp_action REQUIRED)
find_package(base_interfaces_demo REQUIRED)

add_executable(demo01_action_server src/demo01_action_server.cpp)
ament_target_dependencies(
  demo01_action_server
  "rclcpp"
  "rclcpp_action"
  "base_interfaces_demo"
)

add_executable(demo02_action_client src/demo02_action_client.cpp)
ament_target_dependencies(
  demo02_action_client
  "rclcpp"
  "rclcpp_action"
  "base_interfaces_demo"
)

install(TARGETS 
  demo01_action_server
  demo02_action_client
  DESTINATION lib/${PROJECT_NAME})
```

#### 4.编译

终端中进入当前工作空间，编译功能包：

```
colcon build --packages-select cpp03_action
```

#### 5.执行

当前工作空间下，启动两个终端，终端1执行动作服务端程序，终端2执行动作客户端程序。

终端1输入如下指令：

```
. install/setup.bash
ros2 run cpp03_action demo01_action_server
```

终端2输入如下指令：

```
. install/setup.bash
ros2 run cpp03_action demo02_action_client
```

最终运行结果与案例类似。

