# reversal

写一个模型+控制器

| 文件类型   | 作用                    | 注意事项                     |
| ---------- | ----------------------- | ---------------------------- |
| urdf.xacro | 发布机器人的joint和link | transsion是必须的            |
| yaml       | 用来加载控制器          | type类型要和transsmision对应 |
| launch     | 一次启动多个文件        |                              |





问题描述

```c
Could not find resource 'pitch_down_joint' in 'hardware_interface::EffortJointInterface'.
gazebo: [ControllerManager::loadController]: Initializing controller'controllers/arm_controller' failed
```

| 问题                                     | 可能出错点     | 尝试解决方法                                                 | 结果                                                         |
| ---------------------------------------- | -------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 在力矩控制器接口里找不到pitch_down_joint | urdf写得不规范 | 找到在那里发布joint到接口里<br />对比其它可行的joint看区别在那里 | 和预期出错点差不多，参数服务器(=是的吗=）没有收到transsmion的东西因为把它放在文件里<br /><xacro:unless value="${simulation}"><br /></xacro:unless><br />的中间了，然后被忽视了 |

### 收获

- 理解正在使用的文件的每一个东西的作用
- 没有思路时可以从头到尾看一遍
- 利用和相似的但是可用的东西对比



# calibartion

问题描述

关节校准控制器target值无法起作用

| 问题                                                         | 可能出错点                                                   | 尝试解决方法                               | 结果 |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------ | ---- |
| command的输出值一直为当前位置的负值（也就是说它一直想回到零点） | 1.控制器获取的资源方式有问题<br />2.有程序在发sendcommand（0） | 看gimbal_controller怎么使用sendcommand()的 |      |



### gimbal_base

初始化三个东西

- 硬件接口hw
- ros句柄
- 控制器句柄

定义一个yaw的句柄放在controller_nh里？

```
ros::NodeHandle nh_yaw = ros::NodeHandle(controller_nh, "yaw");
```

这是干嘛的？？？

```
hardware_interface::EffortJointInterface* effort_joint_interface =
    robot_hw->get<hardware_interface::EffortJointInterface>();
```

初始化yaw的控制器

```
ctrl_yaw_.init(effort_joint_interface, nh_yaw)
```

创建订阅者往控制器话题发命令

```
cmd_gimbal_sub_ = controller_nh.subscribe<rm_msgs::GimbalCmd>("command", 1, &Controller::commandCB, this);
```

最后更新命令

```
ctrl_yaw_.setCommand(yaw_des, yaw_vel_des + ctrl_yaw_.joint_.getVelocity() - angular_vel_yaw.z);
```

发布命令

```
ctrl_yaw_.update(time, period);
```

更新命令

```
ctrl_yaw_.joint_.setCommand(ctrl_yaw_.joint_.getCommand() - k_chassis_vel_ * chassis_vel_->angular_->z());
```