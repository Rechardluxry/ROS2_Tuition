### 3.5.3 编码设置话题名称

#### 话题分类

话题的名称的设置是与节点的命名空间、节点的名称有一定关系的，话题名称大致可以分为三种类型:

* 全局话题\(话题参考ROS系统，与节点命名空间平级\)；
* 相对话题\(话题参考的是节点的命名空间，与节点名称平级\)；
* 私有话题\(话题参考节点名称，是节点名称的子级\)。

总之，以编码方式设置话题名称是比较灵活的。本节将介绍如何在 rclcpp 和 rclpy 中分别设置不同类型的话题。

#### 准备

请先分别创建 C++ 与 Python 相关的功能包以及节点，且假定在创建节点时，使用的命名空间为 xxx，节点名称为 yyy。

#### 话题设置

##### 1.1全局话题

**格式：**定义时以`/`开头的名称，和命名空间、节点名称无关。

**rclcpp示例：**`publisher_ = this->create_publisher<std_msgs::msg::String>("/topic/chatter", 10);`

**rclpy示例：**`self.publisher_ = self.create_publisher(String, '/topic/chatter', 10)`

**话题：**话题名称为 /topic/chatter，与命名空间 xxx 以及节点名称 yyy 无关。

##### 1.2相对话题

**格式：**非`/`开头的名称，参考命名空间设置话题名称，和节点名称无关。

**rclcpp示例：**`publisher_ = this->create_publisher<std_msgs::msg::String>("topic/chatter", 10);`

**rclpy示例：**`self.publisher_ = self.create_publisher(String, 'topic/chatter', 10)`

**话题：**话题名称为 /xxx/topic/chatter，与命名空间 xxx 有关，与节点名称 yyy 无关。

##### 1.3私有话题

**格式：**定义时以`~/`开头的名称，和命名空间、节点名称都有关系。

**rclcpp示例：**`publisher_ = this->create_publisher<std_msgs::msg::String>("~/topic/chatter", 10);`

**rclpy示例：**`self.publisher_ = self.create_publisher(String, '~/topic/chatter', 10)`

**话题：**话题名称为 /xxx/yyy/topic/chatter，使用命名空间 xxx 以及节点名称 yyy 作为话题名称前缀。

综上，话题名称设置规则在rclcpp与rclpy中基本一致，且上述规则也同样适用于ros2 run指令与launch文件。

