### 1.4.1 安装VSCode

VSCode全称Visual Studio Code，是微软推出的一款轻量级代码编辑器，免费、开源而且功能强大。它支持几乎所有主流的程序语言的语法高亮、智能代码补全、自定义热键、括号匹配、代码片段、代码对比Diff、GIT 等特性，支持插件扩展，并针对网页开发和云端应用开发做了优化。软件跨平台支持Win、Mac以及Linux。

#### 1.下载 {#1下载}

vscode下载：[https://code.visualstudio.com/Download](https://code.visualstudio.com/Download)。

历史版本下载链接：[https://code.visualstudio.com/updates](https://code.visualstudio.com/updates)。

#### 2.vscode 安装与卸载 {#2vscode-安装与卸载}

##### 2.1 安装 {#21-安装}

**方式1：**双击安装即可\(或右击选择安装\)；

**方式2：**`sudo dpkg -i xxxx.deb`。

##### 2.2 卸载 {#22-卸载}

```
sudo dpkg --purge  code
```

#### 3.VSCode启动

VSCode启动也比较简单，可以直接在Show Applications\(显示应用程序\) 中搜索VSCode直接启动\(也可以将其添加到收藏夹\)。

或者，也可以在终端下进入需要被打开的目录\(比如：前面创建的ROS2工作空间ws00\_helloworld\)，然后输入命令：`code .`。

#### 4.VSCode插件

VSCode支持插件扩展，依赖于VSCode丰富多样的插件，可以大大提高程序开发效率，为了方便ROS2程序开发，我们也需要安装一些插件。

首先点击侧边栏的Extensions\(插件\)选项或者使用快捷键`Ctrl+Shift+X`打开插件窗口，本课程建议安装的插件如下：

![](/assets/1.4.1vscode插件.PNG)

当然，上述只是部分推荐插件，大家可以根据自身需求安装其他扩展。

#### 5.VSCode配置

在VSCode中，cpp文件中的`#include "rclcpp/rclcpp.hpp"`包含语句会抛出异常，这是因为没有设置VSCode配置文件中 includepath属性，可以按照如下步骤解决此问题：

![](/assets/1.4.1配置includepath.gif)

1. 将鼠标移到错误提示语句，此时会出现弹窗；
2. 点击弹窗中的快速修复，会有新的弹窗，再点击`编辑"includePath"设置`；
3. 在新页面中，包含路径属性对应的文本域中，换行输入被包含的路径`/opt/ros/humble/include/**`。

至此，问题修复。

VSCode安装并配置完毕后，大家就可以在其中编写ROS2程序了。当然为了提高编码效率，我们会经常性的使用到一些快捷键，VSCode的快捷键可以在菜单栏的“帮助”中查看。

