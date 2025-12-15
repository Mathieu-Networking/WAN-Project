# Enterprise Campus WAN (Layer 3 Switching & OSPF)

### 1. Project Overview
This project simulates a multi-site enterprise network connecting a Headquarters (HQ) and a Remote Branch over a private WAN link. Unlike traditional Router-on-a-Stick topologies, this design utilizes **Multilayer (Layer 3) Switching** at the Core layer to perform wire-speed Inter-VLAN routing, keeping the edge routers dedicated strictly to WAN connectivity.

**Key Technologies:** OSPF Area 0, Layer 3 Switching (SVIs), DHCP Relay, DMZ Security, Routed Ports.

### 2. Network Topology
<img width="999" height="633" alt="image" src="https://github.com/user-attachments/assets/08f2e68a-da81-46e8-967c-0e898a92ac9b" />


### 3. Architecture & Addressing
The network follows a hierarchical IP scheme to simplify routing table management and troubleshooting.

| Site | Device | Role | IP Subnet | Key Function |
| :--- | :--- | :--- | :--- | :--- |
| **HQ** | HQ-Core (L3 Switch) | Inter-VLAN Routing | 10.1.0.0/16 | Gateway for Sales (VLAN 10) & DMZ (VLAN 50) |
| | HQ-Router | WAN Edge | 172.16.0.1/30 | OSPF & WAN Termination |
| | DHCP Server | Centralized IP Mgmt | 10.1.10.5 | Serves BOTH sites (Local & Remote) |
| **Branch** | Branch-Core (L3 Switch) | Branch Gateway | 10.2.0.0/16 | Gateway for Engineering (VLAN 30) |
| | Branch-Router | WAN Edge | 172.16.0.2/30 | Connects to HQ |

### 4. Key Configurations

#### A. Layer 3 Switching (Core Layer)
Instead of using sub-interfaces on the router, I configured **Switched Virtual Interfaces (SVIs)** on the 3560 Multilayer Switches. I also utilized **Routed Ports** (`no switchport`) for the uplinks to the routers to separate Layer 2 switching from Layer 3 routing.
```cisco
! HQ Core Switch Configuration
ip routing
! SVI for Internal LAN
interface vlan 10
 ip address 10.1.10.1 255.255.255.0
! SVI for DMZ
interface vlan 50
 ip address 10.1.50.1 255.255.255.0
! Routed Port to Edge Router
interface GigabitEthernet0/1
 no switchport
 ip address 10.1.255.2 255.255.255.252
```
B. Dynamic Routing (OSPF)
I deployed OSPF Process 1 to share routes between the HQ and Branch sites. This allows the HQ Core switch to automatically learn about the Branch Engineering subnet, and vice-versa.
router ospf 1
```
 router-id 2.2.2.2
 network 10.1.0.0 0.0.255.255 area 0
 network 10.1.255.0 0.0.0.3 area 0
```
C. Centralized DHCP (DHCP Relay)
To centralize management, a single DHCP server at HQ services both the local LAN and the Remote Branch. I configured IP Helper Addresses on the Branch Gateway to forward broadcast requests across the WAN unicast to the HQ server.
```
! Branch Core Switch - Engineering VLAN
interface vlan 30
 ip helper-address 10.1.10.5
```

D. DMZ Security
I placed the Public Web Server in a dedicated DMZ VLAN (50) and applied an Access Control List (ACL) to ensure the DMZ could not initiate connections to the internal Staff LAN.
```
ip access-list extended SECURE_DMZ
 deny ip 10.1.50.0 0.0.0.255 10.1.10.0 0.0.0.255
 permit ip any any
```
5. Verification
Evidence of DHCP Relay (Branch PC pulling HQ IP):
<img width="802" height="299" alt="image" src="https://github.com/user-attachments/assets/216ae17f-a01a-4dd6-aba3-45361ae1ef70" />


Evidence of OSPF Adjacency:

<img width="600" height="112" alt="image" src="https://github.com/user-attachments/assets/d64a5952-d114-453f-b8ee-1ed1bd8a54fe" />


Evidence of DMZ Segmentation (Ping Fail):

<img width="414" height="130" alt="image" src="https://github.com/user-attachments/assets/3309aecd-2d20-4e55-a73a-c9d82cd91ac1" />
