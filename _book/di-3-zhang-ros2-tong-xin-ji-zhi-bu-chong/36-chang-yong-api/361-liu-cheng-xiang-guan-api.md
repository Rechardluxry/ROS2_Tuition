### 3.6.1 流程相关API

一个典型的 ROS2 程序由以下操作组成：

1. 初始化客户端，必须先于节点创建完成；
2. 创建一个或多个节点；
3. 回旋函数处理节点中回调函数；
4. 释放资源。

不管是 rclcpp 还是 rclpy 基本编写流程都是如上所述，本节将主要介绍这些步骤中实用的相关API。

#### 1.rclcpp

##### 1.1初始化

通过 rmw 实现初始化通信并设置全局信号处理程序。

```cpp
/**
 * 通过函数 rclcpp::contexts::get_global_default_context() 初始化全局 context（上下文）。
 * 通过函数 rclcpp::install_signal_handlers() 安装全局信号处理程序。
 *
 * \param[in] argc 要解析的命令行参数的个数。
 * \param[in] argv 要解析的命令行参数数组。
 * \param[in] init_options 要应用的初始化选项。
 * \param[in] signal_handler_options 用于声明应安装哪些信号处理程序。
 */
void init(
  int argc,
  char const * const * argv,
  const InitOptions & init_options = InitOptions(),
  SignalHandlerOptions signal_handler_options = SignalHandlerOptions::All);
```

##### 1.2节点创建

创建节点对象指针。

```cpp
rclcpp::Node::make_shared();
```

或者也可以自定义节点类，然后创建节点对象指针，节点构造函数 3.6.2 介绍。

##### 1.3回旋函数

回旋函数用于执行节点中的回调函数。

```cpp
/// 创建一个默认的单线程执行器并回旋指定的节点（循环执行）。 
/** \param[in] node_ptr 被回旋的节点的指针。 */
void rclcpp::spin(rclcpp::Node::SharedPtr node_ptr);
```

```
/// 创建一个默认的单线程执行器并回旋指定的节点（只执行依次）。
/** \param[in] node_ptr 被回旋的节点的指针。 */
void rclcpp::spin_some(rclcpp::Node::SharedPtr node_ptr);
```

```
/// 执行任务直至 future 完成，除非超时或 rclcpp 被中断。
/*
* \param[in] node_ptr 被回旋的节点的指针。
* \param[in] future 保存未来返回结果的对象。
* \param[in] timeout 超时时间。
*/
template<typename NodeT = rclcpp::Node, typename FutureT, typename TimeRepT = int64_t,
  typename TimeT = std::milli>
rclcpp::FutureReturnCode spin_until_future_complete(
  std::shared_ptr<NodeT> node_ptr,
  const FutureT & future,
  std::chrono::duration<TimeRepT, TimeT> timeout = std::chrono::duration<TimeRepT, TimeT>(-1));
```

函数 rclcpp::spin\_some\(\) 一般会结合循环使用，且循环条件是当前客户端是否运行正常，rclcpp 状态可以通过rclcpp::ok\(\)函数判断。

```
/**
 * 当程序其他部分调用了 rclcpp::shutdown() 或者由于rclcpp信号处理程序接收到了关闭信号，
 * 比如在终端中使用了 ctrl + c 快捷键。会返回false。
 * 
 * 如果 cpntext 取值 nullptr，则使用全局 context，即由 rclcpp::init() 初始化的 context。
 *
 * \param[in] context 被关闭的 context 对象指针。
 * \return context 被关闭时返回 true，否则返回 false。
 */
bool rclcpp::ok(rclcpp::Context::SharedPtr context = nullptr);
```

##### 1.4释放资源

关闭 context （上下文对象）。

```
/**
 * 如果为 context 传参 nullptr,那么就关闭通过 rclcpp::init() 创建的全局 context。
 * 如果使用全局 context，则信号处理程序也会被卸载。
 *
 * 执行该函数时，也将回调 on_shutdown() 函数。
 *
 * \param[in] context 被关闭的 context。
 * \param[in] reason 传递给关闭 context 函数的字符串。
 * \return 如果关闭成功，则为 true，如果 context 已关闭，则为 false。
 */
bool rclcpp::shutdown(
  rclcpp::Context::SharedPtr context = nullptr,
  const std::string & reason = "user called rclcpp::shutdown()");
```

#### 2.rclpy

##### 1.1初始化

初始化 rclpy 客户端。

    def init(
        *,
        args: Optional[List[str]] = None,
        context: Context = None,
        domain_id: Optional[int] = None,
        signal_handler_options: Optional[SignalHandlerOptions] = None,
    ) -> None:
        """
        为给定的 context 初始化 ROS2 通信。
        :param args: 命令行参数列表。
        :param context: 用于初始化的 context， 如果是 ``None``,那么通过函数 .get_default_context 获取默认 context。
        :param domain_id: domain id值。
        :param signal_handler_options: 声明要安装的信号处理程序。
        :return: None
        """

##### 1.2节点创建

创建节点对象。

    def create_node(
        node_name: str,
        *,
        context: Context = None,
        cli_args: List[str] = None,
        namespace: str = None,
        use_global_arguments: bool = True,
        enable_rosout: bool = True,
        start_parameter_services: bool = True,
        parameter_overrides: List[Parameter] = None,
        allow_undeclared_parameters: bool = False,
        automatically_declare_parameters_from_overrides: bool = False
    ) -> 'Node':
        """
        创建 Node 对象。

        :param node_name: 节点名称。
        :param context: 与节点关联的context, 如果是 ``None`` 那么使用默认的全局 context。
        :param cli_args: 节点使用的命令行参数。应用于 ROS2 节点，参数之前包含隐式的 `--ros-args`。
        :param namespace: 与节点关联的命名空间。
        :param use_global_arguments: 节点是否使用进程范围的命令行参数。
        :param enable_rosout: rosout 日志记录是否可用。
        :param start_parameter_services: 是否启用参数服务。
        :param parameter_overrides: 一个`Parameter`对象列表，用于覆盖在此节点上声明的参数的初始值。
        :param allow_undeclared_parameters: 是否允许未声明的参数，默认为 False（不允许）。
        :param automatically_declare_parameters_from_overrides: 如果为 True，则“参数覆盖”将用于在创建期间隐式声明节点上的参数，默认为 False。
        :return: 节点对象.
        """

或者也可以自定义节点类，然后创建节点对象，节点构造函数 3.6.2 介绍。

##### 1.3回旋函数

回旋函数用于执行节点中的回调函数。

```py
def spin(node: 'Node', executor: 'Executor' = None) -> None:
    """
    执行任务并阻塞，直到与执行器关联的上下文关闭。
    回调函数将由提供的执行器执行。

    :param node: 需要被回旋执行的节点。
    :param executor: 要使用的执行器，如果取值 ``None`` 那么使用全局执行器。
    :return: None
    """
```

    def spin_once(node: 'Node', *, executor: 'Executor' = None, timeout_sec: float = None) -> None:
        """
        只要回调在超时到期之前准备好，提供的执行程序就会执行一次回调。
        如果没有提供Executor（即。None），则使用全局执行器。如果全局执行程序有一个部分完成的协程，则完成的工作可能是针对提供的节点以外的节点。

        :param node: 需要被回旋执行的节点。
        :param executor: 要使用的执行器，如果取值 ``None`` 那么使用全局执行器。
        :param timeout_sec: 超时时间（以秒为单位），如果是``None``则无限期等待。如果为 0，则不要等待。
        :return: None
        """

```py
def spin_until_future_complete(
    node: 'Node',
    future: Future,
    executor: 'Executor' = None,
    timeout_sec: float = None
) -> None:
    """
    执行任务，直到 future 完成。

    回调和其他工作将由提供的执行程序执行，直到future.done() 返回True或与执行程序关联的 context 关闭。

    :param node: 添加到执行程序以执行任务的节点。
    :param future: 要等待的 future 对象。
    :param executor: 要使用的执行器，如果取值 ``None`` 那么使用全局执行器。
    :param timeout_sec: 超时时间（以秒为单位），如果是``None``则无限期等待。如果为 0，则不要等待。
    :return: None
    """
```

```
def ok(*, context=None):
    """
    当程序其他部分调用了 rclpy.shutdown() 或者由于rclpy信号处理程序接收到了关闭信号，
    比如在终端中使用了 ctrl + c 快捷键。会返回false。

    如果 cpntext 取值 None，则使用全局 context，即由 rclpy.init() 初始化的 context。

    :param node: 被判断的 context。
    :return: context 被关闭时返回 true，否则返回 false。
    """
```

##### 1.4释放资源

```py
def shutdown(*, context: Context = None, uninstall_handlers: Optional[bool] = None) -> None:
    """
    关闭之前初始化的 context。

    :param context: 与节点关联的context, 如果是 ``None``,那么通过函数 .get_default_context 获取默认 context。
    :param uninstall_handlers:
        如果是 `None`，默认关闭上下文时将卸载信号处理程序。
        如果是 `True`，信号处理程序将被卸载。
        如果是 `False`，信号处理程序不会被卸载。
    :return: None
    """
```



