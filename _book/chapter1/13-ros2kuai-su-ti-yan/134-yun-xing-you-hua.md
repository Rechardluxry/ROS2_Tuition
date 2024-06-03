### 1.3.4 运行优化

每次终端中执行工作空间下的节点时，都需要调用`. install/setup.bash`指令，使用不便，优化策略是，可以将该指令的调用添加进`~/setup.bash`，操作格式如下：

```
echo "source /{工作空间路径}/install/setup.bash" >> ~/.bashrc
```

示例：

```
echo "source /home/ros2/ws00_helloworld/install/setup.bash" >> ~/.bashrc
```

以后再启动终端时，无需手动再手动刷新环境变量，使用更方便。

