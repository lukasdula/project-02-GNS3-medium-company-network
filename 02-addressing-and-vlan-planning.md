
<br>

# **2 - Addressing and VLAN Planning**

<br>

## **2.1 Introduction**

This chapter defines the IPv4 addressing plan and VLAN layout for the medium-sized company network. The topology uses a dedicated /30 point-to-point subnet for the WAN link between R1 and the ISP, ensuring a clear separation between internal and external routing.

Each internal segment receives its own IPv4 subnet, with default gateways assigned on the routing interface to support inter-VLAN communication. The addressing plan is structured to keep all functional zones isolated, consistent, and easy to expand.

<br>

## **2.2 IP Addressing and Subnet Overview**

| Device / Interface | Description                             | IP Address      | Subnet Mask     | Default Gateway | Assigned Network |
| ------------------ | --------------------------------------- | --------------- | --------------- | --------------- | ---------------- |
| ISP Router Gi0/5   | ISP → R1 point-to-point link            | 203.0.113.6     | 255.255.255.252 | N/A             | 203.0.113.4/30   |
| Router R1 Gi0/5    | R1 → ISP point-to-point link            | 203.0.113.5     | 255.255.255.252 | N/A             | 203.0.113.4/30   |
| Router R1 Gi0/0    | R1 → R2 internal routing link           | 10.32.0.1       | 255.255.255.252 | N/A             | 10.32.0.0/30     |
| Router R2 Gi0/0    | R2 → R1 internal routing link           | 10.32.0.2       | 255.255.255.252 | N/A             | 10.32.0.0/30     |
| ISP Router Gi0/1   | ISP → Simulated Internet host           | 198.51.100.5    | 255.255.255.252 | N/A             | 198.51.100.4/30  |
| Internet Host e0   | Simulated external host                 | 198.51.100.6    | 255.255.255.252 | 198.51.100.5    | 198.51.100.4/30  |
| Xubuntu-Server     | Core services server (Server VLAN)      | 10.32.10.10     | 255.255.255.0   | 10.32.10.1      | 10.32.10.0/24    |
| Xubuntu-Admin      | Administrative workstation (Admin VLAN) | 10.32.20.10     | 255.255.255.0   | 10.32.20.1      | 10.32.20.0/24    |
| PC1                | Executive PC (Executive VLAN)           | DHCP 10.32.30.x | 255.255.255.0   | 10.32.30.1      | 10.32.30.0/24    |
| PC2                | Development PC                          | DHCP 10.32.40.x | 255.255.255.0   | 10.32.40.1      | 10.32.40.0/24    |
| PC3                | Development PC                          | DHCP 10.32.40.x | 255.255.255.0   | 10.32.40.1      | 10.32.40.0/24    |
| PC4                | Finance PC                              | DHCP 10.32.50.x | 255.255.255.0   | 10.32.50.1      | 10.32.50.0/24    |
| PC5                | Logistic PC                             | DHCP 10.32.60.x | 255.255.255.0   | 10.32.60.1      | 10.32.60.0/24    |
| PC6                | Warehouse PC                            | DHCP 10.32.70.x | 255.255.255.0   | 10.32.70.1      | 10.32.70.0/24    |
| PC7                | Sales PC                                | DHCP 10.32.80.x | 255.255.255.0   | 10.32.80.1      | 10.32.80.0/24    |
| Printer            | Network Printer                         | 10.32.90.10     | 255.255.255.0   | 10.32.90.1      | 10.32.90.0/24    |

Notes: The DHCP scopes for the dynamic VLAN segments are defined in the DHCP configuration chapter. Each VLAN receives a dedicated IP range where the DHCP server allocates addresses from the dynamic pool.

<br>

## **2.3 VLAN Planning**

| **VLAN ID** | **VLAN Name** | **Description / Purpose**              | **Assigned Network** | **Default Gateway**               | **Assigned Devices** | **Switch Assoc** |
| ----------- | ------------- | -------------------------------------- | -------------------- | --------------------------------- | -------------------- | ---------------- |
| 10          | Server        | Server infrastructure (Xubuntu-Server) | 10.32.10.0/24        | 10.32.10.1                        | Xubuntu-Server       | SW1              |
| 20          | Admin         | Administrative workstation             | 10.32.20.0/24        | 10.32.20.1                        | Xubuntu-Admin        | SW2              |
| 30          | Executive     | Executive PC                           | 10.32.30.0/24        | 10.32.30.1                        | PC1                  | SW3              |
| 40          | Development   | Development PCs                        | 10.32.40.0/24        | 10.32.40.1                        | PC2, PC3             | SW3              |
| 50          | Finance       | Finance workstation                    | 10.32.50.0/24        | 10.32.50.1                        | PC4                  | SW4              |
| 60          | Logistic      | Logistic workstation                   | 10.32.60.0/24        | 10.32.60.1                        | PC5                  | SW5              |
| 70          | Warehouse     | Warehouse workstation                  | 10.32.70.0/24        | 10.32.70.1                        | PC6                  | SW5              |
| 80          | Sales         | Sales workstation                      | 10.32.80.0/24        | 10.32.80.1                        | PC7                  | SW5              |
| 90          | Printer       | Network printer                        | 10.32.90.0/24        | 10.32.90.1                        | Printer              | SW4              |
| 99          | Management    | Management and OSPF backup transit     | 10.32.99.0/24        | 10.32.99.1 (R1) / 10.32.99.2 (R2) | SW1, SW2             | SW1, SW2         |

<br>

## **2.4 Conclusion**

This addressing and VLAN plan defines a structured layout for this project. Each VLAN receives a dedicated /24 subnet that corresponds to a specific functional zone, such as server services, administration, development, executive, finance, logistic, warehouse, sales, and a separate printer segment. The separation of these segments supports clear security boundaries and simplifies policy enforcement.

The point-to-point link toward the ISP keeps internal routing independent from the simulated internet, while the private 10.32.0.0/16 space provides additional room for future subnets. With this design in place, the network is ready for the implementation of inter-VLAN routing, DHCP, security policies, and access control in the following chapters.


---

<br>

**Next part:**