### 3.6.3 Duration

#### 1.rclcpp 中的 Duration

**示例：**创建 Duration 对象，并调用其函数。

```cpp
#include "rclcpp/rclcpp.hpp"

using namespace std::chrono_literals;

int main(int argc, char const *argv[])
{
    rclcpp::init(argc,argv);
    auto node = rclcpp::Node::make_shared("duration_node");

    // 创建 Duration 对象
    rclcpp::Duration du1(1s);
    rclcpp::Duration du2(2,500000000);

    RCLCPP_INFO(node->get_logger(),"s = %.2f, ns = %ld", du2.seconds(),du2.nanoseconds());

    rclcpp::shutdown();
    return 0;
}
```

#### 2.rclpy 中的 Duration

**示例：**创建 Duration 对象，并调用其函数。

```py
import rclpy
from rclpy.duration import Duration

def main():
    rclpy.init()    

    node = rclpy.create_node("duration_demo")
    du1 = Duration(seconds = 2,nanoseconds = 500000000)
    node.get_logger().info("ns = %d" % du1.nanoseconds)

    rclpy.shutdown()

if __name__ == "__main__":

    main()
```



