### 3.6.1 Rate

第2章话题通信案例中，要求话题发布方按照一定的频率发布消息，我们实现时是通过定时器来控制发布频率的，其实，除了定时器之外，ROS2 中还提供了 Rate 类，通过该类对象也可以控制程序的运行频率。

#### 1.rclcpp 中的 Rate

**示例：**周期性输出一段文本。

```cpp
#include "rclcpp/rclcpp.hpp"

using namespace std::chrono_literals;

int main(int argc, char ** argv)
{
  rclcpp::init(argc,argv);
  auto node = rclcpp::Node::make_shared("rate_demo");
  // rclcpp::Rate rate(1000ms); // 创建 Rate 对象方式1
  rclcpp::Rate rate(1.0); // 创建 Rate 对象方式2
  while (rclcpp::ok())
  {
    RCLCPP_INFO(node->get_logger(),"hello rate");
    // 休眠
    rate.sleep();
  }

  rclcpp::shutdown();
  return 0;
}
```

#### 2.rclpy 中的 Rate

rclpy 中的 Rate 对象可以通过节点创建，Rate 对象的 sleep\(\) 函数需要在子线程中执行，否咋会阻塞程序。

**示例：**周期性输出一段文本。

```py
import rclpy
import threading
from rclpy.timer import Rate

rate = None
node = None

def do_some():
    global rate
    global node
    while rclpy.ok():
        node.get_logger().info("hello ---------")
        # 休眠
        rate.sleep()

def main():
    global rate
    global node
    rclpy.init()    
    node = rclpy.create_node("rate_demo")
    # 创建 Rate 对象
    rate = node.create_rate(1.0)
    
    # 创建子线程
    thread = threading.Thread(target=do_some)
    thread.start()

    rclpy.shutdown()

if __name__ == "__main__":
    main()
```



