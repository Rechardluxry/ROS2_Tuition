### 2.3.3 服务通信（C++）

#### 1.服务端实现

功能包cpp02\_service的src目录下，新建C++文件demo01\_server.cpp，并编辑文件，输入如下内容：

```cpp
/*  
  需求：编写服务端，接收客户端发送请求，提取其中两个整型数据，相加后将结果响应回客户端。
  步骤：
    1.包含头文件；
    2.初始化 ROS2 客户端；
    3.定义节点类；
      3-1.创建服务端；
      3-2.处理请求数据并响应结果。
    4.调用spin函数，并传入节点对象指针；
    5.释放资源。
*/

// 1.包含头文件；
#include "rclcpp/rclcpp.hpp"
#include "base_interfaces_demo/srv/add_ints.hpp"

using base_interfaces_demo::srv::AddInts;

using std::placeholders::_1;
using std::placeholders::_2;

// 3.定义节点类；
class MinimalService: public rclcpp::Node{
  public:
    MinimalService():Node("minimal_service"){
      // 3-1.创建服务端；
      server = this->create_service<AddInts>("add_ints",std::bind(&MinimalService::add, this, _1, _2));
      RCLCPP_INFO(this->get_logger(),"add_ints 服务端启动完毕，等待请求提交...");
    }
  private:
    rclcpp::Service<AddInts>::SharedPtr server;
    // 3-2.处理请求数据并响应结果。
    void add(const AddInts::Request::SharedPtr req,const AddInts::Response::SharedPtr res){
      res->sum = req->num1 + req->num2;
      RCLCPP_INFO(this->get_logger(),"请求数据:(%d,%d),响应结果:%d", req->num1, req->num2, res->sum);
    }
};

int main(int argc, char const *argv[])
{
  // 2.初始化 ROS2 客户端；
  rclcpp::init(argc,argv);

  // 4.调用spin函数，并传入节点对象指针；
  auto server = std::make_shared<MinimalService>();
  rclcpp::spin(server);

  // 5.释放资源。
  rclcpp::shutdown();
  return 0;
}
```

#### 2.客户端实现

功能包cpp02\_service的src目录下，新建C++文件demo02\_client.cpp，并编辑文件，输入如下内容：

```cpp
/*  
  需求：编写客户端，发送两个整型变量作为请求数据，并处理响应结果。
  步骤：
    1.包含头文件；
    2.初始化 ROS2 客户端；
    3.定义节点类；
      3-1.创建客户端；
      3-2.等待服务连接；
      3-3.组织请求数据并发送；
    4.创建对象指针调用其功能,并处理响应；
    5.释放资源。

*/
// 1.包含头文件；
#include "rclcpp/rclcpp.hpp"
#include "base_interfaces_demo/srv/add_ints.hpp"

using base_interfaces_demo::srv::AddInts;
using namespace std::chrono_literals;

// 3.定义节点类；
class MinimalClient: public rclcpp::Node{
  public:
    MinimalClient():Node("minimal_client"){
      // 3-1.创建客户端；
      client = this->create_client<AddInts>("add_ints");
      RCLCPP_INFO(this->get_logger(),"客户端创建，等待连接服务端！");
    }
    // 3-2.等待服务连接；
    bool connect_server(){
      while (!client->wait_for_service(1s))
      {
        if (!rclcpp::ok())
        {
          RCLCPP_INFO(rclcpp::get_logger("rclcpp"),"强制退出！");
          return false;
        }

        RCLCPP_INFO(this->get_logger(),"服务连接中，请稍候...");
      }
      return true;
    }
    // 3-3.组织请求数据并发送；
    rclcpp::Client<AddInts>::FutureAndRequestId send_request(int32_t num1, int32_t num2){
      auto request = std::make_shared<AddInts::Request>();
      request->num1 = num1;
      request->num2 = num2;
      return client->async_send_request(request);
    }


  private:
    rclcpp::Client<AddInts>::SharedPtr client;
};

int main(int argc, char ** argv)
{
  if (argc != 3){
    RCLCPP_INFO(rclcpp::get_logger("rclcpp"),"请提交两个整型数据！");
    return 1;
  }

  // 2.初始化 ROS2 客户端；
  rclcpp::init(argc,argv);

  // 4.创建对象指针并调用其功能；
  auto client = std::make_shared<MinimalClient>();
  bool flag = client->connect_server();
  if (!flag)
  {
    RCLCPP_INFO(rclcpp::get_logger("rclcpp"),"服务连接失败！");
    return 0;
  }

  auto response = client->send_request(atoi(argv[1]),atoi(argv[2]));

  // 处理响应
  if (rclcpp::spin_until_future_complete(client,response) == rclcpp::FutureReturnCode::SUCCESS)
  {
    RCLCPP_INFO(client->get_logger(),"请求正常处理");
    RCLCPP_INFO(client->get_logger(),"响应结果:%d!", response.get()->sum);

  } else {
    RCLCPP_INFO(client->get_logger(),"请求异常");
  }

  // 5.释放资源。
  rclcpp::shutdown();
  return 0;
}
```

#### 3.编辑配置文件

##### 1.packages.xml

在创建功能包时，所依赖的功能包已经自动配置了，配置内容如下：

```
<depend>rclcpp</depend>
<depend>base_interfaces_demo</depend>
```

##### 2.CMakeLists.txt

CMakeLists.txt 中服务端和客户端程序核心配置如下：

```
find_package(ament_cmake REQUIRED)
find_package(rclcpp REQUIRED)
find_package(base_interfaces_demo REQUIRED)

add_executable(demo01_server src/demo01_server.cpp)
ament_target_dependencies(
  demo01_server
  "rclcpp"
  "base_interfaces_demo"
)
add_executable(demo02_client src/demo02_client.cpp)
ament_target_dependencies(
  demo02_client
  "rclcpp"
  "base_interfaces_demo"
)

install(TARGETS 
  demo01_server
  demo02_client
  DESTINATION lib/${PROJECT_NAME})
```

#### 4.编译

终端中进入当前工作空间，编译功能包：

```
colcon build --packages-select cpp02_service
```

#### 5.执行

当前工作空间下，启动两个终端，终端1执行服务端程序，终端2执行客户端程序。

终端1输入如下指令：

```
. install/setup.bash
ros2 run cpp02_service demo01_server
```

终端2输入如下指令：

```
. install/setup.bash
ros2 run cpp02_service demo02_client 100 200
```

最终运行结果与案例类似。

