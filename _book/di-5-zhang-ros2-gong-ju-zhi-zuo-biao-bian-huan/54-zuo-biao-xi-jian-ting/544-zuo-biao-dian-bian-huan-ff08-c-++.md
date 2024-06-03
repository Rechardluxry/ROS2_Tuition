### 5.4.4 坐标点变换（C++）

#### 1.坐标点变换实现

功能包 cpp04\_tf\_listener 的 src 目录下，新建 C++ 文件 demo02\_message\_filter.cpp，并编辑文件，输入如下内容：

```cpp
/*  
  需求：将雷达感知到的障碍物的坐标点数据（相对于 laser 坐标系），
       转换成相对于底盘坐标系（base_link）的坐标点。

  步骤：
    1.包含头文件；
    2.初始化 ROS 客户端；
    3.定义节点类；
      3-1.创建tf缓存对象指针；
      3-2.创建tf监听器；
      3-3.创建坐标点订阅方并订阅指定话题；
      3-4.创建消息过滤器过滤被处理的数据；
      3-5.为消息过滤器注册转换坐标点数据的回调函数。
    4.调用 spin 函数，并传入对象指针；
    5.释放资源。

*/
// 1.包含头文件；
#include <geometry_msgs/msg/point_stamped.hpp>
#include <message_filters/subscriber.h>

#include <rclcpp/rclcpp.hpp>
#include <tf2_ros/buffer.h>
#include <tf2_ros/create_timer_ros.h>
#include <tf2_ros/message_filter.h>
#include <tf2_ros/transform_listener.h>
// #ifdef TF2_CPP_HEADERS
//   #include <tf2_geometry_msgs/tf2_geometry_msgs.hpp>
// #else
//   #include <tf2_geometry_msgs/tf2_geometry_msgs.h>
// #endif

#include <tf2_geometry_msgs/tf2_geometry_msgs.hpp>

using namespace std::chrono_literals;

// 3.定义节点类；
class MessageFilterPointListener : public rclcpp::Node
{
public:
  MessageFilterPointListener(): Node("message_filter_point_listener")
  {

    target_frame_ = "base_link";

    typedef std::chrono::duration<int> seconds_type;
    seconds_type buffer_timeout(1);
    // 3-1.创建tf缓存对象指针；
    tf2_buffer_ = std::make_shared<tf2_ros::Buffer>(this->get_clock());
    auto timer_interface = std::make_shared<tf2_ros::CreateTimerROS>(
      this->get_node_base_interface(),
      this->get_node_timers_interface());
    tf2_buffer_->setCreateTimerInterface(timer_interface);
    // 3-2.创建tf监听器；
    tf2_listener_ = std::make_shared<tf2_ros::TransformListener>(*tf2_buffer_);

    // 3-3.创建坐标点订阅方并订阅指定话题；
    point_sub_.subscribe(this, "point");
    // 3-4.创建消息过滤器过滤被处理的数据；
    tf2_filter_ = std::make_shared<tf2_ros::MessageFilter<geometry_msgs::msg::PointStamped>>(
      point_sub_, *tf2_buffer_, target_frame_, 100, this->get_node_logging_interface(),
      this->get_node_clock_interface(), buffer_timeout);
    // 3-5.为消息过滤器注册转换坐标点数据的回调函数。
    tf2_filter_->registerCallback(&MessageFilterPointListener::msgCallback, this);
  }

private:
  void msgCallback(const geometry_msgs::msg::PointStamped::SharedPtr point_ptr)
  {
    geometry_msgs::msg::PointStamped point_out;
    try {
      tf2_buffer_->transform(*point_ptr, point_out, target_frame_);
      RCLCPP_INFO(
        this->get_logger(), "坐标点相对于base_link的坐标:(%.2f,%.2f,%.2f)",
        point_out.point.x,
        point_out.point.y,
        point_out.point.z);
    } catch (tf2::TransformException & ex) {
      RCLCPP_WARN(
        // Print exception which was caught
        this->get_logger(), "Failure %s\n", ex.what());
    }
  }
  std::string target_frame_;
  std::shared_ptr<tf2_ros::Buffer> tf2_buffer_;
  std::shared_ptr<tf2_ros::TransformListener> tf2_listener_;
  message_filters::Subscriber<geometry_msgs::msg::PointStamped> point_sub_;
  std::shared_ptr<tf2_ros::MessageFilter<geometry_msgs::msg::PointStamped>> tf2_filter_;
};

int main(int argc, char * argv[])
{
  // 2.初始化 ROS 客户端；
  rclcpp::init(argc, argv);
  // 4.调用 spin 函数，并传入对象指针；
  rclcpp::spin(std::make_shared<MessageFilterPointListener>());
  // 5.释放资源。
  rclcpp::shutdown();
  return 0;
}
```

#### 2.编辑配置文件

##### 1.package.xml

在创建功能包时，所依赖的部分功能包已经自动配置了，不过为了实现坐标点变换，还需要添加依赖包`tf2_geometry_msgs`和

`message_filters`，修改后的配置内容如下：

```XML
<depend>rclcpp</depend>
<depend>tf2</depend>
<depend>tf2_ros</depend>
<depend>geometry_msgs</depend>
<depend>tf2_geometry_msgs</depend>
<depend>message_filters</depend>
```

##### 2.CMakeLists.txt

CMakeLists.txt 文件修改后的内容如下：

```
# find dependencies
find_package(ament_cmake REQUIRED)
find_package(rclcpp REQUIRED)
find_package(tf2 REQUIRED)
find_package(tf2_ros REQUIRED)
find_package(geometry_msgs REQUIRED)
find_package(tf2_geometry_msgs REQUIRED)

add_executable(demo01_tf_listener src/demo01_tf_listener.cpp)

ament_target_dependencies(
  demo01_tf_listener
  "rclcpp"
  "tf2"
  "tf2_ros"
  "geometry_msgs"
)


add_executable(demo02_message_filter src/demo02_message_filter.cpp)
ament_target_dependencies(
  demo02_message_filter
  "rclcpp"
  "tf2"
  "tf2_ros"
  "geometry_msgs"
  "tf2_geometry_msgs"
  "message_filters"
)

install(TARGETS demo01_tf_listener
  demo02_message_filter
  DESTINATION lib/${PROJECT_NAME})
```

#### 3.编译

终端中进入当前工作空间，编译功能包：

```
colcon build --packages-select cpp04_tf_listener
```

#### 4.执行

在当前工作空间下，启动三个终端，终端1输入如下命令发布雷达（laser）相对于底盘（base\_link）的静态坐标变换：

```
. install/setup.bash 
ros2 run cpp03_tf_broadcaster demo01_static_tf_broadcaster 0.4 0 0.2 0 0 0 base_link laser
```

终端2输入如下命令发布障碍物相对于雷达（laser）的坐标点：

```
. install/setup.bash 
ros2 run cpp03_tf_broadcaster demo03_point_publisher
```

终端3输入如下命令执行坐标点变换：

```
. install/setup.bash 
ros2 run cpp04_tf_listener demo02_message_filter
```

终端3将输出坐标点相对于 base\_link 的坐标，具体结果请参考案例2。

