# slam-mapping-localization-mohamed-abdelaal

A ROS 2 package demonstrating SLAM mapping and localization using
[`slam_toolbox`](https://github.com/SteveMacenski/slam_toolbox) with a
TurtleBot3 in Gazebo simulation.

## Package Structure

```
slam_toolbox_demo/
├── config/
│   ├── mapping.yaml         # slam_toolbox parameters for mapping mode
│   └── localization.yaml    # slam_toolbox parameters for localization mode
├── launch/
│   ├── mappinglaunch.launch.py       # starts slam_toolbox in mapping (async) mode
│   └── localizationlaunch.launch.py  # starts slam_toolbox in localization mode
├── map/
│   ├── turtlebot3_world_map.yaml
│   └── turtlebot3_world_map.pgm
├── posegraph/
│   ├── turtlebot3_world.posegraph
│   └── turtlebot3_world.data
├── CMakeLists.txt
├── package.xml
└── README.md
```

## 1. Setup Instructions

### Prerequisites
- ROS 2 Jazzy
- Gazebo (TurtleBot3 simulation packages)
- `slam_toolbox`:
  ```bash
  sudo apt install ros-$ROS_DISTRO-slam-toolbox
  ```
- TurtleBot3 simulation packages:
  ```bash
  sudo apt install ros-$ROS_DISTRO-turtlebot3 ros-$ROS_DISTRO-turtlebot3-simulations
  ```

### Clone and build
```bash
cd ~/workspaces/slam-mapping-localization-mohamed-abdelaal
colcon build
source install/setup.bash
```

> Repeat `source install/setup.bash` in every new terminal you open for this
> workspace.

## 2. How to Test the Nodes

### Step 1 — Launch the TurtleBot3 Gazebo simulation
```bash
export TURTLEBOT3_MODEL=burger
ros2 launch turtlebot3_gazebo turtlebot3_world.launch.py
```

### Step 2 — Run SLAM mapping
In a new terminal (after sourcing the workspace):
```bash
ros2 launch slam_toolbox_demo mappinglaunch.launch.py
```
Open RViz and add the `/map` display to watch the map build in real time.
Drive the robot around (e.g. with `teleop_twist_keyboard`) to cover the
whole environment:
```bash
ros2 run turtlebot3_teleop teleop_keyboard
```

### Step 3 — Save the map
Once the environment is fully mapped:
```bash
ros2 run nav2_map_server map_saver_cli -f src/slam_toolbox_demo/map/turtlebot3_world_map
```
This produces `turtlebot3_world_map.yaml` and `turtlebot3_world_map.pgm`
(already included in this repo under `map/`).

Shut down the mapping node afterward (Ctrl+C).

### Step 4 — Run localization on the saved map
With the Gazebo simulation still running (or restarted), launch:
```bash
ros2 launch slam_toolbox_demo localizationlaunch.launch.py
```
This loads `config/localization.yaml`, which points `slam_toolbox` at the
saved map/posegraph so the robot can localize against it instead of
building a new map.

### Step 5 — Set the initial pose in RViz
1. In RViz, click **2D Pose Estimate** and click/drag on a **wrong**
   location on the map to see the laser scan mismatch the map.
2. Click **2D Pose Estimate** again, this time on the robot's **actual**
   position (matching its real position/orientation in Gazebo), and
   confirm the laser scan now aligns with the map walls.
3. Drive the robot around and confirm the map stays fixed — only the
   robot's estimated pose updates as it moves.

## 3. Expected Output

**During mapping:**
- RViz shows the `/map` topic being built incrementally as the robot moves.
- The terminal running `mappinglaunch.launch.py` logs the `slam_toolbox`
  lifecycle transitions (`configuring` → `inactive` → `activating`).

**During localization:**
- Before a correct pose estimate: the laser scan (red/colored points)
  does **not** align with the saved map's walls.
- After a correct pose estimate: the laser scan snaps onto the map's
  wall lines.
- Moving the robot afterward keeps the map static — only the robot's
  pose (and the laser scan around it) moves.

## 4. Demo

**Mapping demo:**
https://drive.google.com/file/d/1S0riir9yRzqvlUzFmSvJGbhMg1nvsGH-/view?usp=sharing

**Localization demo:**
https://drive.google.com/file/d/1aBNOIPTOdUXdWRXPqWiCYdh4PUBjbuBT/view?usp=sharing

The mapping demo shows the robot mapping the TurtleBot3 world from empty
to a complete map, then saving it.

The localization demo shows the saved map being loaded, a deliberately
wrong pose estimate followed by a corrected one, and the robot driving
around with the map remaining fixed in RViz.
