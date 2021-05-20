# How to Remap
## Explanation of the exercise
This is a simple example of how to remap correctly. In this case, we are going to edit the fake_turtlebot.launch file to 
move the robot from RBX1 package with our keyboard.

## Download and install RBX1 repository
First, go to a workspace folder. 

```
cd ~/<yourWorkspace>/src
```

Now, copy the follow line in the terminal:

```
git clone https://github.com/pirobot/rbx1.git
```

Install a needed repository:

```
sudo apt-get install python-rosinstall
```

Now we have a copy of the RBX1 repository in our workspace, it's time to compile it

```
cd <yourWorkspace>
catkin_make>
```

In case of presenting any issue, try install the next ROS dependencies:

```
sudo apt-get install ros-kinetic-arbotix-python
```


## How to know which topics remap?

First, run ROS in the terminal

```
roscore
```

Then, run the launch file:

```
roslaunch rbx1_bringup fake_turtlebot.launch
```

Now, in other terminal, we will observe the active topics with the command:

```
rostopic list
```

The answer must be something like this:
```
julianroa18@julianroa18-Aspire-E5-575G:~$ rostopic list
/cmd_vel
/diagnostics
/joint_states
/odom
/rosout
/rosout_agg
/tf
/tf_static
```

Note the topic **"/cmd_vel"**, we will need it later. That is our first topic to remap.
To know which is the second topic we need, run the turtle_teleop_key node in another terminal:

```
rosrun turtlesim turtle_teleop_key
```

Displays the list of active topics again:

```
rostopic list
```

Now the list is similar to:

```
julianroa18@julianroa18-Aspire-E5-575G:~$ rostopic list
/cmd_vel
/diagnostics
/joint_states
/odom
/rosout
/rosout_agg
/tf
/tf_static
/turtle1/cmd_vel
```

The last one, **"/turtle1/cmd_vel"** is the second topic that we will remap
Save that topics for the next step


## REMAP

Remap is a way of **"cheating"** a ROS node, so when it thinks it is subscribing to or publishing to "/topic1" it really does it 
to "/topic2"

Remap works like this

```
<remap from="/different_topic" to="/needed_topic"/>
```

For our example, the correct way is:

```
<remap from="/cmd_vel" to="/turtle1/cmd_vel"/>
```

Now add this line in the fake_turtlebot.launch file and the new file should look like this:

```
<launch>
  <param name="/use_sim_time" value="false" />

  <!-- Load the URDF/Xacro model of our robot -->
  <arg name="urdf_file" default="$(find xacro)/xacro.py '$(find rbx1_description)/urdf/turtlebot.urdf.xacro'" />
   
  <param name="robot_description" command="$(arg urdf_file)" />
    
  <node name="arbotix" pkg="arbotix_python" type="arbotix_driver" output="screen" clear_params="true">
      <rosparam file="$(find rbx1_bringup)/config/fake_turtlebot_arbotix.yaml" command="load" />
      <param name="sim" value="true"/>
      <remap from="/cmd_vel" to="/turtle1/cmd_vel"/>
  </node>
  
  <node name="robot_state_publisher" pkg="robot_state_publisher" type="state_publisher">
      <param name="publish_frequency" type="double" value="20.0" />
  </node>
  
</launch>
```

## How to show the result

Save the changes in the .launch file and kill all the procces in the terminal, except roscore.
Run the .launch again, and then copy the next command in the terminal

```
rqt_graph
```

Finally, execute the next line to do the control

```
rosrun rviz rviz -d `rospack find rbx1_nav`/sim.rviz
```

Move the robot with you keyboard, if you want to change the velocity of the robot, execute the next line in the terminal:

```
rosparam set teleop_turtle/scale_linear 0.5
```

Where 0.5 is the new velocity. Execute again and enjoy :)


## Collaborators
- Gerson Hernández
- Kevin Parra 
- Daniel Cruz
- Julián Roa
