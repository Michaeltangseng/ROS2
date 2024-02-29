# ROS2
ROS2学习

====================================================================
屏幕亮度调节：
	xrandr -q
	xrandr --output HDMI-0 --brightness 0.8

Vscode里调用copilot	
	
2.ROS学习网站
https://fishros.org.cn/forum/

3.在dev_ws根目录下，使用pip包自动寻找目录下面文件所需要的所有安装依赖并安装
rosdepc install -i --from-path src --rosdistro humble -y
安装sudo apt install python3-colcon-ros
对更目录下的所有代码进行编译colcon build，生成build/install/log等文件夹和文件
配置环境变量 source install/local_setup.sh(缺点是不能永久)--解决方法：home/.bashrc,最后一行写入source ~/dev_ws/install/local_setup.sh

4.创建多个功能包
cd ~/dev_ws/src
ros2 pkg create --build-type ament_cmake learning_pkg_c
ros2 pkg create --build-type ament_python learning_pkg_python

5.编译功能包
cd ~/dev_ws/src
colcon build
source install/local_setup.bash

6.--------------------------------------------------------------------------------------------------------------
古月居-P7-节点 node
在~/dev_ws/ learning_node/ learning_node/ node_helloworld.py里面编写node类，setup.py设置对应的
entry_points={
'console_scripts': [
'node_helloworld = learning_node.node_helloworld:main',
'node_helloworld_class = learning_node.node_helloworld_class:main',
'node_object = learning_node.node_object:main',
'node_object_webcam = learning_node.node_object_webcam:main',
],
}
配置完成后，在工作空间里编译（colcon build，编译文件在/dev_ws/install/learning_node/lib/learning_node下，相关代码在~/dev_ws/install/learning_node/lib/python3.10/site-packages/learning_node文件夹下，所以每次配置完后需要重新编译）/运行（ros2 run learning_node node_helloworld）
cd  ~/dev_ws
ros2 run learning_node node_helloworld

调试opencv：
sudo apt install python3-opencv
ros2 run learning_node node_object

继续优化opencv代码：
ros2 run learning_node node_object_webcam#注意设置摄像头

7.--------------------------------------------------------------------------------------------------------------
古月居-P8-话题 topic
发布：ros2 run learning_topic topic_helloworld_pub
订阅：ros2 run learning_topic topic_helloworld_sub

实现复杂的话题通信（opencv摄像头识别苹果）
发布：ros2 run learning_topic topic_webcam_pub
订阅：ros2 run learning_topic topic_webcam_sub

rqt结构：
rqt_graph

8.--------------------------------------------------------------------------------------------------------------
古月居-P9-服务（话题间的相互回应）service
客户端Client：ros2 run learning_service service_adder_server
服务器端Server：ros2 run learning_service service_adder_client 2 3

9.--------------------------------------------------------------------------------------------------------------
古月居-P10-通信接口
接口定义方式：
话题（.msg文件）：通信数据int32 x / int32 y
服务（.srv）：请求数据int 64 a / int64 b              应答数据int64 sum
动作（.action）：目标bool enable                        结果bool finish                反馈int32 state

查看ROS自定义的msg：目录～/opt/ros/humble/share搜索msg

查询接口：
ros2 interface 
ros2 interface list
ros2 interface show sensor_msgs/msg/Image
ros2 interface show turtlesim/action/RotateAbsolute
查看功能包的接口ros2 interface package learning_interface

查询摄像头的服务接口
ros2 usb_cam usb_cam_node_exe取得相机发布图像话题
ros2 run learning_service service_object_server：订阅图像话题，开启识别，提供目标查询的服务
ros2 run learning_service service_object_client：在需要目标位的时候发出请求

查看古月居定义的接口目录：
定义格式：/home/jf/dev_ws/src/GuYueJu_learn_bilibili/ros2_21_tutorials/learning_interface/srv/AddTwoInts.srv
配置接口：在CMakeLists.txt里面，写入自定义的信息
rosidl_generate_interfaces(${PROJECT_NAME}
"msg/ObjectPosition.msg"
"srv/AddTwoInts.srv"
"srv/GetObjectPosition.srv"
"action/MoveCircle.action"
)



查询摄像头的话题通信接口
mul /home/jf/dev_ws/src/GuYueJu_learn_bilibili/ros2_21_tutorials/learning_interface/msg/ObjectPosition.msg

对应的/home/jf/dev_ws/src/GuYueJu_learn_bilibili/ros2_21_tutorials/learning_topic/learning_topic/interface_object_pub.py

ros2 usb_cam usb_cam_node_exe取得相机发布图像话题
ros2 run learning_topic interface_object_pub ：订阅图像话题，开启识别，提供目标查询的服务
ros2 run learning_topic interface_object_sub ：在需要目标位的时候发出请求


10.--------------------------------------------------------------------------------------------------------------
古月居-P11-动作
查看动作信息ros2 action

机器人画圆动作的实现（动作通信）
ros2 run learning_action action_move_server
ros2 run learning_action action_move_client

目录/home/jf/dev_ws/src/GuYueJu_learn_bilibili/ros2_21_tutorials/learning_interface/action/MoveCircle.action
对应/home/jf/dev_ws/src/GuYueJu_learn_bilibili/ros2_21_tutorials/learning_action/learning_action/action_move_server.py

11.--------------------------------------------------------------------------------------------------------------
古月居-P12-参数
以小乌龟为例：
ros2 param list
ros2 param descirbe turtlesim background_b
ros2 param get turtlesim background_b
ros2 param set turtlesim background_b 100

保存节点下的所有参数：
ros2 param dump turtlesim
ros2 param dump turtlesim >> turtlesim.yaml
ros2 param load t turtlesim  turtlesim.yaml

参数编程：
ros2 run learning_parameter param_declare
ros2 param get param_declare robot_name turtle
目录/home/jf/dev_ws/src/GuYueJu_learn_bilibili/ros2_21_tutorials/learning_parameter/learning_parameter/param_declare.py

参数在机器人中的应用
ros2 run usb_cam usb_cam_node_exe
ros2 run learning_parameter param_object_detect
ros2 param set param_object_detect red_h_upper 180
目录/home/jf/dev_ws/src/GuYueJu_learn_bilibili/ros2_21_tutorials/learning_parameter/learning_parameter/param_object_detect.py

12.--------------------------------------------------------------------------------------------------------------
古月居-P14-DDS
在命令行中配置DDS：
ros2 topic pub /chatter std_msgs/msg/Int32 “data: 42” --qos-reliability best_effort
第二个终端：ros2 topic echo /chatter --qos-reliability reliable
第二个终端：ros2 topic echo /chatter --qos-reliability best_effort
第二个终端：ros2 topic info /chatter --verbose

DDS编程示例（话题QoS）：
ros2 run learning_qos qos_helloworld_pub
ros2 run learning_qos qos_helloworld_sub
目录/home/jf/dev_ws/src/GuYueJu_learn_bilibili/ros2_21_tutorials/learning_qos/learning_qos/qos_helloworld_pub.py

13.--------------------------------------------------------------------------------------------------------------
古月居-P15-launch文件
ros2 launch learning_launch simple.launch.py
目录/home/jf/dev_ws/src/GuYueJu_learn_bilibili/ros2_21_tutorials/learning_launch/launch/simple.launch.py

ros2 run rviz2 rviz2 -d /home/jf/dev_ws/src/GuYueJu_learn_bilibili/ros2_21_tutorials/learning_launch/rviz/turtle_rviz.rviz

ros2 launch learning_launch parameters.launch.py
目录/home/jf/dev_ws/src/GuYueJu_learn_bilibili/ros2_21_tutorials/learning_launch/launch/parameters.launch.py

ros2 launch learning_launch parameters_yaml.launch.py
目录/home/jf/dev_ws/src/GuYueJu_learn_bilibili/ros2_21_tutorials/learning_launch/config/turtlesim.yaml

一个launch文件启动多个launch文件
目录/home/jf/dev_ws/src/GuYueJu_learn_bilibili/ros2_21_tutorials/learning_launch/launch/namespaces.launch.py

在设置完毕后，需要在setup.py里进行配置，使其能在build后将这些文件拷贝到build目录
data_files=[
('share/ament_index/resource_index/packages',
['resource/' + package_name]),
('share/' + package_name, ['package.xml']),
(os.path.join('share', package_name, 'launch'), glob(os.path.join('launch', '*.launch.py'))),
(os.path.join('share', package_name, 'config'), glob(os.path.join('config', '*.*'))),
(os.path.join('share', package_name, 'rviz'), glob(os.path.join('rviz', '*.*'))),
],

14.--------------------------------------------------------------------------------------------------------------
古月居-P16-TF坐标系
海龟跟随中的坐标变换
sudo apt install ros-humble-turtle-tf2-py 
sudo apt install ros-humble-tf2-tools
sudo pip3 install transforms3d
ros2 launch turtle_tf2_py turtle_tf2_demo.launch.py
ros2 run turtlesim turtle_teleop_key
绘制坐标系结构ros2 run tf2_tools view_frames
显示坐标系间相对数值关系ros2 run tf2_ros tf2_echo turtle2 turtl1
ros2 run rviz2 rviz2 -d $(ros2 pkg prefix –share turtle_tf2_py)/rviz/turtle_rviz.rviz

静态TF变换
ros2 run learning_tf static_tf_broadcaster
查看坐标系关系ros2 run tf2_tools view_frames
代码设置相对关系：目录/home/jf/dev_ws/src/GuYueJu_learn_bilibili/ros2_21_tutorials/learning_tf/learning_tf/static_tf_broadcaster.py
class StaticTFBroadcaster(Node):
def __init__(self, name):
super().__init__(name) # ROS2节点父类初始化
self.tf_broadcaster = StaticTransformBroadcaster(self) # 创建一个TF广播器对象

static_transformStamped = TransformStamped() # 创建一个坐标变换的消息对象
static_transformStamped.header.stamp = self.get_clock().now().to_msg() # 设置坐标变换消息的时间戳
static_transformStamped.header.frame_id = 'world' # 设置一个坐标变换的源坐标系
static_transformStamped.child_frame_id = 'house' # 设置一个坐标变换的目标坐标系
static_transformStamped.transform.translation.x = 10.0 # 设置坐标变换中的X、Y、Z向的平移
static_transformStamped.transform.translation.y = 5.0 
static_transformStamped.transform.translation.z = 0.0
quat = tf_transformations.quaternion_from_euler(0.0, 0.0, 0.0) # 将欧拉角转换为四元数（roll, pitch, yaw）
static_transformStamped.transform.rotation.x = quat[0] # 设置坐标变换中的X、Y、Z向的旋转（四元数）
static_transformStamped.transform.rotation.y = quat[1]
static_transformStamped.transform.rotation.z = quat[2]
static_transformStamped.transform.rotation.w = quat[3]

self.tf_broadcaster.sendTransform(static_transformStamped) 

TF监听
查询两个坐标系间的关系ros2 run learning_tf tf_listener
终端1：ros2 run learning_tf static_tf_broadcaster
终端2：ros2 run learning_tf tf_listener （见tf_listener.py）
显示出坐标系的信息，与代码里设置的一致

海龟跟随功能的实现
ros2 launch learning_tf turtle_following_demo.launch.py
ros2 run turtlesim turtle_teleop_key
目录/home/jf/dev_ws/src/GuYueJu_learn_bilibili/ros2_21_tutorials/learning_tf/launch/turtle_following_demo.launch.py

15.--------------------------------------------------------------------------------------------------------------
古月居-P17-URDF
<link>：描述刚体部分的外观和物理属性Size/color/shape/inertial matrix collision properties
<joint>：
continuous:无限旋转
revolute:有限旋转
prismatic:平移滑动，有位置极限
fiexd:固定
floating:浮动关节，允许平移/旋转
plannar:平面关节，允许在平面正交方向上平移或旋转

ros2 launch learning_urdf display.launch.py
目录/home/jf/dev_ws/src/GuYueJu_learn_bilibili/ros2_21_tutorials/learning_urdf/launch/display.launch.py
目录/home/jf/dev_ws/src/GuYueJu_learn_bilibili/ros2_21_tutorials/learning_urdf/urdf/mbot_base.urdf

查看URDF结构
urdf_to_graphviz mbot_base.urdf
查看结构.pdf
cd ~/dev_ws/src/GuYueJu_learn_bilibili/ros2_21_tutorials/learning_urdf/urdf
urdf_to_graphviz mbot_base.urdf

16.--------------------------------------------------------------------------------------------------------------
古月居-P18-Gazebo
安装:
sudo apt install ros-humble-gazebo-*
ros2 launch gazebo_ros gazebo.launch.py(似乎不可用，gazebo界面卡住了---原因：没有source gazebo，需要source  /usr/share/gazebo/setup.sh，或在.bashsrc最后一行添加source  /usr/share/gazebo/setup.sh)

xacro文件
目录/home/jf/dev_ws/src/GuYueJu_learn_bilibili/ros2_21_tutorials/learning_gazebo/urdf/mbot_base_gazebo.xacro

运动仿真
ros2 launch learning_gazebo load_urdf_into_gazebo.launch.py
ros2 run teleop_twist_keyboard teleop_twist_keyboard

新一代gazebo Ignition
sudo apt install ros-humble-ros-ign
ros2 launch ros-ign_gazebo_demos rgbd_camara_bridge.launch.py

17.--------------------------------------------------------------------------------------------------------------
古月居-P19-Rviz
ros2 run rviz2 rviz2

摄像头插件：libgazebo_ros_camara.so
目录 /home/jf/dev_ws/src/GuYueJu_learn_bilibili/ros2_21_tutorials/learning_gazebo/urdf/sensors/camera_gazebo.xacro

启动
终端1：ros2 launch learning_gazebo load_mbot_camera_into_gazebo.launch.py
终端2：ros2 run rviz2 rviz2
终端3：ros2 run teleop_twist_keyboard teleop_twist_keyboard

kinect点云相机
目录/home/jf/dev_ws/src/GuYueJu_learn_bilibili/ros2_21_tutorials/learning_gazebo/urdf/sensors/kinect_gazebo.xacro

启动
终端1：ros2 launch learning_gazebo load_mbot_rgbd_into_gazebo.launch.py
终端2：ros2 run rviz2 rviz2
终端3：ros2 run teleop_twist_keyboard teleop_twist_keyboard

雷达
目录/home/jf/dev_ws/src/GuYueJu_learn_bilibili/ros2_21_tutorials/learning_gazebo/urdf/sensors/lidar_gazebo.xacro
终端1：ros2 launch learning_gazebo load_mbot_laser_into_gazebo.launch.py
终端2：ros2 run rviz2 rviz2
终端3：ros2 run teleop_twist_keyboard teleop_twist_keyboard

18.--------------------------------------------------------------------------------------------------------------
古月居-P20-RQT
安装与启动
sudo apt install ros-humble-rqt
rqt
终端2：ros2 run turtlesim_node
终端3：ros2 run turtlesim turtle_teleop_key


Github上的ROS2 视觉检测demo


================================================================================================================
