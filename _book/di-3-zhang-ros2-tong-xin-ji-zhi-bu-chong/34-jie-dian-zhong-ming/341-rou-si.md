### 3.4.1 ros2 run设置节点名称

#### 1.ros2 run设置命名空间 {#1rosrun设置命名空间}

##### 1.1设置命名空间演示 {#11设置命名空间演示}

**语法：**ros2 run 包名 节点名 --ros-args --remap \_\_ns:=命名空间

**示例：**

```
ros2 run turtlesim turtlesim_node --ros-args --remap __ns:=/t1
```

##### 1.2运行结果 {#12运行结果}

使用`ros2 node list`查看节点信息，显示结果：

```
/t1/turtlesim
```

#### 2.ros2 run名称重映射 {#2rosrun名称重映射}

##### 2.1为节点起别名 {#21为节点起别名}

**语法：** ros2 run 包名 节点名 --ros-args --remap \_\_name:=新名称

或

ros2 run 包名 节点名 --ros-args --remap \_\_node:=新名称

**示例：**

```
ros2 run turtlesim turtlesim_node --ros-args --remap __name:=turtle1
```

##### 2.2运行结果 {#22运行结果}

使用`ros2 node list`查看节点信息，显示结果：

```
/turtle1
```

#### 3.ros2 run命名空间与名称重映射叠加 {#3rosrun命名空间与名称重映射叠加}

##### 3.1设置命名空间同时名称重映射 {#31设置命名空间同时名称重映射}

语法: ros2 run 包名 节点名 --ros-args --remap \_\_ns:=新名称 --remap \_\_name:=新名称

```
ros2 run turtlesim turtlesim_node --ros-args --remap __ns:=/t1 --remap __name:=turtle1
```

##### 3.2运行结果 {#32运行结果}

使用`ros2 node list`查看节点信息，显示结果：

```
/t1/turtle1
```



