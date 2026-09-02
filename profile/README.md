# Cooperative Slung-Load UAV Software Architecture
A ROS 2 and PX4-based distributed software framework designed for cooperative cable-suspended payload transportation using multi-rotor UAV formations. This project provides a full-stack solution encompassing physical simulation, distributed swarm communication middleware, high-level formation control, and multi-agent validation pipelines in both Software-In-The-Loop (SITL) and Hardware-In-The-Loop (HITL) environments.

Developed at Sapienza University of Rome (Department of Aerospace and Mechanical Engineering).

![SITL & HITL physical simulation environment](./figures/mission.png)

# Key System Features

- Custom Physics Simulator
  Real-time dynamic simulation integrated with PX4 Autopilot for SITL & HITL validation. Supports multi-instance quadcopter flights and cooperative slung-load configurations via multibody dynamics.

- Decentralized Formation Strategy:
  Computes distributed optimal control laws locally on each UAV, eliminating single-point-of-failure architectures while tracking global reference trajectories.

- Hierarchical Nested-Loop Control:
  Decouples fleet-level cooperative maneuvers (outer loop) from individual flight stabilization (inner loop), guaranteeing system-wide stability during complex tasks.

- ROS2 & DDS-Native Execution:
  Leverages ROS2's distributed DDS middleware for modular, peer-to-peer inter-agent communication, treating each quadrotor as an autonomous computational node.

![Control Architecture](./figures/control_architecture.png)

# Useful Resources
- ![ROS2 Documentation](https://docs.ros.org/en/humble/index.html)
- ![PX4 Autopilot Documentation](https://docs.px4.io/main/en/)
- 
<!--

**Here are some ideas to get you started:**

🙋‍♀️ A short introduction - what is your organization all about?
🌈 Contribution guidelines - how can the community get involved?
👩‍💻 Useful resources - where can the community find your docs? Is there anything else the community should know?
🍿 Fun facts - what does your team eat for breakfast?
🧙 Remember, you can do mighty things with the power of [Markdown](https://docs.github.com/github/writing-on-github/getting-started-with-writing-and-formatting-on-github/basic-writing-and-formatting-syntax)
-->
