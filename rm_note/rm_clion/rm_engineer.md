# rm_engineer

> 理解middlewear必须在理解action的前提之下

## Action

> In some cases, however, if the service takes a long time to execute, the user might want the ability to cancel the request during execution or  get periodic feedback about how the request is progressing. The `actionlib` package provides tools to create servers that execute long-running  goals that can be preempted. It also provides a client interface in  order to send requests to the server. 
>
> 在长时间运行的任务中，需要定时得到反馈和允许中途取消目标

### Action.msg

```c++
Goal

Feedback:传输过程值

Result
```

.action生成.msg

```c
rosrun actionlib_msgs genaction.py -o msg/ action/Averaging.action
```



### Creat Action Server

创建server，确定名字和回调函数

> /engineer_middleware/move_steps
>
> /namespace/server name

> 这回调函数怎么看哇

```c
as_(
      nh_, "move_steps", [this](auto&& PH1) { executeCB(std::forward<decltype(PH1)>(PH1)); }, false),
```

对比官方教程

```c
as_(
    nh_, name, boost::bind(&FibonacciAction::executeCB, this, _1), false),
```

#### More

设置多个回调函数，目标接受时和抢占回调函数

```c
as_(nh_, name, false),
action_name_(name)
{
//register the goal and feeback callbacks     
    as_.registerGoalCallback(boost::bind(&AveragingAction::goalCB, this)); as_.registerPreemptCallback(boost::bind(&AveragingAction::preemptCB, this));
```



### Creat Action Client

```c
action_client_("/engineer_middleware/move_steps", true)
{
  ROS_INFO("Waiting for middleware to start.");
  action_client_.waitForServer();
  ROS_INFO("Middleware started.");
```

创建server对应的client然后等待开启

官方教程的senderGoal，在团队代码的motion.h里

## engineer_arm_config

### config



### launch

#### engineer_moveit_controller_manager.launch.xml

把moveit_controller_manager，和ros_controllers.yaml加载到参数服务器

也就是仿真用的hw

#### fake_moveit_controller_manager.launch.xml

也是仿真用的？

#### load_move_group.launch

引用两个launch

planning_context.launch

用来解算的东西合集

- 是否加载urdf，srdf
- 加载joint_limit.yaml文件
  - 设置最大加速度和速度
- 加载kinematics.yaml
  - 运动解算器的种类和超时时间

#### move_group.launch

> 里面也有planning_context.launch的相关东西，但是上面是false也就是不加载urdf那些

- move_group setting

> 把后面的东西都设置里，后面的直接拿着用

```
<arg name="pipeline" default="ompl"/>
<arg name="allow_trajectory_execution" default="true"/>
<arg name="fake_execution" default="false"/>
<arg name="max_safe_path_cost" default="1"/>
<arg name="jiggle_fraction" default="0."/>
==在规划阶段考虑到的干扰==
<arg name="publish_monitored_planning_scene" default="true"/>

<arg name="capabilities" default=""/>
<arg name="disable_capabilities" default=""/>

<arg name="load_robot_description" default="true"/>
```

- planning functionality
- trajectory execution fuctionality
- start the move_group action server





## engineer_middleware

## src

### engineer_middleware.cpp



```c
if (nh.hasParam("steps_list") && nh.hasParam("scenes_list"))
  {
    XmlRpc::XmlRpcValue steps_list;
    XmlRpc::XmlRpcValue scenes_list;
    nh.getParam("steps_list", steps_list);
    nh.getParam("scenes_list", scenes_list);
    ROS_ASSERT(steps_list.getType() == XmlRpc::XmlRpcValue::Type::TypeStruct);
    ROS_ASSERT(scenes_list.getType() == XmlRpc::XmlRpcValue::Type::TypeStruct);
    for (XmlRpc::XmlRpcValue::ValueStruct::const_iterator it = steps_list.begin(); it != steps_list.end(); ++it)
     //iterator迭代器，在可以在不了解集合内部数据结构的情况下直接遍历，这样可以使得集合内部的的数据不暴露
    {
      step_queues_.insert(
          std::make_pair(it->first, StepQueue(it->second, scenes_list, tf_, arm_group_, chassis_interface_, hand_pub_,card_pub_, gimbal_pub_, gpio_pub_)));
        //在遍历中间不断的传入<序号，stepQueue>
        //stepQueue包含序号，scence？tf缓存和arm_group和各种Publiser
    }
```



```c
std::unordered_map<std::string, StepQueue> step_queues_;
```



- 不断修正base link和odom坐标系误差

```c
middleware.run(ros::Time::now() - last);
```

<details>
void run(ros::Duration period)
{
  geometry_msgs::TransformStamped current;
  try
  {
    current = tf_.lookupTransform("map", "base_link", ros::Time(0));
    //不断发布最新base link 和map之间的相对坐标系给current
  }
  catch (tf2::TransformException& ex)
  {
    ROS_WARN("%s", ex.what());
    return;
  }
  // Transform xy error under map frame to velocity under chassis
  geometry_msgs::Vector3 error;
  error.x = goal_.pose.position.x - current.transform.translation.x;
  error.y = goal_.pose.position.y - current.transform.translation.y;
  try
  {
    tf2::doTransform(error, error, tf_.lookupTransform("base_link", "map", ros::Time(0)));
  }
  catch (tf2::TransformException& ex)
  {
    ROS_WARN("%s", ex.what());
  }
  double roll, pitch, yaw_current, yaw_goal;
  quatToRPY(current.transform.rotation, roll, pitch, yaw_current);
  quatToRPY(goal_.pose.orientation, roll, pitch, yaw_goal);
  error_yaw_ = angles::shortest_angular_distance(yaw_current, yaw_goal);
  geometry_msgs::Twist cmd_vel_{};
  cmd_vel_.linear.x = pid_x_.computeCommand(error.x, period);
  cmd_vel_.linear.y = pid_y_.computeCommand(error.y, period);
  cmd_vel_.angular.z = pid_yaw_.computeCommand(error_yaw_, period);
  ;
  vel_pub_.publish(cmd_vel_);
  error_pos_ = std::abs(error.x) + std::abs(error.y);
  error_yaw_ = std::abs(error_yaw_);
}</details>


## include

### middleware.h

```
const actionlib::SimpleActionServer<rm_msgs::EngineerAction>::GoalConstPtr& goal
```

bianlizhegezhizhen

```
step_queue
```

run()

### step_queue.h

将step,scenes,tf和其他信息加载到queue_中。



### step.h

- 在step类的构造函数中对

```
const XmlRpc::XmlRpcValue& step
```

这个step成员进行不断读取，例如

```
if (step.hasMember("arm"))
    {
      if (step["arm"].hasMember("joints"))
        arm_motion_ = new JointMotion(step["arm"], arm_group);
      else
        arm_motion_ = new EndEffectorMotion(step["arm"], arm_group, tf);
    }
```

如果有相对应的成员值，就开对应motion

- 在bool move()中

把构造函数中加载的motion返回的值来判断，看是否都完成目标了

- 在stop()中

停止所有可执行motion：

arm，hand，chassis

- 在isFinish()中

  判断是否完成，和move()中方法一样

- ```c
  void deleteScene(std::vector<std::string> object_id)
  ```

  

### motion.h

- MotionBase

  ```
  virtual bool move() = 0;
  virtual bool isFinish() = 0;
  virtual void stop() = 0;  
  ```

  定义一个运动需要的三个基本函数

  执行，判断完成，停止

  - MoveitMotionBase

    ```
    bool isFinish() override
    {
      if (isReachGoal())
        countdown_--;
      else
        countdown_ = 5;
      return countdown_ < 0;
    ```

    通过对父类的三个函数的重写和对MoveitInterface的引用来达到运动的目的

    还加了一个motion.h

    ```
    virtual bool isReachGoal() = 0;
    ```

    - EndEffectorMotion
      - 构造函数
        - 在motion这个类中间寻找看使用的是什么方法来记录位置
          - 四元素
          - 欧拉角
          - 笛卡尔坐标系
      - move（）
        - 设置目标位置的位置/方向
      - isReachGoal
        - 通过当前坐位置和目标位置的坐标运算看是否在容忍值之内来判断是否达到目标
    - JointMotion
      - 通过加载各个关节的值到target中然后执行
      - 通过逐个判断各个关节的位置和目标的差来判断是否到达位置

  - PublishMotion

    - HandMotion
    - GpioMotion
    - JointPositionMotion
    - GimbalMotion

  - ChassisMotion



### chassis_interface.h

```
nh_base_motion
```

ju bing cong .yaml limian na



base_link to map

error = 



#### TransformStamped.msg

> such as transfer matrix

- Header header
  - unit 32 seq
  - time stamp
  - string frame_id
- string child_frame_id
- Transform transform
  - Vector3 translation
    - float64 x
    - float64 y
    - float64 z
  - Quaternion rotation
    - float64 x
    - float64 y
    - float64 z
    - float64 w



```
tf2_ros::Buffer& tf_
```

tf_ store a transfer matrix, which translate base_link to map

current

- 

## config

### engineer.yaml

设置x,y,yaw的pid

### scenes_list.yaml

(对应Move it里面的Planning Scene ROS API

> https://ros-planning.github.io/moveit_tutorials/doc/planning_scene_ros_api/planning_scene_ros_api_tutorial.html

内容

> - Adding and removing objects into the world
> - Attaching and detaching objects to the robot

- 增减物品
- 抓取放下物品



- 兑换站的三种情况

- 大矿石岛的三种情况

- 小矿石岛的二种情况

- 空接矿石的三种情况

  ---

  （18*18 *18）

- 空接矿石（相对于末端link6）

- 正常矿石（相对于末端link6）

- 空接（相对于base_link）



1. 为什么会有这多情况
2. orientation为什么会有四个维度
3. attach仅仅是区分空不空接吗
4. 为什么dimension有一些是18* 18*18有些不是
5. 格式在哪里应用的



### step_list.yaml

设置慢，正常，快三个速度和加速度

设置基础动作组

完整动作组

- 加载场景
- 移动云台
- 打开吸盘
- 移动底盘
- 机械臂准备动作
- 机械臂前伸让取矿





# 运行

在节点所在的cpp中

```c
Middleware middleware(nh);
```

开始该类的构造函数

完成对参数文件的读取

```c
if (nh.hasParam("steps_list") && nh.hasParam("scenes_list"))
```

和step_queues_的加载

```c
step_queues_.insert(std::make_pair(it->first, StepQueue(it->second, scenes_list, tf_, arm_group_, chassis_interface_, hand_pub_,card_pub_, gimbal_pub_, gpio_pub_)));
```

最后开启action

```c
as_.start();
 //(actionlib::SimpleActionServer<rm_msgs::EngineerAction> as_;)
```

+++在节点cpp的while函数中一直维持底盘的坐标系监测

```c
middleware.run(ros::Time::now() - last);
```





# engineer_manual.cpp

```c
void EngineerManual::runStepQueue(const std::string& step_queue_name)
{
  rm_msgs::EngineerGoal goal;
  goal.step_queue_name = step_queue_name;
  if (action_client_.isServerConnected())
  {
    if (operating_mode_ == MANUAL)
      action_client_.sendGoal(goal, boost::bind(&EngineerManual::actionDoneCallback, this, _1, _2),
                              boost::bind(&EngineerManual::actionActiveCallback, this),
                              boost::bind(&EngineerManual::actionFeedbackCallback, this, _1));
    operating_mode_ = MIDDLEWARE;
  }
  else
    ROS_ERROR("Can not connect to middleware");
}

void EngineerManual::actionFeedbackCallback(const rm_msgs::EngineerFeedbackConstPtr& feedback)
{
  trigger_change_ui_->update("queue", feedback->current_step);
  if (feedback->total_steps != 0)
    time_change_ui_->update("progress", ros::Time::now(), ((double)feedback->finished_step) / feedback->total_steps);
  else
    time_change_ui_->update("progress", ros::Time::now(), 0.);
}
通常不寫弧度單位
void EngineerManual::actionDoneCallback(const actionlib::SimpleClientGoalState& state,
                                        const rm_msgs::EngineerResultConstPtr& result)
{
  ROS_INFO("Finished in state [%s]", state.toString().c_str());
  ROS_INFO("Result: %i", result->finish);
  operating_mode_ = MANUAL;
}
```



![在这里插入图片描述](https://img-blog.csdnimg.cn/20200528170819546.png#pic_center)











写.yaml配置轨迹控制器，通过urdf和launch文件给关节加载控制器。

通过IKfast插件



# Engineer.action

```c++
# goal
string step_queue_name
---
# result
bool finish
---
# feedback
uint8 total_steps
uint8 finished_step
string current_step
```



# 





# Chassis

加载middleware，chassis_interface，拿.yaml文件里面的pid进行初始化。会一直对底盘进行

run()

> 在client还未发goal时，current和goal都是x,所以保持为0

> motion.h
>
> ```
> bool move() override
>   {
>     interface_.setGoal(target_);
>     return true;
>   }
> ```
>
> 



> step.h
>
> ```
> if (chassis_motion_)
>   success &= chassis_motion_->move();
> 
> ```





> step_queue.h
>
> chassis_interface_.setCurrentAsGoal();



current change because lookupTransform????! 