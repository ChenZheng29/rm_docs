# TF2

```c
$ rosrun tf2_tools view_frames.py
```

听5s的tf数据然后生成pdf

```
evince frames.pdf
```

可以看到生成的图

```
rosrun tf tf_echo [reference_frame] [target_frame]
```

看两个坐标系的相对变化





# Writing a tf2 broadcaster

msg的坐标值来源于头文件的引用

```c
#include <turtlesim/Pose.h>
```

```c
static tf2_ros::TransformBroadcaster br;
  geometry_msgs::TransformStamped transformStamped;
  
  transformStamped.header.stamp = ros::Time::now();
  transformStamped.header.frame_id = "world";
  transformStamped.child_frame_id = turtle_name;
  transformStamped.transform.translation.x = msg->x;
  transformStamped.transform.translation.y = msg->y;
  transformStamped.transform.translation.z = 0.0;
  tf2::Quaternion q;
  q.setRPY(0, 0, msg->theta);
  transformStamped.transform.rotation.x = q.x();
  transformStamped.transform.rotation.y = q.y();
  transformStamped.transform.rotation.z = q.z();
  transformStamped.transform.rotation.w = q.w();

  br.sendTransform(transformStamped);
```

上图中的msg消息如下

### TransformStamped Message

（tf坐标系的专用消息格式，用来实现从父世界到子世界的转变）

- [std_msgs/Header](http://docs.ros.org/en/lunar/api/std_msgs/html/msg/Header.html) header

  （元数据的标注类型）

  - uint32 seq

    （sequence ID序列号？）

  - time stamp

    ​	（时间戳，一般获取当前时间ros::Time::now();）

  - string frame_id

    ​	（父世界的id）

- string child_frame_id

  （子世界id）

- [geometry_msgs/Transform](http://docs.ros.org/en/lunar/api/geometry_msgs/html/msg/Transform.html) transform

  （用于自由空间内的两个坐标系的转换）

  - [geometry_msgs/Vector3](http://docs.ros.org/en/lunar/api/geometry_msgs/html/msg/Vector3.html) translation

    （仅表示一个向量，对于转换没有什么用）

    > \# This represents a vector in free space. 
    > \# It is only meant to represent a direction. Therefore, it does not
    > \# make sense to apply a translation to it (e.g., when applying a 
    > \# generic rigid transformation to a Vector3, tf2 will only apply the
    > \# rotation). If you want your data to be translatable too, use the
    > \# geometry_msgs/Point message instead.（）
    >
    > - float64 x
    > - float64 y
    > - float64 z

  - [geometry_msgs/Quaternion](http://docs.ros.org/en/lunar/api/geometry_msgs/html/msg/Quaternion.html) rotation

  - 四元数：简单的超复数，可表示为：a+bi+cj+dk

    - float64 x
    - float64 y
    - float64 z
    - float64 w



# Writing a tf2 listener

tf发布a相对于b的位置：

```c
transformStamped = tfBuffer.lookupTransform("turtle2", "turtle1",ros::Time(0));
```

#### ros::time(0)和ros::Time:now()的区别

- time(0)是最新一次缓存区有可发布数据的时间
- time::now()是当前时间
  - 但是当前时间并不一定有数据可以发布所以会传lookupTransform的第四个值:ros::Duration(3.0)即容忍时间，最多可以等待的时间

而这两个时间可能也只差了few milliseconds，所以在实际中多用time（0）



#### Time advanced 

```c
try{
    ros::Time now = ros::Time::now();
    ros::Time past = now - ros::Duration(5.0);
    transformStamped = tfBuffer.lookupTransform("turtle2", now,
                             "turtle1", past,
                             "world", ros::Duration(1.0));
```

可以获得两个坐标系在一段时间前的相对变化

# Adding a frame

可以和上面的boardcaster比较一下

区别很小只有父，子的坐标名字和x,y,z的值有区别。

```c
tf2_ros::TransformBroadcaster tfb;
  geometry_msgs::TransformStamped transformStamped;


  transformStamped.header.frame_id = "turtle1";
  transformStamped.child_frame_id = "carrot1";
  transformStamped.transform.translation.x = 0.0;
  transformStamped.transform.translation.y = 2.0;
  transformStamped.transform.translation.z = 0.0;
  tf2::Quaternion q;
        q.setRPY(0, 0, 0);
  transformStamped.transform.rotation.x = q.x();
  transformStamped.transform.rotation.y = q.y();
  transformStamped.transform.rotation.z = q.z();
  transformStamped.transform.rotation.w = q.w();
```

