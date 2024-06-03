### 2.2.4 话题通信自定义接口消息

自定义接口消息的流程与在功能包中编写可执行程序的流程类似，主要步骤如下：

1. 创建并编辑 `.msg`文件；
2. 编辑配置文件；
3. 编译；
4. 测试。

接下来，我们可以参考案例2编译一个msg文件，该文件中包含学生的姓名、年龄、身高等字段。

#### 1.创建并编辑 .msg 文件

功能包base\_interfaces\_demo下新建 msg 文件夹，msg文件夹下新建Student.msg文件，文件中输入如下内容：

```
string   name
int32    age
float64  height
```

#### 2.编辑配置文件

##### 1.package.xml文件

在package.xml中需要添加一些依赖包，具体内容如下：

```
<build_depend>rosidl_default_generators</build_depend>
<exec_depend>rosidl_default_runtime</exec_depend>
<member_of_group>rosidl_interface_packages</member_of_group>
```

##### 2.CMakeLists.txt文件

为了将`.msg`文件转换成对应的C++和Python代码，还需要在CMakeLists.txt中添加如下配置：

```
find_package(rosidl_default_generators REQUIRED)

rosidl_generate_interfaces(${PROJECT_NAME}
  "msg/Student.msg"
)
```

#### 3.编译

终端中进入当前工作空间，编译功能包：

```
colcon build --packages-select base_interfaces_demo
```

#### 4.测试

编译完成之后，在工作空间下的install目录下将生成`Student.msg`文件对应的C++和Python文件，我们也可以在终端下进入工作空间，通过如下命令查看文件定义以及编译是否正常：

```
. install/setup.bash
ros2 interface show base_interfaces_demo/msg/Student
```

正常情况下，终端将会输出与`Student.msg`文件一致的内容。

---



