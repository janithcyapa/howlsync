# HowlSync
## Project Plan
---
### Phase 1: The Core Proof of Concept (Point-to-Point)

**Goal:** Prove that the ESP32-C6 can actually achieve your target sub-0.5m accuracy before introducing the chaos of a mesh network.

* **Research Side:**
* Study 802.11mc (FTM) multi-path fading and Line-of-Sight (LoS) vs. Non-Line-of-Sight (NLoS) error characteristics.
* Research basic statistical filtering (e.g., moving average, median filters, or simple 1D Kalman filters) to smooth raw Time-of-Flight (ToF) and RSSI data.


* **Coding Side:**
* **Skip OTA for now.** Write a simple bash or Python script using `esptool.py` to flash multiple boards connected via USB hubs simultaneously.
* Code a strict 2-node setup: One FTM Initiator and one FTM Responder.
* Output the raw ToF distance and RSSI to the serial monitor. Walk around your lab with a tape measure and graph the actual error vs. calculated distance.



### Phase 2: The Mesh Backbone & Data Flow

**Goal:** Establish a self-healing network where nodes can dynamically join, leave, and share payloads without dropping FTM capabilities.

* **Research Side:**
* Investigate Espressif’s `ESP-WIFI-MESH` vs. standard Wi-Fi AP/STA multiplexing.
* Determine the routing topology (Tree vs. Graph) and how you will handle the "Hidden Node Problem" when UAVs are out of range of each other but connected via a middle-man.


* **Coding Side:**
* Implement basic auto-connect/disconnect logic for 3 to 5 nodes.
* Implement mesh-based Time Sync (TSF). Elect a "Root Node" to broadcast its internal microsecond clock to synchronize the pack.
* Verify that FTM bursts do not crash the mesh routing. Code a routine where nodes successfully pass dummy coordinate data to each other.



### Phase 3: Spatial Localization (The Swarm Geometry)

**Goal:** Translate raw point-to-point distances into a unified 2D/3D local coordinate system. This is the hardest mathematical phase.

* **Research Side:**
* Simulate your localization algorithm in Python (using Matplotlib/NumPy) before touching C.
* Research **Multidimensional Scaling (MDS)** or **Trilateration algorithms**. If Node A knows its distance to B, C, and D, how do you calculate their relative $[x, y]$ coordinates without a GPS anchor?
* Study graph optimization techniques to reduce the $1-2m$ physical error down to the $<0.5m$ range using the shared data of the entire pack.


* **Coding Side:**
* Port your successful Python localization algorithm into C/C++ on the ESP-IDF.
* Format the localized mesh data into a clean, structured C-struct array (e.g., `[Node_ID, X_pos, Y_pos, Z_pos, Timestamp, Confidence_Metric]`).



### Phase 4: Swarm Integration & Infrastructure

**Goal:** Connect HowlSync to HowlOS and deploy fleet-management tools.

* **Research Side:**
* Study SPI DMA architecture on both the ESP32-C6 (Slave) and the STM32H750 (Master) to ensure zero-block data transfer.


* **Coding Side:**
* Implement the high-speed SPI bridge to fire an interrupt and push the coordinate array to the primary Zephyr controller.
* **Now implement OTA.** With the core protocol proven, write the bootloader partition logic to allow the swarm to distribute firmware updates across the mesh to one another.



---