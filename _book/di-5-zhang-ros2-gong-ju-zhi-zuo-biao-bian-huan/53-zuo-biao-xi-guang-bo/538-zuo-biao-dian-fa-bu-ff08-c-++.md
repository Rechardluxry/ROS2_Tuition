### 5.3.8 坐标点发布（C++）

#### 1.话题发布实现

功能包 cpp03\_tf\_broadcaster 的 src 目录下，新建 C++ 文件 demo03\_point\_publisher.cpp，并编辑文件，输入如下内容：

```cpp
/*  
    需求：发布雷达坐标系中某个坐标点相对于雷达（laser）坐标系的位姿。
    步骤：
        1.包含头文件；
        2.初始化 ROS 客户端；
        3.定义节点类；
            3-1.创建坐标点发布方；
            3-2.创建定时器；
            3-3.组织并发布坐标点消息。
        4.调用 spin 函数，并传入对象指针；
        5.释放资源。


*/
// 1.包含头文件；
#include "rclcpp/rclcpp.hpp"
#include "geometry_msgs/msg/point_stamped.hpp"

using namespace std::chrono_literals;

// 3.定义节点类；
class MinimalPointPublisher: public rclcpp::Node {
public:
    MinimalPointPublisher(): Node("minimal_point_publisher"),x(0.1){
        // 3-1.创建坐标点发布方；
        point_pub_ = this->create_publisher<geometry_msgs::msg::PointStamped>("point",10);
        // 3-2.创建定时器；
        timer_ = this->create_wall_timer(0.1s,std::bind(&MinimalPointPublisher::on_timer, this));
    }
private:
    void on_timer(){
        // 3-3.组织并发布坐标点消息。
        geometry_msgs::msg::PointStamped point;
        point.header.frame_id = "laser";
        point.header.stamp = this->now();
        x += 0.004;
        point.point.x = x;
        point.point.y = 0.0;
        point.point.z = 0.1;        
        point_pub_->publish(point);
    }
    rclcpp::Publisher<geometry_msgs::msg::PointStamped>::SharedPtr point_pub_;
    rclcpp::TimerBase::SharedPtr timer_;
    double_t x;
};

int main(int argc, char const *argv[])
{
    // 2.初始化 ROS 客户端；
    rclcpp::init(argc,argv);
    // 4.调用 spin 函数，并传入对象指针；
    rclcpp::spin(std::make_shared<MinimalPointPublisher>());
    // 5.释放资源。
    rclcpp::shutdown();
    return 0;
}
```

#### 2.编辑配置文件

package.xml 无需修改，CMakeLists.txt 文件需要添加如下内容：

```
add_executable(demo03_point_publisher src/demo03_point_publisher.cpp)
ament_target_dependencies(
  demo03_point_publisher
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
  demo03_point_publisher
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

终端2输入如下命令发布障碍物相对于雷达（laser）的坐标点：

```
. install/setup.bash 
ros2 run cpp03_tf_broadcaster demo03_point_publisher
```

#### 5.rviz2 查看坐标系关系

参考 **5.3.2 静态广播器（命令）**内容启动并配置 rviz2，显示坐标变换后，再添加 PointStamped 插件并将其话题设置为 /point，最终显示结果与案例演示类似。

![](/assets/5.3.8rviz2添加坐标点.gif)

