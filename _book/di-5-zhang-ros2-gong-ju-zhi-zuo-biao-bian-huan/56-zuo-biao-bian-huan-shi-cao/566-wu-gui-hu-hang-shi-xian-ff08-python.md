### 5.6.6 乌龟护航实现（Python）

#### 1.编写 launch 文件

功能包 py05\_exercise 的 launch 目录下新建文件exer02\_turte\_escort.launch.xml，并编辑文件，输入如下内容：

```xml
<launch>
    <!-- 乌龟准备 -->
    <node pkg="turtlesim" exec="turtlesim_node" name="t1" />
    <node pkg="py05_exercise" exec="exer01_tf_spawn_py" name="t2" />
    <node pkg="py05_exercise" exec="exer01_tf_spawn_py" name="t3" >
        <param name="turtle_name" value="turtle3"/>
        <param name="x" value="2.0"/>
        <param name="y" value="8.0"/>
    </node>
    <node pkg="py05_exercise" exec="exer01_tf_spawn_py" name="t4" >
        <param name="turtle_name" value="turtle4"/>
        <param name="x" value="8.0"/>
        <param name="y" value="2.0"/>
    </node>
    <!-- 发布坐标变换 -->
    <node pkg="py05_exercise" exec="exer02_tf_broadcaster_py" name="tf_broadcaster1_py">
    </node>
    <node pkg="py05_exercise" exec="exer02_tf_broadcaster_py" name="tf_broadcaster2_py">
        <param name="turtle_name" value="turtle2" />
    </node>
    <node pkg="py05_exercise" exec="exer02_tf_broadcaster_py" name="tf_broadcaster3_py">
        <param name="turtle_name" value="turtle3" />
    </node>
    <node pkg="py05_exercise" exec="exer02_tf_broadcaster_py" name="tf_broadcaster4_py">
        <param name="turtle_name" value="turtle4" />
    </node>
    <!-- 发布乌龟目标坐标 -->
    <node pkg="tf2_ros" exec="static_transform_publisher" name="goal_tf1" args="--x -1 --frame-id turtle1 --child-frame-id goal1"/>
    <node pkg="tf2_ros" exec="static_transform_publisher" name="goal_tf2" args="--y 1 --frame-id turtle1 --child-frame-id goal2"/>
    <node pkg="tf2_ros" exec="static_transform_publisher" name="goal_tf3" args="--y -1 --frame-id turtle1 --child-frame-id goal3"/>
    <!-- 监听坐标变换 -->
    <node pkg="py05_exercise" exec="exer03_tf_listener_py" name="tf_listener_py1">
        <param name="target_frame" value="turtle2" />
        <param name="source_frame" value="goal1" />
    </node>
    <node pkg="py05_exercise" exec="exer03_tf_listener_py" name="tf_listener_py2">
        <param name="target_frame" value="turtle3" />
        <param name="source_frame" value="goal2" />
    </node>
    <node pkg="py05_exercise" exec="exer03_tf_listener_py" name="tf_listener_py3">
        <param name="target_frame" value="turtle4" />
        <param name="source_frame" value="goal3" />
    </node>
</launch>
```

#### 2.编译

终端中进入当前工作空间，编译功能包：

```
colcon build --packages-select py05_exercise
```

#### 3.执行

当前工作空间下启动终端，输入如下命令运行launch文件：

```
. install/setup.bash 
ros2 launch py05_exercise exer02_turtle_escort.launch.xml
```

此时窗口中会新生成三只乌龟，向原生乌龟 turtle1 运动，并会组成固定队形。

再新建终端，启动 turtlesim 键盘控制节点：

```
ros2 run turtlesim turtle_teleop_key
```

该终端下可以通过键盘控制 turtle1 运动，并且护航的乌龟会按照特定队形跟随 turtle1 运动。最终的运行结果与演示案例类似。

