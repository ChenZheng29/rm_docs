rm_control
rm_msgs: 自定义的 ROS 话题消息、服务、动作
rm_common: 常用函数、算法、裁判系统收发、ui

rm_hw: 同名节点通过 SocketCAN 与执行器进行通信获取数据并发送指令，在实车运行时提供硬件接
口给控制器
rm_dbus: dbus 数据接收节点
rm_gazebo: 同名 Gazebo Plugin 在仿真运行时提供硬件接口给控制器
rm_controllers
rm_chassis_controllers: 麦克纳姆轮、舵轮、平衡车的底盘控制器
rm_orientation_controllers: 获取 imu 姿态以修正整车实际姿态的控制器
rm_gimbal_controllers: 内置枪管角解算模型的云台控制器
rm_shooter_controllers: 操作摩擦轮，拨弹盘完成发射控制器的控制器
rm_calibration_controllers: 校准执行器位置的控制器
robot_state_controller: robot_state_publisher [8] 的高性能版，高频维护 tf
gpio_controller: 读写 gpio 的控制器
mimic_joint_controller: 使指定 joint 的位置模仿另一 joint 的控制器
rm_description: 所有机器人的 URDF，定义了：机器人各坐标系关系、电机与关节的映射、关节的限位、仿
真需要的物理属性
rm_detection: 装甲板和能量机关视觉识别
rm_stone: 矿石和障碍块视觉识别
rm_track: 选择并预测目标的位置速度，并发送给云台控制器
rm_manual: 普通机器人决策

rm_fsm: 哨兵机器人决策





## rm_common

### command_sender.h

#### CommandSenderBa.h

