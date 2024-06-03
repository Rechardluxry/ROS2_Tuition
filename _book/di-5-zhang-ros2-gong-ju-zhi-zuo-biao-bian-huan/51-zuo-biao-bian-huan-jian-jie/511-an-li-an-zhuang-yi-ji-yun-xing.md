### 5.1.1 案例安装以及运行

关于坐标变换的实现有一个经典的“乌龟跟随”案例，在学习坐标变换的具体知识点之前，建议先安装并运行此案例。

#### 1.安装

首先安装“乌龟跟随”案例的功能包以及依赖项。

安装方式1（二进制方式安装）：

```
sudo apt-get install ros-humble-turtle-tf2-py ros-humble-tf2-tools ros-humble-tf-transformations
```

安装方式2（克隆源码并构建）：

```
git clone https://github.com/ros/geometry_tutorials.git -b ros2
```

此外，还需要安装一个名为 `transforms3d` 的 Python 包，它为 `tf_transformations`包提供四元数和欧拉角变换功能，安装命令如下：

```
sudo apt intall python3-pip
pip3 install transforms3d
```

#### 2.执行

启动两个终端，终端1输入如下命令：

```
ros2 launch turtle_tf2_py turtle_tf2_demo.launch.py
```

该命令会启动 turtlesim\_node 节点，turtlesim\_node 节点中自带一只小乌龟 turtle1，除此之外还会新生成一只乌龟 turtle2，turtle2 会运行至 turtle1 的位置。

终端2输入如下命令：

```
ros2 run turtlesim turtle_teleop_key
```

该终端下可以通过键盘控制 turtle1 运动，并且 turtle2 会跟随 turtle1 运动（参考引言部分的**案例1**）。

