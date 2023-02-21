# ENGINEER_SERVO

```
topic: "/servo_server/delta_twist_cmds"
```

```
rostopic pub -r 100 -s /servo_server/delta_twist_cmds geometry_msgs/TwistStamped "header: auto
twist:
  linear:
    x: 0.0
    y: 0.01
    z: -0.01
  angular:
    x: 0.0
    y: 0.0
    z: 0.0"
```





## ur_simulated_config.yaml

| 参数                                                         | 解释                                                  | 问题                       |
| ------------------------------------------------------------ | ----------------------------------------------------- | -------------------------- |
| ues_gazebo                                                   | 仿真还是真的                                          |                            |
| command_in_type                                              | 输入的命令是范围还是速度                              |                            |
| ==scale:<br />  linear<br />  rotational<br />  joint==      | 最大的线速度和角速度                                  | 只有joint最后起作用吗？    |
| ==low_pass_filter_coeff==                                    | 对LPF的数据信任程度（越高越大）                       | 如果为零就会引入很多噪声？ |
| publish_period                                               | 名字                                                  |                            |
| ==low_latency_mode==                                         | 低延时，如果收到命令立刻发布，忽略publish_period      |                            |
| command_out_type                                             | 发送的消息类型我们的是trajectory_msgs/JointTrajectory |                            |
| publish_joint_positions<br />publish_joint_velocities<br />publish_joint_accelerations | 为了节省带宽一般只选择前两个                          |                            |
| move_group_name                                              | 名字                                                  |                            |
| ==planning_frame==                                           | 规划的坐标系一般是base_link                           |                            |
| ==ee_frame_name==                                            | effector，末端执行器                                  |                            |
| ==robot_link_command_frame==                                 | 一般不是base或者effector                              | ?                          |
| incoming_command_timeout                                     | 如果一段时间没收到消息自动停止                        |                            |
| num_outgoint_halt_msgs_to_publish                            | 如果ROS丢了消息数达到这个就关                         |                            |
| lower_singularity_threshold                                  | 当条件数达到这个时开始减速（接近奇点）                |                            |
| hard_stop_singularity_threshold                              | 当条件数达到这个的时候停止                            |                            |
| joint_topic                                                  | joint_states                                          |                            |
| status_topic                                                 | status                                                |                            |
| command_out_topic                                            | 往哪一个话题发数据                                    |                            |
| ==check_collisions==                                         | 是否开启碰撞检测                                      |                            |
| collision_check_rate                                         | 名字                                                  |                            |
| collision_check_type                                         | threshold_distance以什么进行碰撞检测                  |                            |
| self_collision_proximity_threshold<br />scene_collision_proximity_threshold | 检测开始范围                                          |                            |
| collision_distance_safety_factor                             | 碰撞安全系数，在低延时的时候可以搞大点                |                            |
| min_allowable_collision_distance                             | 如果一个碰撞范围在这之内就停止检测                    |                            |

