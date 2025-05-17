# robot_workshop

## Preparation

Before you begin, make sure you have [Docker](https://docs.docker.com/get-docker/) installed on your computer. Docker is required to run the development container (devcontainer) that provides all the tools and dependencies you need for this workshop.

- **Install Docker:**
  - Follow the official [Docker installation guide](https://docs.docker.com/get-docker/) for your operating system (Windows, macOS, or Linux).
  - After installation, you can verify Docker is working by running `docker --version` in your terminal.

- **Install Visual Studio Code (VS Code) or Cursor:**
  - Download and install [VS Code](https://code.visualstudio.com/) **or** [Cursor](https://www.cursor.so/).
  - Cursor is an alternative editor built on top of VS Code, so it supports all the same extensions and workflows, including Dev Containers.
  - Install the [Dev Containers extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers) from the marketplace (this works in both VS Code and Cursor).

- **Open the Project Folder in VS Code or Cursor:**
  - Open this folder in your chosen editor. You should see a pop-up asking if you want to "Reopen in Container" or "Open in Dev Container".
  - Click the button to open the folder in a devcontainer. The editor will automatically build and start a Docker container with all the required tools and dependencies installed for you.

This setup ensures you have a consistent environment with everything you need for robotics development, regardless of your host operating system.

Follow [this section](https://code.visualstudio.com/docs/devcontainers/containers#_installation) to get vscode or cursor setup with the devcontainer extension.

Then open this folder in vscode, and a pop up for opening the workspace in a devcontainer will show.

This will download and install a bunch of packages needed for what we want to do.

## Bring up

### 1. Get the submodules

If you haven't already, make sure to initialize and update the git submodules. This will pull in the TurtleBot 4 simulator source code:

```sh
git submodule update --init --recursive
```

### 2. Build the workspace (if needed)

If you are making changes to the source or want to build from source, you can build the workspace inside the devcontainer:

```sh
source /opt/ros/humble/setup.bash
colcon build --symlink-install
```

### 3. Source the workspace

After building, source the workspace so ROS 2 can find the packages:

```sh
source install/setup.bash
```

### 4. Launch the TurtleBot 4 Simulator with Gazebo and RViz

You can follow the official [TurtleBot 4 Simulator tutorial](https://turtlebot.github.io/turtlebot4-user-manual/software/turtlebot4_simulator.html) for more details and options.

To launch the simulator with default settings (Gazebo + TurtleBot 4):

```sh
ros2 launch turtlebot4_ignition_bringup turtlebot4_ignition.launch.py
```

To launch with RViz:

```sh
ros2 launch turtlebot4_ignition_bringup turtlebot4_ignition.launch.py rviz:=true
```

You can customize the launch with additional arguments (see the [official documentation](https://turtlebot.github.io/turtlebot4-user-manual/software/turtlebot4_simulator.html) for all options).

### Driving the Robot with the Keyboard

You can drive the TurtleBot 4 around the simulated environment using your keyboard. This is done with the `teleop_twist_keyboard` ROS 2 package, which sends velocity commands to the robot.

**1. Install teleop_twist_keyboard (if not already installed):**

If you are in the devcontainer, this package may already be installed. If not, you can install it with:

```sh
sudo apt update && sudo apt install ros-humble-teleop-twist-keyboard
```

**2. Run teleop_twist_keyboard:**

Open a new terminal (make sure your ROS 2 environment is sourced):

```sh
source install/setup.bash
ros2 run teleop_twist_keyboard teleop_twist_keyboard --ros-args -r /cmd_vel:=/cmd_vel
```

- Use the on-screen instructions to drive the robot with your keyboard (WASD keys for movement).
- Make sure the simulator is running before starting teleop.

You can also use the built-in GUI teleop panel in the simulator if you prefer a graphical interface.

## Generating a Map for Navigation (SLAM in Simulation)

You can generate a map of your simulated environment using SLAM (Simultaneous Localization and Mapping) and save it for later use with navigation. This is useful for autonomous navigation tasks.

### 1. Launch the Simulator with SLAM and RViz

Start the simulator with SLAM, Nav2, and RViz enabled:

```sh
source /opt/ros/humble/setup.bash
source install/setup.bash
ros2 launch turtlebot4_ignition_bringup turtlebot4_ignition.launch.py slam:=true nav2:=true rviz:=true
```

- This will bring up the simulator, start SLAM mapping, and open RViz for visualization.

### 2. Drive the Robot to Map the Environment

- Use the teleoperation tools (keyboard, joystick, or GUI) to drive the TurtleBot 4 around the environment.
- Watch RViz to see the map being built in real time.
- Try to cover all areas you want included in the map.

### 3. Save the Map

Once you are satisfied with the map, save it using the following command in a new terminal (make sure your ROS 2 environment is sourced):

```sh
source /opt/ros/humble/setup.bash
source install/setup.bash
ros2 service call /slam_toolbox/save_map slam_toolbox/srv/SaveMap "name:
  data: 'map_name'"
```

- This will save `map_name.pgm` and `map_name.yaml` in your current directory.

**Tip:** If you are using namespaces, you may need to call the map saver tool directly:

```sh
source /opt/ros/humble/setup.bash
source install/setup.bash
ros2 run nav2_map_server map_saver_cli -f "map_name" --ros-args -p map_subscribe_transient_local:=true -r __ns:=/namespace
```

### 4. Use the Map for Navigation

- You can now use this saved map for navigation by launching the simulator with localization and Nav2, passing your map as a parameter.

For more details, see the [official mapping tutorial](https://turtlebot.github.io/turtlebot4-user-manual/tutorials/generate_map.html).

### Navigating the Robot with a Saved Map (Localization & Nav2)

Once you have generated and saved a map, you can use it to autonomously navigate the robot using the Nav2 stack. This process uses localization (not SLAM) to determine the robot's position on the map and plan paths to goals.

#### 1. Launch the Simulator with Localization and Nav2

Start the simulator, enabling Nav2 and localization, and pass your saved map as a parameter. Replace `map_name.yaml` with the path to your saved map file:

```sh
source /opt/ros/humble/setup.bash
source install/setup.bash
ros2 launch turtlebot4_ignition_bringup turtlebot4_ignition.launch.py nav2:=true slam:=false localization:=true rviz:=true map:=/absolute/path/to/map_name.yaml
```

- This will bring up the simulator, start localization and Nav2, and open RViz for visualization.
- If you want to use a specific world, add the `world:=your_world` argument.

#### 2. Set the Initial Pose in RViz

Before sending navigation goals, you must set the robot's initial pose on the map:
- In the RViz window, select the **2D Pose Estimate** tool (usually an icon with an arrow).
- Click and drag on the map to indicate the robot's approximate starting position and orientation.

#### 3. Send Navigation Goals

- Use the **Nav2 Goal** tool in RViz (icon with an arrow and a flag) to click and drag on the map where you want the robot to go.
- The robot will plan a path and attempt to drive to the goal.

#### 4. (Optional) Echo Clicked Points

You can use the **Publish Point** tool in RViz to publish coordinates to the `/clicked_point` topic. To see these points in the terminal:

```sh
source /opt/ros/humble/setup.bash
source install/setup.bash
ros2 topic echo /clicked_point
```

#### More Information

For more details and advanced options (such as using custom worlds or multiple robots), see the [official TurtleBot 4 navigation tutorial](https://turtlebot.github.io/turtlebot4-user-manual/tutorials/navigation.html).

## Index

- [Git](#git): Version control for tracking and collaborating on code.
- [Docker](#docker): Containerization for consistent environments.
- [Devcontainers](#devcontainers): Ready-to-use coding environments using Docker.
- [ROS 2](#ros-2): Tools and libraries for building robot applications.
- [Simulation](#simulation): Test robot code in a virtual world before real hardware.

---

## Git

Git is a tool that helps us keep track of changes in our code. It allows us to save versions of our work, collaborate with others, and go back to previous versions if something goes wrong. Think of it as a time machine for your code!

### Why do we want version control?
Version control helps us keep track of every change we make to our code. If we make a mistake, we can go back to an earlier version. It also makes it easy to work with others, since everyone can see what changes have been made and when.

### What does "adding" mean?
When you "add" a file in Git, you're telling Git to start tracking changes to that file. This is the first step before saving your changes (committing).
```sh
git add filename.txt
```
Or to add all changed files:
```sh
git add .
```

### What does "committing" mean?
"Committing" means saving a snapshot of your changes in Git. Each commit is like a checkpoint you can return to later. You should commit often, with a short message describing what you changed.
```sh
git commit -m "Describe what you changed"
```

### What is branching?
Branching lets you work on new features or experiments without changing the main code. You can try out ideas safely, and later combine (merge) your changes back into the main branch if you want.
Create a new branch:
```sh
git checkout -b my-feature-branch
```
Switch to an existing branch:
```sh
git checkout main
```

### How do I make a pull request on GitHub?
A pull request is a way to ask for your changes to be added to a project on GitHub. You push your branch to GitHub, then open a pull request. Others can review your code, discuss it, and approve it before it gets added to the main project.
Push your branch to GitHub:
```sh
git push origin my-feature-branch
```
Then, go to the repository on GitHub and click "Compare & pull request" to start the process.

---

## Docker

Docker lets us package our code and all the things it needs to run (like libraries and tools) into a single container. This means our code will work the same way on any computer, making it easier to share and set up.

### Finding ROS 2 and Robotics Docker Images

You can find ready-made Docker images for ROS 2 and other robotics tools on [Docker Hub](https://hub.docker.com/). Search for images using keywords like `ros`, `ros2`, or the specific robotics tool you need.

For example, to find official ROS 2 images:
- Go to https://hub.docker.com/_/ros
- Or search in your terminal:
  ```sh
  docker search ros
  ```

### Running a ROS 2 Docker Image

Once you find an image, you can run it as a container. For example, to run the official ROS 2 Humble image:
```sh
docker run -it --rm ros:humble
```
- `-it` lets you interact with the container.
- `--rm` removes the container when you exit.
- `ros:humble` is the image name and tag.

You can also mount your code into the container:
```sh
docker run -it --rm -v $(pwd):/workspace ros:humble
```
This mounts your current directory into `/workspace` inside the container.

### Making Your Own Dockerfile

If you need a custom environment, you can create your own Dockerfile. A Dockerfile is a text file with instructions for building a Docker image.

Example `Dockerfile` for ROS 2:
```Dockerfile
# Use the official ROS 2 base image
FROM ros:humble

# Install extra packages
RUN apt-get update && \
    apt-get install -y ros-humble-turtlebot4-simulator

# Set the default command
CMD ["bash"]
```
To build your image:
```sh
docker build -t my-ros2-image .
```
To run it:
```sh
docker run -it --rm my-ros2-image
```

### Dockerfile, Image, and Container: What's the Difference?

- **Dockerfile**: A recipe (text file) that tells Docker how to build an image.
- **Docker image**: The built package (like a snapshot of an OS + software) created from the Dockerfile. You can share or reuse images.
- **Docker container**: A running instance of an image. You can have many containers from the same image, each isolated from the others.

**Summary:**
- Write a Dockerfile → Build an image → Run containers from that image.

---

## Devcontainers

A devcontainer is a special setup that uses Docker to create a ready-to-use coding environment. When you open this project in VS Code with the devcontainer extension, it automatically sets up everything you need, so you can start coding right away without worrying about installing lots of software.

### What is a Devcontainer?
A devcontainer lets you define your development environment as code. This means everyone on your team gets the same tools, libraries, and settings, no matter their computer or OS. It's especially useful for robotics and ROS 2, where dependencies can be tricky.

### How Does the `.devcontainer` Folder Work?
The `.devcontainer` folder contains files that describe your development environment:
- `devcontainer.json`: The main configuration file. It tells VS Code how to build and start your container, what extensions to install, and more.
- `Dockerfile` (optional): If you need a custom environment, you can add a Dockerfile here to install extra tools or packages.

Example structure:
```
.devcontainer/
  devcontainer.json
  Dockerfile
```

### Customizing Your Devcontainer
You can add extra packages or tools by editing the `Dockerfile` in `.devcontainer`.

For example, to add a ROS 2 package:
```Dockerfile
# In .devcontainer/Dockerfile
FROM ros:humble

RUN apt-get update && \
    apt-get install -y ros-humble-turtlebot4-simulator
```

You can also change settings in `devcontainer.json`, like which VS Code extensions to install automatically:
```json
// In .devcontainer/devcontainer.json
{
  "name": "ROS 2 Devcontainer",
  "build": {
    "dockerfile": "Dockerfile"
  },
  "extensions": [
    "ms-iot.vscode-ros",
    "ms-python.python"
  ]
}
```

### Rebuilding or Updating the Devcontainer
If you change the Dockerfile or `devcontainer.json`, you need to rebuild the devcontainer:
- In VS Code, open the Command Palette (`Ctrl+Shift+P` or `Cmd+Shift+P`)
- Type and select: **Dev Containers: Rebuild and Reopen in Container**

This will rebuild your environment with the new settings or packages.

### Example: Adding a ROS 2 Tool
Suppose you want to add `ros-humble-rqt` to your devcontainer. Edit your `.devcontainer/Dockerfile`:
```Dockerfile
RUN apt-get update && \
    apt-get install -y ros-humble-rqt
```
Then rebuild the devcontainer as above.

---

## ROS 2

ROS 2 (Robot Operating System 2) is a set of tools and libraries that help us build robot applications. It makes it easier to control robots, process sensor data, and make robots do useful things. ROS 2 is widely used in both research and industry.

### What is ROS 2?
ROS 2 is an open-source framework for building robot software. It provides communication tools, drivers, algorithms, and developer tools to help you create complex robotics systems. ROS 2 improves on ROS 1 with better support for real-time, security, multi-robot systems, and more.

Learn more: [Official ROS 2 Documentation (Humble)](https://docs.ros.org/en/humble/index.html)

### Key Concepts in ROS 2

#### Nodes
- **What:** Nodes are the basic building blocks of a ROS 2 system. Each node is a process that performs computation, such as reading sensors or controlling motors.
- **Example:** A camera node publishes images, while a motor controller node subscribes to velocity commands.
- [Learn more about Nodes](https://docs.ros.org/en/humble/Tutorials/Concepts/Nodes.html)

#### Topics
- **What:** Topics are named channels that nodes use to send and receive messages asynchronously. They are used for streaming data like sensor readings or commands.
- **Example:** A laser scanner node publishes distance data on a `/scan` topic, and a navigation node subscribes to it.
- [Learn more about Topics](https://docs.ros.org/en/humble/Tutorials/Concepts/Topics.html)

#### Launch Files
- **What:** Launch files are scripts (usually in Python or XML) that start up multiple nodes and set parameters all at once. They make it easy to bring up complex systems.
- **Example:** A launch file can start your robot's driver, sensor nodes, and visualization tools with one command.
- [Learn more about Launch Files](https://docs.ros.org/en/humble/Tutorials/Intermediate/Launch/Creating-Launch-Files.html)

#### Services
- **What:** Services provide synchronous, request/reply communication between nodes. They are used for tasks that need a response, like requesting a sensor reading or resetting a robot.
- **Example:** A node can call a service to reset the robot's odometry, and the service node will reply when done.
- [Learn more about Services](https://docs.ros.org/en/humble/Tutorials/Concepts/Services.html)

#### Actions
- **What:** Actions are for long-running tasks that provide feedback and can be canceled, such as moving a robot arm to a position.
- **Example:** A navigation node can send a goal to a robot to move to a location, receive progress updates, and cancel if needed.
- [Learn more about Actions](https://docs.ros.org/en/humble/Tutorials/Concepts/Actions.html)

See the [ROS 2 Concepts](https://docs.ros.org/en/humble/Tutorials/Concepts/Nodes.html) and [Beginner Tutorials](https://docs.ros.org/en/humble/Tutorials.html) for more details.

### Getting Started with ROS 2
- **Installation**: Follow the [official installation guide](https://docs.ros.org/en/humble/Installation.html) for your OS.
- **Tutorials**: Start with the [Beginner: CLI tools](https://docs.ros.org/en/humble/Tutorials/Beginner-CLI-Tools.html) and [Beginner: Client libraries](https://docs.ros.org/en/humble/Tutorials/Beginner-Client-Libraries.html) tutorials.
- **Creating Packages**: Learn how to [create a workspace and package](https://docs.ros.org/en/humble/Tutorials/Beginner-Client-Libraries/Creating-A-Workspace/Creating-A-Workspace.html).
- **Writing Nodes**: Try the [simple publisher and subscriber tutorials](https://docs.ros.org/en/humble/Tutorials/Beginner-Client-Libraries/Writing-A-Simple-Py-Publisher-And-Subscriber.html) (Python) or [C++ version](https://docs.ros.org/en/humble/Tutorials/Beginner-Client-Libraries/Writing-A-Simple-Cpp-Publisher-And-Subscriber.html).

### More Resources
- [ROS 2 Documentation (Humble)](https://docs.ros.org/en/humble/index.html)
- [ROS 2 Tutorials](https://docs.ros.org/en/humble/Tutorials.html)
- [ROS 2 Concepts](https://docs.ros.org/en/humble/Tutorials/Concepts/Nodes.html)
- [ROS 2 Installation Guide](https://docs.ros.org/en/humble/Installation.html)

For more advanced topics, see the [How-to Guides](https://docs.ros.org/en/humble/How-To-Guides.html) and [Intermediate/Advanced Tutorials](https://docs.ros.org/en/humble/Tutorials.html).

---

## Simulation

Simulation lets us test our robot code in a virtual world before trying it on a real robot. This is safer and faster, and helps us find and fix problems early. In this workshop, we'll use simulation to see how our code would work on a real robot, without needing any hardware.

### Popular Simulators for ROS 2

Below are some of the most widely used simulators that integrate with ROS 2, covering mobile robots, robot arms, autonomous vehicles, and aerial robots:

#### Gazebo (Ignition/Garden/Classic)
- **What:** The most popular open-source robotics simulator, with strong ROS 2 integration. Supports mobile robots, robot arms, sensors, and custom environments.
- **Use Cases:** Mobile robots, robot arms, multi-robot systems, sensors.
- **ROS 2 Integration:** [gazebo_ros_pkgs](https://github.com/ros-simulation/gazebo_ros_pkgs) and [Ignition Gazebo ROS 2](https://ignitionrobotics.org/docs/ros2)
- **Learn more:** [Gazebo Website](https://gazebosim.org/)

#### NVIDIA Isaac Sim
- **What:** High-fidelity, GPU-accelerated simulator for robotics AI, manipulation, and navigation. Built on Omniverse.
- **Use Cases:** Robot arms, mobile robots, synthetic data generation, AI training.
- **ROS 2 Integration:** [Isaac Sim ROS 2 Bridge](https://docs.omniverse.nvidia.com/isaacsim/latest/ros2_tutorials.html)
- **Learn more:** [Isaac Sim Website](https://developer.nvidia.com/isaac-sim)

#### CARLA
- **What:** Open-source simulator for autonomous driving research. Simulates vehicles, sensors, pedestrians, and urban environments.
- **Use Cases:** Autonomous vehicles, sensor simulation, perception, planning.
- **ROS 2 Integration:** [carla-ros-bridge](https://github.com/carla-simulator/ros-bridge)
- **Learn more:** [CARLA Website](https://carla.org/)

#### Webots
- **What:** Versatile, easy-to-use simulator for mobile robots, arms, and drones. Good for education and research.
- **Use Cases:** Mobile robots, robot arms, drones, swarm robotics.
- **ROS 2 Integration:** [Webots ROS 2 package](https://github.com/cyberbotics/webots_ros2)
- **Learn more:** [Webots Website](https://cyberbotics.com/)

#### CoppeliaSim (V-REP)
- **What:** Powerful simulator for robot arms, mobile robots, and complex mechanisms. Supports physics engines and scripting.
- **Use Cases:** Robot arms, mobile robots, industrial automation.
- **ROS 2 Integration:** [CoppeliaSim ROS Interface](https://github.com/CoppeliaRobotics/simExtROS2)
- **Learn more:** [CoppeliaSim Website](https://www.coppeliarobotics.com/)

#### Aerial Robot Simulators
- **AirSim:**
  - **What:** Open-source simulator for drones and cars, developed by Microsoft. Supports photorealistic environments and physics.
  - **Use Cases:** Drones (UAVs), autonomous vehicles, AI research.
  - **ROS 2 Integration:** [AirSim ROS Wrapper](https://github.com/microsoft/AirSim/tree/main/ros)
  - **Learn more:** [AirSim Website](https://microsoft.github.io/AirSim/)
- **RotorS:**
  - **What:** Gazebo-based simulator for multicopter UAVs. Good for research and algorithm development.
  - **Use Cases:** Drones, aerial robotics research.
  - **ROS 2 Integration:** Community packages and Gazebo plugins (primarily ROS 1, but can be adapted for ROS 2).
  - **Learn more:** [RotorS GitHub](https://github.com/ethz-asl/rotors_simulator)

---

**Tip:** Most simulators support running alongside ROS 2 nodes, allowing you to test perception, planning, and control algorithms in a safe, repeatable environment. Choose the simulator that best matches your robot type and research needs.