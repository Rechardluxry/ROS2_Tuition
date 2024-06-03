### 5.3.5 动态广播器（C++）

#### 1.广播实现

功能包 cpp03\_tf\_broadcaster 的 src 目录下，新建 C++ 文件 demo02\_dynamic\_tf\_broadcaster.cpp，并编辑文件，输入如下内容：

```cpp
/*   
  需求：编写动态坐标变换程序，启动 turtlesim_node 以及 turtle_teleop_key 后，该程序可以发布
       乌龟坐标系到窗口坐标系的坐标变换，并且键盘控制乌龟运动时，乌龟坐标系与窗口坐标系的相对关系
       也会实时更新。

  步骤：
    1.包含头文件；
    2.初始化 ROS 客户端；
    3.定义节点类；
      3-1.创建动态坐标变换发布方；
      3-2.创建乌龟位姿订阅方；
      3-3.根据订阅到的乌龟位姿生成坐标帧并广播。
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
class MinimalDynamicFrameBroadcaster : public rclcpp::Node
{
public:
  MinimalDynamicFrameBroadcaster(): Node("minimal_dynamic_frame_broadcaster")
  {
    // 3-1.创建动态坐标变换发布方；
    tf_broadcaster_ = std::make_unique<tf2_ros::TransformBroadcaster>(*this);

    std::string topic_name = "/turtle1/pose";

    // 3-2.创建乌龟位姿订阅方；
    subscription_ = this->create_subscription<turtlesim::msg::Pose>(
      topic_name, 10,
      std::bind(&MinimalDynamicFrameBroadcaster::handle_turtle_pose, this, _1));
  }

private:
  // 3-3.根据订阅到的乌龟位姿生成坐标帧并广播。   
  void handle_turtle_pose(const turtlesim::msg::Pose & msg)
  {
    // 组织消息
    geometry_msgs::msg::TransformStamped t;
    rclcpp::Time now = this->get_clock()->now();

    t.header.stamp = now;
    t.header.frame_id = "world";
    t.child_frame_id = "turtle1";

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
};

int main(int argc, char * argv[])
{
  // 2.初始化 ROS 客户端；
  rclcpp::init(argc, argv);
  // 4.调用 spin 函数，并传入对象指针；
  rclcpp::spin(std::make_shared<MinimalDynamicFrameBroadcaster>());
  // 5.释放资源。
  rclcpp::shutdown();
  return 0;
}
```

#### 2.编辑配置文件

package.xml 无需修改，CMakeLists.txt 文件需要添加如下内容：

```
add_executable(demo02_dynamic_tf_broadcaster src/demo02_dynamic_tf_broadcaster.cpp)
ament_target_dependencies(
  demo02_dynamic_tf_broadcaster
  "rclcpp"
  "tf2"
  "tf2_ros"
  "geometry_msgs"
  "turtlesim"
)
```

文件中 install 修改为如下内容：

```
install(TARGETS demo01_static_tf_broadcaster
  demo02_dynamic_tf_broadcaster
  DESTINATION lib/${PROJECT_NAME})
```

#### 3.编译

终端中进入当前工作空间，编译功能包：

```
colcon build --packages-select cpp03_tf_broadcaster
```

#### 4.执行

启动两个终端，终端1输入如下命令：

```
ros2 run turtlesim turtlesim_node
```

终端2输入如下命令：

```
ros2 run turtlesim turtle_teleop_key
```

再在当前工作空间下，启动终端，输入如下命令：

```
. install/setup.bash 
ros2 run cpp03_tf_broadcaster demo02_dynamic_tf_broadcaster
```

#### 5.rviz2 查看坐标系关系

参考 **5.3.2 静态广播器（命令）**内容启动并配置 rviz2（Global Options 中的 Fixed Frame 设置为 world），通过键盘控制乌龟运动，最终执行结果与案例2类似。

