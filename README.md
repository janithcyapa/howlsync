<div align="center">
  <img src="docs/assets/howlsync-logo.png" alt="HowlSync Logo" width="300" />

  # HowlSync
  **Decentralized Swarm Communication & Spatial Localization**

  [![License](https://img.shields.io/badge/License-Apache_2.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)
  [![Framework](https://img.shields.io/badge/Framework-ESP--IDF-red.svg)](https://docs.espressif.com/projects/esp-idf/en/latest/esp32c6/)
  [![Hardware](https://img.shields.io/badge/Hardware-ESP32--C6-black.svg)](https://www.espressif.com/en/products/socs/esp32-c6)
  [![Ecosystem](https://img.shields.io/badge/Ecosystem-ArcticHowl-00E5FF.svg)](#)
</div>

---

## Overview

**HowlSync** is the dedicated communication and localization engine for the ArcticHowl swarm robotics ecosystem. Built entirely on the ESP-IDF framework and targeted specifically at the ESP32-C6 architecture, it provides a highly resilient, self-healing mesh network for multi-agent robotic fleets. 

Operating completely independent of the robot's primary control loop, HowlSync ensures that computationally heavy networking tasks never interrupt critical flight or drive dynamics. By piping the final localization matrix directly to the primary controller via a high-speed SPI/DMA link, HowlSync acts as an invisible, high-precision tether connecting the entire pack.

---

## Core Capabilities

* **Dynamic Mesh Topology:** A decentralized, auto-connecting, and self-healing node network. Drones and rovers can enter or drop out of the swarm seamlessly without breaking the pack's communication structure.
* **Precision Spatial Mapping:** Leverages 802.11mc (Wi-Fi Fine Time Measurement) Time-of-Flight (ToF) paired with RSSI graph networks to calculate relative distances between all active nodes in real time.
* **GPS-Denied Navigation:** By mapping the swarm's geometry purely through node-to-node signal propagation, HowlSync dramatically reduces spatial error to the sub-0.5m range, enabling complex swarm maneuvers indoors, underground, or in heavily jammed environments.
* **Hardware Determinism:** Designed to offload network polling from the primary Zephyr-based flight/drive controller (HowlOS). It simply delivers the finished spatial data array precisely when the Kalman filters need it.

---

## Architecture Context

HowlSync is designed to run on the **WolfNav Frost** hardware layer (ESP32-C6) and serves as the bridge between the physical swarm and the internal logic.

1. **The Network (ESP32-C6):** Runs HowlSync, constantly pinging neighboring nodes using FTM/802.11mc and maintaining the mesh graph.
2. **The Link (SPI + DMA):** HowlSync triggers a hardware interrupt to the main controller and blasts the updated relative coordinate matrix over a high-speed SPI bus.
3. **The Brain (STM32/Zephyr):** The primary flight/drive controller (running HowlOS) immediately ingests the matrix into its Extended Kalman Filters (EKF) without ever dropping a motor control cycle.

---

## Directory Structure

```text
howlsync/
├── CMakeLists.txt
├── main/
│   ├── CMakeLists.txt
│   ├── main.c                  # Entry point and RTOS task initialization
│   ├── ftm_localization.c      # 802.11mc ToF and RSSI distance calculations
│   ├── mesh_network.c          # ESP-WIFI-MESH topology management
│   └── spi_bridge.c            # SPI slave/master DMA link to HowlOS
├── components/                 # Custom ESP-IDF components
├── include/                    # Public headers
└── sdkconfig.defaults          # Default ESP32-C6 configuration (Wi-Fi 6, FTM enabled)
