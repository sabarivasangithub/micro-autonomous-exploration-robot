# Micro Autonomous Exploration Robot

A wheeled robot that autonomously explores an unknown environment, builds a 2D LiDAR map, and returns to its starting position using ROS 2.

> **Status:** Early development. The architecture and features below are planned. Development starts in simulation, followed by physical robot testing.

## Goals and scope

- Explore reachable indoor space and avoid detected obstacles.
- Build and save a 2D occupancy grid map.
- Return to the recorded starting position and orientation.

The initial design assumes differential drive on flat floors. Precise docking and automatic charging are future work.

## Software stack

- **Platform:** Ubuntu 24.04, ROS 2 Jazzy, Gazebo Harmonic, and `ros_gz`.
- **Mapping and navigation:** SLAM Toolbox, Nav2, and RViz2.
- **Languages:** Python (`rclpy`) for custom autonomy; existing drivers or C++ for hardware integration.

Use the same versions and confirm hardware driver compatibility. See the [Gazebo compatibility guide](https://gazebosim.org/docs/harmonic/ros_installation/).

For Setup and simulation commands, refer : README_commands.md file

## ROS 2 nodes

| Node or component | Responsibility | Main interfaces |
| --- | --- | --- |
| LiDAR driver | Publish laser scans | `/scan` |
| Base controller | Drive wheels and provide encoder odometry | `/cmd_vel`, `/odom`, TF |
| `robot_state_publisher` | Publish robot link transforms | `/tf`, `/tf_static` |
| SLAM Toolbox | Build the map and estimate global pose | `/scan`, `/map`, TF |
| Nav2 nodes | Plan and execute navigation | `/navigate_to_pose`, velocity commands |
| `exploration_node` | Find and rank exploration viewpoints | `/map`, `/exploration/candidates` |
| `mission_manager_node` | Manage goals, return home, and request map saving | Nav2 actions, map-saving service |
| Map saver | Export the map to files | `/map` |

Reuse existing mapping, navigation, and driver packages. Write the exploration and mission manager nodes. Only the mission manager sends autonomous navigation goals.

Required transform chain: `map` → `odom` → `base_link` → `laser_frame`. SLAM Toolbox publishes the first transform, base odometry the second, and the robot model supplies the LiDAR mounting transform. Each transform must have one publisher. Keep AMCL disabled during active SLAM mapping.

Integration reference: [Nav2 with SLAM Toolbox](https://docs.nav2.org/jazzy/tutorials/general_tutorials/navigation2_with_slam/navigation2_with_slam/).

## Exploration and return home

1. Wait for valid sensor data, transforms, a map, and active navigation; record the starting pose in the `map` frame.
2. Find **frontiers**, the boundaries between known free space and unknown space. Select reachable viewpoints in free space with sufficient robot clearance.
3. Send goals through Nav2, monitor results, and retry failed goals within limits.
4. Stop exploration when no reachable frontiers remain across several valid map updates, a time limit expires, or return is requested.
5. Cancel the exploration goal, save a map checkpoint, and navigate home while keeping SLAM active. A saving failure must not prevent return.
6. Confirm successful arrival within configured position and heading tolerances, stop, and save the final map. Report navigation or saving failures.

Exhausting reachable frontiers does not guarantee complete coverage. SLAM error limits return accuracy; precise docking may need an additional base sensor.

## Planned repository layout

| Path | Contents |
| --- | --- |
| `src/mae_description/` | Robot model and dimensions |
| `src/mae_simulation/` | Gazebo worlds and simulated sensors |
| `src/mae_hardware/` | Motor and encoder interfaces |
| `src/mae_autonomy/` | Exploration and mission nodes |
| `src/mae_bringup/` | Launch files and configuration |
| `maps/`, `docs/` | Example maps, wiring, and test results |

Exclude `build/`, `install/`, `log/`, Python caches, and large sensor recordings through `.gitignore`.

## Setup

Install the selected stack, `colcon`, and `rosdep`. Once ROS packages exist under `src/` and `rosdep` is initialized, run from the repository root:

```bash
source /opt/ros/jazzy/setup.bash
rosdep update
rosdep install --from-paths src --ignore-src -r -y
colcon build --symlink-install
source install/setup.bash
```

Launch commands will be added after verification. Use simulation time consistently in Gazebo and real time on hardware.

## Roadmap

- [ ] Verify simulated driving, LiDAR, odometry, and transforms.
- [ ] Map manually and navigate to selected goals.
- [ ] Demonstrate a commanded return home.
- [ ] Add frontier exploration, failure handling, and map saving.
- [ ] Validate full missions in simulation, then on hardware.

Measure return error and test blocked routes, sensor loss, and cancellation. Before hardware autonomy, verify calibration, speed limits, motor command timeout, and emergency stop. A horizontal LiDAR does not reliably detect drop-offs.

## Contributors and workflow

- **[Sabari Vasan Jayabarathi / sabarivasangithub]**
- **[Abdulla Fadly Mohamed ajuwath / GitHub username]**
  
##Stay tuned!
