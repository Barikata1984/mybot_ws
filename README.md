# mybot_ws
URDF model for Gazebo integrated with ROS

# Update for Melodic
Updating http://www.moorerobots.com/blog/post/3 to run on ROS Melodic (Ubuntu 18.04)

## Simulating the ROS Navigation Stack (part 3)

### Mapping and Navigation: Turtlebot3 (Optional)
In ROS melodic, turtlebot3 has replaced the older turtlebot packages.  This
version of the tutorial takes you through map building/making with turtlebot3,
then with our `mybot` robot.

#### Setup
1. First, install the turtlebot3 packages:
`sudo apt install ros-melodic-turtlebot3_/*`
`sudo apt install ros-melodic-slam-gmapping ros-melodic-gmapping ros-melodic-openslam-gmapping`

#### Creating the map
2. I had to change the turtlebot3 gazebo to run with no gui (slow computer):
```
cp `rospack find turtlebot3_gazebo`/launch/turtlebot3_world.launch `rospack find rover_gazebo`/launch
```
add
`  <arg name="gui" default="false"/>` below launch
change
`  <arg name="gui" value="true"/>`
to
`  <arg name="gui" value="$(arg gui)"/>`
finally,
```bash
export TURTLEBOT3_MODEL=waffle
roslaunch rover_gazebo turtlebot3_world.launch gui:=true/false
```

3. In Terminal 2, start map building (also starts rviz)
export TURTLEBOT3_MODEL=waffle
`roslaunch turtlebot3_slam turtlebot3_slam.launch slam_methods:=gmapping`

4. In Terminal 3, start teleop
export TURTLEBOT3_MODEL=waffle
`roslaunch turtlebot3_teleop turtlebot3_teleop_key.launch`

#### Saving the map
5. In Terminal 4, save the map to some file path
`rosrun map_server map_saver -f /tmp/test_map`


#### Loading the map
Close all previous terminals and run the following commands below.  Once loaded, use rviz to set navigation waypoints and the robot should move autonomously.
5. Install local planner (if needed)
`sudo apt install ros-melodic-dwa-local-planner`

6. In Terminal 1, launch the Gazebo world
`roslaunch rover_gazebo turtlebot3_world.launch`

7. In Terminal 2, start map building (will start rviz too)
`roslaunch turtlebot3_navigation turtlebot3_navigation.launch map_file:=/tmp/test_map.yaml`

8. In rviz, estimate initial pose - click `2D Pose Estimate` and click the approximate location of the robot on the map, and drag to indicate the direction.

9. In Terminal 3, start teleop and move the robot around.  The estimated positions should converge on the true position pretty quickly.
`roslaunch turtlebot3_teleop turtlebot3_teleop_key.launch`

10. In rviz, send a few `2D Navigation Goals` (click on `2D Nav Goal and click/drag to set position/orientation) and watch the robot autonomously navigate to the goal.

Close all terminals, you are done with turtlebot3.

### Mapping and Navigation: Our Robot
We follow the same steps for our own differential drive robot.

#### Creating the map
Run the following commands.  Use teleop to move the robot around and create an accurate and thorough map.

1. Change the robot description in turtlebot3_world.launch:
    * roscd rover_description
    * cp launch/turtlebot3_world.launch rover_tb3_world.launch
    * change line 17 from:
      ` <param name="robot_description" command="$(find xacro)/xacro --inorder $(find turtlebot3_description)/urdf/turtlebot3_$(arg model).urdf.xacro" /> `
to

      ` <param name="robot_description" command="$(find xacro)/xacro --inorder $(find rover_description)/urdf/rover.xacro" /> `


2. Copy and modify `turtlebot3_slam.launch`
    * cp `rospack find turtlebot3_slam`/launch/turtlebot3_slam.launch `rospack find rover_navigation`/launch/rover_slam.launch
    * change line 8-11 from:
```
  <!-- TurtleBot3 -->
  <include file="$(find turtlebot3_bringup)/launch/turtlebot3_remote.launch">
    <arg name="model" value="$(arg model)" />
  </include>
```
to
```
  <!-- MyBot -->
  <param name="robot_description" command="$(find xacro)/xacro.py '$(find rover_description)/urdf/rover.xacro'"/>
  <node name="joint_state_publisher" pkg="joint_state_publisher" type="joint_state_publisher">
    <param name="use_gui" value="False"/>
  </node>
  <node name="robot_state_publisher" pkg="robot_state_publisher" type="robot_state_publisher"/>
  <!-- mybot has chassis, turtlebot has base_footprint -->
  <node pkg="tf" type="static_transform_publisher" name="base" args="0 0 0 0 0 0 chassis base_footprint 100" />
```

4. In `rover_description/urdf/rover.gazebo` change the scan topic in line 90 from:
   ` /rover/laser/scan` to `/scan`
 to match what turtlebot_slam expects.

2. In Terminal 1, launch the gazebo world
`roslaunch rover_gazebo rover_tb3_world.launch`

3. In Terminal 2, start map building (also starts rviz)
`roslaunch rover_navigation rover_slam.launch slam_methods:=gmapping`

4. In Terminal 3, start teleop
`roslaunch turtlebot3_teleop turtlebot3_teleop_key.launch`

### Saving the map
5. In Terminal 4, save the map to some file path
`rosrun map_server map_saver -f /tmp/test_map`

### Loading the map
Close all previous terminals and run the following commands below.  Once loaded, use rviz to set navigation waypoints and the robot should move autonomously.

5. Copy and modify `turtlebot3_navigation.launch`

    * cp `rospack find turtlebot3_navigation`/launch/turtlebot3_navigation.launch `rospack find rover_navigation`/launch/turtlebot3_navigation.launch
    * change line 8-11 from:
```
  <!-- TurtleBot3 -->
  <include file="$(find turtlebot3_bringup)/launch/turtlebot3_remote.launch">
    <arg name="model" value="$(arg model)" />
  </include>
```
to
```
  <!-- MyBot -->
  <param name="robot_description" command="$(find xacro)/xacro.py '$(find rover_description)/urdf/rover.xacro'"/>
  <node name="joint_state_publisher" pkg="joint_state_publisher" type="joint_state_publisher">
    <param name="use_gui" value="False"/>
  </node>
  <node name="robot_state_publisher" pkg="robot_state_publisher" type="robot_state_publisher"/>
  <!-- mybot has chassis, turtlebot has base_footprint -->
  <node pkg="tf" type="static_transform_publisher" name="base" args="0 0 0 0 0 0 chassis base_footprint 100" />
```

6. In Terminal 1, launch the Gazebo world
`roslaunch rover_gazebo rover_tb3_world.launch`

7. In Terminal 2, start map building (will start rviz too)
`roslaunch rover_navigation rover_navigation.launch map_file:=/tmp/test_map.yaml`

8. In rviz, estimate initial pose - click `2D Pose Estimate` and click the approximate location of the robot on the map, and drag to indicate the direction.

9. In Terminal 3, start teleop and move the robot around.  The estimated positions should converge on the true position pretty quickly.
`roslaunch turtlebot3_teleop turtlebot3_teleop_key.launch`

10. In rviz, send a few `2D Navigation Goals` (click on `2D Nav Goal and click/drag to set position/orientation) and watch the robot autonomously navigate to the goal.

Close all terminals, you are done with turtlebot3.

