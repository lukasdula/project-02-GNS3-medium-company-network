# **Medium Company Network Infrastructure**

<br>


## **Introduction and Objectives**

This next project created in GNS3 presents a medium-sized company network infrastructure. It expands on my earlier work and demonstrates a more advanced, multi-layer topology using dynamic routing, segmented VLAN design, Linux-based internal services and structured security policies. The goal is to model a realistic company environment where routing, switching, server services and access control operate together as one stable system.

The network uses VLAN segmentation with inter-VLAN routing via subinterfaces, OSPF single-area routing, DHCP relay, local DNS and HTTP services hosted on Xubuntu Server, NAT/PAT for external connectivity and a set of layered security controls including port security, basic device hardening and ACL rules. VLAN 99 functions not only as the management network but also as the OSPF backup path—if the primary point-to-point link between R1 and R2 fails, OSPF automatically shifts adjacency to VLAN 99 and restores full routing, ensuring continuous inter-VLAN communication. The project also incorporates advanced switching features such as RSTP operation, PortFast for edge ports and BPDU handling, reflecting realistic Layer 2 behavior in enterprise environments. Troubleshooting tasks were included intentionally to simulate real operational issues.

This project also demonstrates practical work with GNS3 VM integration, Cisco IOSv and IOSv-L2 appliances, Linux server configuration and structured diagnostics across the entire topology.

<br>


## **Topology Diagram**

![](images/Pasted%20image%2020251202222432.png)


<br>


## **Network Zones**

Zones clearly define device roles and which parts of the network communicate together.

- **ISP Zone** – simulated external connectivity
- **Server Zone (VLAN 10)** – Xubuntu server providing DNS, DHCP and HTTP
- **Admin Zone (VLAN 20)** – Xubuntu-Admin workstation with full administrative access
- **User VLANs** – Executive, Development, Finance, Printers, Logistics, Warehouse, Sales
- **Routing/Switching Zone** – Routers R1/R2 and all Layer 2 interconnections


<br>


## **Project Structure**

1. [Network Topology and Devices](01-network-topology-and-devices.md)
2. [Addressing and VLAN Planning](02-addressing-and-vlan-planning.md)
3. [Basic Device Configuration](03-basic-device-configuration.md)
4. [VLAN and Trunk Configuration](04-vlan-and-trunk-configuration.md)
5. [Advanced Switching Features](05-advanced-switching-features.md)
6. [Inter-VLAN Routing and OSPF](06-inter-vlan-routing-and-ospf.md)
7. [Core Server Services](07-core-server-services.md)
8. [NAT/PAT and External Connectivity](08-nat-pat-configuration.md)
9. [Wireshark Monitoring](09-wireshark-monitoring.md)
10. [Network Security](10-network-security.md)
11. [Troubleshooting](11-troubleshooting.md)
12. [Conclusion and Summary](12-conclusion-and-summary.md)

<br>


## **Key Project Features**

- VLAN segmentation and trunk links across all switches
- Inter-VLAN routing with subinterfaces
- Switching features including STP, RSTP, PortFast, and BPDU behavior
- OSPF single-area internal routing with VLAN 99 as backup path
- DHCP relay for all VLANs
- DNS and HTTP services on Linux
- NAT/PAT translation for outbound traffic
- Port security and administrative hardening
- ACL-based segmentation between VLANs
- Full diagnostics and packet analysis in Wireshark
- Troubleshooting of routing, DHCP, VLAN and addressing issues

<br>


## **Tools and Environment**


- **GNS3 version 2.2.54**
    
- **Xubuntu VM** (kernel-based QEMU virtual machine inside GNS3)
    
- **Cisco IOSv Router**
    
    - *VIOS-ADVENTERPRISEK9-M, Version 15.9(3)M6*
        
- **Cisco IOSv-L2 Switch**
    
    - *vios_l2-ADVENTERPRISEK9-M, Version 15.2(20170321)*
        
- **Visual Studio Code** (documentation editing)
    
- **Obsidian** (notes, summaries and screenshots)



<br>


## **Author’s Note**

This project was a very important step in my learning. It is the biggest and most complex network I have built so far in GNS3. It was also the first time I worked with OSPF in a network of this size and it showed me how routing really behaves when there are two routers and more paths to choose from.

I found it very interesting to implement VLAN 99 as the management network and also as a backup path for OSPF. When I tested what happens if the main link between R1 and R2 goes down, I learned how OSPF moves the traffic to the backup path and how priority decides which route becomes active. This gave me real practical skill in understanding routing failover and how important a well planned design is.

Some parts were difficult, especially setting up BIND9 for DNS. I had moments when it was really stressful, but solving these problems helped me understand the network much better. This project also showed me how much I have improved compared to my first project.

Working on this topology gave me confidence and a clearer idea of what I want to try next. I am excited to see what the next challenge will be in my upcoming project.


---

<br>


© 2025 – Lukas Dula | Home Network Project & Portfolio
