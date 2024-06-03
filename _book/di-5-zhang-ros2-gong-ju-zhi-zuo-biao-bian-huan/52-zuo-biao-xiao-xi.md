## 5.2 坐标相关消息

坐标变换的实现其本质是基于话题通信的发布订阅模型的，发布方可以发布坐标系之间的相对关系，订阅方则可以监听这些消息，并实现不同坐标系之间的变换。显然的根据之前介绍，在话题通信中，接口消息作为数据载体在整个通信模型中是比较重要的一部分，本节将会介绍坐标变换中常用的两种接口消息：`geometry_msgs/msg/TransformStamped`和`geometry_msgs/msg/PointStamped`。

前者用于描述某一时刻两个坐标系之间相对关系的接口，后者用于描述某一时刻坐标系内某个坐标点的位置的接口。在坐标变换中，会经常性的使用到坐标系相对关系以及坐标点信息。

#### 1.geometry\_msgs/msg/TransformStamped {#1geometrymsgstransformstamped}

通过如下命令查看接口定义：

```
ros2 interface show geometry_msgs/msg/TransformStamped
```

接口定义解释：

```
std_msgs/Header header
    builtin_interfaces/Time stamp     # 时间戳
        int32 sec
        uint32 nanosec
    string frame_id                   # 父级坐标系

string child_frame_id                 # 子级坐标系

Transform transform                   # 子级坐标系相对于父级坐标系的位姿
    Vector3 translation               # 三维偏移量
        float64 x
        float64 y
        float64 z
    Quaternion rotation               # 四元数
        float64 x 0
        float64 y 0
        float64 z 0
        float64 w 1
```

四元数类似于欧拉角用于表示坐标系的相对姿态。

#### 2.geometry\_msgs/msg/PointStamped {#2geometrymsgspointstamped}

通过如下命令查看接口定义：

```
ros2 interface show geometry_msgs/msg/PointStamped
```

接口定义解释：

```
std_msgs/Header header
    builtin_interfaces/Time stamp    # 时间戳
        int32 sec
        uint32 nanosec
    string frame_id                  # 参考系
Point point                          # 三维坐标
    float64 x
    float64 y
    float64 z
```



