## 3.2 工作空间覆盖

#### 场景

同一工作空间下不允许出现功能包重名的情况，但是当存在多个工作空间时，不同工作空间下的功能包是可以重名的，那么当功能包重名时，会调用哪一个呢？

> 比如：自定义工作空间A存在功能包turtlesim，自定义工作空间B也存在功能包turtlesim，当然系统自带工作空间也存在turtlesim，如果调用turtlesim包，会调用哪个工作空间中的呢？

#### 概念

所谓工作空间覆盖，是指不同工作空间存在重名功能包时，重名功能包的调用会产生覆盖的情况。

#### 作用

没什么用，这种情况是需要极力避免出现的。

#### 演示

1.分别在不同的工作空间下创建turtlesim功能包。

终端下进入ws00\_helloworld的src目录，新建功能包：

```
ros2 pkg create turtlesim --node-name turtlesim_node
```

为了方便查看演示结果，将默认生成的 turtlesim\_node.cpp 中的打印内容修改为：`ws00_helloworld turtlesim\n`

终端下进入ws01\_plumbing的src目录，新建功能包：

```
ros2 pkg create turtlesim --node-name turtlesim_node
```

为了方便查看演示结果，将默认生成的 turtlesim\_node.cpp 中的打印内容修改为：`ws01_plumbing turtlesim\n`

2.在 ~/.bashrc 文件下追加如下内容：

```
source /home/ros2/ws00_helloworld/install/setup.bash
source /home/ros2/ws01_plumbing/install/setup.bash
```

修改完毕后，保存并关闭文件。

3.新建终端，输入如下指令：

```
ros2 run turtlesim turtlesim_node
```

输出结果为：`ws01_plumbing turtlesim`，也即执行的是 ws01\_plumbing 功能包下的 turtlesim，而 ws00\_helloworld 下的 turtlesim 与内置的 turtlesim 被覆盖了。

#### **原因** {#原因}

这与~/.bashrc中不同工作空间的setup.bash文件的加载顺序有关：

1.ROS2 会解析 ~/.bashrc 文件，并生成全局环境变量 AMENT\_PREFIX\_PATH 与 PYTHONPATH，两个环境变量取值分别对应了 ROS2 中 C++ 和 Python 功能包，环境变量的值由功能包名称组成；

![](/assets/3.2空间覆盖.PNG)

2.两个变量的值的设置与 ~/.bashrc 中的 setup.bash 的配置顺序有关，对于自定义的工作空间而言，后配置的优先级更高，主要表现在后配置的工作空间的功能包在环境变量值组成的前部，而前配置工作空间的功能包在环境变量值组成的后部分，如果更改两个自定义工作空间在 ~/.bashrc 中的配置顺序，那么变量值也将相应更改，但是 ROS2 系统工作空间的配置始终处于最后。

3.调用功能包时，会按照 AMENT\_PREFIX\_PATH 或 PYTHONPATH 中包配置顺序从前往后依次查找相关功能包，查找到功能包时会停止搜索，也即配置在前的会优先执行。

#### **隐患** {#隐患}

前面提到，工作空间覆盖的情况是需要极力避免出现的，因为导致一些安全隐患：

1. 可能会出现功能包调用混乱，出现实际调用与预期调用结果不符的情况；
2. 即便可以通过 ~/.bashrc 来配置不同工作空间的优先级，但是经过测试，修改 ~/.bashrc 文件之后不一定马上生效，还需要删除工作空间下build与install目录重新编译，才能生效，这个过程繁琐且有不确定性。

综上，在实际工作中，需要制定明确的包命名规范，避免包重名情况。

