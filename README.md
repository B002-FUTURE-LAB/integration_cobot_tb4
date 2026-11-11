# Cobot + TurtleBot 4 Integration Overlay

Third-layer package to visualize **TurtleBot 4** and **MyCobot 280** in the same coordinate frame (`map`) without altering their existing workspaces.

## Concept

- **TurtleBot 4** continues running natively on ROS 2 Jazzy, Gazebo Harmonic / physical hardware, Nav2, and FastDDS.
- **MyCobot 280** continues running on ROS 2 Galactic, MoveIt 2, CycloneDDS, and the Jetson Nano hardware bridge.
- This overlay adds:
  - A static transform `map -> mycobot_base_link` at `x=-0.338416188955307`, `y=1.1060110330581665`, `z=0.80`;
  - Visual 3D room furniture, floor, LiDAR-height map obstacle strips, and the MyCobot table at that position;
  - The MyCobot 280 URDF model inside the Jazzy RViz scene;
  - Real-time mirroring of MyCobot `/joint_states` over UDP into `/mycobot/joint_states` on the Jazzy graph.

## Anchor Coordinates

```text
map
└── mycobot_base_link
    x:   -0.338416188955307
    y:    1.1060110330581665
    z:    0.80
    yaw:  0.0
```

`z=0.80` represents the tabletop height where the MyCobot base is mounted.

## Quick Start

1. Start TurtleBot 4 (simulation or physical localization/Nav2):

```bash
cd ~/B002_Future_Lab_Bots/turtlebot4_jazzy
./sim/run_lab_world.sh true true 0.0 0.0 0.0
```

2. In another terminal, launch the host overlay:

```bash
cd ~/B002_Future_Lab_Bots/integration_cobot_tb4
./scripts/run_host_overlay.sh start
```

This command also starts the live joint-state UDP bridge:

```text
Galactic/container /joint_states -> UDP -> Jazzy/host /mycobot/joint_states
```

If TurtleBot 4 is running inside the `tb4_sim` container, use:

```bash
cd ~/B002_Future_Lab_Bots/integration_cobot_tb4
./scripts/install_overlay_into_tb4_container.sh
./scripts/launch_overlay_in_tb4_container.sh static
```

3. In TurtleBot 4's RViz, set `Fixed Frame = map` and add:
- `TF`
- `MarkerArray` on `/cobot_tb4_overlay/markers`
- `RobotModel` using `/mycobot/robot_description` to see the MyCobot URDF in the same RViz window.

## Dedicated Integrated 3D View Window

With TurtleBot 4 / Nav2 running on the host, open a dedicated 3D RViz window:

```bash
cd ~/B002_Future_Lab_Bots/integration_cobot_tb4
./scripts/run_3d_view.sh
```

This command:
- Reads the active map from `/map_server` (`yaml_filename`);
- Generates 3D obstacle geometry and semantic room furniture from the saved map;
- Publishes the static anchor `map -> mycobot_base_link`;
- Starts/restarts the MyCobot real joint-state bridge if the `mycobot_ros2` container is running;
- Opens a dedicated RViz window showing TurtleBot 4, MyCobot 280, and the 3D laboratory environment.

## MyCobot Joint-State Bridge

To manage only the UDP bridge that mirrors the real/MoveIt arm state into Jazzy:

```bash
cd ~/B002_Future_Lab_Bots/integration_cobot_tb4
./scripts/start_mycobot_joint_exporter.sh status
./scripts/start_mycobot_joint_exporter.sh start
./scripts/start_mycobot_joint_exporter.sh stop
```

This bridge does not require a ROS 2 distro bridge between Galactic and Jazzy; it serializes `sensor_msgs/JointState` as JSON over UDP and republishes `/mycobot/joint_states` in Jazzy while keeping arm control in the original MoveIt 2 stack.

## Scope & Boundaries

- Does not modify TurtleBot 4 Nav2, Gazebo, or mission routines.
- Does not modify MyCobot MoveIt 2, hardware bridge, or visual servoing.
- Provides a clean, distro-independent 3D visualization and TF integration in RViz.
