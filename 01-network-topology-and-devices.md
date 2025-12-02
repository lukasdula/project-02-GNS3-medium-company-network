
<br>

# **1 - Network Topology and Devices**

<br>

## **1.1  Network Topology and Devices**

This first chapter introduces the structured medium-sized company network used in this project. The design organizes all hosts, servers and user groups into separate logical zones, each connected through two core routers. Switching, routing and basic services follow a clear layout so that every part of the environment has a defined role.

All internal segments connect to the core routing layer, which provides inter-VLAN routing and forwards external traffic to the ISP. The following sections describe the topology diagram, network zones, device list and all physical connections used in this project.

<br>

## **1.2  Topology Diagram**

![](images/Pasted%20image%2020251124001241.png)

<br>

## **1.3  Network Zones**

The topology is divided into the following areas:

- **ISP and internet access**
    
- **Core routing layer** (R1 and R2)
    
- **Administration zone** (Xubuntu-Admin)
    
- **Server zone** (Xubuntu-Server)
    
- **Executive zone** (PC1-Executive)
    
- **Development zone** (PC2-Development, PC3-Development)
    
- **Finance zone** (PC4-Finance)
    
- **Logistic zone** (PC5-Logistic)
    
- **Warehouse zone** (PC6-Warehouse)
    
- **Sales zone** (PC7-Sales)
    
- **Printer zone** (Printer)
    
- **Network management zone** (VLAN 99)
    

<br>

## **1.4  Used Devices**

| **Device**       | **Image / Type** | **Interfaces**     | **Access** | **Purpose**                                                 |
| ---------------- | ---------------- | ------------------ | ---------- | ----------------------------------------------------------- |
| Router R1        | Cisco IOSv       | 6x GigabitEthernet | Telnet     | Core routing device; inter-VLAN routing; uplink to ISP.     |
| Router R2        | Cisco IOSv       | 6x GigabitEthernet | Telnet     | Secondary routing device; distribution for access switches. |
| ISP Router       | Cisco IOSv       | 6x GigabitEthernet | Telnet     | Point-to-point link to R1 and simulated WAN access.         |
| Switches SW1–SW5 | Cisco IOSv-L2    | 4x GigabitEthernet | Telnet     | Layer 2 switching; access ports and trunk links.            |
| Xubuntu-Admin    | QEMU Xubuntu VM  | 1x GigabitEthernet | VNC        | Administration workstation for network management.          |
| Xubuntu-Server   | QEMU Xubuntu VM  | 1x GigabitEthernet | VNC        | Internal server for DHCP, DNS and HTTP                      |
| PC1–PC7          | VPCS Hosts       | 1x Ethernet        | Console    | Workstations placed in VLANs.                               |
| Printer          | GNS3 Printer     | 1x Ethernet        | Console    | Network printer in a dedicated VLAN.                        |
| Internet Host    | VPCS             | 1x Ethernet        | Console    | Simulated external host for NAT/PAT verification.           |

<br>

## **1.5  Connection Overview**

| **Device**     | **Interface** | **Connected to →** | **Peer Interface** | **Purpose**                                                   |
| -------------- | ------------- | ------------------ | ------------------ | ------------------------------------------------------------- |
| **ISP Router** | Gi0/5         | R1                 | Gi0/5              | WAN uplink between ISP and core router R1.                    |
| ISP Router     | Gi0/1         | Internet-Host      | e0                 | Connection to simulated external internet host.               |
| R1             | Gi0/5         | ISP Router         | Gi0/5              | Outbound WAN link.                                            |
| R1             | Gi0/0         | R2                 | Gi0/0              | Internal routing link between R1 and R2.                      |
| R1             | Gi0/4         | SW4                | Gi0/0              | Core uplink to access switch SW4.                             |
| R2             | Gi0/0         | R1                 | Gi0/0              | Routing link between R2 and R1.                               |
| R2             | Gi0/2         | SW2                | Gi0/2              | Core uplink for admin access segment.                         |
| R2             | Gi0/3         | SW3                | Gi0/3              | Core uplink for executive and development segments.           |
| **SW1**        | Gi0/0         | Xubuntu-Server     | e0                 | Access link for internal server VM.                           |
| SW1            | Gi0/3         | SW2                | Gi0/3              | Redundant switch-to-switch link (STP).                        |
| **SW2**        | Gi0/0         | Xubuntu-Admin      | e0                 | Access link for admin workstation.                            |
| SW2            | Gi0/3         | SW1                | Gi0/3              | Redundant switch-to-switch link (STP).                        |
| **SW3**        | Gi0/0         | PC1-Executive      | e0                 | Access link for executive workstation.                        |
| SW3            | Gi0/1         | PC2-Development    | e0                 | Access link for development workstation.                      |
| SW3            | Gi0/2         | PC3-Development    | e0                 | Access link for development workstation.                      |
| SW3            | Gi0/3         | R2                 | Gi0/3              | Uplink to R2 for executive and development VLANs.             |
| **SW4**        | Gi0/2         | PC4-Finance        | e0                 | Access link for finance workstation.                          |
| SW4            | Gi0/1         | Printer            | e0                 | Downlink to SW4 for warehouse/sales/printer.                  |
| SW4            | Gi0/0         | R1                 | Gi0/4              | Uplink to R1 for finance, logistic and warehouse/sales VLANs. |
| SW4            | Gi0/3         | SW5                | Gi0/3              | Downlink to SW5 for warehouse/sales/logistic.                 |
| **SW5**        | Gi0/2         | Logistic           | e0                 | Access link for logistic workstation.                         |
| SW5            | Gi0/0         | PC6-Warehouse      | e0                 | Access link for warehouse workstation.                        |
| SW5            | Gi0/1         | PC7-Sales          | e0                 | Access link for sales workstation.                            |
| SW5            | Gi0/3         | SW4                | Gi0/3              | Uplink to SW4 for warehouse and sales VLANs.                  |

<br>

## **1.6  Conclusion**

This chapter defines the medium-sized company network and summarizes all devices used in the topology. It presents the physical layout, the logical separation of user groups, and the roles of every router, switch and host. The structured segmentation establishes a clear starting point for configuration and verification.

The following chapter -> **02-addressing-and-vlan-planning**, introduces the subnet allocation, VLAN mapping and gateway structure for all segments. Subsequent chapters build on this plan through switching features, routing, core services and security.

<br>

---

<br>

**Next chapter:** [Addressing and VLAN Planning](02-addressing-and-vlan-planning.md)
