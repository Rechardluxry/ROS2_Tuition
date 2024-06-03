### 3.7.1 命令工具

ROS2中常用的命令如下：

* **ros2 node**：节点相关命令行工具
* **ros2 interface**：接口\(msg、srv、action\)消息相关的命令行工具
* **ros2 topic**：话题通信相关的命令行工具
* **ros2 service**：服务通信相关的命令行工具
* **ros2 action**：动作通信相关的命令行工具
* **ros2 param**：参数服务相关的命令行工具

关于命令的使用一般都会提供帮助文档，帮助文档的获取方式如下：

* 可以通过`命令 -h` 或 `命令 --help`的方式查看命令帮助文档，比如：`ros2 node -h`或 `ros2 node --help`。

* 命令下参数的使用也可以通过`命令 参数 -h` 或 `命令 参数 --help`的方式查看帮助文档，比如：`ros2 node list -h`或 `ros2 node list --help`。

#### 1.ros2 node

ros2 node的基本使用语法如下：

```
info  输出节点信息
list  输出运行中的节点的列表
```

#### 2.ros2 interace

ros2 interace的基本使用语法如下：

```
list      输出所有可用的接口消息
package   输出指定功能包下的
packages  输出包含接口消息的功能包
proto     输出接口消息原型
show      输出接口消息定义格式
```

#### 3.ros2 topic

ros2 topic的基本使用语法如下：

```
bw       输出话题消息传输占用的带宽
delay    输出带有 header 的话题延迟
echo     输出某个话题下的消息
find     根据类型查找话题
hz       输出消息发布频率
info     输出话题相关信息
list     输出运行中的话题列表
pub      向指定话题发布消息
type     输出话题使用的接口类型
```

#### 4.ros2 service

ros2 service的基本使用语法如下：

```
call  向某个服务发送请求
find  根据类型查找服务
list  输出运行中的服务列表
type  输出服务使用的接口类型
```

#### 5.ros2 action

ros2 action的基本使用语法如下：

```
info       输出指定动作的相关信息
list       输出运行中的动作的列表
send_goal  向指定动作发送请求
```

#### 6.ros2 param

ros2 param的基本使用语法如下：

```
delete    删除参数
describe  输出参数的描述信息
dump      将节点参数写入到磁盘文件
get       获取某个参数
list      输出可用的参数的列表
load      从磁盘文件加载参数到节点
set       设置参数
```



