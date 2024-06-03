### 5.3.3 静态广播器（C++）

#### 1.广播实现

功能包 cpp03\_tf\_broadcaster 的 src 目录下，新建 C++ 文件 demo01\_static\_tf\_broadcaster.cpp，并编辑文件，输入如下内容：

```cpp
/*  
  需求：编写静态坐标变换程序，执行时传入两个坐标系的相对位姿关系以及父子级坐标系id，
       程序运行发布静态坐标变换。
  步骤：
    1.包含头文件；
    2.判断终端传入的参数是否合法；
    3.初始化 ROS 客户端；
    4.定义节点类；
      4-1.创建静态坐标变换发布方；
      4-2.组织并发布消息。
    5.调用 spin 函数，并传入对象指针；
    6.释放资源。

*/

// 1.包含头文件；
#include <geometry_msgs/msg/transform_stamped.hpp>

#include <rclcpp/rclcpp.hpp>
#include <tf2/LinearMath/Quaternion.h>
#include <tf2_ros/static_transform_broadcaster.h>

using std::placeholders::_1;

// 4.定义节点类；
class MinimalStaticFrameBroadcaster : public rclcpp::Node
{
public:
  explicit MinimalStaticFrameBroadcaster(char * transformation[]): Node("minimal_static_frame_broadcaster")
  {
    // 4-1.创建静态坐标变换发布方；
    tf_publisher_ = std::make_shared<tf2_ros::StaticTransformBroadcaster>(this);

    this->make_transforms(transformation);
  }

private:
  // 4-2.组织并发布消息。
  void make_transforms(char * transformation[])
  {
    // 组织消息
    geometry_msgs::msg::TransformStamped t;

    rclcpp::Time now = this->get_clock()->now();
    t.header.stamp = now;
    t.header.frame_id = transformation[7];
    t.child_frame_id = transformation[8];

    t.transform.translation.x = atof(transformation[1]);
    t.transform.translation.y = atof(transformation[2]);
    t.transform.translation.z = atof(transformation[3]);
    tf2::Quaternion q;
    q.setRPY(
      atof(transformation[4]),
      atof(transformation[5]),
      atof(transformation[6]));
    t.transform.rotation.x = q.x();
    t.transform.rotation.y = q.y();
    t.transform.rotation.z = q.z();
    t.transform.rotation.w = q.w();

    // 发布消息
    tf_publisher_->sendTransform(t);
  }
  std::shared_ptr<tf2_ros::StaticTransformBroadcaster> tf_publisher_;
};

int main(int argc, char * argv[])
{
  // 2.判断终端传入的参数是否合法；
  auto logger = rclcpp::get_logger("logger");

  if (argc != 9) {
    RCLCPP_INFO(
      logger, "运行程序时请按照：x y z roll pitch yaw frame_id child_frame_id 的格式传入参数");
    return 1;
  }

  // 3.初始化 ROS 客户端；
  rclcpp::init(argc, argv);
  // 5.调用 spin 函数，并传入对象指针；
  rclcpp::spin(std::make_shared<MinimalStaticFrameBroadcaster>(argv));
  // 6.释放资源。
  rclcpp::shutdown();
  return 0;
}
```

#### 2.编辑配置文件

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
find_package(ament_cmake REQUIRED)
find_package(rclcpp REQUIRED)
find_package(tf2 REQUIRED)
find_package(tf2_ros REQUIRED)
find_package(geometry_msgs REQUIRED)
find_package(turtlesim REQUIRED)

add_executable(demo01_static_tf_broadcaster src/demo01_static_tf_broadcaster.cpp)
ament_target_dependencies(
  demo01_static_tf_broadcaster
  "rclcpp"
  "tf2"
  "tf2_ros"
  "geometry_msgs"
  "turtlesim"
)

install(TARGETS demo01_static_tf_broadcaster
  DESTINATION lib/${PROJECT_NAME})
```

#### 3.编译

终端中进入当前工作空间，编译功能包：

```
colcon build --packages-select cpp03_tf_broadcaster
```

#### 4.执行

当前工作空间下，启动两个终端，终端1输入如下命令发布雷达（laser）相对于底盘（base\_link）的静态坐标变换：

```
. install/setup.bash 
ros2 run cpp03_tf_broadcaster demo01_static_tf_broadcaster 0.4 0 0.2 0 0 0 base_link laser
```

终端2输入如下命令发布摄像头（camera）相对于底盘（base\_link）的静态坐标变换：

```
. install/setup.bash 
ros2 run cpp03_tf_broadcaster demo01_static_tf_broadcaster -0.5 0 0.4 0 0 0 base_link camera
```

#### 5.rviz2 查看坐标系关系

参考 **5.3.2 静态广播器（命令）**内容启动并配置 rviz2，最终执行结果与案例1类似。

