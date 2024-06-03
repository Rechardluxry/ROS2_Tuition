### 2.5.3 参数服务（C++）

#### 1.参数服务端

功能包cpp04\_param的src目录下，新建C++文件demo01\_param\_server.cpp，并编辑文件，输入如下内容：

```cpp
/*
    需求：编写参数服务端，设置并操作参数。
    步骤：
        1.包含头文件；
        2.初始化 ROS2 客户端；
        3.定义节点类；
            3-1.声明参数；
            3-2.查询参数；
            3-3.修改参数；
            3-4.删除参数。
        4.创建节点对象指针，调用参数操作函数，并传递给spin函数；
        5.释放资源。

*/

// 1.包含头文件；
#include "rclcpp/rclcpp.hpp"

// 3.定义节点类；
class MinimalParamServer: public rclcpp::Node{
    public:
        MinimalParamServer():Node("minimal_param_server",rclcpp::NodeOptions()
                .allow_undeclared_parameters(true)
                ){       
        }
        // 3-1.声明参数；
        void declare_param(){
            // 声明参数并设置默认值
            this->declare_parameter("car_type","Tiger"); 
            this->declare_parameter("height",1.50); 
            this->declare_parameter("wheels",4);   
            // 需要设置 rclcpp::NodeOptions().allow_undeclared_parameters(true),否则非法 
            this->set_parameter(rclcpp::Parameter("undcl_test",100));
        }
        // 3-2.查询参数
        void get_param(){
            RCLCPP_INFO(this->get_logger(),"------------------查----------------");
            // 获取指定
            rclcpp::Parameter car_type = this->get_parameter("car_type");
            RCLCPP_INFO(this->get_logger(),"car_type:%s",car_type.as_string().c_str());
            RCLCPP_INFO(this->get_logger(),"height:%.2f",this->get_parameter("height").as_double());
            RCLCPP_INFO(this->get_logger(),"wheels:%ld",this->get_parameter("wheels").as_int());
            RCLCPP_INFO(this->get_logger(),"undcl_test:%ld",this->get_parameter("undcl_test").as_int());
            // 判断包含
            RCLCPP_INFO(this->get_logger(),"包含car_type? %d",this->has_parameter("car_type"));
            RCLCPP_INFO(this->get_logger(),"包含car_typesxxxx? %d",this->has_parameter("car_typexxxx"));
            // 获取所有
            auto params = this->get_parameters({"car_type","height","wheels"});
            for (auto &param : params)
            {
                RCLCPP_INFO(this->get_logger(),"name = %s, value = %s", param.get_name().c_str(), param.value_to_string().c_str());

            }
        }
        // 3-3.修改参数
        void update_param(){
            RCLCPP_INFO(this->get_logger(),"------------------改----------------");
            this->set_parameter(rclcpp::Parameter("height",1.75));
            RCLCPP_INFO(this->get_logger(),"height:%.2f",this->get_parameter("height").as_double());
        }
        // 3-4.删除参数
        void del_param(){
            RCLCPP_INFO(this->get_logger(),"------------------删----------------");
            // this->undeclare_parameter("car_type");
            // RCLCPP_INFO(this->get_logger(),"删除操作后，car_type还存在马? %d",this->has_parameter("car_type"));
            RCLCPP_INFO(this->get_logger(),"删除操作前，undcl_test存在马? %d",this->has_parameter("undcl_test"));
            this->undeclare_parameter("undcl_test");
            RCLCPP_INFO(this->get_logger(),"删除操作前，undcl_test存在马? %d",this->has_parameter("undcl_test"));
        }
};

int main(int argc, char ** argv)
{
    // 2.初始化 ROS2 客户端；
    rclcpp::init(argc,argv);

    // 4.创建节点对象指针，调用参数操作函数，并传递给spin函数；
    auto paramServer= std::make_shared<MinimalParamServer>();
    paramServer->declare_param();
    paramServer->get_param();
    paramServer->update_param();
    paramServer->del_param();
    rclcpp::spin(paramServer);

    // 5.释放资源。
    rclcpp::shutdown();
    return 0;
}
```

#### 2.参数客户端

功能包cpp04\_param的src目录下，新建C++文件demo02\_param\_client.cpp，并编辑文件，输入如下内容：

```cpp
/*
    需求：编写参数客户端，获取或修改服务端参数。
    步骤：
        1.包含头文件；
        2.初始化 ROS2 客户端；
        3.定义节点类；
            3-1.查询参数；
            3-2.修改参数；
        4.创建节点对象指针，调用参数操作函数；
        5.释放资源。
*/

// 1.包含头文件；
#include "rclcpp/rclcpp.hpp"

using namespace std::chrono_literals;

// 3.定义节点类；
class MinimalParamClient: public rclcpp::Node {
    public:
        MinimalParamClient():Node("paramDemoClient_node"){
            paramClient = std::make_shared<rclcpp::SyncParametersClient>(this,"minimal_param_server");
        }
        bool connect_server(){
            // 等待服务连接
            while (!paramClient->wait_for_service(1s))
            {
                if (!rclcpp::ok())
                {
                   return false;
                }  
                RCLCPP_INFO(this->get_logger(),"服务未连接");
            }

            return true;

        }

        // 3-1.查询参数；
        void get_param(){
            RCLCPP_INFO(this->get_logger(),"-----------参数客户端查询参数-----------");
            double height = paramClient->get_parameter<double>("height");
            RCLCPP_INFO(this->get_logger(),"height = %.2f", height);
            RCLCPP_INFO(this->get_logger(),"car_type 存在吗？%d", paramClient->has_parameter("car_type"));
            auto params = paramClient->get_parameters({"car_type","height","wheels"});
            for (auto &param : params)
            {
                RCLCPP_INFO(this->get_logger(),"%s = %s", param.get_name().c_str(),param.value_to_string().c_str());
            }


        }
        // 3-2.修改参数；
        void update_param(){
            RCLCPP_INFO(this->get_logger(),"-----------参数客户端修改参数-----------");
            paramClient->set_parameters({rclcpp::Parameter("car_type","Mouse"),
            rclcpp::Parameter("height",2.0),
            //这是服务端不存在的参数，只有服务端设置了rclcpp::NodeOptions().allow_undeclared_parameters(true)时，
            // 这个参数才会被成功设置。
            rclcpp::Parameter("width",0.15),
            rclcpp::Parameter("wheels",6)});
        }

    private:
        rclcpp::SyncParametersClient::SharedPtr paramClient;
};

int main(int argc, char const *argv[])
{
    // 2.初始化 ROS2 客户端；
    rclcpp::init(argc,argv);

    // 4.创建节点对象指针，调用参数操作函数；
    auto paramClient = std::make_shared<MinimalParamClient>();
    bool flag = paramClient->connect_server();
    if(!flag){
        return 0;
    }
    paramClient->get_param();
    paramClient->update_param();
    paramClient->get_param();

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
```

##### 2.CMakeLists.txt

CMakeLists.txt中参数服务端和参数客户端程序核心配置如下：

```
find_package(rclcpp REQUIRED)

add_executable(demo01_param_server src/demo01_param_server.cpp)
ament_target_dependencies(
  demo01_param_server
  "rclcpp"
)
add_executable(demo02_param_client src/demo02_param_client.cpp)
ament_target_dependencies(
  demo02_param_client
  "rclcpp"
)

install(TARGETS 
  demo01_param_server
  demo02_param_client
  DESTINATION lib/${PROJECT_NAME})
```

#### 4.编译

终端中进入当前工作空间，编译功能包：

```
colcon build --packages-select cpp04_param
```

#### 5.执行

当前工作空间下，启动两个终端，终端1执行参数服务端程序，终端2执行参数客户端程序。

终端1输入如下指令：

```
. install/setup.bash
ros2 run cpp04_param demo01_param_server
```

终端2输入如下指令：

```
. install/setup.bash
ros2 run cpp04_param demo02_param_client
```

最终运行结果与案例类似。

