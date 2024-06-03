### 3.6.2 节点相关API

节点是 ROS2 程序中的基本单元，可以创建各种类型的通信对象。

#### 1.Node构造函数

创建具有指定名称的新节点。

```
/**
 * \param[in] node_name 节点名称。
 * \param[in] options 节点创建的附加选项。
 * \throws InvalidNamespaceError 命名空间无效异常。
 */
rclcpp::Node::Node(
  const std::string & node_name,
  const NodeOptions & options = NodeOptions());
/**
 * \param[in] node_name 节点名称。
 * \param[in] namespace_ 节点命名空间。
 * \param[in] options 节点创建的附加选项。
 * \throws InvalidNamespaceError 命名空间无效异常。
 */
rclcpp::Node::Node(
  const std::string & node_name,
  const std::string & namespace_,
  const NodeOptions & options = NodeOptions());
```

#### 2.常用函数

```
/// Get the name of the node.
/** \return The name of the node. */
RCLCPP_PUBLIC
const char *
get_name() const;

/// Get the namespace of the node.
/**
 * This namespace is the "node's" namespace, and therefore is not affected
 * by any sub-namespace's that may affect entities created with this instance.
 * Use get_effective_namespace() to get the full namespace used by entities.
 *
 * \sa get_sub_namespace()
 * \sa get_effective_namespace()
 * \return The namespace of the node.
 */
RCLCPP_PUBLIC
const char *
get_namespace() const;

/// Get the fully-qualified name of the node.
/**
 * The fully-qualified name includes the local namespace and name of the node.
 * \return fully-qualified name of the node.
 */
RCLCPP_PUBLIC
const char *
get_fully_qualified_name() const;

/// Get the logger of the node.
/** \return The logger of the node. */
RCLCPP_PUBLIC
rclcpp::Logger
get_logger() const;
```

#### 3.话题通信

    /// Create and return a Publisher.
    /**
     * The rclcpp::QoS has several convenient constructors, including a
     * conversion constructor for size_t, which mimics older API's that
     * allows just a string and size_t to create a publisher.
     *
     * For example, all of these cases will work:
     *
     * ```cpp
     * pub = node->create_publisher<MsgT>("chatter", 10);  // implicitly KeepLast
     * pub = node->create_publisher<MsgT>("chatter", QoS(10));  // implicitly KeepLast
     * pub = node->create_publisher<MsgT>("chatter", QoS(KeepLast(10)));
     * pub = node->create_publisher<MsgT>("chatter", QoS(KeepAll()));
     * pub = node->create_publisher<MsgT>("chatter", QoS(1).best_effort().durability_volatile());
     * {
     *   rclcpp::QoS custom_qos(KeepLast(10), rmw_qos_profile_sensor_data);
     *   pub = node->create_publisher<MsgT>("chatter", custom_qos);
     * }
     * ```
     *
     * The publisher options may optionally be passed as the third argument for
     * any of the above cases.
     *
     * \param[in] topic_name The topic for this publisher to publish on.
     * \param[in] qos The Quality of Service settings for the publisher.
     * \param[in] options Additional options for the created Publisher.
     * \return Shared pointer to the created publisher.
     */
    template<
      typename MessageT,
      typename AllocatorT = std::allocator<void>,
      typename PublisherT = rclcpp::Publisher<MessageT, AllocatorT>>
    std::shared_ptr<PublisherT>
    create_publisher(
      const std::string & topic_name,
      const rclcpp::QoS & qos,
      const PublisherOptionsWithAllocator<AllocatorT> & options =
      PublisherOptionsWithAllocator<AllocatorT>()
    );

    /// Create and return a Subscription.
    /**
     * \param[in] topic_name The topic to subscribe on.
     * \param[in] qos QoS profile for Subcription.
     * \param[in] callback The user-defined callback function to receive a message
     * \param[in] options Additional options for the creation of the Subscription.
     * \param[in] msg_mem_strat The message memory strategy to use for allocating messages.
     * \return Shared pointer to the created subscription.
     */
    template<
      typename MessageT,
      typename CallbackT,
      typename AllocatorT = std::allocator<void>,
      typename SubscriptionT = rclcpp::Subscription<MessageT, AllocatorT>,
      typename MessageMemoryStrategyT = typename SubscriptionT::MessageMemoryStrategyType
    >
    std::shared_ptr<SubscriptionT>
    create_subscription(
      const std::string & topic_name,
      const rclcpp::QoS & qos,
      CallbackT && callback,
      const SubscriptionOptionsWithAllocator<AllocatorT> & options =
      SubscriptionOptionsWithAllocator<AllocatorT>(),
      typename MessageMemoryStrategyT::SharedPtr msg_mem_strat = (
        MessageMemoryStrategyT::create_default()
      )
    );

#### 4.服务通信

```
/// Create and return a Client.
/**
 * \param[in] service_name The topic to service on.
 * \param[in] qos_profile rmw_qos_profile_t Quality of service profile for client.
 * \param[in] group Callback group to call the service.
 * \return Shared pointer to the created client.
 */
template<typename ServiceT>
typename rclcpp::Client<ServiceT>::SharedPtr
create_client(
  const std::string & service_name,
  const rmw_qos_profile_t & qos_profile = rmw_qos_profile_services_default,
  rclcpp::CallbackGroup::SharedPtr group = nullptr);

/// Create and return a Service.
/**
 * \param[in] service_name The topic to service on.
 * \param[in] callback User-defined callback function.
 * \param[in] qos_profile rmw_qos_profile_t Quality of service profile for client.
 * \param[in] group Callback group to call the service.
 * \return Shared pointer to the created service.
 */
template<typename ServiceT, typename CallbackT>
typename rclcpp::Service<ServiceT>::SharedPtr
create_service(
  const std::string & service_name,
  CallbackT && callback,
  const rmw_qos_profile_t & qos_profile = rmw_qos_profile_services_default,
  rclcpp::CallbackGroup::SharedPtr group = nullptr);
```

定时器

```
/// Create a timer.
/**
 * \param[in] period Time interval between triggers of the callback.
 * \param[in] callback User-defined callback function.
 * \param[in] group Callback group to execute this timer's callback in.
 */
template<typename DurationRepT = int64_t, typename DurationT = std::milli, typename CallbackT>
typename rclcpp::WallTimer<CallbackT>::SharedPtr
create_wall_timer(
  std::chrono::duration<DurationRepT, DurationT> period,
  CallbackT callback,
  rclcpp::CallbackGroup::SharedPtr group = nullptr);
```



