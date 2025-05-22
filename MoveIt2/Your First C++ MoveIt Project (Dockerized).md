
>**Warning:** Most features in MoveIt won't work properly since additional parameters are required for full Move Group functionality. For a full setup, please continue with the  [Move Group C++ Interface Tutorial](https://moveit.picknik.ai/main/doc/tutorials/your_first_project/your_first_project.html)


### 1. First, make sure the Docker container is spun up

Your shell should have a prompt like ```root@justin-PC:~/ws_moveit#```.

```bash
# Run this with or without --rm (see Docker setup note)
DOCKER_IMAGE=humble-humble-tutorial-source \
docker compose run --rm --name moveit2_gpu gpu
```


### 2. Mounting the Host Workspace (`~/ws_moveit`) Inside Docker

Because we're using Docker, currently our ```ws_moveit``` workspace is inaccessible to the host machine.  We need to set up a volume so the same code lives on our host AND our container.

##### 2.1 Modify ```docker-compose.yml``` so the host folder is mounted into the container

```yaml
services:
	cpu:
		image: moveit/moveit2:${DOCKER IMAGE}
		# ... existing lines ...
		volumes:
			- /home/justin/ws_moveit:/ws_moveit # <-- add this line
			- /tmp/.X11-unix:/tmp/.X11-unix # existing
			- $XAUTHORITY:/root/.Xauthority # existing
	gpu:
		image: moveit/moveit2:${DOCKER_IMAGE}
		# ... existing lines ...
		volumes:
			- /home/justin/ws_moveit:/ws_moveit # <-- add this line
			- /tmp/.X11-unix:/tmp/.X11-unix # existing
			- $XAUTHORITY:/root/.Xauthority # existing
```

##### 2.2. Restart the container (so it sees the new volume mount)

```bash
# shutdown old container
docker compose down

# restart
DOCKER_IMAGE=humble-humble-tutorial-source \
docker compose run --rm --name moveit2_gpu gpu
```

##### 2.3. Verify the mount

If all goes well, you should be able to see ```docker-compose.yml``` while *inside the container shell:*

==**SUPER IMPORTANT NOTE:**== The mount point will be in ```~/ws_moveit/ws_moveit```, **not** ```~/ws_moveit```.

```bash
# Restart container 
(base) justin@justin-PC:~/ws_moveit/src$ DOCKER_IMAGE=humble-humble-tutorial-source docker compose run --rm --name moveit2_gpu gpu

# Huh? Where is docker-compose.yml?
root@justin-PC:~/ws_moveit# ls
build  install  log  src

# Ahh, there it is!
root@justin-PC:~/ws_moveit# cd /ws_moveit/
root@justin-PC:/ws_moveit# ls
docker-compose.yml  install  src
root@justin-PC:/ws_moveit# 
```

##### 2.4. ==Now we're all good!==

From now on, we need to work in ```/ws_moveit```, ***not*** ```~/ws_moveit```. Honestly, you can delete ```~/ws_moveit``` at this point (if you screwed up and made one in the first place, like me).

```bash
# INCORRECT
root@justin-PC:~# cd ws_moveit/
root@justin-PC:~/ws_moveit# ls <-- THIS IS THE WRONG DIRECTORY
build  install  log  src

# Goodbye!
root@justin-PC:~# rm -rf ws_moveit/

# CORRECT
root@justin-PC:~# cd /ws_moveit/
root@justin-PC:/ws_moveit# ls
docker-compose.yml  install  src
```


### 3. Create a New ROS 2 Package

This command will create ```package.xml```, ```CMakeLists.txt```, and an empty ```src/hello_moveit.cpp```. See the [ROS 2 package creation tutorial](https://docs.ros.org/en/humble/Tutorials/Beginner-Client-Libraries/Creating-Your-First-ROS2-Package.html) for details.

```bash
# Verify you're in the correct src directory
cd /ws_moveit/src

# Create a new package using ROS2 command line tools
ros2 pkg create \
  --build-type ament_cmake \
  --dependencies moveit_ros_planning_interface rclcpp \
  --node-name hello_moveit hello_moveit # <node_name> <package_name>
```

- ```ament_cmake``` is the usual C++ build system in ROS 2.

- ```moveit_ros_planning_interface``` dependency gives us the ```MoveGroupInterface``` class.

- ```rclcpp``` is the ROS 2 C++ client library - this is our second dependency.

- ```--node-name hello_moveit hello_moveit``` sets the default executable name (and the C++ node's name inside your code), as well as the actual package name

##### Terminal Output:
```bash
root@justin-PC:~# cd /ws_moveit/src/
root@justin-PC:/ws_moveit/src# ros2 pkg create \
  --build-type ament_cmake \
  --dependencies moveit_ros_planning_interface rclcpp \
  --node-name hello_moveit hello_moveit # <node_name> <package_name>

going to create a new package
package name: hello_moveit              # pkg name we specified
destination directory: /ws_moveit/src   # correct workspace dir!
package format: 3
version: 0.0.0

description: TODO: Package description
maintainer: ['root <root@todo.todo>']   # put yo email here
licenses: ['TODO: License declaration'] # grrr MIT bad zoe bad
build type: ament_cmake
dependencies: ['moveit_ros_planning_interface', 'rclcpp']
node_name: hello_moveit                 # node name we specified
creating folder ./hello_moveit
creating ./hello_moveit/package.xml
creating source and include folder
creating folder ./hello_moveit/src
creating folder ./hello_moveit/include/hello_moveit
creating ./hello_moveit/CMakeLists.txt
creating ./hello_moveit/src/hello_moveit.cpp
```


### 4. Create a ROS Node and Executor (Boilerplate)

>"The first block of code is a bit of boilerplate, but you should be used to seeing this from the ROS2 tutorials" - Official MoveIt2 docs

Open the new source file created at:

```
ws_moveit/src/hello_moveit/src/hello_moveit.cpp
``` 

Replace the standard "Hello World" code with the following:

```cpp
#include <memory>
#include <rclcpp/rclcpp.hpp>
#include <moveit/move_group_interface/move_group_interface.h>

int main(int argc, char * argv[])
{
  // Initialize ROS 2
  rclcpp::init(argc, argv);

  // Create a Node (with param override support)
  auto const node = std::make_shared<rclcpp::Node>(
    "hello_moveit",
    rclcpp::NodeOptions()
      .automatically_declare_parameters_from_overrides(true)
  );

  // Create a ROS logger for organized output
  auto const logger = rclcpp::get_logger("hello_moveit");

  // Next step goes here

  // Shutdown ROS
  rclcpp::shutdown();
  return 0;
}
```

##### ==Rant==
Whoever put:

```
#include <moveit/move_group_interface/move_group_interface.hpp>
```

Instead of the CORRECT:

```
#include <moveit/move_group_interface/move_group_interface.h>
```

inside the ***official*** MoveIt2 tutorial code cost me 4+ hours today, and should be executed immediately.


### 5. Rebuild and Launch

Rebuild our ```hello_moveit``` package using colcon inside the container.

```bash
cd /ws_moveit

# Build the package
colcon build --mixin debug

Starting >>> hello_moveit
Finished <<< hello_moveit [4.63s]                     
Summary: 1 package finished [4.71s] # success!
```

If it builds successfully, then source the workspace environment and run the program. 
- Since we're on Docker, no need to open a new terminal like the official tutorial says.

```bash
# Source the setup
source install/setup.bash
ros2 run hello_moveit hello_moveit # should run & exit with no errors
```


### 6. Plan and Execute using ```MoveGroupInterface```

In place of the comment that says "Next step goes here", add the following code:

```cpp
// Create the MoveIt MoveGroup Interface
using moveit::planning_interface::MoveGroupInterface;
auto move_group_interface = MoveGroupInterface(node, "panda_arm");

// Set a target Pose
auto const target_pose = []{
  geometry_msgs::msg::Pose msg;
  msg.orientation.w = 1.0;
  msg.position.x = 0.28;
  msg.position.y = -0.2;
  msg.position.z = 1.0;
  return msg;
}();
move_group_interface.setPoseTarget(target_pose);

// Create a plan to that target pose
auto const [success, plan] = [&move_group_interface]{
  moveit::planning_interface::MoveGroupInterface::Plan msg;
  auto const ok = static_cast<bool>(move_group_interface.plan(msg));
  return std::make_pair(ok, msg);
}();

// Execute the plan
if(success) {
  move_group_interface.execute(plan);
} else {
  RCLCPP_ERROR(logger, "Planning failed!");
}
```

Next, rebuild the code:

```bash
# Build
cd /ws_moveit
colcon build --mixin debug
source install.setup.bash
```

We need to reuse the demo launch file from the previous tutorial to start RViz and the ```MoveGroup``` node. ===You need multiple Docker terminals for this! ===

##### Terminal 2 (new)
In a second terminal, source the workspace and run the demo launch file.

```bash
# Connect to the SAME Docker container
docker exec -it moveit2_gpu bash

# Inside the container:
cd /ws_moveit
source install/setup.bash
ros2 launch moveit2_tutorials demo.launch.py # Launches RViz
```

Then, in the ```Displays``` window under ```MotionPlanning / Planning Request```, uncheck the ```Query Goal State``` box. You should see something like this:

![[tutorial1_before.png]]

##### Terminal 3 (new)
In a third terminal, source the workspace and run the program.

```bash
# Connect to the SAME Docker container
docker exec -it moveit2_gpu bash

# Inside the container:
cd /ws_moveit
source install/setup.bash
ros2 run hello_moveit hello_moveit
```

The arm should now reposition itself to the following pose:

![[tutorial1_after.png]]