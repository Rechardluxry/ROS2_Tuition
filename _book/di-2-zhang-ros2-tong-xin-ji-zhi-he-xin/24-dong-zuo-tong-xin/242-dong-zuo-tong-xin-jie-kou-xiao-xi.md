### 2.4.2 动作通信接口消息

定义动作接口消息与定义话题或服务接口消息流程类似，主要步骤如下：

1. 创建并编辑`.action`文件；
2. 编辑配置文件；
3. 编译；
4. 测试。

接下来，我们可以参考案例编写一个action文件，该文件中包含请求数据\(一个整型字段\)、响应数据\(一个整型字段\)和连续反馈数据\(一个浮点型字段\)。

#### 1.创建并编辑 .action 文件

功能包base\_interfaces\_demo下新建action文件夹，action文件夹下新建Progress.action文件，文件中输入如下内容：

```
int64 num
---
int64 sum
---
float64 progress
```

#### 2.编辑配置文件

##### 1.package.xml

如果单独构建action功能包，需要在package.xml中需要添加一些依赖包，具体内容如下：

```
<buildtool_depend>rosidl_default_generators</buildtool_depend>
<depend>action_msgs</depend>
<member_of_group>rosidl_interface_packages</member_of_group>
```

当前使用的是 base\_interfaces\_demo 功能包，已经为 msg 、srv 文件添加过了一些依赖，所以 package.xml 中添加如下内容即可：

```
<buildtool_depend>rosidl_default_generators</buildtool_depend>
<depend>action_msgs</depend>
```

##### 2.CMakeLists.txt

如果是新建的功能包，与之前定义msg、srv文件同理，为了将`.action`文件转换成对应的C++和Python代码，还需要在CMakeLists.txt 中添加如下配置：

```
find_package(rosidl_default_generators REQUIRED)

rosidl_generate_interfaces(${PROJECT_NAME}
  "action/Progress.action"
)
```

不过，我们当前使用的base\_interfaces\_demo包，那么只需要修改rosidl\_generate\_interfaces函数即可，修改后的内容如下：

```
rosidl_generate_interfaces(${PROJECT_NAME}
  "msg/Student.msg"
  "srv/AddInts.srv"
  "action/Progress.action"
)
```

#### 3.编译

终端中进入当前工作空间，编译功能包：

```
colcon build --packages-select base_interfaces_demo
```

#### 4.测试

编译完成之后，在工作空间下的 install 目录下将生成`Progress.action`文件对应的C++和Python文件，我们也可以在终端下进入工作空间，通过如下命令查看文件定义以及编译是否正常：

```
. install/setup.bash
ros2 interface show base_interfaces_demo/action/Progress
```

正常情况下，终端将会输出与`Progress.action`文件一致的内容。

