### 2.5.2 参数数据类型

在ROS2中，参数由键、值和描述符三部分组成，其中键是字符串类型，值可以是bool、int64、float64、string、byte\[\]、bool\[\]、int64\[\]、float64\[\]、string\[\]中的任一类型，描述符默认情况下为空，但是可以设置参数描述、参数数据类型、取值范围或其他约束等信息。

为了方便操作，参数被封装为了相关类，其中C++客户端对应的类是`rclcpp::Parameter`，Python客户端对应的类是`rclpy.Parameter`。借助于相关API，我们可以实现参数对象创建以及参数属性解析等操作。以下代码提供了参数相关API基本使用的示例。

**C++示例：**

```cpp
...
// 创建参数对象
rclcpp::Parameter p1("car_name","Tiger"); //参数值为字符串类型
rclcpp::Parameter p2("width",0.15); //参数值为浮点类型
rclcpp::Parameter p3("wheels",2); //参数值为整型

// 获取参数值并转换成相应的数据类型
RCLCPP_INFO(rclcpp::get_logger("rclcpp"),"car_name = %s", p1.as_string().c_str());
RCLCPP_INFO(rclcpp::get_logger("rclcpp"),"width = %.2f", p2.as_double());
RCLCPP_INFO(rclcpp::get_logger("rclcpp"),"wheels = %ld", p3.as_int());

// 获取参数的键
RCLCPP_INFO(rclcpp::get_logger("rclcpp"),"p1 name = %s", p1.get_name().c_str());
// 获取参数数据类型
RCLCPP_INFO(rclcpp::get_logger("rclcpp"),"p1 type_name = %s", p1.get_type_name().c_str());
// 将参数值转换成字符串类型
RCLCPP_INFO(rclcpp::get_logger("rclcpp"),"p1 value_to_msg = %s", p1.value_to_string().c_str());
...
```

**Python示例：**

```py
# 创建参数对象
p1 = rclpy.Parameter("car_name",value="Horse")
p2 = rclpy.Parameter("length",value=0.5)
p3 = rclpy.Parameter("wheels",value=4)

# 获取参数值
get_logger("rclpy").info("car_name = %s" % p1.value)
get_logger("rclpy").info("length = %.2f" % p2.value)
get_logger("rclpy").info("wheels = %d" % p3.value)

# 获取参数键
get_logger("rclpy").info("p1 name = %s" % p1.name)
```

关于参数具体的API使用，在后续案例中会有介绍。

