# ROSMASTER X3 PLUS — Robot Project

Personal robotics project built on the Yahboom ROSMASTER X3 PLUS platform.

## Goals
- [ ] Person following via YOLOv8 + depth camera + PID control
- [ ] Object pickup (marker) triggered by voice command
- [ ] Deliver object to person on request

## Hardware
| Component | Details |
|---|---|
| Platform | Yahboom ROSMASTER X3 PLUS |
| Compute | Jetson Orin Nano / Orin NX |
| LiDAR | YDLiDAR (A1) |
| Drive | Mecanum wheels (omnidirectional) |
| Arm | 6DOF robotic arm |
| Voice | Onboard voice recognition module |

## Network
| Item | Value |
|---|---|
| Jetson username | ketson |
| SSH command | ssh ketson@192.168.1.11 |

If IP changed run on Jetson: ip addr show wlP1p1s0

If robot reverts to hotspot mode:
sudo nmcli connection down ROSMASTER
sudo nmcli connection up "HomeWifiName"

## Workspaces
Two separate ROS2 workspaces on the Jetson:

Main: ~/yahboomcar_ros2_ws/yahboomcar_ws
Library: ~/yahboomcar_ros2_ws/software/library_ws

Source before running commands:
source ~/yahboomcar_ros2_ws/yahboomcar_ws/install/setup.bash
source ~/yahboomcar_ros2_ws/software/library_ws/install/setup.bash

## Launch Commands

### Tab 1 — Robot Base
cd ~/yahboomcar_ros2_ws/yahboomcar_ws && source install/setup.bash
ros2 launch yahboomcar_bringup yahboomcar_bringup_X3_launch.py

Expected: IMU free fall warnings — normal, ignore them.

### Tab 2 — LiDAR
cd ~/yahboomcar_ros2_ws/software/library_ws && source install/setup.bash
ros2 launch ydlidar_ros2_driver ydlidar_launch.py

Expected: Now lidar is scanning...

If LiDAR crashes: reseat USB cable, verify ls -la /dev/ydlidar points to ttyUSB0, relaunch.

### Tab 3 — SLAM Mapping
cd ~/yahboomcar_ros2_ws/yahboomcar_ws && source install/setup.bash
ros2 launch yahboomcar_nav map_gmapping_a1_launch.py

Expected: Registering Scans: Done repeating.

### Tab 4 — Save Map
cd ~/yahboomcar_ros2_ws/yahboomcar_ws && source install/setup.bash
ros2 run nav2_map_server map_saver_cli -f ~/maps/home_room

### Copy Map to Mac
Run on Mac terminal:
scp ketson@192.168.1.11:~/maps/home_room.pgm ~/Desktop/home_room.pgm

## Mapping Tips
- Drive slowly
- Hug walls within 1-2 meters
- Cover edges first then middle
- Go through every doorway
- Full living room + hallway + kitchen takes ~10 minutes
- Restart all nodes fresh if LiDAR crashes mid-run

## Battery
| State | Voltage |
|---|---|
| Full charge | 12.6V |
| Healthy | 11.1V - 12.4V |
| Charge soon | 10.5V - 11.0V |
| Critical | below 9.6V |

Charger: CHP-12620 wall adapter 12.6V 2A barrel jack
Charge time from ~10.5V: approximately 1-1.5 hours
Always unplug Deans connector from robot before charging.

## Known Issues
| Issue | Root Cause | Fix |
|---|---|---|
| Continuous beeping + motor lockout | Battery below 9.6V | Charge battery |
| LiDAR timeout/deadlock | USB connection drop | Reseat cable, relaunch |
| RViz fails to open | No display on Jetson | Use UTM VM on Mac (TODO) |
| laser_launch.py not found | Wrong package | Use ydlidar_ros2_driver instead |

## Correct Launch File Names
| Purpose | Package | Launch file |
|---|---|---|
| Robot bringup | yahboomcar_bringup | yahboomcar_bringup_X3_launch.py |
| LiDAR driver | ydlidar_ros2_driver | ydlidar_launch.py |
| SLAM mapping | yahboomcar_nav | map_gmapping_a1_launch.py |

## TODO
- [ ] Set up UTM VM bridged networking for RViz on Mac
- [ ] Redo SLAM map (first attempt too noisy, drove too fast)
- [ ] Set up YOLOv8 person detection node
- [ ] Build PID follow controller
- [ ] Decide arm control approach (ros1_bridge vs direct serial)
- [ ] Train YOLO on marker for pickup task
- [ ] Integrate voice command trigger
