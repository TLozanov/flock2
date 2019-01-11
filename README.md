# `flock2`

`flock2` can fly a swarm of [DJI Tello](https://store.dji.com/product/tello) drones.
`flock2` is built on top of [ROS2](https://index.ros.org/doc/ros2/),
 [flock_vlam](https://github.com/ptrmu/flock_vlam)
 and [tello_ros](https://github.com/clydemcqueen/tello_ros).

## TF Tree

* `map` is the world frame
* `base_link` is the Tello
* `camera_frame` is the front-facing camera on the Tello

## Nodes

### flock_base

Manages flight states and drone actions.

#### Subscribed topics

* `~joy` [sensor_msgs/Joy](http://docs.ros.org/api/sensor_msgs/html/msg/Joy.html)
* `~tello_response` tello_msgs/TelloResponse
* `~flight_data` tello_msgs/FlightData

#### Published topics

* `~cmd_vel` [geometry_msgs/Twist](http://docs.ros.org/api/geometry_msgs/html/msg/Twist.html)
* `~start_mission` [std_msgs/Empty](http://docs.ros.org/api/std_msgs/html/msg/Empty.html)
* `~stop_mission` [std_msgs/Empty](http://docs.ros.org/api/std_msgs/html/msg/Empty.html)

#### Published services

* `~tello_command` tello_msgs/TelloCommand

### filter

Uses a Kalman filter to estimate pose and velocity.
The estimate is published on the `filtered_odom` topic.

Publishes a transform from `map` to `base_link`.

Publishes the estimated path during a mission.

#### Subscribed topics

* `~start_mission` [std_msgs/Empty](http://docs.ros.org/api/std_msgs/html/msg/Empty.html)
* `~stop_mission` [std_msgs/Empty](http://docs.ros.org/api/std_msgs/html/msg/Empty.html)
* `~camera_pose` [geometry_msgs/PoseWithCovarianceStamped](http://docs.ros.org/api/geometry_msgs/html/msg/PoseWithCovarianceStamped.html)

#### Published topics

* `~filtered_odom` [nav_msgs/Odometry](http://docs.ros.org/api/nav_msgs/html/msg/Odometry.html)
* `~estimated_path` [nav_msgs/Path](http://docs.ros.org/api/nav_msgs/html/msg/Path.html)
* `/tf` [tf2_msgs/TFMessage](http://docs.ros.org/api/tf2_msgs/html/msg/TFMessage.html)

### flock_simple_path

Generates a continuous flight path, publishes it, and follows it using a P controller.
Send `start_mission` to run the mission, and `stop_mission` to stop it.

#### Subscribed topics

* `~start_mission` [std_msgs/Empty](http://docs.ros.org/api/std_msgs/html/msg/Empty.html)
* `~stop_mission` [std_msgs/Empty](http://docs.ros.org/api/std_msgs/html/msg/Empty.html)
* `/tf` [tf2_msgs/TFMessage](http://docs.ros.org/api/tf2_msgs/html/msg/TFMessage.html)

#### Published topics

* `~planned_path` [nav_msgs/Path](http://docs.ros.org/api/nav_msgs/html/msg/Path.html)
* `~cmd_vel` [geometry_msgs/Twist](http://docs.ros.org/api/geometry_msgs/html/msg/Twist.html)

## Installation

### 1. Set up your Linux environment

Set up a Ubuntu 18.04 box or VM. This should include ffmpeg 3.4.4 and OpenCV 3.2.

### 2. Set up your Python environment

Use your favorite Python package manager to set up Python 3.6+ and the following packages:

* numpy 1.15.2
* transformations 2018.9.5

### 3. Set up your ROS environment

[Install ROS2 Crystal Clemmys](https://github.com/ros2/ros2/wiki/Installation) with the `ros-crystal-desktop` option.

If you install binaries, be sure to also install the 
[development tools and ROS tools](https://github.com/ros2/ros2/wiki/Linux-Development-Setup#install-development-tools-and-ros-tools)
from the source installation instructions.

Install these additional packages:
~~~
sudo apt install ros-crystal-joystick-drivers ros-crystal-cv-bridge
~~~

### 4. Install flock2, flock_vlam and tello_ros

Download, compile and install the flock packages:
~~~
mkdir -p ~/flock2_ws/src
cd ~/flock2_ws/src
git clone https://github.com/clydemcqueen/flock2.git
git clone https://github.com/ptrmu/flock_vlam.git
git clone https://github.com/clydemcqueen/tello_ros.git
cd ..
source /opt/ros/crystal/setup.bash
colcon build --event-handlers console_direct+
~~~

## Running

### Teleop

`teleop_launch.py` will allow you to fly the drone using a wired XBox One gamepad.

Turn on the drone, connect to `TELLO-XXXXX` via wi-fi, and launch ROS:
~~~
source /opt/ros/crystal/setup.bash
source ~/flock2_ws/install/setup.bash
ros2 launch flock2 teleop_launch.py
~~~

Controls:
* menu button to take off
* view button to land
* left bumper and B to start mission
* left bumper and A to stop mission

## Future Work

The primary goal is to autonomously fly multiple Tello drones in simple patterns.