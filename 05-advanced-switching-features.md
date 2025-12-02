# **5 - Advanced Switching Features**

<br>

## **5.1 Introduction** 

This chapter introduces the advanced Layer 2 switching features used to improve stability, security, and manageability of the network. Rapid Spanning Tree Protocol (RSTP) is enabled across all switches to provide loop prevention and fast convergence. SW1 is selected as the Root Bridge to ensure predictable forwarding paths, while PortFast accelerates host connectivity on access ports. BPDU Guard adds protection by disabling any edge port that receives unexpected BPDUs. The chapter also prepares the switching infrastructure for management by creating ip addresses for VLAN 99, configuring SVIs, enabling LLDP/CDP, and introducing SSH for secure remote access.


<br>

## **5.2 Topology Diagram**

![](images/Pasted%20image%2020251124192517.png)

<br>

## **5.3 Objectives**

1. Enable and verify Rapid Spanning Tree Protocol (RSTP) on all switches.
    
2. Elect SW1 as the Root Bridge and SW2 as the secondary bridge.
    
3. Verify spanning-tree port roles across redundant and access links.
    
4. Apply PortFast on all access ports.
    
5. Configure BPDU Guard to protect edge ports.
    
6. Enable LLDP/CDP for neighbor detection.
    
7. Implement Management VLAN 99 and Configure Management SVIs
    
8. Prepare SSH configuration for secure remote access (testing in later chapter).

<br>


## **5.4 STP (RSTP 802.1w)**


This section enables Rapid Spanning Tree Protocol (RSTP) on all switches in the medium-sized company network. RSTP provides loop prevention and faster convergence in comparison to classic STP and ensures that redundant links between SW1 and SW2 do not create Layer 2 loops. All access switches (SW3, SW4, SW5) also participate in the same spanning-tree domain so that the entire switching topology remains stable.

Configuration – Example on SW1

```plaintext
enable
configure terminal
spanning-tree mode rapid-pvst
end
write

show spanning-tree summary
```
![](images/Pasted%20image%2020251124162843.png)

*The same spanning-tree mode rapid-pvst configuration is applied on all remaining switches (SW2–SW5) to keep a consistent spanning-tree domain across the entire network.*

### **Results**  

Rapid PVST (RSTP) is now active on all switches. The spanning-tree summary output on SW1 confirms that the protocol operates in rapid-pvst mode and that all connected switches participate in a single spanning-tree domain. The network is protected against Layer 2 loops and prepared for root bridge selection and further STP tuning in the next sections.

<br>


## **5.5 Root Bridge Selection (SW1 as Primary, SW2 as Secondary)**

 
This step makes SW1 the Root Bridge for all VLAN spanning-tree instances and sets SW2 as the secondary bridge. Changing the bridge priority ensures stable Layer 2 behavior, predictable forwarding paths, and correct handling of the redundant link. After SW1 becomes the root, all of its ports move to the forwarding state. SW2 then chooses its best root port toward SW1 and blocks the second redundant link to prevent loops.

Configuration – SW1 (Primary Root Bridge)

```plaintext
enable
configure terminal
spanning-tree vlan 1-99 priority 4096
end
write
```
![](images/Pasted%20image%2020251124163452.png)


Configuration – SW2 (Secondary Root Bridge)

```plaintext
enable
configure terminal
spanning-tree vlan 1-99 priority 8192
end
write
```
![](images/Pasted%20image%2020251124163628.png)

### **Results**  

After setting the priorities, SW1 becomes the Root Bridge for all VLANs. The STP output on SW1 shows that it is the root and all its ports are in the forwarding state. SW2 detects SW1 as the root and chooses one root port toward SW1. The second link between SW1 and SW2 becomes an alternate (blocking) port, which prevents loops in the network.

<br>


## **5.6 Spanning-Tree Port Role Verification**


This section verifies spanning-tree port roles for each switch individually. Every switch will have its own heading, command block, and a short Results subsection. The Results for each switch will be added after you send the corresponding CLI screenshot.

### SW1 - Verification

```plaintext
enable
show spanning-tree summary
```
![](images/Pasted%20image%2020251124165053.png)

### **Results**  

The summary output shows that SW1 is operating in rapid-pvst mode and is the root bridge for all VLANs (1–99). All spanning-tree instances have zero blocking ports and all active ports are in the forwarding state, which confirms that SW1 functions as the central root switch for the network.

### SW2 - Verification

```plaintext
enable
show spanning-tree summary
```
![](images/Pasted%20image%2020251124165306.png)

### **Results** (SW2) 

The summary output shows that SW2 is not the root bridge and correctly identifies SW1 as the active root. Each VLAN instance displays one blocking port, which represents the alternate port on the redundant link. All remaining ports are in the forwarding state. This confirms proper STP behavior: SW2 selects a single root port toward SW1 and blocks the backup path to prevent loops.

### SW3 - Verification

```plaintext
enable
show spanning-tree summary
```
![](images/Pasted%20image%2020251124165437.png)

### Results (SW3)

The summary output shows that SW3 is not the root bridge but participates normally in rapid-pvst. Both VLAN instances (30 and 40) have zero blocking ports. All active ports operate in the forwarding state, and the switch has one root port toward the distribution layer. This confirms that SW3 functions correctly as an access switch with stable, loop-free STP behavior.

### SW4 - Verification

```plaintext
enable
show spanning-tree summary
```
![](images/Pasted%20image%2020251124165532.png)

### **Results (SW4)**

The summary output shows that SW4 is not the root bridge and operates normally in rapid-pvst mode. All VLAN instances (50, 60, 70, 80, 90) have zero blocking ports, and every active interface is in the forwarding state. This confirms that SW4 behaves correctly as an access switch, with stable spanning-tree operation and a single root path toward the distribution layer.

### SW5 - Verification

```plaintext
enable
show spanning-tree summary
```
![](images/Pasted%20image%2020251124165632.png)

### Results (SW5) 

The summary output shows that SW5 is not the root bridge and operates correctly in rapid-pvst mode. All VLAN instances (60, 70, 80) have zero blocking ports, and all active interfaces remain in the forwarding state. This confirms that SW5 functions properly as an access switch, with a clean and loop-free spanning-tree state and a single root path toward the distribution layer.

<br>

### **Final Results**

The spanning-tree summaries confirm stable Rapid PVST operation across the network. SW1 is the Root Bridge with all ports forwarding. SW2 selects a single root port toward SW1 and blocks the redundant link. Access switches SW3, SW4, and SW5 show only forwarding ports and a clear root path. The entire STP domain is loop-free and behaves exactly as designed.

<br>

## **5.7 Apply PortFast on All Access Ports**

This step enables PortFast on access ports across all switches. PortFast allows host-facing interfaces to move immediately into the forwarding state, improving connection speed for PCs and servers. It must be used only on access ports that connect to end devices. For demonstration, the configuration example below is shown on SW3.

Configuration Example (SW3 Access Ports)

````plaintext
enable
configure terminal
interface Gi0/0
spanning-tree portfast
interface Gi0/1
spanning-tree portfast
interface Gi0/2
spanning-tree portfast
end
write
````
![](images/Pasted%20image%2020251124170641.png)

### **WARNING!!!** 

***PortFast is applied only on host-facing ports. It must not be enabled on trunk ports or redundant uplinks.***

### **Results**  

PortFast is successfully enabled on the selected access ports. Host connections transition directly to the forwarding state without waiting for the standard STP listening and learning phases.

<br>
## **5.8 Configure BPDU Guard on Edge Ports**


BPDU Guard adds protection to PortFast-enabled access ports. If an unexpected BPDU is received on a host-facing interface, the switch places the port into an error-disabled state. This prevents accidental or malicious connections of switches on edge ports and protects the stability of the spanning-tree domain. In this step, BPDU Guard is applied on all access ports across the network. For demonstration, the configuration example below shows SW5.

Configuration Example (SW5 Access Ports)

```plaintext
enable
configure terminal
interface Gi0/0
spanning-tree bpduguard enable
interface Gi0/1
spanning-tree bpduguard enable
interface Gi0/2
spanning-tree bpduguard enable
end
write
```
![](images/Pasted%20image%2020251124172135.png)


>**Notes:** BPDU Guard is used together with PortFast to secure host-facing interfaces. It must not be enabled on trunk links or redundant uplinks.

### **Results**  

BPDU Guard is successfully enabled on the selected access ports. Any received BPDU will automatically disable the port, preventing potential loops or misconfigurations from devices mistakenly connected to host interfaces.

<br>

## **5.9 Enable LLDP/CDP for Neighbor Detection**


This step enables LLDP and CDP on switches to provide neighbor discovery information. These protocols allow each device to identify directly connected neighbors, simplifying troubleshooting and verification during network configuration. LLDP is an open standard, while CDP is Cisco proprietary. Both are enabled globally for consistent visibility across the switching infrastructure.

Configuration Example (SW4)

```plaintext
enable
configure terminal
lldp run
cdp run
end
write
```
![](images/Pasted%20image%2020251124172708.png)
 
>**Notes:** LLDP and CDP help verify physical topology, interface connections, and device roles. They are used only for discovery and do not affect spanning-tree or forwarding behavior.

Verification Example (SW4)

```plaintext
show lldp neighbors detail
show cdp neighbors detail
```
![](images/Pasted%20image%2020251124172846.png)

### **Results**  

Results LLDP and CDP successfully detect the directly connected neighbor on SW4. Both commands identify SW5 on interface Gi0/3, confirming that the uplink between SW4 and SW5 is active and correctly reported by neighbor discovery protocols. LLDP provides chassis and system details, while CDP confirms platform, port mapping, and duplex settings.

<br>

## **5.10 Management VLAN 99 Transit Configuration**

 
This step prepares Management VLAN 99 as a dedicated transit and management segment for the core layer. VLAN 99 was already created on SW1 and SW2 in the previous chapter and is now used for two purposes: secure remote administration of the core switches via SSH and, later, as a backup Layer 3 transit path for OSPF between R1 and R2. If the primary point-to-point link in the 10.32.0.0/30 network fails, OSPF can form an adjacency over VLAN 99 and keep routing between both sides of the network operational.

Management VLAN 99 Overview (Core Only)

| Device | VLAN ID | SVI IP address | Description                                     |
| ------ | ------- | -------------- | ----------------------------------------------- |
| SW1    | 99      | 10.32.99.11/24 | Core switch 1 management & OSPF in next chapter |
| SW2    | 99      | 10.32.99.12/24 | Core switch 2 management & OSPF next chapter    |

Configuration SW1

```plaintext
enable
configure terminal
interface Vlan 99
ip address 10.32.99.11 255.255.255.0
no shutdown
end
write
```
![](images/Pasted%20image%2020251124184921.png)

Configuration SW2

 ```
enable
configure terminal
interface Vlan 99
ip address 10.32.99.12 255.255.255.0
no shutdown
end
write
 ```
![](images/Pasted%20image%2020251124185026.png)



>**Notes:** Management VLAN 99 is implemented only on the core switches (SW1 and SW2) in this project. It provides a logical Layer 2 path between R1 and R2 through the core, which will be used later when configuring router subinterfaces (for example 10.32.99.1 and 10.32.99.2) and OSPF adjacency over VLAN 99. At that stage, IP default gateways for SW1 and SW2 will point to the appropriate router address in VLAN 99 to allow remote SSH access from the management network.

## **Results** 

VLAN 99 is available on both core switches with dedicated SVI addresses. This prepares a separate management and transit segment that can be used for secure SSH administration and as a backup OSPF transport between R1 and R2 once Layer 3 configuration is introduced in the routing chapters.

<br>

## **5.11 SSH Configuration for Secure Remote Access**


This step prepares secure remote management on the core switches by enabling SSH on SW1 and SW2. SSH allows encrypted administrative access and will be fully tested once inter-VLAN routing is implemented. Only core switches require SSH at this stage, because they host the management SVI (VLAN 99) and act as primary control points for network operations.


Configuration SW1 

 ```
username admin privilege 15 secret project2
ip domain-name company.local
crypto key generate rsa 
2048
ip ssh version 2
line vty 0 4  
login local  
transport input ssh
 ```
![](images/Pasted%20image%2020251124190146.png)

Configuration SW2

 ```
username admin privilege 15 secret project2
ip domain-name company.local
crypto key generate rsa 
2048
ip ssh version 2
line vty 0 4  
login local  
transport input ssh
 ```
![](images/Pasted%20image%2020251124190333.png)

## **Results** 

SSH is now enabled on SW1 and SW2, allowing encrypted remote access through their management SVI addresses in VLAN 99. Full connectivity and login verification will be performed in the next chapter, once routing between all management devices is active.


<br>

## **5.12 Conclusion**

This fifth chapter prepares the switching layer for stable and secure operation. Rapid PVST is enabled on all switches to provide fast convergence and loop prevention. SW1 becomes the Root Bridge, and SW2 acts as the secondary bridge, creating predictable forwarding paths across the topology. PortFast and BPDU Guard are applied on access ports to speed up host connectivity and protect the network from accidental loops. LLDP and CDP give clear visibility of neighbor devices for easier verification and troubleshooting.

Management VLAN 99 is created on the core switches, and each switch receives a dedicated SVI address. This VLAN forms the management layer for future SSH access and acts as a backup L3 transit for OSPF in the next chapter. SSH configuration is completed on SW1 and SW2, preparing the core for secure remote administration once routing is fully enabled.



----

<br>

**Next chapter:** [Inter-VLAN Routing and OSPF](06-inter-vlan-routing-and-ospf.md)
