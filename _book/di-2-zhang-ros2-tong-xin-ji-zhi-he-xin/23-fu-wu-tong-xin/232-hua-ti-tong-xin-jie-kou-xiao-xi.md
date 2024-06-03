### 2.3.2 服务通信接口消息

定义服务接口消息与定义话题接口消息流程类似，主要步骤如下：

1. 创建并编辑 `.srv`文件；
2. 编辑配置文件；
3. 编译；
4. 测试。

接下来，我们可以参考案例编写一个srv文件，该文件中包含请求数据\(两个整型字段\)与响应数据\(一个整型字段\)。

#### 1.创建并编辑 .srv 文件

功能包base\_interfaces\_demo下新建srv文件夹，srv文件夹下新建AddInts.srv文件，文件中输入如下内容：

```
int32 num1
int32 num2
---
int32 sum
```

#### 2.编辑配置文件

##### 1.package.xml 文件

srv文件与msg文件的包依赖一致，如果你是新建的功能包添加srv文件，那么直接参考定义msg文件时package.xml 配置即可。由于我们使用的是base\_interfaces\_demo该包已经为msg文件配置过了依赖包，所以package.xml不需要做修改。

##### 2.CMakeLists.txt 文件

如果是新建的功能包，与之前定义msg文件同理，为了将`.srv`文件转换成对应的C++和Python代码，还需要在CMakeLists.txt中添加如下配置：

```
find_package(rosidl_default_generators REQUIRED)

rosidl_generate_interfaces(${PROJECT_NAME}
  "srv/AddInts.srv"
)
```

不过，我们当前使用的base\_interfaces\_demo包，那么你只需要修改rosidl\_generate\_interfaces函数即可，修改后的内容如下：

```
rosidl_generate_interfaces(${PROJECT_NAME}
  "msg/Student.msg"
  "srv/AddInts.srv"
)
```

#### 3.编译

终端中进入当前工作空间，编译功能包：

```
colcon build --packages-select base_interfaces_demo
```

#### 4.测试

编译完成之后，在工作空间下的 install 目录下将生成`AddInts.srv`文件对应的C++和Python文件，我们也可以在终端下进入工作空间，通过如下命令查看文件定义以及编译是否正常：

```
. install/setup.bash
ros2 interface show base_interfaces_demo/srv/AddInts
```

正常情况下，终端将会输出与`AddInts.srv`文件一致的内容。

