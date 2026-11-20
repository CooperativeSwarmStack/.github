# 802.11s WiFi Mesh with the ALFA AWUS036ACM on Seeed Studio reComputer Industrial J4012

> **Source note.** This guide is an adaptation, for NVIDIA Jetson nodes on Seeed Studio reComputer Industrial J4012, of the article [*ALFA AWUS036ACM: IBSS Ad Hoc and 802.11s Mesh on Raspberry Pi*](https://yupitek.com/en/blog/awus036acm-ibss-mesh-raspberry-pi/) published by Yupitek. The operating principles, commands and values come from the original article, which is written for Raspberry Pi. Items not yet verified on Jetson are listed in the [TODO](#12-todo-items-to-verify) section.

This guide explains how the **MediaTek MT7612U** chipset inside the **ALFA AWUS036ACM** USB dongle makes it possible to build a WiFi mesh network between multiple nodes, with no router or access point. In this case the nodes are **NVIDIA Jetson devices on Seeed Studio reComputer Industrial J4012**.

## Table of Contents

1. [Why a specific adapter is needed](#1-why-a-specific-adapter-is-needed)
2. [The MT7612U chipset and the mt76x2u driver](#2-the-mt7612u-chipset-and-the-mt76x2u-driver)
3. [AWUS036ACM hardware specifications](#3-awus036acm-hardware-specifications)
4. [IBSS (Ad Hoc) vs. 802.11s Mesh](#4-ibss-ad-hoc-vs-80211s-mesh)
5. [Verifying the driver on the Jetson node](#5-verifying-the-driver-on-the-jetson-node)
6. [Setting up the 802.11s mesh](#6-setting-up-the-80211s-mesh)
7. [Optional: Layer 2 routing with batman-adv](#7-optional-layer-2-routing-with-batman-adv)
8. [Persistence across reboots](#8-persistence-across-reboots)
9. [Alternative: IBSS Ad Hoc mode](#9-alternative-ibss-ad-hoc-mode)
10. [Use case: drone / robot cluster](#10-use-case-drone--robot-cluster)
11. [FAQ and troubleshooting](#11-faq-and-troubleshooting)
12. [TODO: items to verify](#12-todo-items-to-verify)

---

## Introduction

IEEE 802.11s is an extension to the IEEE 802.11 standard which allows multiple wireless nodes to connect with each other without having an AP between them. If you have an AP at your home, and you want to copy a file to another laptop B which is connected to the same AP as your own laptop (laptop A), then A transmits the data to the AP, which in turn, transmits it to laptop B. So all the communication between A and B is done via the AP.

IEEE 802.11s, however, allows direct communication between A and B without the need for an AP. However, the real power of IEEE 802.11s manifests itself in the presence of multiple wireless nodes. Using the 802.11s mesh standard, the nodes can form a multi-hop network where all the links of the network are wireless. This means that no wired infrastructure is needed to setup the network. This is very handy in a lot of scenarios, for example, disaster relief teams can carry laptops with IEEE 802.11s support. This would enable them to exchange information with other laptops in the area even if those are not in direct range.

IEEE 802.11 standard defines a Wireless Distribution System. Typical implementations of a WDS are found at universities for example, where multiple APs are installed throughout the campus advertising a single SSID. The APs are connected together using a wired network such as Ethernet, but users can connect to the APs via a wireless connection. Think of IEEE 802.11s as replacing wires with wireless links in the above scenario. That removes the infrastructure cost such as laying wires and maintaining them. Also, it enables flexibility in terms of moving the APs or installing new APs.

## 1. Why a specific adapter is needed

Building a WiFi network between nodes without a router, or a mesh that automatically routes traffic through intermediate nodes, is not possible with most USB WiFi adapters: the kernel driver does not expose the necessary modes.

The **IBSS** and **Mesh Point** modes require the driver to be built on **mac80211**, the official 802.11 software MAC layer of the Linux kernel. Only mac80211-based drivers expose them as usable interface types.

Many USB adapters instead use out-of-kernel drivers with a simplified wireless stack that omits IBSS and mesh support. Even some in-kernel drivers for newer chipsets do not expose these modes.

The **MT7612U** is one of the few chipsets where the answer is a clean yes to both.

## 2. The MT7612U chipset and the mt76x2u driver

The AWUS036ACM uses the MediaTek MT7612U chipset, handled by the **`mt76x2u`** driver, part of the **mt76** driver family maintained by MediaTek and integrated into the mainline Linux kernel.

- **In the kernel since version 4.19** (October 2018)
- Implements the complete Linux mac80211 interface, so it natively supports both IBSS (Ad Hoc) and 802.11s Mesh Point

Driver stack:

```
Your application (ping, iperf3, batman-adv...)
        ↓
nl80211 / cfg80211  ← kernel WiFi configuration API
        ↓
mac80211            ← IEEE 802.11 software MAC layer
        ↓
mt76x2u             ← MT7612U USB hardware driver
        ↓
MediaTek MT7612U    ← Physical chip (2×2 MIMO, USB 3.0)
```

mac80211 implements the full 802.11 state machine: beacon management, frame construction, power save, IBSS peer discovery, and mesh path routing. Drivers built on top of mac80211 automatically inherit all of these capabilities.

### Supported interface modes

| Mode | Supported | Description |
|---|---|---|
| IBSS | ✅ | Ad Hoc peer-to-peer, no AP required |
| Managed (Station) | ✅ | Standard client mode |
| AP | ✅ | Software Access Point (hostapd) |
| AP/VLAN | ✅ | Virtual LAN over AP |
| Monitor | ✅ | Passive capture + packet injection |
| Mesh Point | ✅ | 802.11s multi-hop mesh networking |
| P2P-client | ✅ | Wi-Fi Direct client |
| P2P-GO | ✅ | Wi-Fi Direct Group Owner |

## 3. AWUS036ACM hardware specifications

| Parameter | Value |
|---|---|
| Chipset | MediaTek MT7612U |
| WiFi Standard | IEEE 802.11 a/b/g/n/ac (WiFi 5) |
| Frequency Bands | 2.4 GHz (2.412–2.472 GHz) + 5 GHz (5.15–5.825 GHz) |
| Channel Widths | 20 / 40 / 80 MHz |
| Max Speed (5 GHz) | 867 Mbps (802.11ac) |
| Max Speed (2.4 GHz) | 300 Mbps (802.11n) |
| Combined Speed | AC1200 (867 + 300 Mbps) |
| USB Interface | USB 3.0 Type-A (backwards compatible with USB 2.0) |
| Antennas | 2× RP-SMA female connectors, 2× 5 dBi dual-band dipole (detachable) |
| USB VID/PID | 0e8d:7612 |

Transmit power:

| Standard | TX Power |
|---|---|
| 802.11a | 20 dBm |
| 802.11b | 23 dBm |
| 802.11g | 23 dBm |
| 802.11n | 21 dBm |
| 802.11ac | 20 dBm |

Receive sensitivity:

| Standard | Sensitivity |
|---|---|
| 802.11a | −92 dBm |
| 802.11b | −97 dBm |
| 802.11g | −90 dBm |
| 802.11n | −90 dBm |
| 802.11ac | −86 dBm |

## 4. IBSS (Ad Hoc) vs. 802.11s Mesh

### IBSS — Independent Basic Service Set

The IEEE 802.11 specification for peer-to-peer wireless networking with no central infrastructure.

- Devices communicate directly with each other, with no AP or router involved
- Any two devices on the same SSID and channel can exchange data
- The network is self-contained: no internet connection is needed
- IPs are assigned statically (or via a simple DHCP daemon you run yourself)

**Limitations:**

- All nodes must be within direct radio range of each other: **there is no automatic multi-hop forwarding**
- Standard WPA2 encryption is **not available** in IBSS mode (security is done at the application layer)
- Scales practically to 3–5 nodes before performance degrades

### 802.11s Mesh Point

A separate IEEE amendment that defines true wireless mesh networking. Unlike IBSS, an 802.11s mesh:

- Automatically discovers neighbouring nodes and builds a routing table
- Forwards traffic through intermediate hops to reach nodes that are out of direct range
- Self-heals when a node disappears: other paths are used automatically
- Uses the Hybrid Wireless Mesh Protocol (**HWMP**) for path selection by default

```
  [Node A] ──── [Node B] ──── [Node C]
                   │
                [Node D]
```

Node A reaches C and D via B, automatically. If B fails, the mesh attempts to find alternate routes.

802.11s is the right choice when you have more than 3–4 nodes, when nodes are spread across a larger area, or when you want the network to be resilient without manual management.

### Comparison

| Feature | IBSS (Ad Hoc) | 802.11s Mesh Point |
|---|---|---|
| IEEE Standard | 802.11 IBSS | 802.11s |
| Multi-hop routing | ❌ Manual only | ✅ Automatic (HWMP) |
| Node range | All in direct range | Extends beyond single hop |
| Scalability | 3–5 nodes practical | 10+ nodes with batman-adv |
| Setup complexity | Low | Medium |
| Self-healing | ❌ No | ✅ Yes |
| Best for | Simple 2–3 node setups | Larger distributed systems |

## 5. Verifying the driver on the Jetson node

After plugging the AWUS036ACM into a USB port of the reComputer J4012, run:

```bash
lsusb
```

You should see the entry:

```
Bus 001 Device 003: ID 0e8d:7612 MediaTek Inc. MT7612U 802.11a/b/g/n/ac Wireless Adapter
```

Confirm the driver is active:

```bash
dmesg | grep mt76
```

Expected output (condensed):

```
mt76x2u 1-1.4:1.0: ASIC revision: 76120044
mt76x2u 1-1.4:1.0: Firmware Version: 0.1
mt76x2u 1-1.4:1.0: loaded firmware from mediatek/mt7662u_rom_patch.bin
```

List the interfaces:

```bash
iw dev
```

If the node already has a built-in WiFi interface (`wlan0`), the adapter will typically appear as `wlan1`. In all commands below, replace `wlan1` and `phy1` with the actual names on your system.

Check the supported modes:

```bash
iw phy phy1 info | grep -A 20 "Supported interface modes"
```

Among others, you should see the `* mesh point` entry:

```
Supported interface modes:
     * IBSS
     * managed
     * AP
     * AP/VLAN
     * monitor
     * mesh point
     * P2P-client
     * P2P-GO
```

## 6. Setting up the 802.11s mesh

This example builds a three-node mesh on 5 GHz Channel 36. Each node automatically discovers the others and establishes paths. The steps must be run **on every node**.

**Step 1 — Identify the phy associated with the AWUS036ACM:**

```bash
iw dev
```

```
phy#1
    Interface wlan1
        ...
```

**Step 2 — Add a mesh point interface named `mesh0`:**

```bash
sudo iw phy phy1 interface add mesh0 type mp mesh_id JetsonMesh
```

This creates a virtual interface `mesh0` on the same physical radio as `wlan1`. The original `wlan1` can remain in managed mode (for example for internet access) simultaneously: this is VIF (Virtual Interface) in action.

> `mesh_id` must be identical on all nodes of the network. The name `JetsonMesh` can be replaced with an identifier of your choice.

**Step 3 — Set the operating channel (5 GHz Channel 36):**

```bash
sudo iw dev mesh0 set channel 36
```

**Step 4 — Bring the mesh interface up:**

```bash
sudo ip link set mesh0 up
```

**Step 5 — Assign a unique IP address per node:**

```bash
# Node 1:
sudo ip addr add 10.88.0.1/24 dev mesh0

# Node 2:
sudo ip addr add 10.88.0.2/24 dev mesh0

# Node 3:
sudo ip addr add 10.88.0.3/24 dev mesh0
```

### Verify mesh formation

Check that peers have been discovered:

```bash
iw dev mesh0 station dump
```

Example output (Node 1 sees Node 2 and Node 3):

```
Station xx:xx:xx:xx:xx:02 (on mesh0)
    inactive time:  120 ms
    rx bytes:       4820
    tx bytes:       3560
    signal:         -52 dBm
    tx bitrate:     300.0 MBit/s MCS 15 40MHz
    mesh plink:     ESTAB
    mesh local PS mode: ACTIVE

Station xx:xx:xx:xx:xx:03 (on mesh0)
    mesh plink:     ESTAB
```

Inspect the mesh routing paths:

```bash
iw dev mesh0 mpath dump
```

```
DEST ADDR         NEXT HOP          IFACE   SN      METRIC  QLEN    EXPTIME         DTIM    DRET    FLAGS
xx:xx:xx:xx:xx:02 xx:xx:xx:xx:xx:02 mesh0   32      1158    0       0       100     0       0x4
xx:xx:xx:xx:xx:03 xx:xx:xx:xx:xx:02 mesh0   18      2316    0       0       100     0       0x14
```

In this example Node 3's next hop is Node 2, meaning Node 1 reaches Node 3 through Node 2 automatically.

Ping across the mesh:

```bash
# From Node 1:
ping -c 4 10.88.0.3
```

## 7. Optional: Layer 2 routing with batman-adv

For production deployments, replacing the kernel's built-in HWMP with **batman-adv** provides more advanced routing, better performance under mobility, and compatibility with higher-level networking tools.

```bash
sudo apt install batctl
sudo modprobe batman-adv
```

Configure the mesh interface as a batman-adv slave:

```bash
# Bring mesh0 up without an IP first
sudo ip link set mesh0 up
sudo batctl if add mesh0

# Bring the batman interface up
sudo ip link set bat0 up

# Assign IP to the batman interface (not mesh0)
sudo ip addr add 10.88.0.1/24 dev bat0
```

Check the batman routing tables:

```bash
sudo batctl n    # Neighbours
sudo batctl o    # Originator table (all known nodes)
sudo batctl tg   # Translation table (MAC → IP mapping)
```

## 8. Persistence across reboots

Virtual interfaces created with `iw` **do not persist** across reboots. Create `/etc/systemd/system/mesh-point.service` on each node:

```ini
[Unit]
Description=802.11s Mesh Point Network
After=network.target

[Service]
Type=oneshot
RemainAfterExit=yes
ExecStartPre=/bin/sleep 5
ExecStart=/bin/bash -c '\
  iw phy phy1 interface add mesh0 type mp mesh_id RaspberryMesh && \
  iw dev mesh0 set channel 36 && \
  ip link set mesh0 up && \
  ip addr add 10.88.0.1/24 dev mesh0'
ExecStop=/bin/bash -c '\
  ip link set mesh0 down && \
  iw dev mesh0 del'

[Install]
WantedBy=multi-user.target
```

Enable it:

```bash
sudo systemctl daemon-reload
sudo systemctl enable mesh-point.service
sudo systemctl start mesh-point.service
```

> Change the IP address in the service file for each node (.1, .2, .3, etc.).

## 9. Alternative: IBSS Ad Hoc mode

For direct links between a few nodes, all within radio range of each other, IBSS can be used. Example for two nodes on 5 GHz (Channel 36):

```bash
# Stop NetworkManager / wpa_supplicant from interfering
sudo systemctl stop NetworkManager 2>/dev/null || true
sudo pkill wpa_supplicant 2>/dev/null || true

sudo ip link set wlan1 down
sudo iw dev wlan1 set type ibss
sudo ip link set wlan1 up
sudo iw dev wlan1 ibss join RaspberryMesh 5180
sudo ip addr add 192.168.88.1/24 dev wlan1    # Node 2: 192.168.88.2
```

Frequency reference:

| Frequency | Channel |
|---|---|
| 5180 MHz | 5 GHz, Channel 36 |
| 5200 MHz | 5 GHz, Channel 40 |
| 2412 MHz | 2.4 GHz, Channel 1 (better range, lower speed) |
| 2437 MHz | 2.4 GHz, Channel 6 |

Verify the link:

```bash
iw dev wlan1 link
ping -c 4 192.168.88.2
```

To route traffic between Node 1 and Node 3 through Node 2 in IBSS, enable IP forwarding on Node 2:

```bash
sudo sysctl -w net.ipv4.ip_forward=1
```

## 10. Use case: drone / robot cluster

**Scenario:** multiple units (drones or ground robots) need to share telemetry data and coordinate actions without going through a ground station.

**5 GHz band:** at AC1200 speeds (867 Mbps on 5 GHz), the AWUS036ACM has ample headroom for multiple simultaneous video streams, and the 5 GHz band avoids the crowded 2.4 GHz interference common in urban areas. In the original example each unit streams 1080p H.264 video at ~5 Mbps plus telemetry at <100 Kbps.

**2.4 GHz band:** offers greater range at the expense of speed (TX power up to 23 dBm on 802.11b/g).

**Antennas:** the two RP-SMA connectors are a standard size, so the included 5 dBi antennas can be replaced with high-gain directional antennas to extend range.

With the 802.11s mesh, when units move and the topology changes, paths are updated automatically (even more effectively with batman-adv), with no single point of failure.

## 11. FAQ and troubleshooting

**The `wlan1` interface does not appear.**
Run `iw dev` after plugging in the adapter. If it does not appear within 10 seconds, check `dmesg | grep -i mt76`. If the package is missing, install `iw` with `sudo apt install iw`.

**`iw dev wlan1 set type ibss` returns "Device or resource busy".**
NetworkManager or wpa_supplicant is holding the interface:

```bash
sudo systemctl stop NetworkManager 2>/dev/null || true
sudo pkill wpa_supplicant 2>/dev/null || true
```

**`iw phy phy1 interface add mesh0 type mp` returns "Operation not supported".**
The phy name may not be `phy1`. Run `iw dev` to check which phy the AWUS036ACM is on and use the correct one.

**IBSS is set up but ping fails.** Check that:

- both nodes are on the exact same frequency (e.g., both 5180)
- the nodes have different IP addresses in the same subnet
- no firewall is blocking ICMP (`sudo iptables -L`)
- `iw dev wlan1 link` shows the same IBSS cell ID (BSSID) on both nodes

**The `mesh0` interface disappears after reboot.**
Virtual interfaces created with `iw` do not persist: use the systemd service from section 8.

**Can I use WPA2 encryption over IBSS?**
No: WPA2-Personal (PSK) is not supported in IBSS mode by the Linux kernel. For a secured IBSS you can use application-layer encryption (WireGuard, OpenVPN, SSH tunnels). 802.11s mesh does support **SAE** (WPA3-style authentication) with wpa_supplicant.

**Can the adapter be in mesh/IBSS and managed mode simultaneously?**
Yes, thanks to VIF: `wlan1` can stay managed (for example for internet access) while a second interface is added for the node-to-node network:

```bash
sudo iw phy phy1 interface add adhoc0 type ibss
sudo ip link set adhoc0 up
sudo iw dev adhoc0 ibss join LocalNet 5180
sudo ip addr add 192.168.88.1/24 dev adhoc0
```

**What is the maximum range?**
With the included 5 dBi antennas: 20–50 m outdoors in open air on 5 GHz, and 50–100 m on 2.4 GHz. With high-gain directional antennas, range can reach several hundred metres in line-of-sight conditions.

## 12. TODO: items to verify

The source document is written for Raspberry Pi. The following points are not covered and must be verified on the reComputer Industrial J4012.

- [ ] **Driver on the NVIDIA kernel (L4T).** The "plug and play" claim relies on the mainline kernel and Raspberry Pi OS kernels. The Jetson kernel is an NVIDIA fork with its own configuration: verify that `mt76x2u`, `mac80211` and mesh point support are enabled (`modinfo mt76x2u`, `lsmod` after plugging in the dongle). If not, rebuild the kernel or the modules. Verify the same for `batman-adv`.
- [ ] **Firmware.** Verify that `mt7662u_rom_patch.bin` is present in `/lib/firmware/mediatek/`.
- [ ] **Regulatory domain.** On 5 GHz, Channel 36 in Italy has restrictions (indoor use). Check with `iw reg get` that the domain is `IT` and that the channel is usable for initiating transmission, otherwise mesh or IBSS may fail to start.
- [ ] **Interface and phy names.** If the J4012 has an integrated WiFi/LTE module, `wlan1` and `phy1` may differ.
- [ ] **J4012-specific data** (USB 3.0 ports, power supply): not in the source document, to be added from the Seeed datasheet.