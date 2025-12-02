# **4 - VLAN and Trunk Configuration**


<br>

## **4.1 Introduction**

This chapter defines the VLAN structure and trunking configuration for the medium-sized company network. VLANs segment the environment into functional zones such as server, administration, executive, development, finance, logistic, warehouse, sales and printer segments. Trunk links carry multiple VLANs between switches and toward the core routing layer, ensuring that all relevant segments are transported across the switching topology. This chapter configures VLAN creation, access port assignments, trunking, and verification.


<br>

## **4.2 Topology Diagram**

![](images/Pasted%20image%2020251124153410.png)


<br>

## **4.3 VLAN Overview**

| **VLAN ID** | **Name**    | **Description**                              |
| ----------- | ----------- | -------------------------------------------- |
| 10          | Server      | Server infrastructure                        |
| 20          | Admin       | Administrative workstation                   |
| 30          | Executive   | Executive PC                                 |
| 40          | Development | Development PCs                              |
| 50          | Finance     | Finance workstation                          |
| 60          | Logistic    | Logistic workstation                         |
| 70          | Warehouse   | Warehouse workstation                        |
| 80          | Sales       | Sales workstation                            |
| 90          | Printer     | Network printer                              |
| 99          | Management  | Management VLAN (configured in next chapter) |


<br>

## **4.4 Objectives**

1. Create all VLANs across required switches.
    
2. Assign access ports to their correct VLANs.
    
3. Configure trunk ports and allowed VLAN lists.
    
4. Verify VLAN and trunk configuration.
    


<br>

## **4.5 VLAN Creation on Switches**

The switches require different VLAN sets based on their roles. SW1 and SW2 carry all VLANs because they form a redundant core path and must forward every segment across the topology. VLAN 99 is included for management and becomes fully operational after routing and OSPF configuration in later chapters.

### Switch SW1 (Server -> Switch)

**Configuration:**

```
enable
configure terminal
vlan 10
name Server
exit
vlan 20
name Admin
exit
vlan 30
name Executive
exit
vlan 40
name Development
exit
vlan 50
name Finance
exit
vlan 60
name Logistic
exit
vlan 70
name Warehouse
exit
vlan 80
name Sales
exit
vlan 90
name Printer
exit
vlan 99
name Management
exit
end
write
```
![](images/Pasted%20image%2020251124145010.png)

### **Results**  

SW1 now contains all VLANs required for server access and redundant switching.

<br>

### Switch SW2 (Admin + Core Facing)

**Configuration:**

```
enable
configure terminal
vlan 10
name Server
exit
vlan 20
name Admin
exit
vlan 30
name Executive
exit
vlan 40
name Development
exit
vlan 50
name Finance
exit
vlan 60
name Logistic
exit
vlan 70
name Warehouse
exit
vlan 80
name Sales
exit
vlan 90
name Printer
exit
vlan 99
name Management
exit
end
write
```
![](images/Pasted%20image%2020251124145130.png)

### **Results**  

SW2 includes all VLANs because it connects to both R2 and SW1.

<br>

### Switch SW3 (Executive and Development)

**Configuration:**

```
enable
configure terminal
vlan 30
name Executive
exit
vlan 40
name Development
exit
end
write
```
![](images/Pasted%20image%2020251124145237.png)

### **Results**  

SW3 includes only VLANs for the executive and development segments.


<br>

### Switch SW4 (Finance, Printer, Transit to SW5)

**Configuration:**

```
enable
configure terminal
vlan 50
name Finance
exit
vlan 90
name Printer
exit
vlan 60
name Logistic
exit
vlan 70
name Warehouse
exit
vlan 80
name Sales
exit
end
write
```
![](images/Pasted%20image%2020251124145343.png)

### **Results**  

SW4 includes VLANs for finance, printer, and all transit VLANs leading toward SW5.

<br>

### Switch SW5 (Logistic, Warehouse, Sales)

**Configuration:**

```
enable
configure terminal
vlan 60
name Logistic
exit
vlan 70
name Warehouse
exit
vlan 80
name Sales
exit
end
write
```
![](images/Pasted%20image%2020251124145428.png)

### **Results**  

SW5 includes VLANs needed for logistic, warehouse, and sales workstations.


<br>

## **4.6 Assign Access Ports to VLANs**

### Switch SW1

Server (VLAN 10):

```
enable
configure terminal
interface Gi0/0
switchport mode access
switchport access vlan 10
no shutdown
exit
end
write
```
![](images/Pasted%20image%2020251124145510.png)

### **Results**  

The server access port on SW1 is assigned to VLAN 10.

<br>

### Switch SW2

Admin workstation (VLAN 20):

```
enable
configure terminal
interface Gi0/0
switchport mode access
switchport access vlan 20
no shutdown
exit
end
write
```
![](images/Pasted%20image%2020251124145545.png)

### **Results**  

The admin workstation port is configured for VLAN 20.

<br>

### Switch SW3

Executive (VLAN 30):

```
enable
configure terminal
interface Gi0/0
switchport mode access
switchport access vlan 30
no shutdown
exit
end
write
```
![](images/Pasted%20image%2020251124145631.png)

Development (VLAN 40):

```
enable
configure terminal
interface Gi0/1
switchport mode access
switchport access vlan 40
no shutdown
interface Gi0/2
switchport mode access
switchport access vlan 40
no shutdown
exit
end
write
```
![](images/Pasted%20image%2020251124145703.png)

### **Results**  

Executive and development ports are assigned to their VLANs correctly.

<br>

### Switch SW4

Finance (VLAN 50):

```
enable
configure terminal
interface Gi0/2
switchport mode access
switchport access vlan 50
no shutdown
exit
end
write
```
![](images/Pasted%20image%2020251124145732.png)

Printer (VLAN 90):

```
enable
configure terminal
interface Gi0/1
switchport mode access
switchport access vlan 90
no shutdown
exit
end
write
```
![](images/Pasted%20image%2020251124145754.png)

### **Results**  

SW4 access ports now correctly serve finance and printer VLANs.

<br>

### Switch SW5

Logistic (VLAN 60):

```
enable
configure terminal
interface Gi0/2
switchport mode access
switchport access vlan 60
no shutdown
exit
end
write
```
![](images/Pasted%20image%2020251124145850.png)

Warehouse (VLAN 70):

```
enable
configure terminal
interface Gi0/0
switchport mode access
switchport access vlan 70
no shutdown
exit
end
write
```
![](images/Pasted%20image%2020251124145905.png)

Sales (VLAN 80):

```
enable
configure terminal
interface Gi0/1
switchport mode access
switchport access vlan 80
no shutdown
exit
end
write
```
![](images/Pasted%20image%2020251124145920.png)

### **Results**  

SW5 access ports are assigned to VLANs 60, 70, and 80.


<br>

## **4.7 Configure Trunk Ports**

Trunk ports carry multiple VLANs between switches and routers. SW1 and SW2 use a trunk link that transports all VLANs to preserve redundancy. Other trunks are limited to the VLANs required by their connected segments.

### Redundant link (SW1 <-> SW2)

All VLANs are allowed on the redundant trunk between SW1 and SW2.

**SW1 interface Gi0/3 and Gi0/2:**

```
enable
configure terminal
interface Gi0/3
switchport trunk encapsulation dot1q
switchport mode trunk
switchport trunk allowed vlan 10,20,30,40,50,60,70,80,90,99
no shutdown
exit
interface Gi0/2
switchport trunk encapsulation dot1q
switchport mode trunk
switchport trunk allowed vlan 10,20,30,40,50,60,70,80,90,99
no shutdown
exit 
end
write
```
![](images/Pasted%20image%2020251124150530.png)

**SW2 interface Gi0/3 and Gi0/1:**

```
enable
configure terminal
interface Gi0/3
switchport trunk encapsulation dot1q
switchport mode trunk
switchport trunk allowed vlan 10,20,30,40,50,60,70,80,90,99
no shutdown
exit
interface Gi0/1
switchport trunk encapsulation dot1q
switchport mode trunk
switchport trunk allowed vlan 10,20,30,40,50,60,70,80,90,99
no shutdown
exit
end
write
```
![](images/Pasted%20image%2020251124150640.png)

### R2 <-> SW3 (Executive and Development)

Allowed VLANs 30 and 40.

**SW3 Gi0/3:**

```
enable
configure terminal
interface Gi0/3
switchport trunk encapsulation dot1q
switchport mode trunk
switchport trunk allowed vlan 30,40
no shutdown
exit
end
write
```
![](images/Pasted%20image%2020251124150722.png)

### R2 <-> SW2 (Core link)

All VLANs are allowed toward the core router.

**SW2 Gi0/2:**

```
enable
configure terminal
interface Gi0/2
switchport trunk encapsulation dot1q
switchport mode trunk
switchport trunk allowed vlan 10,20,30,40,50,60,70,80,90,99
no shutdown
exit
end
write
```
![](images/Pasted%20image%2020251124150758.png)


### **R1 <-> SW1 (Core link)**

All VLANs are allowed toward the core router.

**SW1 Gi/0:**

```
enable
configure terminal
interface Gi0/1
switchport trunk encapsulation dot1q
switchport mode trunk
switchport trunk allowed vlan 10,20,30,40,50,60,70,80,90,99
no shutdown
exit
end
write
```
![](images/Pasted%20image%2020251124195200.png)

### R1 <-> SW4 (Finance, Printer, Transit)

Allowed VLANs 50, 60, 70, 80, 90,

**SW4 Gi0/0:**

```
enable
configure terminal
interface Gi0/0
switchport trunk encapsulation dot1q
switchport mode trunk
switchport trunk allowed vlan 50,60,70,80,90
no shutdown
exit
end
write
```
![](images/Pasted%20image%2020251124175815.png)

### SW4 <-> SW5 (Transit to Logistic, Warehouse, Sales)

Allowed VLANs 60, 70, 80.

**SW4 Gi0/3:**

```
enable
configure terminal
interface Gi0/3
switchport trunk encapsulation dot1q
switchport mode trunk
switchport trunk allowed vlan 60,70,80
no shutdown
exit
end
write
```
![](images/Pasted%20image%2020251124150936.png)

**SW5 Gi0/3:**

```
enable
configure terminal
interface Gi0/3
switchport trunk encapsulation dot1q
switchport mode trunk
switchport trunk allowed vlan 60,70,80
no shutdown
exit
end
write
```
![](images/Pasted%20image%2020251124151005.png)


>**Notes:** IOSv-L2 switches in GNS3 require the manual command _switchport trunk encapsulation dot1q_ on every trunk interface because 802.1Q is not enabled by default.

### **Results**  

All trunk ports are configured with IEEE 802.1Q encapsulation and the correct allowed VLAN lists. The redundant trunk between SW1 and SW2 transports all VLANs, while access trunks are limited to the VLANs required by their segments.


<br>

## **4.8 Verification**

The following commands verify VLAN presence, trunk operation, and interface status on each switch.

### Switch SW1

```
enable
show vlan brief
show interfaces trunk
show interfaces status
```
![](images/Pasted%20image%2020251124195726.png)

### Switch SW2

```
enable
show vlan brief
show interfaces trunk
show interfaces status
```
![](images/Pasted%20image%2020251124151843.png)

### Switch SW3

```
enable
show vlan brief
show interfaces trunk
show interfaces status
```
![](images/Pasted%20image%2020251124151945.png)

### Switch SW4

```
enable
show vlan brief
show interfaces trunk
show interfaces status
```
![](images/Pasted%20image%2020251124152129.png)

### Switch SW5

```
enable
show vlan brief
show interfaces trunk
show interfaces status
```
![](images/Pasted%20image%2020251124152221.png)

### **Results**

All switches show the correct VLANs, operational trunk ports, and active access interfaces. The configuration matches the project design, confirming correct Layer 2 segmentation.

<br>

## **4.9 Conclusion**

This chapter defines and implements the VLAN structure for all workstation, server, and transit segments. All required VLANs are created across the switching infrastructure, and access ports are assigned according to the network topology. Trunk links are configured to transport the correct VLAN sets between core and access switches, including the redundant connection between SW1 and SW2.

These configurations complete the fundamental Layer 2 segmentation of the network and provide the switching foundation required for implementing STP, Management VLAN 99, and other advanced switching features introduced in the next chapter.

---

<br>

**Next part:** [Advanced Switching Features](05-advanced-switching-features.md)
