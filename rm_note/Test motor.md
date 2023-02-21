# Test motor

- config

  - rm_hw.yaml

  - ```yaml
    bus:
      - can0
    loop_frequency: 1000
    cycle_time_error_threshold: 0.001
    
    actuators:
      joint1_motor:
        bus: can0
        id: 0x201
        type: rm_3508
        lp_cutoff_frequency: 60
    ```

  - controllers.yaml

  - ```yaml
    controllers:
      joint_state_controller:
        type: joint_state_controller/JointStateController
        publish_rate: 50
      joint1_position_controller:
        type: effort_controllers/JointPositionController
        joint: joint1
        pid:
          {
            p: 30,
            i: 0.0,
            d: 0.8,
            i_clamp_max: 1,
            i_clamp_min: -1,
            antiwindup: true,
          }
      joint1_velocity_controller:
        type: effort_controllers/JointVelocityController
        joint: joint1
        pid: { p: 0.8, i: 0, d: 0.0, i_max: 0.0, i_min: 0.0, antiwindup: true }
    ```

- launch

  - load_rm_hw.launch

  - ```xml
    <launch>
        <!-- push robot_description to factory and spawn robot in gazebo -->
        <param name="robot_description"
               command="$(find xacro)/xacro $(find rm_controls_tutorials)/urdf/rmrobot.urdf.xacro"/>
        <rosparam file="$(find rm_hw)/config/actuator_coefficient.yaml" command="load" ns="rm_hw"/>
        <rosparam file="$(find rm_controls_tutorials)/config/rm_hw.yaml" command="load" ns="rm_hw"/>
        <node name="rm_hw" pkg="rm_hw" type="rm_hw" respawn="false" clear_params="true"/>
        <node name="robot_state_publisher" pkg="robot_state_publisher" type="robot_state_publisher"/>
    </launch>
    ```

  - load_controllers.launch

  - ```xml
    <launch>
        <rosparam file="$(find rm_controls_tutorials)/config/controllers.yaml" command="load"/>
        <!-- load the controllers -->
        <node name="controller_loader" pkg="controller_manager" type="controller_manager"
              respawn="false" output="screen"
              args="load
              controllers/joint_state_controller
              controllers/joint1_position_controller
              controllers/joint1_velocity_controller
    "/>
    </launch>
    ```