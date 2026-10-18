# Jetson Platform Configuration Reference
**File path:** `docs/jetson_configuration.md`  
**Document Version:** 1.1.0  
**Last Updated:** 2026-10-19  
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
| **Thermal Design** | Fanless enclosure with passive heatsink (-20°C to 60°C operating range) |
| **In-Flight Power Supply** | 12 V DC nominal, fed directly via a buck step-down converter connected to the drone's Power Distribution Board (PDB) |
| **Bench Power Supply** | External 12 V – 24 V DC supply connected via 2-pin terminal block |

---

## 2. Hardware Modules & Peripherals

- **LTE / 4G Cellular Connectivity Module:** Installed in the **Mini PCIe** slot and linked to the onboard Nano-SIM card slot for cellular data transmission.
- **Stereo Depth Camera:** **Intel® RealSense™ D435i** connected via USB 3.2 Gen1 (Port 1). Can be used for 3D depth perception, visual odometry/SLAM, and obstacle avoidance.
- **External USB Extension Cable:** Mounted to USB 3.2 Gen1 (Port 2) to provide external chassis access for bench debugging, firmware updates, and direct log downloads.
- **Flight Controller Interface:** **Pixhawk 6X** Autopilot connected via standard RJ-45 Gigabit Ethernet (LAN2) for high-bandwidth MAVLink and ROS 2 communication.
- **Real-Time Clock (RTC):** Integrated RTC powered by an installed CR1220 coin battery for system timestamp preservation during power-off states.

---

## 3. Ports and Interfaces Status Table
<img width="4012" height="1300" alt="image" src="https://github.com/user-attachments/assets/a25c138f-18a2-4b60-8fbf-127586553143" />
<img width="3944" height="1496" alt="image" src="https://github.com/user-attachments/assets/caa0e3a1-7065-4fd8-8330-1313645c5aa9" />


| Physical Interface / Port | In Use | Connected Device | Purpose / Function Description |
| :--- | :--- | :--- | :--- |
| **Ethernet LAN2 (RJ-45 GbE, 10/100/1000 Mbps)** | **Yes** | Pixhawk 6X Autopilot | High-speed telemetry link (MAVLink / Micro-XRCE-DDS) |
| **Ethernet LAN1 (RJ-45 GbE, PoE PSE 802.3af 15W)** | **No** | *-* | Auxiliary network interface with Power over Ethernet capability (e.g., optional IP camera) |
| **USB 3.2 Gen1 (Type-A, Port 1)** | **Yes** | Intel® RealSense™ D435i Camera | eg. RGB-D point cloud streaming, visual-inertial odometry & SLAM |
| **USB 3.2 Gen1 (Type-A, Port 2)** | **Yes** | External USB Cable Extension | External maintenance access for field updates and USB drive logs |
| **USB 3.2 Gen1 (Type-A, Port 3)** | **No** | *-* | Auxiliary USB 3.2 expansion port |
| **USB 2.0 Type-C (Device Mode)** | **No** | *-* | System flashing, recovery mode, and low-level recovery access |
| **USB 2.0 Type-C (Debug UART & RP2040)** | **No** | *-* | Onboard RP2040 serial console debugging and low-level diagnostic access |
| **M.2 Key M (2280 PCIe Gen 4.0 x4)** | **Yes** | 128 GB NVMe M.2 SSD | OS root filesystem |
| **Mini PCIe Slot** | **Yes** | 4G/LTE Cellular Module | Primary slot hosting the 4G/LTE modem for long-range communication |
| **M.2 Key B (3042/3052 PCIe/USB3)** | **No** | *-* | Secondary expansion slot for alternative 4G/5G cellular modems |
| **Nano-SIM Card Slot** | **Yes** | *-* | Mobile network SIM card connected to the Mini PCIe cellular modem |
| **SMD Wi-Fi / Bluetooth Interface** | **No** | *-* | Header/pad array for optional surface-mount Wi-Fi/Bluetooth module |
| **COM DB9 Port (RS-232/RS-422/RS-485)** | **No** | *-* | Switchable industrial serial communications port |
| **CAN Bus Port (Terminal Block)** | **No** | *-* | CAN bus protocol integration / Telemetry passthrough |
| **Digital I/O Terminal Block (4x DI, 4x DO, GNDs)** | **No** | *-* | Digital inputs/outputs (4x DI, 4x DO, isolated & non-isolated GNDs) for payload triggers and status signals |
| **HDMI 2.0 Type-A** | **No** | *-* | Video output interface for bench setup and debugging displays |
| **MIPI CSI Connectors (2x 2-lane 15-pin)** | **No** | *-* | Direct MIPI camera inputs (unused; depth perception handled via USB RealSense) |
| **DC 12V–24V Power Input (2-Pin Terminal Block)** | **Yes** | Drone PDB Step-Down (12 V) | Main DC power supply input from avionics power bus |
| **Fan Connector (5V PWM, 1x Header)** | **No** | *-* | Optional cooling fan power and PWM speed control header (passive cooling utilized) |
| **TPM 2.0 Connector (1x Header)** | **No** | *-* | Connection header for optional Trusted Platform Module (TPM 2.0) hardware security chip |
| **RTC Battery Socket (CR1220)** | **Yes** | CR1220 Coin Cell Battery | Power source for internal system RTC timekeeping |
| **RTC 2-Pin Header** | **No** | *-* | Auxiliary 2-pin connector for external RTC battery wiring |

---

## 4. Software Configuration & Stack

| Software Component | Version / Distribution | Notes & Details |
| :--- | :--- | :--- |
| **Operating System** | Ubuntu 22.04 LTS (Jammy Jellyfish) | 64-bit ARM (`aarch64`) architecture |
| **Jetson Board Support Package (BSP)** | Custom Seeed Studio BSP | Dedicated carrier board support package for reComputer Industrial J4012 |
| **NVIDIA JetPack** | JetPack 6.0 | Custom build targeting Seeed carrier hardware |
| **NVIDIA L4T (Linux for Tegra)** | R36.3.0 | Tegra hardware acceleration drivers and system software |
| **Linux Kernel** | `5.15.x-tegra` | Custom Tegra Linux kernel with real-time patch support |
| **CUDA Toolkit** | *-* | Parallel computing framework for onboard AI tasks |
| **cuDNN** | *-* | Deep learning primitives library |
| **TensorRT** | *-* | High-performance inference engine for Jetson Orin NX |
| **ROS 2 Distribution** | ROS 2 Humble Hawksbill | Primary robotics middleware for perception and navigation nodes |
| **Camera Drivers Stack** | *-* | Intel RealSense SDK 2.0 and official ROS 2 camera node wrapper |
| **Cellular Network Stack** | *-* | LTE modem communication stack and network auto-reconnect services |
| **Autopilot Communications Bridge** | `micro_xrce_dds_agent` | High-speed Ethernet bridge to Pixhawk 6X autopilot |

---

## 5. Maintenance & Version Control Requirements

This document **must** be updated and committed to version control whenever:
1. **Hardware Layout Changes:** Replacing the Mini PCIe LTE module, occupying the M.2 Key B slot, upgrading SSD capacity, or adding secondary camera payloads via MIPI CSI.
2. **Wiring / Interface Routing Changes:** Modifying USB port allocations, changing Ethernet IP assignments, activating digital I/O channels, or re-routing power terminals.
3. **Software Environment Changes:** Re-flashing JetPack, upgrading Ubuntu packages, kernel modifications, or ROS 2 workspace updates.
