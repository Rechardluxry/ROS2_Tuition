### 3.6.4 Time 与 Duration 运算

#### 1.rclcpp 中的运算

**示例：**Time 以及 Duration 的相关运算。

```cpp
#include "rclcpp/rclcpp.hpp"

int main(int argc, char const *argv[])
{
    rclcpp::init(argc,argv);
    auto node = rclcpp::Node::make_shared("time_opt_demo");

    rclcpp::Time t1(1,500000000);
    rclcpp::Time t2(10,0);

    rclcpp::Duration du1(3,0);
    rclcpp::Duration du2(5,0);

    // 比较
    RCLCPP_INFO(node->get_logger(),"t1 >= t2 ? %d",t1 >= t2);
    RCLCPP_INFO(node->get_logger(),"t1 < t2 ? %d",t1 < t2);
    // 数学运算
    rclcpp::Time t3 = t2 + du1;
    rclcpp::Time t4 = t1 - du1;
    rclcpp::Duration du3 = t2 - t1;

    RCLCPP_INFO(node->get_logger(), "t3 = %.2f",t3.seconds());  
    RCLCPP_INFO(node->get_logger(), "t4 = %.2f",t4.seconds()); 
    RCLCPP_INFO(node->get_logger(), "du3 = %.2f",du3.seconds()); 

    RCLCPP_INFO(node->get_logger(),"--------------------------------------");
    // 比较
    RCLCPP_INFO(node->get_logger(),"du1 >= du2 ? %d", du1 >= du2);
    RCLCPP_INFO(node->get_logger(),"du1 < du2 ? %d", du1 < du2);
    // 数学运算
    rclcpp::Duration du4 = du1 * 3.0;
    rclcpp::Duration du5 = du1 + du2;
    rclcpp::Duration du6 = du1 - du2;

    RCLCPP_INFO(node->get_logger(), "du4 = %.2f",du4.seconds()); 
    RCLCPP_INFO(node->get_logger(), "du5 = %.2f",du5.seconds()); 
    RCLCPP_INFO(node->get_logger(), "du6 = %.2f",du6.seconds()); 

    rclcpp::shutdown();
    return 0;
}
```

#### 2.rclpy 中的运算

**示例：**Time 以及 Duration 的相关运算。

```py
import rclpy
from rclpy.time import Time
from rclpy.duration import Duration

def main():
    rclpy.init()
    node = rclpy.create_node("time_opt_node")
    t1 = Time(seconds=10)
    t2 = Time(seconds=4)

    du1 = Duration(seconds=3)
    du2 = Duration(seconds=5)

    # 比较
    node.get_logger().info("t1 >= t2 ? %d" % (t1 >= t2))
    node.get_logger().info("t1 < t2 ? %d" % (t1 < t2))
    # 数学运算
    t3 = t1 + du1
    t4 = t1 - t2    
    t5 = t1 - du1

    node.get_logger().info("t3 = %d" % t3.nanoseconds)
    node.get_logger().info("t4 = %d" % t4.nanoseconds)
    node.get_logger().info("t5 = %d" % t5.nanoseconds)

    # 比较
    node.get_logger().info("-" * 80)
    node.get_logger().info("du1 >= du2 ? %d" % (du1 >= du2))
    node.get_logger().info("du1 < du2 ? %d" % (du1 < du2))

    rclpy.shutdown()

if __name__ == "__main__":
    main()
```



