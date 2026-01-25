# **6 - Inter VLAN Routing and OSPF**

<br>

## **6.1 Introduction**  

This chapter explains how the network moves from basic VLAN separation to full Layer 3 routing. The chapter shows, in a simple and clear way, how R1 and R2 provide communication between VLANs and how OSPF delivers dynamic routing for the whole network.

First, router-on-a-stick subinterfaces are created on R1 and R2. Each VLAN receives one gateway IP address on the correct router. After basic Layer 3 connectivity is established, OSPF operates as the routing protocol that shares routing information between the routers. All VLANs, the point-to-point link and the management network in VLAN 99 are placed in Area 0.

The management network (VLAN 99) has two purposes: it provides routed access to switches and also acts as a secondary OSPF path. If the main link between R1 and R2 goes down, OSPF redirects traffic through VLAN 99 so inter-VLAN communication continues without interruption.

<br>

## **6.2 Topology Diagram**

![](images/Pasted%20image%2020260125064337.png)

<br>



## **6.3 Objectives**

1. Configure 802.1Q subinterfaces on R1 and R2 only for VLANs where each router provides the default gateway.
    
2. **Configure default gateways for the management network (VLAN 99) on SW1 and SW2.**
    
3. Create OSPF routing on R1 and R2, including Router ID configuration, Area 0 assignment and advertising all VLAN and transit networks.
    
4. Establish OSPF adjacency between R1 and R2 using the primary point-to-point link 10.32.0.0/30.
    
5. Implement Management VLAN 99 on both routers and include it in OSPF as a backup transit path.
    
6. Simulate primary link failure and confirm that OSPF automatically transitions to VLAN 99 as the secondary routing path.
    
7. Configure external routing by adding a default route on R1 towards the ISP and a return static route on the ISP to enable full connectivity for NAT/PAT in a later chapter.

<br>

## **6.4 Configure Subinterfaces on R1 and R2**


This objective focuses on creating router-on-a-stick subinterfaces on R1 and R2. Each router receives only the VLANs for which it provides the default gateway. This ensures clean Layer 3 separation and predictable routing across the network. All subinterfaces use 802.1Q encapsulation.



#### R1 Configuration

```
enable
configure terminal
interface Gi0/1.10
encapsulation dot1Q 10
ip address 10.32.10.1 255.255.255.0
no shutdown
exit
interface Gi0/4.50
encapsulation dot1Q 50
ip address 10.32.50.1 255.255.255.0
no shutdown
exit
interface Gi0/4.60
encapsulation dot1Q 60
ip address 10.32.60.1 255.255.255.0
no shutdown
exit
interface Gi0/4.70
encapsulation dot1Q 70
ip address 10.32.70.1 255.255.255.0
no shutdown
exit
interface Gi0/4.80
encapsulation dot1Q 80
ip address 10.32.80.1 255.255.255.0
no shutdown
exit
interface Gi0/4.90
encapsulation dot1Q 90
ip address 10.32.90.1 255.255.255.0
no shutdown
exit
interface Gi0/1.99
encapsulation dot1Q 99
ip address 10.32.99.1 255.255.255.0
no shutdown
exit
end
write memory
```
![](images/Pasted%20image%2020251124223846.png)

#### R2 Configuration

```
enable
configure terminal
interface Gi0/2.20
encapsulation dot1Q 20
ip address 10.32.20.1 255.255.255.0
no shutdown
exit
interface Gi0/3.30
encapsulation dot1Q 30
ip address 10.32.30.1 255.255.255.0
no shutdown
exit
interface Gi0/3.40
encapsulation dot1Q 40
ip address 10.32.40.1 255.255.255.0
no shutdown
exit
interface Gi0/2.99
encapsulation dot1Q 99
ip address 10.32.99.2 255.255.255.0
no shutdown
exit
end
write memory
```
![](images/Pasted%20image%2020251124224847.png)

### **Results**

All VLANs now receive a correct default gateway on the appropriate router. Inter-VLAN routing becomes possible once OSPF is configured in the next objective.

<br>

### **Verification**

#### R1 Verification

```
show ip interface brief
```
![](images/Pasted%20image%2020251124224440.png)

#### R2 Verification

```
show ip interface brief
```
![](images/Pasted%20image%2020251124225024.png)

<br>

## **6.5 Configure Default Gateways for VLAN 99 on SW1 and SW2**


This part ensures that both switches in the management network (VLAN 99) use the correct routed next-hop devices. SW1 sends traffic to R1, and SW2 sends traffic to R2. This step is required for routed management access and for enabling the OSPF backup path through VLAN 99.

#### SW1 Configuration

```
enable
configure terminal
ip default-gateway 10.32.99.1
end
write memory
```
![](images/Pasted%20image%2020251124225630.png)

#### SW2 Configuration

```
enable
configure terminal
ip default-gateway 10.32.99.2
end
write memory
```
![](images/Pasted%20image%2020251124225702.png)

### **Results**

Both switches can now reach their respective routers as the next-hop device for management traffic. This makes routed management possible and prepares the management network for OSPF-based failover.

<br>

### **Verification**

#### SW1 Verification

```
ping 10.32.99.1
```
![](images/Pasted%20image%2020251124230201.png)

#### SW2 Verification

```
ping 10.32.99.2
```
![](images/Pasted%20image%2020251124230121.png)

<br>

## **6.6 Create OSPF Routing on R1 and R2**

OSPF provides dynamic routing for all internal networks in this topology. R1 and R2 advertise the VLAN gateway networks, the management network (VLAN 99), and the point-to-point transit link. Both routers operate in Area 0 to keep routing simple and consistent across the whole network.

#### R1 Configuration

```
enable
configure terminal
router ospf 1
router-id 1.1.1.1
network 10.32.10.0 0.0.0.255 area 0
network 10.32.50.0 0.0.0.255 area 0
network 10.32.60.0 0.0.0.255 area 0
network 10.32.70.0 0.0.0.255 area 0
network 10.32.80.0 0.0.0.255 area 0
network 10.32.90.0 0.0.0.255 area 0
network 10.32.99.0 0.0.0.255 area 0
network 10.32.0.0 0.0.0.3 area 0
exit
end
write memory
```
![](images/Pasted%20image%2020251124230850.png)

#### R2 Configuration

```
enable
configure terminal
router ospf 1
router-id 2.2.2.2
network 10.32.20.0 0.0.0.255 area 0
network 10.32.30.0 0.0.0.255 area 0
network 10.32.40.0 0.0.0.255 area 0
network 10.32.99.0 0.0.0.255 area 0
network 10.32.0.0 0.0.0.3 area 0
exit
end
write memory
```
![](images/Pasted%20image%2020251124230924.png)

### **Results**

OSPF routing is now active on both routers. VLAN networks, management networks, and the transit network are successfully advertised. The routers are prepared to form adjacency in the next objective.

<br>

### **Verification**

#### R1 Verification

```
show ip ospf neighbor
```
![](images/Pasted%20image%2020251124231024.png)
 ```
show ip route ospf
 ```
![](images/Pasted%20image%2020251124231101.png)

### **Results (R1)**

R1 successfully forms OSPF neighbor relationships with R2 over both the primary point-to-point link and the management VLAN 99. All VLAN networks assigned to R2 are correctly learned through OSPF. At this stage, both paths are active, but VLAN 99 is intended to serve as the backup path. The next step will include adjusting the OSPF cost on the VLAN 99 interface to ensure it acts as the secondary route.

#### R2 Verification

```
show ip ospf neighbor
show ip route ospf
```
![](images/Pasted%20image%2020251124231142.png)

### **Results (R2)**

R2 successfully establishes OSPF neighbor relationships with R1 across the point-to-point link and the management VLAN 99. All VLAN networks provided by R1 are correctly received through OSPF. Both links are currently equal in cost, but VLAN 99 is designed to be the backup path. In the following step, the OSPF cost will be increased on the VLAN 99 interface to ensure it functions purely as the fallback route.


--- 

<br>

### **6.6.1 Adjusting OSPF Path Costs for Primary and Backup Links**

This section configures OSPF interface costs so that the point-to-point link between R1 and R2 becomes the primary routing path, while VLAN 99 functions strictly as the backup path.

#### R1 Configuration

```
enable
configure terminal
interface Gi0/1.99
ip ospf cost 50
exit
interface Gi0/0
ip ospf cost 10
exit
end
write memory
```
![](images/Pasted%20image%2020251124232351.png)

#### R2 Configuration

```
enable
configure terminal
interface Gi0/2.99
ip ospf cost 50
exit
interface Gi0/0
ip ospf cost 10
exit
end
write memory
```
![](images/Pasted%20image%2020251124232437.png)

### **Results**

The point-to-point link now carries the lowest cost and becomes the preferred OSPF path between R1 and R2. VLAN 99 remains available, but it is used only when the primary link fails.

#### **Verification**

```
show ip route ospf
```
![](images/Pasted%20image%2020251124232556.png)

OSPF now prefers the point-to-point link as the primary path, while VLAN 99 remains available only as the backup route.

<br>

## **6.7 Simulate Primary Link Failure and VLAN 99 Failover**


This objective verifies that OSPF automatically switches from the primary point-to-point link to the backup path in VLAN 99 when the main link between R1 and R2 goes down. The test confirms that dynamic routing keeps inter-VLAN communication available even during a link failure.

### Configuration

#### Step 1 - Shut down the primary link on R2

```
enable
configure terminal
interface GigabitEthernet0/0
shutdown
exit
end
write memory
```
![](images/Pasted%20image%2020251124234226.png)

#### Step 2 - (Optional) Restore the primary link on R2

This step is used after verification to return the topology to the normal production state.

```
enable
configure terminal
interface GigabitEthernet0/0
no shutdown
exit
end
write memory
```

<br>

### Verification

#### R1 Verification

Check that OSPF has moved the neighbor relationship and routes to the VLAN 99 path.

```
show ip ospf neighbor
show ip route ospf
```
![](images/Pasted%20image%2020251124234324.png)

#### R2 Verification

Confirm that R2 also uses VLAN 99 as the only OSPF path to R1.

```
show ip ospf neighbor
show ip route ospf
```
![](images/Pasted%20image%2020251124234505.png)

### Results

After GigabitEthernet0/0 on R2 is shut down, OSPF removes the routes that use the point-to-point link as the next hop. Both routers keep a single OSPF neighbor relationship over the management network in VLAN 99, and all learned routes point to the backup subinterfaces. This behavior confirms that VLAN 99 successfully operates as the secondary routing path when the primary link fails.

<br>

## **6.8 External Routing Configuration**


External routing connects the internal medium-company network to the simulated ISP. R1 acts as the gateway router, forwarding all non-local traffic toward the ISP. A default route is configured on R1, and a corresponding return route is added on the ISP to ensure two‑way connectivity.

### Configuration

#### R1 Default Route

```
configure terminal
ip route 0.0.0.0 0.0.0.0 203.0.113.6
end
write memory
```
![](images/Pasted%20image%2020251125001729.png)
#### ISP Return Route

```
configure terminal
ip route 10.32.0.0 255.255.0.0 203.0.113.5
end
write memory
```
![](images/Pasted%20image%2020251125001821.png)

R2 Default Route


```
ip route 0.0.0.0 0.0.0.0 10.32.0.1
```
![](images/Pasted%20image%2020251126003520.png)

### Results

R2 now forwards all unknown traffic to R1 via the transit network 10.32.0.0/30. Devices in VLAN 20, 30 and 40 can reach the ISP router and the Internet Host after NAT/PAT is applied on R1.


### **Verification**

```
show ip route
ping 198.51.100.5
ping 10.32.10.10
```
![](images/Pasted%20image%2020251125002240.png)


<br>

## **6.9 Concuslion**

The chapter defines how the network moves from basic VLAN separation to full Layer 3 routing. Subinterfaces are created on both routers so that each VLAN receives a correct default gateway. The switches in VLAN 99 obtain routed management access through their assigned next-hop addresses. OSPF is enabled on R1 and R2, advertising all gateway networks and the transit link in Area 0.  
Path costs are adjusted so the point-to-point link becomes the primary OSPF path, while VLAN 99 acts only as the backup. When the primary link is shut down, OSPF immediately shifts traffic to VLAN 99, keeping all inter-VLAN communication available.  
Finally, a default route on R1 and a return route on the ISP prepare the network for full external connectivity and future NAT/PAT configuration.


--- 

<br>

**Next part:** [Core Server Services](07-core-server-services.md)
