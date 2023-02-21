# Moveit！

### 现状：

在编写动作组时，自由度低，仅有两种方式

- 指定末端相对于固定坐标系的位姿
- 指定各个joint的值

### 由此出现的问题

- 动作之间衔接僵硬
- 在复杂场景下解算成功率低

### 为了解决上述问题，采取方案

- 完善rm_engineer的轨迹部分，提高编写动作的自由度

### 预期效果

- 可以根据不同场景，选择不同的编写方式
  - 简单（单个动作，无碰撞风险）：指定各个joint值
  - 中等（多个动作衔接）：由简单动作给出轨迹点，然后自定义插值位置，速度，加速度
  - 拓展：引入示教（自动记录轨迹信息）并且后期有可视化界面来方便调整（参考Altas那篇知乎文章）

### 任务规划

- 了解现阶段队伍使用的方法，接口。

  - joint           

    ```
    interface_.setJointValueTarget(target_);
    ```

  - endeffort  

    ```
    interface_.setPoseTarget(target_);
    ```

  - 调用MoveGroupInterface来控制电机达到直接的角度或者末端达到指定的位姿

  - 现有的speed和accel是一个缩放因子，具体的位置，速度和加速度并未接管

  - 在接口里用move()实现目标位置的规划和执行

- 找MoveGroupInterface相关介绍

  - 得到了相关可以调用的函数，寻找到几个可能有用的

- 在Moveit Tutorials中找运动规划的相关内容

  - OMPL（The Open Motion Planning Library）
    - config
      - longest_valid_segment_fraction(最长有效碰撞段)
        - 将连续的边缘线段化用于检测碰撞，将 longest_valid_segment_fraction设置得太低，碰撞检查/运动规划将非常缓慢。  设置得太高会错过小或窄物体周围的碰撞。  

## 相关知识

- 四元数
  - 欧拉角的缺点
    - 不易在任意方向的旋转轴插值
    - 万向节死锁？
    - 旋转次序无法确定
  - 轴角的引入
    - 为了解决欧拉角缺点1
    - [x,y,z,theta]前三旋转轴矢量，后表示转角
      - 缺点：不能够进行简单的插值

- 四元数表达
  - [x,y,z,w]
  - 优点：在四维空间中表达，方便归一化和插值，确定旋转次序
- 轨迹表达moveit_msgs/RobotTrajectoryMessage
  - trajectory_msgs/JointTrajectory joint_trajectory
    - Header header
      - uint32 seq
      - time stamp
      - string frame_id
    - string[] joint_names
    - JointTrajectoryPoint[] points
      - float64[] positions
      - float64[] velocities
      - float64[] accelerations
      - float64[] effort
      - duration time_from_start
  - trajectory_msgs/MultiDOFJointTrajectory multi_dof_joint_trajectory
    - [trajectory_msgs/MultiDOFJointTrajectoryPoint[\]](http://docs.ros.org/en/noetic/api/trajectory_msgs/html/msg/MultiDOFJointTrajectoryPoint.html) points
      - [geometry_msgs/Transform[\]](http://docs.ros.org/en/noetic/api/geometry_msgs/html/msg/Transform.html) transforms
        - [geometry_msgs/Vector3](http://docs.ros.org/en/noetic/api/geometry_msgs/html/msg/Vector3.html) translation
        - [geometry_msgs/Quaternion](http://docs.ros.org/en/noetic/api/geometry_msgs/html/msg/Quaternion.html) rotation
      - [geometry_msgs/Twist[\]](http://docs.ros.org/en/noetic/api/geometry_msgs/html/msg/Twist.html) velocities
        - [geometry_msgs/Vector3](http://docs.ros.org/en/noetic/api/geometry_msgs/html/msg/Vector3.html) linear
        - [geometry_msgs/Vector3](http://docs.ros.org/en/noetic/api/geometry_msgs/html/msg/Vector3.html) angular
      - [geometry_msgs/Twist[\]](http://docs.ros.org/en/noetic/api/geometry_msgs/html/msg/Twist.html) accelerations
      - duration time_from_start

### 轨迹规划

轨迹：机械手臂末端点或参做点的位置、速度、加速度对时间的历程

目的：给定起始点和中点，定义中间点，得到关节的驱动数值的连续平滑曲线

理想轨迹：位置连续，速度连续（控制加速度变化，保证关节负担，以此保证精度）

> 笛卡尔坐标系（直角空间坐标系）和关节坐标系之间的相互转换对应正逆运动学的解算

#### 关节坐标系流程图

![img](https://img-blog.csdn.net/20180908093354241?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2FpYzE5OTk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

1）知道位置，就能通过逆向运动学（IK），反向求解出有六个自由度的机械臂它六个转轴需要的角度。

2）拟合曲线，就是轨迹规划啦，得到这些点连续平滑的函数表示（多种）。就是找函数关系（横纵轴的映射）。

3）再借由顺向运动学（FK）验证这个函数能否经过我们在（1）定义好的点，使得机械臂通过我们规定的点，达到我们想要的位置，以此测试函数是否正确。

4）检验可行性，比如这个轨迹是不是可能会撞到现实空间中的某些东西之类的。

#### 笛卡尔坐标系流程图

![img](https://img-blog.csdn.net/20180908095515806?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2FpYzE5OTk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)



1）拟合曲线，直接做轨迹规划，找出他们的函数式。

2）逆向求解出达到这个位置需要机械臂的每个转轴转动的度数，检验求解出的角度是否超出了机械臂转动角度的最大范围等。

3）检验可行性，检测该轨迹在现实中是否会碰到障碍物。

#### 两种坐标系比较

- 关节坐标系优点：计算量小
- 笛卡尔坐标系：更加直观，在第二步就获得轨迹，但是对算力要求高，需要计算大量点的逆解

> 两个一个是生成连续的joint角度值，一个是生成连续的空间曲线

#### 多项式拟合曲线

多项式，每一次就是多项式空间中的一个基，然后多个基长成空间

对于轨迹规划，需要保证位移，速度，加速度相对于时间是连续的，所以最小需要三次多项式，（对于三次多项式，可以保证连续，但是可能会产生突变，突变对于实际有无影响就要考虑电机硬件）

#### Moveit！生成轨迹特点

默认为等距轨迹，需要结合硬件看是否要改成等时的。

- 理解move_group.plan()后moveit如何与ompl进行的交互，然后ompl如何完成的算法并将结果返回给moveit的。
- Moveit 中产生的trajectory Data 只能在程序内部看到，并不能通过话题、消息来看到，其是通过 follow_joint_trajectory 进行封装之后，才可以通过相应的话题进行查看。
- ros controller  是采用五次多项式对机械臂的轨迹点进行插补的。Moveit中的得到的规划路径消息是根据轨迹用TOPP计算出速度加速度，也就是说如果现在有一条轨迹的position，也是可以用TOPP算出这条轨迹的速度和加速度值，然后自己同话题发布出去。

## 可能用到的

MoveGroupInterface

```
setNumPlanningAttempts（）
```

> 用来寻找最短路径的次数，默认为1

可能可以通过在没有找到的之前一直找来达到解算成功的目的，因为在调试中出现现象：同一个情况下规划多次就可以成功，而且报错显示规划所用到的时间很短很短

拓展：有没有可能，规划是否成功的依据不是找到最短路径？



```
setPlannerId (const std::string &planner_id) 
```

换规划器试试？



```
setApproximateJointValueTarget (const geometry_msgs::Pose &eef_pose, const std::string &end_effector_link="")
```

在规定目标不能实现的时候自动采用近似值



```
getTrajectoryConstraints () 
```

可以看看已有的约束



在config中将

longest_valid_segment_fraction设置调高（目前是0.005，意思是状态空间的0.005以上就不检查）



在轨迹中加入中间点，利于解算

- 将两个点之间的路径规划出来然后将轨迹合并，再重新插值？

plan()用来得到当前值到目标值的轨迹信息

引入中间点，然后在起始点plan一次到中点，在中点plan一次到目标点，得到两次的轨迹，然后相加轨迹采用下面：

```
moveit::core::MoveItErrorCode execute(const moveit_msgs::RobotTrajectory& trajectory);
```

区别于

```
moveit::core::MoveItErrorCode execute(const Plan& plan);
```

输入值分别是trajectory和plan，plan包含了起始位姿和规划时间

问题是：只需要轨迹的位置信息，速度和加速度需要重新规划



这个好像也可以试试，可以加入中间点

```
double moveit::planning_interface::MoveGroupInterface::computeCartesianPath 	
```

