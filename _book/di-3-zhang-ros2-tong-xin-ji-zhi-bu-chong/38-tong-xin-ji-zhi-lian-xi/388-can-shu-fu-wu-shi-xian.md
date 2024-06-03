### 3.8.8 参数服务实现

#### 1.参数客户端实现

功能包cpp07\_exercise的src目录下，新建C++文件exe06\_param.cpp，并编辑文件，输入如下内容：

```cpp
/*
   需求：修改turtlesim_node的背景颜色。
   步骤：
       1.包含头文件；
       2.初始化 ROS2 客户端；
       3.定义节点类；
            3-1.创建参数客户端；
            3-2.连接参数服务端；
            3-3.更新参数。
       4.创建对象指针,并调用其函数；
       5.释放资源。
*/
// 1.包含头文件；
#include "rclcpp/rclcpp.hpp"

using namespace std::chrono_literals;
// 3.定义节点类；
class ExeParamClient: public rclcpp::Node{
public:
    ExeParamClient():Node("exe_param_client"),red(0){
        // 3-1.创建参数客户端；
        param_client = std::make_shared<rclcpp::SyncParametersClient>(this,"/turtlesim");
    }
    // 3-2.连接参数服务端；
    bool connect_server(){
        while (!param_client->wait_for_service(1s))
        {
            if (!rclcpp::ok())
            {
                RCLCPP_INFO(this->get_logger(),"终端退出!");
                return false;
            }

            RCLCPP_INFO(this->get_logger(),"参数服务连接中，请稍等......");
        }
        return true;
    }
    // 3-3.更新参数。
    void update_param(){
        red = param_client->get_parameter<int32_t>("background_r");
        rclcpp::Rate rate(30.0);
        int i = red;
        while (rclcpp::ok())
        {
            i < 255 ? red += 5 : red -= 5;
            i += 5;
            if(i >= 510) i = 0;

            // RCLCPP_INFO(this->get_logger(),"red = %d", red);
            param_client->set_parameters({rclcpp::Parameter("background_r",red)});
            rate.sleep();
        }

    }
private:
    rclcpp::SyncParametersClient::SharedPtr param_client;
    int32_t red;
};

int main(int argc, char const *argv[])
{
    // 2.初始化 ROS2 客户端；
    rclcpp::init(argc,argv);

    // 4.创建对象指针,并调用其函数；
    auto param_client = std::make_shared<ExeParamClient>();
    if(!param_client->connect_server()) return 1;
    param_client->update_param();

    // rclcpp::spin(param_client);
    // 5.释放资源。
    rclcpp::shutdown();
    return 0;
}
```

#### 2.launch文件

功能包cpp07\_exercise的launch目录下，新建launch文件exe06\_param.launch.py，并编辑文件，输入如下内容：

```py
from launch import LaunchDescription
from launch_ros.actions import Node

def generate_launch_description():
    # 创建turtlesim_node节点
    turtle = Node(package="turtlesim",executable="turtlesim_node")
    # 创建背景色修改节点
    param = Node(package="cpp07_exercise",executable="exe06_param")

    return LaunchDescription([turtle,param])
```

#### 3.编辑配置文件

package.xml 无需修改，CMakeLists.txt 文件需要添加如下内容：

```
add_executable(exe06_param src/exe06_param.cpp)
ament_target_dependencies(
  exe06_param
  "rclcpp"
  "turtlesim"
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
  exe06_param
  DESTINATION lib/${PROJECT_NAME})
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
ros2 launch cpp07_exercise exe06_param.launch.py
```

指令执行后，将生成turtlesim\_node节点对应的窗口，窗口背景色会动态改变，最终运行结果与演示案例类似。

