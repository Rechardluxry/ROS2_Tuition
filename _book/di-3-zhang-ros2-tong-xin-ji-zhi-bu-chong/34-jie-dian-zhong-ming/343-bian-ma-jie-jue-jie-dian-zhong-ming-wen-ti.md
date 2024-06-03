### 3.4.3 编码设置节点名称

在 rclcpp 和 rclpy 中，节点类的构造函数中，都分别提供了设置节点名称与命名空间的参数。

#### 1.rclcpp中的相关API

rclcpp中节点类的构造函数如下：

```
Node (const std::string &node_name, const NodeOptions &options=NodeOptions())
Node (const std::string &node_name, const std::string &namespace_, const NodeOptions &options=NodeOptions())
```

构造函数1中可以直接通过node\_name设置节点名称，构造函数2既可以通过node\_name设置节点名称也可以通过namespace\_设置命名空间。

#### 2.rclpy中的相关API

rclpy中节点类的构造函数如下：

```
Node(node_name, *,
   context=None,
   cli_args=None,
   namespace=None, 
   use_global_arguments=True, 
   enable_rosout=True, 
   start_parameter_services=True, 
   parameter_overrides=None, 
   allow_undeclared_parameters=False, 
   automatically_declare_parameters_from_overrides=False)
```

构造函数中可以使用node\_name设置节点名称，namespace设置命名空间。

