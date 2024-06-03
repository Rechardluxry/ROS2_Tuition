### 1.2.3 安装colcon构建工具

colcon是一个命令行工具，用于改进编译，测试和使用多个软件包的工作流程。它实现过程自动化，处理需求并设置环境以便于使用软件包。ROS2中便是使用colcon作为包构建工具的，但是ROS2中没有默认安装colcon，需要自行安装，安装命令如下：

```
sudo apt install python3-colcon-common-extensions
```

安装完colcon之后，就可以在ROS2中编写应用程序了，下一节我们将介绍ROS2版本的HelloWorld实现。

