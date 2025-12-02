# **3 - Basic Device Configuration**


<br>

## **3.1 Introduction**

This part defines the baseline configuration applied to all core routing and switching devices. The goal is to establish stable IPv4 addressing on point-to-point links, assign static addresses to the management hosts, apply consistent hostnames, and document interface descriptions used across all routers and switches. No VLANs, subinterfaces, OSPF, DHCP, or security features are configured at this stage.


<br>

## **3.2 Topology Diagram**

![](/images/Pasted%image%20251202232911.png)

<br>

## **3.3 Objectives**

1. Apply static IPv4 addresses to the Xubuntu-Server and Xubuntu-Admin.
    
2. Assign hostnames to all routers and switches.
    
3. Configure IPv4 addresses on all router point-to-point links.
    
4. Apply interface descriptions to all routing and switching devices.
    
5. Verify connectivity using interface checks, ping tests, and CDP neighbors.
    

<br>

## **3.4 Static IPv4 Addressing for Xubuntu Hosts**

Static IPv4 configuration is applied to ensure both management devices remain reachable before VLAN routing and DHCP services are introduced.

### Xubuntu-Server (VLAN 10)

Open configuration file:

 ```
 sudo nano /etc/netplan/01-network-manager-all.yaml
 ```
![](images/Pasted%20image%2020251123220142.png)

**Configuration:**

```
network:
  version: 2
  renderer: NetworkManager
  ethernets:
    ens3:
      addresses: [10.32.10.10/24]
      nameservers:
        addresses: [10.32.10.1]
      routes:
        - to: 0.0.0.0/0
          via: 10.32.10.1
```
![](images/Pasted%20image%2020251123220123.png)


Apply configuration:

```
sudo netplan apply
```
![](images/Pasted%20image%2020251123220415.png)

Verification:

```
ip a
```
![](images/Pasted%20image%2020251123220350.png)

### **Results**  

The server uses a static management IP

### Xubuntu-Admin (VLAN 20)

**Configuration:**

```
network:
  version: 2
  renderer: NetworkManager
  ethernets:
    ens3:
      addresses: [10.32.20.10/24]
      nameservers:
        addresses: [10.32.20.1]
      routes:
        - to: 0.0.0.0/0
          via: 10.32.20.1

```
![](images/Pasted%20image%2020251123212856.png)

Apply configuration:

```
sudo netplan apply
```
![](images/Pasted%20image%2020251123213311.png)

Verification:

```
ip a
```
![](images/Pasted%20image%2020251123213331.png)

### **Results**  

The admin workstation is correctly configured


>**Notes:** Static IPv4 configuration can also be set using the Xubuntu GUI (NetworkManager). When configured through the GUI, the static address is saved as a persistent connection profile and is automatically applied after reboot, the same way as the netplan configuration.


<br>

## **3.5 Hostname Configuration**

All routers and switches receive unique hostnames based on the device list from Chapter 1. The configuration example below illustrates the format used.

### Example (Router R1)

```
enable
configure terminal
hostname R1
exit
write
```
![](images/Pasted%20image%2020251123221224.png)

<br>

### Device Hostname Table

| Device     | Hostname |
| ---------- | -------- |
| Router R1  | R1       |
| Router R2  | R2       |
| ISP Router | ISP      |
| SW1        | SW1      |
| SW2        | SW2      |
| SW3        | SW3      |
| SW4        | SW4      |
| SW5        | SW5      |

> Notes: VPCS devices do not require manual hostname configuration. Each VPCS node automatically automatically uses its GNS3 topology label as its device name and does not need additional hostname commands.


### Results

Hostnames are consistently assigned across all routers and switches, ensuring clear device identification throughout the network. 

<br>

## **3.6 IPv4 Configuration on Router Interfaces**

This section assigns IPv4 addresses to all router point-to-point interfaces. Each link receives a unique /30 subnet to ensure clear separation between networks and reliable detection of directly connected devices. These addresses form the core routing backbone and provide connectivity between the internal network and the simulated ISP.

### Router R1

```
enable
configure terminal
interface Gi0/0
ip address 10.32.0.1 255.255.255.252
no shutdown
interface Gi0/5
ip address 203.0.113.5 255.255.255.252
no shutdown
exit
end
write
```
![](images/Pasted%20image%2020251123224845.png)
### Router R2

```
enable
configure terminal
interface Gi0/0
ip address 10.32.0.2 255.255.255.252
no shutdown
exit
end
write
```
![](images/Pasted%20image%2020251123224537.png)

### ISP Router

```
enable
configure terminal
interface Gi0/5
ip address 203.0.113.6 255.255.255.252
no shutdown
interface Gi0/1
ip address 198.51.100.5 255.255.255.252
no shutdown
exit
end
write
```
![](images/Pasted%20image%2020251123224509.png)

### Internet Host

```
ip 198.51.100.6 255.255.255.252 198.51.100.5
```
![](images/Pasted%20image%2020251123224645.png)

### Results

All router point-to-point interfaces are configured with correct IPv4 addressing, and each link is operational after verification.  
This section applies IPv4 addressing to point-to-point links between routers and to the simulated Internet segment.

<br>

## **3.7 Interface Descriptions**


Interface descriptions make it easy to see what each port is used for.  
They help with reading the configuration and checking that devices are connected correctly.

### Router R1 (Example)

```
interface Gi0/5
description Link to ISP 
interface Gi0/0
description Link to R2 
interface Gi0/4
description Downlink to SW4 
```


### Switch SW3 (Example)

```
interface Gi0/0
 description PC1 Executive
interface Gi0/1
 description PC2 Development
interface Gi0/2
 description PC3 Development
interface Gi0/3
 description Uplink to R2
```
![](images/Pasted%20image%2020251123225640.png)

### Port Description Mapping

| **Device** | **Description Style**                                           |
| ---------- | --------------------------------------------------------------- |
| R1         | Uplink to ISP, Link to R2, **Downlink to SW1**, Downlink to SW4 |
| R2         | Link to R1, Downlink to SW2, Downlink to SW3                    |
| ISP        | Link to R1, Link to Internet Host                               |
| SW1        | Xubuntu-Server, Redundant link to SW2, **Uplink to R1**         |
| SW2        | Xubuntu-Admin, Uplink to R2, Redundant link to SW1              |
| SW3        | Executive/Development PCs, Uplink to R2                         |
| SW4        | Finance PC, Printer, Downlink to SW5, Uplink to R1              |
| SW5        | Logistic, Warehouse, Sales PCs, Uplink to SW4                   |


<br>

## **3.8 Connectivity Verification**

This section checks that router interfaces are working correctly and that basic connectivity is available.  

The interface summary shows the status of each port, and simple ping tests confirm that the router can reach the ISP and external host.

### Router R1 – Interface Summary

```
show ip interface brief
```
![](images/Pasted%20image%2020251123232457.png)

### Ping Tests

R1 → ISP:

```
ping 203.0.113.6
```
![](images/Pasted%20image%2020251123232523.png)

ISP → Internet Host:

```
ping 198.51.100.6
```
![](images/Pasted%20image%2020251123232545.png)

R1 → R2:

```
ping 10.32.0.2
```
![](images/Pasted%20image%2020251123232739.png)


All connectivity checks confirm that point-to-point interfaces operate correctly. R1 successfully reaches the ISP router, the ISP reaches the Internet Host, and R1 reaches R2 on the internal P2P link.  
Interface summaries show all assigned IPv4 addresses in the _up/up_ state, proving that the physical and Layer 3 configuration is applied correctly.

<br>

## **3.9 Extra diagnostic**


This diagnostic verifies neighbor visibility using CDP on the router.

* The goal is to confirm correct physical cabling and Layer 2 connectivity between the core routers.

### CDP Neighbors (Example on R2)

A CDP check is executed on **R2** to verify detection of its directly connected neighbor, **R1**, over the point-to-point core link.

```
show cdp neighbors
```
![](images/Pasted%20image%2020251123232933.png)

### Results

- R2 successfully detects **R1** on the expected interface **Gi0/0**, confirming correct physical connectivity on the core link.
    
- CDP does **not** display the switches at this stage because they do not yet have fully configured management interfaces or CDP-relevant settings.
    
- The output confirms that the Layer 2 connection between the routers matches the intended network design and that the point-to-point link is functioning as expected.

<br>

## **3.10 Conclusion**

This chapter completes the initial configuration of all core devices.  
Hostnames, interface descriptions, and IPv4 addresses are assigned to ensure a clear and organized network layout.  

Connectivity checks confirm that router interfaces operate correctly and that point-to-point links function as expected. The devices are now fully prepared for the next stages of VLAN configuration, routing, and service deployment.


---

<br>

**Next chapter:** [VLAN and Trunk Configuration](04-vlan-and-trunk-configuration.md)
