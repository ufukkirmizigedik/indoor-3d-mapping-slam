# Indoor 3D Mapping (SLAM) — Livox Mid-360 + NVIDIA Jetson

Real-time 3D indoor mapping system using **Livox Mid-360 LiDAR** and **NVIDIA Jetson**, running **ROS2 Humble** with the **FAST-LIO** algorithm.

## Demo

[![Watch the demo](https://img.youtube.com/vi/Km7jlLZztEM/0.jpg)](https://youtu.be/Km7jlLZztEM)

---

## Hardware & Software

| Component | Details |
|---|---|
| Compute | NVIDIA Jetson Nano |
| Sensor | Livox Mid-360 LiDAR |
| OS | Ubuntu 22.04 LTS |
| Middleware | ROS2 Humble |
| Algorithm | [FAST-LIO (ROS2 port)](https://github.com/Ericsii/FAST_LIO_ROS2) |

---

## How It Works

```
Livox Mid-360 → Ethernet → Jetson → ROS2 → FAST-LIO → RViz (real-time 3D map)
```

FAST-LIO (Fast LiDAR-Inertial Odometry) fuses LiDAR point cloud data with IMU to build a real-time 3D map of the environment.

---

## Setup Guide

### 1. Network Configuration

The Livox Mid-360 communicates via Ethernet. Assign a static IP to the Jetson:

```bash
sudo ifconfig enP8p1s0 192.168.1.50 netmask 255.255.255.0 up
```

Verify connection:
```bash
ping 192.168.1.160
```

---

### 2. Install Livox ROS2 Driver

```bash
mkdir -p ~/livox_ws/src
cd ~/livox_ws/src
git clone https://github.com/Livox-SDK/livox_ros_driver2.git
```

**⚠️ Important — Patch CMakeLists.txt:**

There is a known CMake `LIVOX_INTERFACES_INCLUDE_DIRECTORIES` path bug in ROS2 Humble.  
Open `~/livox_ws/src/livox_ros_driver2/CMakeLists.txt` and replace the `get_target_property` line with:

```cmake
set(LIVOX_INTERFACES_INCLUDE_DIRECTORIES
    "${CMAKE_CURRENT_BINARY_DIR}/rosidl_generator_c"
    "${CMAKE_CURRENT_BINARY_DIR}/rosidl_generator_cpp")
```

**Configure IP Address:**

Edit `~/livox_ws/src/livox_ros_driver2/config/MID360_config.json`:
- Set `"ip"` inside `lidar_configs` → `"192.168.1.160"`
- Set all IPs under `"host_net_info"` → `"192.168.1.50"`

**Build:**

```bash
cd ~/livox_ws
colcon build --cmake-args -DROS_EDITION=ROS2 -DHUMBLE_ROS=HUMBLE
```

---

### 3. Install FAST-LIO

```bash
sudo apt update
sudo apt install -y libgoogle-glog-dev

mkdir -p ~/fastlio_ws/src
cd ~/fastlio_ws/src
git clone https://github.com/Ericsii/FAST_LIO_ROS2.git
cd FAST_LIO_ROS2

# Critical: initialize submodules (ikd-Tree), otherwise build will fail
git submodule update --init --recursive

cd ~/fastlio_ws
source ~/livox_ws/install/setup.bash
colcon build --symlink-install --cmake-args -DROS_EDITION=ROS2 -DHUMBLE_ROS=HUMBLE
```

---

### 4. Run the System

**Terminal 1 — Start Livox driver:**
```bash
source ~/livox_ws/install/setup.bash
ros2 launch livox_ros_driver2 msg_MID360_launch.py
```

**Terminal 2 — Start FAST-LIO + RViz:**
```bash
source ~/fastlio_ws/install/setup.bash
ros2 launch fast_lio mapping.launch.py config_file:=mid360.yaml
```

RViz opens automatically and shows real-time 3D point cloud generation.  
Check odometry data:
```bash
ros2 topic echo /Odometry
```

---

## Notes

- This guide includes a patch for a known CMake bug in the Livox driver when compiling under ROS2 Humble
- Submodule initialization (`git submodule update --init --recursive`) is critical — skipping it causes missing `ikd-Tree` source errors
- Tested on NVIDIA Jetson Nano with Ubuntu 22.04

---

## Author

**Ufuk Kırмızıgedik**  
Electronics Engineer & Automation Developer  
📧 ufukkirmizigedik1984@gmail.com  
💬 Telegram: [@K_Ufuk](https://t.me/K_Ufuk)
