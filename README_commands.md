## Setup and usage

Target environment: **Ubuntu 24.04, ROS 2 Jazzy, Gazebo Harmonic**. These commands assume ROS 2 Jazzy and its apt repository are already installed. Run each section when needed; keep long-running demos in their own terminals.

### 1. Install tools and dependencies

```bash
sudo apt update
sudo apt install git gh build-essential python3-colcon-common-extensions python3-rosdep \
  ros-jazzy-demo-nodes-cpp ros-jazzy-demo-nodes-py \
  ros-jazzy-ros-gz ros-jazzy-navigation2 ros-jazzy-nav2-bringup \
  ros-jazzy-slam-toolbox
```

Load ROS in **every new terminal**:

```bash
source /opt/ros/jazzy/setup.bash
```

Check the environment:

```bash
cat /etc/os-release
echo "$ROS_DISTRO"
gz sim --versions
```

### 2. Connect GitHub and clone the project

For a new computer, authenticate through the browser, then configure Git authentication:

```bash
gh auth login --hostname github.com --git-protocol https --web
gh auth setup-git
```

Clone once; skip cloning if this folder already contains your repository:

```bash
git clone https://github.com/sabarivasangithub/micro-autonomous-exploration-robot.git ~/micro-autonomous-exploration-robot
cd ~/micro-autonomous-exploration-robot
```

Set commit identity once for this repository, replacing the placeholders:

```bash
git config user.name "Your Name"
git config user.email "your-email@example.com"
git remote -v
```

Before adding project files, create a working branch once:

```bash
git switch -c feature/first-map
```

If that branch already exists, use `git switch feature/first-map`.

### 3. Test ROS communication

Publisher — terminal 1:

```bash
source /opt/ros/jazzy/setup.bash
ros2 run demo_nodes_cpp talker
```

Subscriber — terminal 2:

```bash
source /opt/ros/jazzy/setup.bash
ros2 run demo_nodes_py listener
```

The listener should print messages from the talker on `/chatter`. Stop each program with **Ctrl + C**.

### 4. Navigate using the demo map

```bash
source /opt/ros/jazzy/setup.bash
ros2 launch nav2_bringup tb3_simulation_launch.py headless:=False
```

In RViz, use **2D Pose Estimate** to set the robot's approximate starting position and heading. Once navigation is active, use **Nav2 Goal** to choose a destination in free space.

### 5. Build a new map with SLAM

Stop the previous simulation with **Ctrl + C** and wait for it to exit before launching mapping mode:

```bash
source /opt/ros/jazzy/setup.bash
ros2 launch nav2_bringup tb3_simulation_launch.py headless:=False slam:=True use_sim_time:=True
```

For a fresh SLAM session, no **2D Pose Estimate** is needed. Send nearby **Nav2 Goal** destinations in known free space, one at a time, and watch the map grow. Goal selection is manual at this stage; autonomous exploration and return home remain planned.

### 6. Save the map

Keep the mapping simulation running. In a second terminal:

```bash
source /opt/ros/jazzy/setup.bash
cd ~/micro-autonomous-exploration-robot
mkdir -p maps
ros2 run nav2_map_server map_saver_cli -f maps/first_map
ls maps
```

Expected outputs: `first_map.pgm` (map image) and `first_map.yaml` (map metadata). Reusing `first_map` replaces that saved map; choose a different name to keep another version.

### 7. Save changes to GitHub

From the project folder, check the branch and edits:

```bash
git branch --show-current
git status
git diff
```

After both map files save successfully, commit and push from `feature/first-map`:

```bash
git add maps/first_map.pgm maps/first_map.yaml
git commit -m "Add first map from simulated robot"
git push -u origin feature/first-map
```

For README edits, stage `README.md` with `git add README.md` before committing. `git diff` shows unstaged edits to tracked files; `git diff --staged` shows changes selected for the next commit.

Open a pull request on GitHub to merge the working branch into `main`. After merging, return to a clean local working tree and update:

```bash
git switch main
git pull --ff-only origin main
git status
```

References: [Nav2 quickstart](https://docs.nav2.org/jazzy/getting_started/quickstart/quickstart/), [mapping with SLAM](https://docs.nav2.org/jazzy/tutorials/general_tutorials/navigation2_with_slam/navigation2_with_slam/), and [GitHub flow](https://docs.github.com/en/get-started/using-github/github-flow).
