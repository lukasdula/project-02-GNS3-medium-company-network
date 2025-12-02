# **12 - Conclusion and Summary**

<br>

## **Final Evaluation**

This second GNS3 project defines a complete medium-sized company network that operates as a fully integrated routed and switched infrastructure. The design includes VLAN segmentation, Layer 2 trunking, inter-VLAN routing, OSPF routing, DHCP relay, local DNS and HTTP services, NAT/PAT translation and a structured security policy. All components work together as one system.

The topology contains two routers (R1 and R2), several Layer 2 switches, an internal Xubuntu Server providing DNS, DHCP and HTTP, an Xubuntu-Admin workstation and multiple user devices across defined VLANs. The addressing plan uses /24 subnets and a /30 link between routers. VLAN 99 functions as both the management network and the OSPF transit path.

Inter-VLAN routing is handled through 802.1Q subinterfaces on both routers. OSPF runs across the routed /30 link and the VLAN 99 trunk, so both routers exchange routing information and maintain reachability for all internal networks. DHCP relay on R2 forwards broadcasts to the central DHCP server, and DNS/HTTP services operate correctly inside the server VLAN.

NAT and PAT on R1 translate internal addresses when traffic goes toward the ISP. All internal devices appear on the outside using the public IPv4 address assigned to R1. PAT overload separates sessions and ensures proper return traffic. This stage uses a more advanced ACL policy to define which internal networks are allowed to be translated.

Wireshark monitoring confirms correct ICMP, DNS, TCP and ARP packet patterns on internal links and shows the NAT-translated packets on the WAN interface. Packet captures on the R1–R2 link and the VLAN 99 trunk validate stable OSPF neighbour formation and consistent routing updates.

Troubleshooting demonstrates realistic faults: a missing default route on R2, an incorrect SVI address on SW1 in VLAN 99 and a wrong DHCP helper-address on R2. Each issue is identified, corrected and verified. After all fixes, the network functions normally.

This network provides a strong model for expanding into more advanced routing setups, refined security rules, improved server services and larger multi-switch designs.

<br>

## **Project Overview**

1. **Network Topology and Devices****: The network includes two routers, multiple Layer 2 switches, an internal server and several clients. VLANs separate user groups, server infrastructure and management.
    
2. **Addressing and VLAN Planning**: Each VLAN uses a dedicated /24 IPv4 subnet. Router-to-router links use /30 addressing. VLAN 99 defines management and OSPF transit.
    
3. **Basic Device Configuration**: Hostnames, interface addressing and administrative settings define the base layer of connectivity.
    
4. **VLAN and Trunk Configuration**: All switches create VLANs and assign access and trunk ports according to the design.
    
5. **Advanced Switching Features**: STP ensures loop-free switching, trunk links carry multiple VLANs reliably and MAC learning maintains stable Layer 2 forwarding.
    
6. **Inter-VLAN Routing and OSPF**: R1 and R2 use 802.1Q subinterfaces to route between all VLANs, and OSPF maintains dynamic routing across the /30 link and VLAN 99 transit path.
    
7. **Core Server Services**: Xubuntu Server provides DNS, DHCP and HTTP services. R2 forwards DHCP broadcasts using relay.
    
8. **NAT/PAT and External Configuration**: R1 translates internal IPv4 addresses to the public address on the ISP-connected interface.
    
9. **Wireshark Monitoring**: Packet captures validate ICMP, DNS, ARP, TCP flows and NAT/PAT translation.
    
10. **Network Security**: basic security, Port security protects access ports, SSH and passwords secure management, and ACLs control VLAN communication while keeping server and admin access available.
    
11. **Troubleshooting**: Routing, SVI configuration and DHCP relay faults are identified, fixed and verified.
    


<br>

---

<br>

**Back to project overview:** 

