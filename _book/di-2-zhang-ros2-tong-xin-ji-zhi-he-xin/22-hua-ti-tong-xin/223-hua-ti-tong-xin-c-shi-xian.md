### 2.2.5 话题通信之自定义消息（C++）

**准备**

> C++文件中包含自定义消息相关头文件时，可能会抛出异常，可以配置VSCode中c\_cpp\_properties.json文件，在文件中的 includePath属性下添加一行："${workspaceFolder}/install/base\_interfaces\_demo/include/\*\*"
>
> 添加完毕后，包含相关头文件时，就不会抛出异常了，其他接口文件或接口包的使用也与此同理。

#### 1.发布方实现

功能包cpp01\_topic的src目录下，新建C++文件demo01\_talker\_stu.cpp，并编辑文件，输入如下内容：

```cpp
/*  
  需求：以某个固定频率发送文本学生信息，包含学生的姓名、年龄、身高等数据。
*/

// 1.包含头文件；
#include "rclcpp/rclcpp.hpp"
#include "base_interfaces_demo/msg/student.hpp"

using namespace std::chrono_literals;
using base_interfaces_demo::msg::Student;
// 3.定义节点类；
class MinimalPublisher : public rclcpp::Node
{
  public:
    MinimalPublisher()
    : Node("student_publisher"), count_(0)
    {
      // 3-1.创建发布方；
      publisher_ = this->create_publisher<Student>("topic_stu", 10);
      // 3-2.创建定时器；
      timer_ = this->create_wall_timer(500ms, std::bind(&MinimalPublisher::timer_callback, this));
    }

  private:
    void timer_callback()
    {
      // 3-3.组织消息并发布。
      auto stu = Student();
      stu.name = "张三";
      stu.age = count_++;
      stu.height = 1.65;
      RCLCPP_INFO(this->get_logger(), "学生信息:name=%s,age=%d,height=%.2f", stu.name.c_str(),stu.age,stu.height);
      publisher_->publish(stu);

    }
    rclcpp::TimerBase::SharedPtr timer_;
    rclcpp::Publisher<Student>::SharedPtr publisher_;
    size_t count_;
};

int main(int argc, char * argv[])
{
  // 2.初始化 ROS2 客户端；
  rclcpp::init(argc, argv);
  // 4.调用spin函数，并传入节点对象指针。
  rclcpp::spin(std::make_shared<MinimalPublisher>());
  // 5.释放资源；
  rclcpp::shutdown();
  return 0;
}
```

#### 2.订阅方实现

功能包cpp01\_topic的src目录下，新建C++文件demo04\_listener\_stu.cpp，并编辑文件，输入如下内容：

```cpp
/*  
    需求：订阅发布方发布的学生消息，并输出到终端。
*/

// 1.包含头文件；
#include "rclcpp/rclcpp.hpp"
#include "base_interfaces_demo/msg/student.hpp"

using std::placeholders::_1;
using base_interfaces_demo::msg::Student;
// 3.定义节点类；
class MinimalSubscriber : public rclcpp::Node
{
  public:
    MinimalSubscriber()
    : Node("student_subscriber")
    {
      // 3-1.创建订阅方；
      subscription_ = this->create_subscription<Student>("topic_stu", 10, std::bind(&MinimalSubscriber::topic_callback, this, _1));
    }

  private:
    // 3-2.处理订阅到的消息；
    void topic_callback(const Student & msg) const
    {
      RCLCPP_INFO(this->get_logger(), "订阅的学生消息：name=%s,age=%d,height=%.2f", msg.name.c_str(),msg.age, msg.height);
    }
    rclcpp::Subscription<Student>::SharedPtr subscription_;
};

int main(int argc, char * argv[])
{
  // 2.初始化 ROS2 客户端；
  rclcpp::init(argc, argv);
  // 4.调用spin函数，并传入节点对象指针。
  rclcpp::spin(std::make_shared<MinimalSubscriber>());
  // 5.释放资源；
  rclcpp::shutdown();
  return 0;
}
```

#### 3.编辑配置文件

package.xml无需修改，CMakeLists.txt文件需要添加如下内容：

```
add_executable(demo03_talker_stu src/demo03_talker_stu.cpp)
ament_target_dependencies(
  demo03_talker_stu
  "rclcpp"
  "std_msgs"
  "base_interfaces_demo"
)

add_executable(demo04_listener_stu src/demo04_listener_stu.cpp)
ament_target_dependencies(
  demo04_listener_stu
  "rclcpp"
  "std_msgs"
  "base_interfaces_demo"
)
```

文件中install修改为如下内容：

```
install(TARGETS 
  demo01_talker_str
  demo02_listener_str
  demo03_talker_stu
  demo04_listener_stu
  DESTINATION lib/${PROJECT_NAME})
```

#### 4.编译

终端中进入当前工作空间，编译功能包：

```
colcon build --packages-select cpp01_topic
```

#### 5.执行

当前工作空间下，启动两个终端，终端1执行发布程序，终端2执行订阅程序。

终端1输入如下指令：

```
. install/setup.bash
ros2 run cpp01_topic demo03_talker_stu
```

终端2输入如下指令：

```
. install/setup.bash 
ros2 run cpp01_topic demo04_listener_stu
```

最终运行结果与案例2类似。

---



