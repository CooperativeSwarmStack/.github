# Jetson Platform Configuration Reference
**File path:** `docs/jetson_configuration.md`  
**Document Version:** 1.0.0  
**Last Updated:** 2026-10-18  
**Status:** Active Reference Document  
---
## 1. Hardware Platform Overview

| Parameter | Technical Specification |
| :--- | :--- |
| **System Model** | Seeed Studio reComputer Industrial J4012 |
| **NVIDIA SoM Module** | NVIDIA Jetson Orin™ NX (16 GB) |
| **AI Performance** | Up to 100 TOPS |
| **GPU Architecture** | 1024-core NVIDIA Ampere with 32 Tensor Cores |
| **CPU Architecture** | 8-core Arm® Cortex®-A78AE v8.2 64-bit (2 MB L2 + 4 MB L3) |
| **System RAM** | 16 GB 128-bit LPDDR5 (102.4 GB/s bandwidth) |
| **Carrier Board** | Seeed Studio reComputer Industrial J4012 Carrier Board |
| **Primary Mass Storage** | 128 GB NVMe M.2 2280 SSD (PCIe Gen 4.0 x4, M.2 Key M slot) |
| **In Flight Power Supply** | 12 V DC nominal, fed directly via a buck step-down converter connected to the drone's Power Distribution Board (PDB) |
| **Bench Power Supply** | External 11 V – 19 V DC (5 A current-limited) supply connected via 2-pole terminal block |

---
## 2. Additional Hardware Modules & Peripherals
* **LTE / 4G Cellular Connectivity Module:** Installed in the **M.2 Key B** slot and linked to the onboard Nano-SIM card slot. Provides long-range cellular telemetry, remote control links, and cloud/swarm data routing.
* **Stereo Depth Camera:** **Intel® RealSense™ D435i** connected via USB 3.2 Gen1. Used for 3D depth perception, visual odometry/SLAM, and obstacle avoidance.
* **External USB Extension Cable:** Mounted to provide external chassis access to a secondary USB 3.2 port for bench debugging, firmware updates, and direct log downloads.
* **Flight Controller Interface:** **Pixhawk 6X** Autopilot connected via standard RJ-45 Gigabit Ethernet (non-PoE) for high-bandwidth MAVLink and ROS 2 communication.
---
## 3. Ports and Interfaces Status Table

| Physical Interface / Port | Available | In Use | Connected Device | Purpose / Function Description |
| :--- | :--- | :--- | :--- | :--- |
| **Ethernet LAN2 (RJ-45 GbE, Non-PoE)** | Yes | **Yes** | Pixhawk 6X Autopilot | High-speed telemetry link (MAVLink / Micro-XRCE-DDS / ROS 2) |
| **Ethernet LAN1 (RJ-45 GbE, PoE PSE)** | Yes | **No** | *Unused* | Auxiliary network interface / Optional IP camera connection |
| **USB 3.2 Gen1 (Port 1)** | Yes | **Yes** | Intel® RealSense™ D435i Camera | RGB-D point cloud streaming, visual-inertial odometry & SLAM |
| **USB 3.2 Gen1 (Port 2)** | Yes | **Yes** | External USB Cable Extension | External maintenance access for field updates and USB drive logs |
| **USB 3.2 Gen1 (Port 3)** | Yes | **No** | *Unused* | Auxiliary USB expansion port |
| **USB 2.0 Type-C (Device Mode)** | Yes | **No** | *Unused* | System flashing, recovery mode, and low-level diagnostic access |
| **USB 2.0 Type-C (Debug UART)** | Yes | **No** | *Unused* | Onboard RP2040 serial console debugging |
| **M.2 Key M (2280 PCIe Gen 4.0 x4)** | Yes | **Yes** | 256 GB NVMe SSD | OS root filesystem, ROS 2 workspace, logs, and AI models |
| **M.2 Key B (3042/3052 PCIe/USB3)** | Yes | **Yes** | 4G/LTE Cellular Module | Cellular modem interface for mobile telemetry and swarm networking |
| **Nano-SIM Card Slot** | Yes | **Yes** | SIM Card | Mobile network SIM card for LTE modem data connection |
| **Mini PCIe / M.2 Key E Slot** | Yes | **No** | *Unused* | Secondary expansion slot (e.g., Wi-Fi/Bluetooth or LoRa) |
| **COM DB9 Port (RS-232/422/485)** | Yes | **No** | *Unused* | Industrial serial communications port |
| **CAN Bus Port (Terminal Block)** | Yes | **No** | *Unused* | CAN bus protocol integration / Telemetry passthrough |
| **16-Pin Terminal Block (DIO)** | Yes | **No** | *Unused* | Digital Input/Output (4x DI, 4x DO) for payload triggers and indicators |
| **HDMI 2.0 / 2.1 Type-A** | Yes | **No** | *Unused* | Display output for bench development and setup |
| **MIPI CSI Connectors (2x 2-lane)** | Yes | **No** | *Unused* | Direct MIPI camera inputs (unused; depth perception handled via USB RealSense) |
| **2-Pin Terminal / DC Jack (Power In)** | Yes | **Yes** | Drone PDB Step-Down (12 V) | Main 12 V DC power supply input from avionics power bus |

---
## 4. Software Configuration & Stack

| Software Component | Version / Distribution | Notes & Details |
| :--- | :--- | :--- |
| **Operating System** | Ubuntu 22.04 LTS (Jammy Jellyfish) | 64-bit ARM (`aarch64`) architecture |
| **Jetson Board Support Package (BSP)** | Custom Seeed Studio BSP | Dedicated carrier board support package for reComputer Industrial J4012 |
| **NVIDIA JetPack** | JetPack 6.0 | Custom build targeting the Seeed carrier hardware |
| **NVIDIA L4T (Linux for Tegra)** | R36.3.0 | Tegra hardware acceleration drivers and system software |
| **Linux Kernel** | `5.15.x-tegra` | Custom Tegra Linux kernel with real-time patch support |
| **CUDA Toolkit** | CUDA 12.2 | Parallel computing framework for onboard AI tasks |
| **cuDNN** | cuDNN 8.9.x | Deep learning primitives library |
| **TensorRT** | TensorRT 8.6.x | High-performance inference engine for Jetson Orin NX |
| **ROS 2 Distribution** | ROS 2 Humble Hawksbill | Primary robotics middleware for perception and navigation nodes |
| **Camera Drivers Stack** | `librealsense2` + `realsense2_camera` | Intel RealSense SDK 2.0 and official ROS 2 camera node wrapper |
| **Cellular Network Stack** | `ModemManager` / `NetworkManager` / `qmi_wwan` | LTE modem communication stack and network auto-reconnect services |
| **Autopilot Communications Bridge** | `micro_xrce_dds_agent` / `mavros2` | High-speed Ethernet bridge to Pixhawk 6X autopilot |

---
## 5. Maintenance & Version Control Requirements
This document **must** be updated and committed to version control whenever:
1. **Hardware Layout Changes:** Replacing the M.2 LTE module, upgrading SSD capacity, or adding secondary camera payloads.
2. **Wiring / Interface Routing Changes:** Modifying USB port allocations, changing Ethernet IP assignments, or re-routing power terminals.
3. **Software Environment Changes:** Re-flashing JetPack, upgrading Ubuntu packages, kernel modifications, or ROS 2 workspace updates.
