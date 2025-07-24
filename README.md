### Pozyx – system launch and corridor tests (23.07.2025)

After configuring the gateway, a stable connection to the cloud was achieved (“Connected to Pozyx Cloud”). The initial alerts (Idle, no ranging/position) were eliminated by:
1. Reactivating anchors (#5816, #5846, #5870, #5875) – all online.  
2. Autocalibrate / Ranging only – each anchor has 3/3 connections with the others.  
3. Completing the X, Y, Z coordinates.  
4. Start/Run positioning – status: `positioner running`.  
5. Activating 3 tags in the same setup.

**Final state:** `positioner running`, `4/4 anchors functional`, `3 active tags`.

**Test scenario:** anchors mounted on the corridor ceiling; tags placed on chairs for calibration/ranging, then moved around. Verification performed in Pozyx Cloud (floor map).

<img src="pozyx_tag_aplication.png" alt="Pozyx Cloud UI – positioner running, 4/4 anchors functional, 3 active tags">
<div class="caption">Fig. 1. View in Pozyx Cloud – positioner running, 4/4 anchors functional, 3 active tags.</div>
<table>
  <tr>
    <img src="pozyx_tag_1.png" alt="Corridor with ceiling‑mounted anchors and tags on chairs">
    <div class="caption">Fig. 3. Anchor layout in the corridor; tags used for calibration and range testing.</div>
    <table class="img2x2">
  </tr>
  <tr>
    <td>
      <img src="pozyx_tag_2.png" alt="Corridor with ceiling‑mounted anchors and tags on chairs">
      <div class="caption">Fig. 3. Anchor layout in the corridor; tags used for calibration and range testing.</div>
    </td>
    <td>
      <img src="pozyx_tag_3.png" alt="Two Pozyx tags on a desk, one with an active LED">
      <div class="caption">Fig. 4. Pair of Pozyx tags – checking LED status and readiness for measurements.</div>
    </td>
  </tr>
</table>

<div class="video-block">
  <video controls playsinline muted>
    <source src="PierwszaPróba.mp4" type="video/mp4">
    Your browser does not support HTML5 video. Download the file: <a href="PierwszaPróba.mp4">PierwszaPróba.mp4</a>
  </video>
  <div class="caption">Video 1. First corridor walk‑through with a tag.</div>
</div>


### RTLS System Assumptions

The system is designed as a modular real‑time architecture for receiving and integrating data from various positioning technologies. Key assumptions:

- Support for multiple location data sources over UDP (Pozyx, Ubisense)  
- Data processing on a server (VM or containers)  
- Unified message format to the model `{id, x, y, z, t}`  
- Data access via REST API and WebSocket  
- External frontend (HTTP client) independent of the backend  
- Possibility of integration with MES / ERP / BI systems  

---

## UWB Technology and RTLS Systems

### What is UWB?

**Ultra‑Wideband (UWB)** is a wireless technology that enables very precise distance measurements and real‑time location tracking (RTLS). Operating in the 3.1–10.6 GHz band, it delivers accuracies on the order of a few centimetres. Thanks to its very short pulses, UWB does not interfere with typical Wi‑Fi or Bluetooth networks and is resistant to multipath effects.

### Positioning Approaches: TDoA and AoA

**TDoA (Time Difference of Arrival)**  
The position is calculated from the time‑of‑arrival differences of signals reaching the anchors. It requires precise time synchronisation between devices, typically achieved via Ethernet and protocols like PTP. This method performs well in open areas with a large number of anchors.

**AoA (Angle of Arrival)**  
The position is determined by measuring the angle at which the signal reaches the receiving antenna. AoA systems do not need time synchronisation between devices but rely on directional antennas and advanced signal‑processing algorithms. This approach performs well in flexible, industrial installations.


## Ubisense and Pozyx Systems

### Ubisense

Ubisense is a mature, industrial UWB‑based positioning system that uses TDoA. It enables precise real‑time tracking of objects in industrial and warehouse environments. Communication with the system takes place via multicast UDP using the OTW‑40 protocol.

<img src="Ubisense_logo.png" width="30%">

<img src="ubisense_sprzet.png" width="80%">

### Pozyx

Pozyx is a modern, modular UWB RTLS that relies on the AoA method. It simplifies rapid industrial deployments, supports the UDP protocol, and can connect directly through a gateway without requiring a central server. Pozyx anchors can be powered via PoE and managed remotely through the web interface.

<img src="pozyx_logo.png" width="80%">

<img src="pozyx_sprzet.jpg" width="80%">



### Hardware – installed devices

Below are photos from the physical deployment of the system:

#### Ubisense: controllers with three Ethernet ports for synchronization

Each Ubisense controller provides:
- dedicated ports for synchronization between anchors (time and position),
- separate Ethernet ports for management and power (PoE).

<img src="ubisense_1.jpg" width="80%">

<img src="ubisense_2.jpg" width="80%">

#### Pozyx: anchor with an Ethernet bus (power + communication)

The Pozyx system uses a simplified architecture:
- each anchor has an Ethernet port with PoE support,
- time synchronization is handled over the network (PTP),
- UDP data are transmitted directly from the Pozyx gateway.

<img src="pozyx_anchors.jpg" width="80%">



## RTLS Network Topology (Ubisense + Pozyx)

In this variant, the RTLS backend runs inside a single virtual machine. UDP messages from the external positioning systems (Pozyx and Ubisense) are sent to it. Each protocol has its own receiver that parses the messages and passes them on for unified processing. The results are exposed via an API (REST or WebSocket), and visualization is handled on the client side.

![RTLS topology](./topologia3.png)

### Key components:
- `Pozyx Gateway` and `Ubisense App` – send location data as UDP messages  
- `UDP Receiver (Pozyx)` – parses Pozyx‑specific UDP messages  
- `UDP Receiver (Ubisense)` – parses OTW‑40 messages  
- `Data → common format` – data‑normalization layer `{id, x, y, z, t}`  
- `REST / WebSocket API` – provides real‑time data  
- `Client application (map)` – frontend runs outside the VM and visualizes the location  

The system can run on any hypervisor (e.g., VirtualBox, Proxmox, KVM), and UDP communication takes place within the local subnet.

The Pozyx controller is directly connected to four Pozyx anchors, which are powered and communicate over Ethernet.

### Connection list

Router → PoE Switch #1  
PoE Switch #1 → RTLS Pozyx Controller  
PoE Switch #1 → Ubisense Anchor (x2)  
PoE Switch #1 → PoE Switch #2  
PoE Switch #2 → RTLS Ubisense Controller  
PoE Switch #2 → Ubisense Anchor (x2)  
RTLS Pozyx Controller → Pozyx Anchor (x4)



## Possible application solutions

RTLS data (from Ubisense and Pozyx) can be received and processed by a common collection application. The main goal is to unify the positioning data sent over UDP and make them available in real time to downstream modules (e.g., API, visualization, MES).

Below are two independent approaches to implementing the application – one based on virtual machines and the other on containers with a Python interpreter.

### Solution 1: UDP on virtual machines

Each component runs on a separate virtual machine (e.g., VirtualBox, Proxmox, KVM). The machines are connected via an internal bridge network, and UDP packets are exchanged between private addresses.

![RTLS topology](./topologiaWirtualki.png)

### Key components:
- `Pozyx Gateway` and `Ubisense App` – send location data as UDP messages  
- `UDP Receiver (Pozyx)` – parses Pozyx‑specific UDP messages  
- `UDP Receiver (Ubisense)` – parses OTW‑40 messages  
- `Data → common format` – data‑normalization layer `{id, x, y, z, t}`  
- `REST / WebSocket API` – provides real‑time data  
- `Client application (map)` – frontend runs outside the VM and visualizes the location  

The system can run on any hypervisor (e.g., VirtualBox, Proxmox, KVM), and UDP communication takes place within the local subnet.

### Solution 2: UDP processing in containers with the frontend outside the server

The diagram below shows how location data are received from the UDP network in Docker/Podman containers and exposed to external applications. The frontend runs outside the container host and connects to the API provided by the backend.

The setup assumes:
- two UDP data sources (described earlier),
- containers: *ingest*, *normalizer*, *API*,
- a frontend that runs independently (e.g., browser, MES, dashboard).

![RTLS topology](./topologiaKontener.png)

The architecture is fully modular and ready for deployment with Docker Compose or Podman Pod. Internal container communication can be based on asynchronous queues (e.g., `asyncio`), sockets, or local TCP. The API exposes location data in the unified format `{id, x, y, z, t}`.
