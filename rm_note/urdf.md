# urdf

- 底盘坐标轴朝向
- 注意单位，（stl给的是mm，要转换成m）
- gazebo原点和fusion原点重合（对角线中心）
- 先出底盘，然后一个link一个link的加



# urdf编写流程

在编写urdf前，需要对实车的结构有一定了解，没有实车时可通过图纸或者向设计者了解 在出urdf时，必须要和机械组的组员面对面出，将每一个link载入，在rviz中观察偏移量以及朝向位置有没有出现错误，最后再在gazebo中检查惯量，确保一个link的stl以及参数都没有问题再出下一个link，直到全部正确为止



### 

# link

每一个link需要包含visual、collision、inertial三个标签

#### visual

在visual标签中载入机械组给出的stl模型 示例:

```
<visual>
   <origin xyz="0 0 0" rpy="0 0 0"/>
   <geometry>
       <mesh filename="package://rm_description/meshes/sentry/catapult.stl" scale="0.001 0.001 0.001"/>
   </geometry>
</visual>
```

其中origin xyz rpy中可调整stl模型相对于xyz轴或rpy轴的偏移量，这个调整仅提供视觉上的偏移，不会对实际仿真有影响，所以一般不会进行修改

> ​		scale="0.001 0.001 0.001
>
> 缩放比例，把stl文件的单位mm换成米？
> 

#### inertial

inertial标签中需要填入机械组给出的link

- 质量
- 质心
- 惯性矩阵（==要填入质心处的惯性矩阵==） 

```
<inertial>
    <mass value="451.658e-3"/>
    <origin xyz="-10.414e-3 -18.637e-3 -4.243e-3"/>   
    <inertia ixx="9.679e-4" ixy="-3.152e-4" ixz="-5857.572e-9" iyy="2.66e-3"
             iyz="46492.352e-9" izz="2.71e-3"/>
</inertial>
```

#### 

#### collision

simpler collision models are often used to reduce computation time

碰撞箱不应该直接用stl文件，而是简单的立体几何来减少计算时间

```
<collision>
    <origin xyz="0 0 0" rpy="0 0 0"/>
    <geometry>
        <mesh filename="package://rm_description/meshes/sentry/catapult.stl" scale="0.001 0.001 0.001"/>
    </geometry>
</collision>
```

### 

# joint

urdf中每两个link需要一个joint进行连接，joint要有子link以及父link

```
<parent link="base_link"/>
<child link="catapult_link"/>
```



#### joint类型

joint常用的类型有平动以及转动，清楚joint的运动情况，选择正确的关节类型 转动关节: 

- continous 		没有限位的转动关节，可以绕单轴无限旋转 
- revolute 		   有限位的转动关节，有一定的旋转角度限制 平动关节: 
- prismatic 		 平动关节，沿某一轴滑动的关节 
- fixed      			固定关节:不允许运动的关节 示例:

```
<joint name="drive_wheel_joint" type="revolute">
```

#### 

#### axis

设置关节围绕哪一个轴运动

```
<axis xyz="0 1 0"/>
```



#### origin

joint中写入的偏移量是指子link的坐标系到父link坐标系的偏移量，这项数据需要机械组给出并且要测量准确 示例:

```
<origin xyz="0.7e-3 -77e-3 -49.8e-3" rpy="0 0 0"/>
```







- ## stl规范

  - 机械组给出的stl不能超过2M

  - 有电机操纵的部分需要单独写一个link并出stl（可以把电机当成joint来看） 跟随它运动的部分包含控制其运动的电机归属于这个的link

- ## 数据规划

  - 机械所给出的stl属于高度简化的模型，但每一个link的质量、质心以及惯性矩阵是需要未简化前的 填入的所有数据都要换算成国际单位m



- ## 坐标系规范

  - base_link的坐标系出在几何中心，其他link的坐标系出在与父link连接处









```
common:
  speed:
    slowly: &SLOWLY
      speed: 0.1
      accel: 0.1
      timeout: 10.
    normally: &NORMALLY
      speed: 0.4
      accel: 0.4
      timeout: 6.
    quickly: &QUICKLY
      speed: 0.7
      accel: 0.7
      timeout: 4.

  tolerance:
    normal_tolerance: &NORMAL_TOLERANCE
      tolerance_joints: [ 0.06, 0.006, 0.006, 0.01, 0.01, 0.01 ]
    bigger_tolerance: &BIGGER_TOLERANCE
      tolerance_joints: [ 0.01, 0.01, 0.01, 0.2, 0.2, 0.2 ]

  gripper:
    open_gripper: &OPEN_GRIPPER
      state: false
    close_gripper: &CLOSE_GRIPPER
      state: true

  arm:
    normal_home: &NORMAL_HOME
      joints: [ 0., 0.02,  0.007, -0.1566, 0., 3.14 ]
      common:
        <<: *NORMALLY
      tolerance:
        <<: *NORMAL_TOLERANCE
    stored_home: &STORE_HOME
      joints: [ 0.26, 0.01, 0., -0.348, 0., 3.14 ]
      common:
        <<: *NORMALLY
      tolerance:
        <<: *NORMAL_TOLERANCE
    pick_wait:  &PICK_WAIT
      frame: base_link
      position: [ 0.283, 0.225, 0.836 ]
      rpy: [ -2.15,-1.560,-2.538 ]
      common:
        <<: *NORMALLY

  gimbal:
    sky_gimbal: &SKY_POS
      frame: gimbal_base
      position: [ -0.2, -0.0008, 1 ]
    back_gimbal: &BACK_POS
      frame: gimbal_base
      position: [ -2 ,-0.1, -15 ]
    big_stone_gimbal: &BIG_STONE_POS
      frame: gimbal_base
      position: [ 2, 0.01, 0.1 ]
    walking_pos: &WALKING_POS
      frame: gimbal_base
      position: [ -2 ,0.01, -15 ]

steps_list:
  ################### COMMON STEP####################
  DELETE_SCENE:
    - step: "delete_scene"
  ################### GIMBAL ####################
  SKY_GIMBAL:
    - step: "open gripper"
      gimbal:
        <<: *SKY_POS
  WALK_GIMBAL:
    - step: "open gripper"
      gimbal:
        <<: *WALKING_POS

  BIG_STONE_GIMBAL:
    - step: "open gripper"
      gimbal:
        <<: *BIG_STONE_POS

  BACK_GIMBAL:
    - step: "gimbal"
      gimbal:
        <<: *BACK_POS

  ################### GRIPPER ####################
  OPEN_GRIPPER:
    - step: "open gripper"
      gripper:
        <<: *OPEN_GRIPPER

  CLOSE_GRIPPER:
    - step: "close gripper"
      gripper:
        <<: *CLOSE_GRIPPER
  ################### CHASSIS ####################
  CHASSIS_FORWARD:
    - step: "forward"
      chassis:
        frame: base_link
        position: [ 0.3, 0. ]
        yaw: 0.0
        chassis_tolerance_position_: 0.025
        chassis_tolerance_angular_: 0.1

  CHASSIS_BACKWARD:
    - step: "backward"
      chassis:
        frame: base_link
        position: [ -0.1, 0. ]
        yaw: 0.0
        chassis_tolerance_position_: 0.025
        chassis_tolerance_angular_: 0.1

  CHASSIS_LEFT:
    - step: "left"
      chassis:
        frame: base_link
        position: [ 0., 0.5 ]
        yaw: 0.0
        chassis_tolerance_position_: 0.1
        chassis_tolerance_angular_: 0.01

  CHASSIS_RIGHT:
    - step: "right"
      chassis:
        frame: base_link
        position: [ 0., -0.5 ]
        yaw: 0.0
        chassis_tolerance_position_: 0.1
        chassis_tolerance_angular_: 0.01

  CHASSIS_ROTATE:
    - step: "rotate"
      chassis:
        frame: base_link
        position: [ 0., 0. ]
        yaw: 3.14
        chassis_tolerance_position_: 0.1
        chassis_tolerance_angular_: 0.01

  ################### CHASSIS ####################
  ARM_RIGHT1:
    - step: "forward"
      servo:
        frame: tools_link
        position: [ 0., 0.,   0. ]
        orientation: [ 0., 0., 0. ]
        servo_tolerance_position: 0.06
        servo_tolerance_angular: 0.1

  ################### HOME ####################

  HOME0:
    - step: "home with no stone"
      arm:
        <<: *NORMAL_HOME
  HOME1:
    - step: "home with one stone"
      arm:
        <<: *NORMAL_HOME
  HOME2:
    - step: "home with two stone"
      arm:
        <<: *NORMAL_HOME


  ################### GROUND ####################
  GROUND_STONE0:
    - step: "ground"
      arm:
        joints: [ 0., 0.02,  0.007, -0.05, 0., 0. ]
        common:
          <<: *NORMALLY
        tolerance:
          <<: *NORMAL_TOLERANCE

  ###################    SKY_BIG_ISLAND    ####################
  SKY_BIG_ISLAND0:
    - step: "arm_wait"
      arm:
        <<: *PICK_WAIT
  SKY_BIG_ISLAND00:
    - step: "up"
      arm:
        joints: [ 0.435, 0.266, 0.007, -1.496, -0.005, 0 ]
        common:
          <<: *NORMALLY
        tolerance:
          <<: *NORMAL_TOLERANCE
  SKY_BIG_ISLAND000:
    - step: "arm_wait"
      arm:
        <<: *PICK_WAIT
    ###################    BIG_ISLAND    ####################
    ###################    MID    ################
  BIG_ISLAND0:
    - step: "arm_wait"
      arm:
        <<: *PICK_WAIT
  BIG_ISLAND00:
    - step: "add scenes"
      scene_name: MID_BIG_ISLAND
    - step: "arm_ready"
      arm:
        frame: base_link
        position: [ 0.743, 0.007, 0.748 ]
        rpy: [ -0.555,1.539, 2.581 ]
        common:
          <<: *NORMALLY
        tolerance_position: 0.001
    - step: "change gripper state"
    - step: "delete_scene"
  BIG_ISLAND000:
    - step: "get stone"
      arm:
        joints: [0.299, 0.266, 0.007, -1.526, -0.005, 3.119]
        common:
          <<: *NORMALLY
        tolerance:
          <<: *NORMAL_TOLERANCE
    - step: "up"
      arm:
        joints: [0.435, 0.266, 0.007, -1.496, -0.005, 3.119]
        common:
          <<: *NORMALLY
        tolerance:
          <<: *NORMAL_TOLERANCE
    - step: "reversal"
      arm:
        joints: [0.433, 0.042, 0.007, -1.502, -0.001, 0.014]
        common:
          <<: *NORMALLY
        tolerance:
          <<: *BIGGER_TOLERANCE
    - step: "move to wait "
      arm:
        frame: base_link
        position: [ 0.283, 0.225, 0.836 ]
        rpy: [ -2.15,-1.560,-2.538 ]
        common:
          <<: *SLOWLY
  ###################    SMALL_ISLAND    ####################
  LF_SMALL_ISLAND0:
    - step: "arm_wait"
      arm:
        <<: *PICK_WAIT
      gripper:
        <<: *CLOSE_GRIPPER
  LF_SMALL_ISLAND00:
    - step: "add scenes"
      scene_name: MID_SMALL_ISLAND
    - step: "arm_ready"
      arm:
        frame: base_link
        position: [ 0.507, 0.265, 0.722 ]
        rpy: [ -0.649,1.506, 2.485 ]
        common:
          <<: *NORMALLY
        tolerance_position: 0.001
    - step: "change gripper state"
      gripper:
         <<: *OPEN_GRIPPER
    - step: "delete_scene"
  LF_SMALL_ISLAND000:
    - step: "move arm down to gain stone"
      arm:
        frame: base_link
        position: [ 0.507, 0.265, 0.6401 ]
        rpy: [ -0.750,1.510, 2.383 ]
        common:
          <<: *QUICKLY
        tolerance_position: 0.01
    - step: "move arm up with stone and go right"
      arm:
        frame: base_link
        position: [ 0.504, 0.265, 0.811 ]
        rpy: [ -0.330, 1.441, 2.809 ]
        common:
          <<: *NORMALLY
        tolerance_position: 0.001
    - step: "go right"
      arm:
        frame: base_link
        position: [ 0.506, 0.209, 0.812 ]
        rpy: [ -0.331, 1.439, 2.815 ]
        common:
          <<: *NORMALLY
        tolerance_position: 0.001
    - step: "reversal"
      arm:
        frame: base_link
        position: [ 0.506, 0.209, 0.811 ]
        rpy: [ 0.042,-1.443, -0.033 ]
        common:
          <<: *NORMALLY
        tolerance_position: 0.001
  LF_SMALL_ISLAND0000:
    - step: "move to wait "
      arm:
        frame: base_link
        position: [ 0.283, 0.225, 0.836 ]
        rpy: [ -2.15,-1.560,-2.538 ]
        common:
          <<: *NORMALLY
    ###################    MID    ################
  MID_SMALL_ISLAND0:
    - step: "arm_wait"
      arm:
        <<: *PICK_WAIT
  MID_SMALL_ISLAND00:
    - step: "add scenes"
      scene_name: MID_SMALL_ISLAND
    - step: "arm_ready"
      arm:
        frame: base_link
        position: [ 0.507, 0.008, 0.723 ]
        rpy: [ -0.498,1.515, 2.639 ]
        common:
          <<: *NORMALLY
        tolerance_position: 0.001
    - step: "change gripper state"
    - step: "backward"
      chassis:
        frame: base_link
        position: [ -0.09, 0. ]
        yaw: 0.0
        chassis_tolerance_position_: 0.025
        chassis_tolerance_angular_: 0.1
    - step: "delete_scene"
  MID_SMALL_ISLAND000:
    - step: "up"
      arm:
        joints: [0.2540476892306656, 0.03155902847607778, 0.005682599990380954, -1.5497683296037037, 0.09603447921700581, 3.1295657735691975]
        common:
          <<: *SLOWLY
        tolerance:
          <<: *BIGGER_TOLERANCE
    - step: "up"
      arm:
        joints: [ 0.2550476892306656, 0.03255902847607778, 0.005682599990380954, -1.5497683296037037, 0.09603447921700581, 3.1295657735691975 ]
        common:
          <<: *SLOWLY
        tolerance:
          <<: *BIGGER_TOLERANCE
    - step: "up"
      arm:
        joints: [ 0.2560476892306656, 0.03355902847607778, 0.005682599990380954, -1.5497683296037037, 0.09603447921700581, 3.1295657735691975 ]
        common:
          <<: *SLOWLY
        tolerance:
          <<: *BIGGER_TOLERANCE
    - step: "up"
      arm:
        joints: [ 0.2570476892306656, 0.03455902847607778, 0.005682599990380954, -1.5497683296037037, 0.09603447921700581, 3.1295657735691975 ]
        common:
          <<: *SLOWLY
        tolerance:
          <<: *BIGGER_TOLERANCE
    - step: "up"
      arm:
        joints: [ 0.2580476892306656, 0.03555902847607778, 0.005682599990380954, -1.5497683296037037, 0.09603447921700581, 3.1295657735691975 ]
        common:
          <<: *SLOWLY
        tolerance:
          <<: *BIGGER_TOLERANCE
    - step: "up"
      arm:
        joints: [ 0.2590476892306656, 0.03655902847607778, 0.005682599990380954, -1.5497683296037037, 0.09603447921700581, 3.1295657735691975 ]
        common:
          <<: *SLOWLY
        tolerance:
          <<: *BIGGER_TOLERANCE
    - step: "up"
      arm:
        joints: [ 0.26000476892306656, 0.03755902847607778, 0.005682599990380954, -1.5497683296037037, 0.09603447921700581, 3.1295657735691975 ]
        common:
          <<: *SLOWLY
        tolerance:
          <<: *BIGGER_TOLERANCE
    - step: "up"
      arm:
        joints: [ 0.2610476892306656, 0.03855902847607778, 0.005682599990380954, -1.5497683296037037, 0.09603447921700581, 3.1295657735691975 ]
        common:
          <<: *SLOWLY
        tolerance:
          <<: *BIGGER_TOLERANCE

    ###################    RT    ################
  RT_SMALL_ISLAND0:
    - step: "arm_wait"
      arm:
        <<: *PICK_WAIT
  RT_SMALL_ISLAND00:
    - step: "add scenes"
      scene_name: MID_SMALL_ISLAND
    - step: "arm_ready"
      arm:
        frame: base_link
        position: [ 0.507, -0.252, 0.718 ]
        rpy: [ -0.516, 1.498, 2.615 ]
        common:
          <<: *NORMALLY
        tolerance_position: 0.001
    - step: "change gripper state"
    - step: "delete_scene"
  RT_SMALL_ISLAND000:
    - step: "move arm down to gain stone"
      arm:
        frame: base_link
        position: [ 0.504, -0.250, 0.640 ]
        rpy: [ -0.524,1.502, 2.614 ]
        common:
          <<: *QUICKLY
        tolerance_position: 0.01
    - step: "move arm up with stone and go right"
      arm:
        frame: base_link
        position: [ 0.504, -0.250, 0.812 ]
        rpy: [ -0.334,1.465, 2.802 ]
        common:
          <<: *NORMALLY
        tolerance_position: 0.001
    - step: "go right"
      arm:
        frame: base_link
        position: [ 0.505, -0.290, 0.811 ]
        rpy: [ -0.328,1.464, 2.803 ]
        common:
          <<: *NORMALLY
        tolerance_position: 0.001
    - step: "reversal"
      arm:
        frame: base_link
        position: [ 0.506, -0.295, 0.806 ]
        rpy: [ -0.137,-1.467, 0.108 ]
        common:
          <<: *SLOWLY
        tolerance_position: 0.001
  RT_SMALL_ISLAND0000:
    - step: "move to wait "
      arm:
        frame: base_link
        position: [ 0.283, 0.225, 0.836 ]
        rpy: [ -2.15,-1.560,-2.538 ]
        common:
          <<: *NORMALLY
      ###################    SMALL_ISLAND    ####################

      ###################   JQ   ####################
```