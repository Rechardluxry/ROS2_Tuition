### 1.2.1 安装ROS2

整体而言，ROS2的安装步骤不算复杂，大致步骤如下：

1. 准备1：设置语言环境；
2. 准备2：启动Ubuntu universe存储库；
3. 设置软件源；
4. 安装ROS2；
5. 配置环境。

请注意：虽然安装比较简单，但是安装过程比较耗时，需要耐心等待。

#### 1.准备1：设置语言环境

请先检查本地语言环境是否支持UTF-8编码，可调用如下指令检查并设置UTF-8编码：

```
locale  # 检查是否支持 UTF-8

sudo apt update && sudo apt install locales
sudo locale-gen en_US en_US.UTF-8
sudo update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8
export LANG=en_US.UTF-8

locale  # 验证设置是否成果
```

注意：语言环境可以不同，但必须支持UTF-8编码。

#### 2.准备2：启动Ubuntu universe存储库

常用的启动Ubuntu universe存储库方式有两种：图形化操作与命令行操作。

##### 方式1：图形化操作

请打开软件与更新\(Software & Updates\)窗口，确保启动了universe存储库，以保证可以下载”社区维护的免费和开源软件“，操作示例如下：

![](/assets/1.2.1启动universe存储库.png)

##### 方式2：命令行操作

首先，通过如下命令检查是否已经启动了Ubuntu universe存储库：

```
apt-cache policy | grep universe
 500 http://us.archive.ubuntu.com/ubuntu jammy/universe amd64 Packages
     release v=22.04,o=Ubuntu,a=jammy,n=jammy,l=Ubuntu,c=universe,b=amd64
```

如果没有如上所示的输出，那么请调用如下命令启动Ubuntu universe存储库：

```
sudo apt install software-properties-common
sudo add-apt-repository universe
```

#### 3.设置软件源

先将ROS 2 apt存储库添加到系统，用apt授权我们的GPG密钥：

```
sudo apt update && sudo apt install curl gnupg lsb-release
sudo curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key -o /usr/share/keyrings/ros-archive-keyring.gpg
```

> 提示：如果抛出连接 **raw.githubusercontent.com  **失败的异常信息，解决方式请参考本节最后部分补充2内容。

然后将存储库添加到源列表：

```
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] http://packages.ros.org/ros2/ubuntu $(source /etc/os-release && echo $UBUNTU_CODENAME) main" | sudo tee /etc/apt/sources.list.d/ros2.list > /dev/null
```

#### 4.安装ROS2

首先更新apt存储库缓存：

```
sudo apt update
```

然后升级已安装的软件\(ROS2软件包建立在经常更新的Ubuntu系统上，在安装新软件包之前请确保您的系统是最新的\)：

```
sudo apt upgrade
```

安装桌面版ROS2\(建议\)，包含：ROS、RViz、示例与教程，安装命令如下：

```
sudo apt install ros-humble-desktop
```

或者，也可以安装基础版ROS2，包含：通信库、消息包、命令行工具，但是没有 GUI 工具，安装命令如下：

```
sudo apt install ros-humble-ros-base
```

#### 5.配置环境

终端下，执行ROS2程序时，需要调用如下命令配置环境：

```
source /opt/ros/humble/setup.bash
```

每次新开终端时，都得执行上述命令，或者也可以执行如下命令，将配置环境指令写入 ”~/.bashrc“ 文件，那么每次新启动终端时，不需要在手动配置环境：

```
echo "source /opt/ros/humble/setup.bash" >> ~/.bashrc
```

到目前为止，ROS2就已经安装且配置完毕了。

---

#### 补充1：关于卸载\(谨慎操作\)

ROS2安装完毕之后，如果想卸载ROS2，可以执行如下命令：

```
sudo apt remove ~nros-humble-* && sudo apt autoremove
```

还可以再删除ROS2对应的存储库：

```
sudo rm /etc/apt/sources.list.d/ros2.list
sudo apt update
sudo apt autoremove
# Consider upgrading for packages previously shadowed.
sudo apt upgrade
```

#### 补充2：关于 **raw.githubusercontent.com 连接失败的处理**

安装ROS2过程中，执行到步骤3也即设置软件源时，可能会抛出异常。

**异常提示：**curl: \(7\) Failed to connect to raw.githubusercontent.com port 443: 拒绝连接。

**异常原因：**DNS被污染。

**解决思路：**查询错误提示中域名的IP地址，然后修改 /etc/hosts 文件，添加域名与IP映射。

**具体实现：**

1.访问 [https://www.ipaddress.com/](https://www.ipaddress.com/) 并输入域名 **raw.githubusercontent.com**，查询 ip 地址。

![](/assets/1.2.1域名IP地址查询1.PNG)

![](/assets/1.2.1域名IP地址查询2.PNG)

查询到的ip地址可能有多个，记录任意一个地址即可。

2.修改/etc/hosts文件：

```
sudo gedit /etc/hosts
```

添加ip和域名映射到hosts文件，保存并退出。

![](/assets/1.2.1域名IP地址映射.PNG)

操作完毕后，终端再次运行安装指令即可正常执行

