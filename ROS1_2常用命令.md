# ROS1 / ROS2 常用指令

## 工作空间 Workspace

创建文件夹  

  ```bash
  mkdir -p ws/src  
  cd ws/src
  ```

---

- ROS1  

  ```bash
  cd ws/src
  catkin_init_workspace # 初始化workspace
  ```

- ROS2

  ```bash
  ws/src 中创建功能包 就会自动初始化workspace
  ```

  - **src**，代码空间，未来编写的代码、脚本，都需要人为的放置到这里；
  - **install**，安装空间，放置编译得到的可执行文件和脚本；
  - build，编译空间，保存编译过程中产生的中间文件；
  - log，日志空间，编译和运行过程中，保存各种警告、错误、信息等日志。

## 创建包 Package

- ROS1

  ```bash
  cd ~/ws/src
  catkin_create_pkg <pkg_name> <dependencies1> <dependencies2> ...
  ```

- ROS2

  ```bash
  cd ~/ws/src
  ros2 pkg create --build-type ament_cmake <package_name> # 创建c++包
  ros2 pkg create --build-type ament_python <package_name> # 创建python包

  ros2 pkg create --build-type ament_cmake --node-name <node_name> <package_name> # 创建包，并在包内src文件夹下创建<node_name>.cpp文件

  ros2 pkg create --build-type ament_cmake --license Apache-2.0 <package_name>
  ```

  - C++功能包
    - package.xml文件包含功能包的版权描述，和各种**依赖**的声明。
    - CMakeLists.txt文件是**编译规则**，C++代码需要编译才能运行，所以必须要在该文件中设置如何编译，使用CMake语法。

  - Python功能包
    - package.xml文件包含功能包的版权描述，和各种**依赖**的声明。
    - setup.py文件里边也包含一些版权信息，除此之外，还有“**entry_points**”配置的程序入口

### CmakeLists.txt

- 生成可执行节点 Node

  ```bash
  add_executable(<name> [source1] [source2] ...) 
  # 以提供的源文件编译生成一个可执行文件 <name>。
  # 在这些文件中，除了主程序的入口（如 main()），其他文件提供的是可被调用的模块或逻辑单元。

  target_link_libraries(${PROJECT_NAME}_node
    ${catkin_LIBRARIES}  # catkin_LIBRARIES 是通过 find_package(...) 找到的 ROS 库
  )
  或
  target_link_libraries(<target>
    [item1] [item2] [...]  # [item1] [item2] ...：是要链接到该目标上的库（静态库、动态库或变量表示的库列表
  )
  ```

- msg相关
  .msg 文件必须放在 ws/src/package/msg 文件夹中，否则编译时找不到

  ```bash
  # 指定依赖组件（必须包含 message_generation）
  find_package(catkin REQUIRED COMPONENTS
    roscpp
    std_msgs  # # 如果你的 .msg 中使用了 std_msgs/String 等标准类型
    message_generation
  )
  # 声明消息文件，根据 .msg 文件生成相应的 C++ / Python 语言的消息接口代码
  add_message_files(
    FILES
    <message1>.msg
    <message2>.msg
    ...
  )
  # 设置依赖项
  generate_messages(
    DEPENDENCIES
    std_msgs  # 如果你的 .msg 中使用了 std_msgs/String 等标准类型
  )

  add_dependencies(<target> depend1 depend2 ...) # 控制构建顺序，确保依赖的生成代码已完成
  # <target>：你通过 add_executable() 或 add_library() 定义的目标
  # depend1, depend2：该目标依赖的其他构建目标，这些目标必须先被构建

  例如
  add_dependencies(node ${PROJECT_NAME}_generate_messages_cpp)  # 
  ```

## 编译 Build

- ROS1

    ```bash
    cd ~/ws
    catkin_make [-j8]# 将生成 build, devel 两个文件夹 [-j8]使用8核编译

    (可选)catkin_make install # 将生成 install 文件夹
    ```

- ROS2

    ```bash
    sudo apt install python3-colcon-common-extensions  # 安装colcon
    ```

    ```bash
    cd ~/ws
    colcon build

    colcon build --packages-select <package_name>   # 只编译给定功能包 (如果不想构建特定的包，则在对应包内放置一个名为 COLCON_IGNORE 的空文件)

    colcon build --symlink-install  # 建立符号链接，避免修改一次python文件就得重新编译
    ```

清除编译历史

```bash
cd ws
sudo rm -rf install/ build/ log/   # 清除工作空间中的编译历史
```

## 设置环境变量

- ROS1

  ```bash
  source devel/setup.bash
  或者
  echo "source ~/ws/devel/setup.bash" >> ~/.bashrc
  ```

- ROS2

    ```bash
    source install/local_setup.bash
    或者
    echo "source ~/ws/install/local_setup.bash" >> ~/.bashrc
    # setup.bash会把外部的多个工作空间囊括进来，如果多个工作空间中有相同名字的功能包可能就会互相冲突。
    # local_setup.bash只会source脚本所在目录。这样只会查找当前install目录下的执行文件和依赖。
    ```

## 节点 Node

- ROS1
  
    ```bash
    roscore

    rosrun <package_name> <executable_name>

    rosnode list
    ```

- ROS2

    ```bash
    ros2 run <package_name> <executable_name>

    ros2 run <package_name> <executable_name> --ros-args --remap __node:=my_turtle  # 重映射节点名

    ros2 node list  # 列出所有运行中的节点
    ros2 node info <node_name>  # 查看节点信息（topic,service,action）
    ```

## 话题 Topic

- ROS1

    ```bash
    rostopic list

    rostopic pub <topic_name> <msg_type> '<args>'    # 发布话题
    ```

- ROS2

    ```bash
    ros2 topic list
    ros2 topic type <topic_name>    # 查看话题消息的类型
    ros2 topic list -t  # 返回话题列表，在括号中附加话题消息类型

    ros2 topic pub <topic_name> <msg_type> '<args>'  # 发布消息，并指定消息内容
    
    ros2 topic echo <topic_name>    # 输出话题发布的消息
    ros2 topic info <topic_name>   # 查看话题信息
    ros2 topic hz <topic_name>  # 输出已有话题的发布频率
    ros2 topic bw <topic_name>  # 输出已有话题的带宽
    ros2 topic find <topic_type>  # 查找特定类型的所有话题
    
    ros2 interface show <msg_type>  # 输出消息的格式定义
    ```

## 服务 Service

- ROS1

    ```bash
    rosservice call <service_name>  # 调用服务
    ```

- ROS2

    ```bash
    ros2 service list
    ros2 service type <service_name>    # 查看服务的类型
    ros2 service list -t

    ros2 service find <srv_type>    # 查找特定类型的所有服务
    ros2 service call <service_name> <srv_type> <arguments> # 调用服务
    # 例如：ros2 service call /spawn turtlesim/srv/Spawn "{x: 2, y: 2, theta: 0.2, name: ''}"

    ros2 interface show <srv_type>  # 输出服务类型的定义
    ```

## 动作 Action

- ROS1
  
- ROS2

    ```bash
    ros2 action list
    ros2 action list -t
    ros2 action info <action_name> 

    ros2 action send_goal <action_name> <action_type> <values> [--feedback]  # <values> need to be in YAML format.

    ros2 interface show <action_name>:
        goal
        ---
        result
        ---
        feedback
    ```

## 参数 Parameter

- 命令行操作参数

    ```bash
    rosparam list   # 查看参数列表
    
    rosparam get <param_name>  # 查询某个参数的值
    rosparam set <param_name> <value>  # 修改某个参数的值
    
    rosparam dump <file_name>  # 将参数保存到文件中
    rosparam load <file_name>  # 从文件中加载参数
    
    rosparam delete <param_name>  # 删除某个参数
    ```

    ```bash
    ros2 param list # 查看参数列表

    ros2 param describe <node_name> <param_name> # 查看某个参数的描述信息

    ros2 param get <node_name> <param_name> # 查询某个参数的值
    ros2 param set <node_name> <param_name> <value> # 修改某个参数的值

    ros2 param dump <node_name> >> <file.yaml> # 将某个节点的参数保存到参数文件中
    ros2 param load <node_name> <file.yaml> # 一次性加载某一个文件中的所有参数

    ros2 run <package_name> <executable_name> --ros-args --params-file <file_name> # 节点启动时加载参数文件
    ```

- 参数编程：在程序中声明、设置、修改参数

## 通信接口

- 话题 .msg
  
    ```bash
    fieldtype fieldname
    或者
    fieldtype fieldname fielddefaultvalue  # 字段默认值
    或者
    constanttype CONSTANTNAME=constantvalue  # 常量定义
    或者
    another_pkg/AnotherMessage msg  # 嵌套消息定义
    ```

  fieldtype可以是：
  - a built-in-type

    | 类型名称 | C++        | Python            | DDS type |
    |----------|------------|-------------------|----------|
    | bool     | bool       | builtins.bool     | boolean  |
    | byte     | uint8_t    | builtins.bytes*   | octet    |
    | char     | char       | builtins.int*     | char     |
    | float32  | float      | builtins.float*   | float    |
    | float64  | double     | builtins.float*   | double   |
    | int8     | int8_t     | builtins.int*     | octet    |
    | uint8    | uint8_t    | builtins.int*     | octet    |
    | int16    | int16_t    | builtins.int*     | short    |
    | uint16   | uint16_t   | builtins.int*     | unsigned short |
    | int32    | int32_t    | builtins.int*     | long     |
    | uint32   | uint32_t   | builtins.int*     | unsigned long |
    | int64    | int64_t    | builtins.int*     | long long |
    | uint64   | uint64_t   | builtins.int*     | unsigned long long |
    | string   | std::string| builtins.str      | string   |
    | wstring  | std::u16string | builtins.str   | wstring  |
    | static array | std::array<T, N> | builtins.list*    | T[N]           |
    | unbounded dynamic array | std::vector | builtins.list | sequence       |
    | bounded dynamic array | customclass<T, N> | builtins.list* | sequence<T, N> |
    | bounded string | std::string | builtins.str*     | string         |

    ```bash
    int32[] unbounded_integer_array  # 每个内置类型都可以用来定义数组
    int32[5] five_integers_array
    int32[<=5] up_to_five_integers_array

    string string_of_unbounded_size
    string<=10 up_to_ten_characters_string

    string[<=5] up_to_five_unbounded_strings
    string<=10[] unbounded_array_of_strings_up_to_ten_characters_each
    string<=10[<=5] up_to_five_strings_up_to_ten_characters_each
    
    uint8 x 42
    int16 y -2000
    string full_name "John Doe"  # 字符串值必须用单引号或双引号定义。 目前字符串值没有转义
    int32[] samples [-200, -100, 0, 100, 200]

    int32 X=123  # 常量名称必须大写
    int32 Y=-123
    string FOO="foo"
    string EXAMPLE='bar'

    another_pkg/AnotherMessage msg
    ```

  - names of Message descriptions defined on their own, such as “geometry_msgs/PoseStamped”

  Field names
  字段名称必须为小写字母数字字符，并使用下划线分隔单词。字段名称必须以字母开头，且不得以下划线结尾或包含两个连续的下划线。但是常量名称必须大写。

- 服务 .srv

  由`---`分割的两部分:request和response

    ```bash
    # request constants
    int8 FOO=1
    int8 BAR=2
    # request fields
    int8 foobar
    another_pkg/AnotherMessage msg
    ---
    # response constants
    uint32 SECRET=123456
    # response fields
    another_pkg/YetAnotherMessage val
    CustomMessageDefinedInThisPackage value
    uint32 an_integer
    ```

- 动作 .action

    ```bash
    <request_type> <request_fieldname>
    ---
    <response_type> <response_fieldname>
    ---
    <feedback_type> <feedback_fieldname>
    ```

```bash
ros2 interface list                    # 查看系统接口列表
ros2 interface show <interface_name>   # 查看某个接口的详细定义
ros2 interface package <package_name>  # 查看某个功能包中的接口定义
```

## 分布式通信

- 分布式网络分组——DOMAIN机制

    ```bash
    export ROS_DOMAIN_ID=<your_domain_id>：设置ROS的域ID，默认为0，可用于多机器人系统的通信
    ```

    要在 shell 会话之间维持此设置，可以将命令添加到 shell 启动脚本中：

    ```bash
    echo "export ROS_DOMAIN_ID=<your_domain_id>" >> ~/.bashrc
    ```

## DDS(Data Distribution Service) - ROS2引入

- QoS(Quality of Service)：DDS中另外一个重要特性，包括延迟、带宽、可靠性等。
  
  - 查看ROS2系统中每一个发布者或者订阅者的QoS策略：

    ```bash
    ros2 topic info <topic_name> --verbose
    ```

  - 命令行配置 QoS

    ```bash
    ros2 topic pub /chatter std_msgs/msg/Int32 "data: 42" --qos-reliability best_effort 
    ros2 topic echo /chatter --qos-reliability best_effort
    ```

  - 编程配置 QoS

## 启动 Launch

- ROS1: .launch文件(XML)

- [ROS2:  .launch文件(XML), .yaml(YAML), .py文件](https://book.guyuehome.com/ROS2/3.%E5%B8%B8%E7%94%A8%E5%B7%A5%E5%85%B7/3.1_Launch/)

    ```bash
    ros2 launch <package_name> <launch_file_name>
    ros2 launch <package_name> <launch_file_name> param:=value # 设置传递给启动文件的参数
    ```
补充:  
- [ROS中的launch启动文件](https://caodong-street.github.io/2021/10/18/ros-zhong-de-launch-qi-dong-wen-jian/)

  - output=”screen”：将节点的标准输出/错误打印到终端屏幕(调试阶段，实时查看节点输出信息)，默认输出为日志文档。  
      - 标准输出 (stdout)：一般是 ROS_INFO, **printf**, **cout** 等；  
      - 标准错误 (stderr)：一般是 ROS_WARN, ROS_ERROR, ROS_FATAL, cerr 等。  
  
  - respawn=”true”：复位属性，该节点停止时，会自动重启，默认为false。  
  - required=”true”：必要节点，当该节点终止时，launch文件中的其他节点也被终止。  
  - ns=”namespace”：命名空间，为节点内的相对名称添加命名空间前缀。  
  - args=”arguments”：节点需要的输入参数，类似于launch文件内部的局部变量，仅限于launch文件使用，便于launch文件的重构，与ROS节点内部的实现没有关系。  
  - param: parameter是ROS系统运行中的参数，存储在参数服务器中。

## 其他

- ROS2
  
  - 自动安装依赖

    ```bash
    cd ws
    rosdep install -i --from-path src --rosdistro <distro_name> -y：自动根据packages.xml安装依赖  
    ```

    ```bash
    ros2 pkg executables <package_name>：检查package中可执行的程序  
    ```
