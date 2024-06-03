### 3.5.1 ros2 run 修改话题名称

#### 1.ros2 run设置命名空间 {#1rosrun设置命名空间}

该实现与**3.4.1 ros2 run设置节点名称**中演示的语法使用一致。

##### 1.1设置命名空间演示 {#11设置命名空间演示}

**语法：**ros2 run 包名 节点名 --ros-args --remap \_\_ns:=命名空间

**示例：**

```
ros2 run turtlesim turtlesim_node --ros-args --remap __ns:=/t1
```

##### 1.2运行结果 {#12运行结果}

使用`ros2 topic list`查看节点信息，显示结果：

```
/t1/turtle1/cmd_vel
/t1/turtle1/color_sensor
/t1/turtle1/pose
```

节点下的话题已经添加了命名空间前缀。

#### 2.ros2 run话题名称重映射 {#2rosrun名称重映射}

##### 2.1为话题起别名 {#21为节点起别名}

**语法：** ros2 run 包名 节点名 --ros-args --remap 原话题名称:=新话题名称

**示例：**

```
ros2 run turtlesim turtlesim_node --ros-args --remap /turtle1/cmd_vel:=/cmd_vel
```

##### 2.2运行结果 {#22运行结果}

使用`ros2 topic list`查看节点信息，显示结果：

```
/cmd_vel
/turtle1/color_sensor
/turtle1/pose
```

节点下的话题/turtle1/cmd\_vel已经被修改为了/cmd\_vel。

**注意：**

当为节点添加命名空间时，节点下的所有非全局话题都会前缀命名空间，而重映射的方式只是修改指定话题。

