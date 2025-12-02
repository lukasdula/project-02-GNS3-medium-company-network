# **11 - Troubleshooting**

<br>

## **11.1 Introduction**

This troubleshooting chapter demonstrates how simulated network faults are identified and corrected in a medium-size routed and switched topology. Each issue represents a realistic configuration error that affects Layer 3 reachability, management access, or DHCP relay operation. The diagnostics focus on simple tests such as ping, interface inspection, routing table checks, and verification commands on routers and switches. The goal is to confirm the root cause and restore normal connectivity.

<br>

## **11.2 Objectives**

This troubleshooting chapter covers three independent faults:

1. R2 is missing the default route required for forwarding traffic to external networks.
    
2. SW1 uses an incorrect IP address on the management SVI for VLAN 99.
    
3. R2 contains an incorrect DHCP relay configuration for VLAN 40 (wrong helper-address).
    

Each fault is diagnosed, corrected, and verified to show that the network returns to normal operation.

<br>

## **11.3 Missing Default Route on R2**


This troubleshooting section addresses a routing issue where devices located behind R2 (VLAN 20, VLAN 30, VLAN 40) are unable to reach the ISP or external networks. The issue is caused by the absence of a valid default route on R2, preventing it from forwarding outbound traffic.

### **Symptoms**

- Devices in VLAN 20/30/40 cannot ping ISP (198.51.100.5) or Internet Host (198.51.100.6).
    
- Error message: **Destination Host Unreachable**.
    
- The unreachable reply originates from **10.32.20.1** (R2).
    

### **1) Diagnostics on R2**

#### Ping Test from VLAN 20

```
ping 198.51.100.5
Destination Host Unreachable
```
![](images/Pasted%20image%2020251201174237.png)

#### Ping Test from VLAN 30

```
ping 198.51.100.6
Destination Host Unreachable
```
![](images/Pasted%20image%2020251201174315.png)

#### Routing Table on R2

```
show ip route
Gateway of last resort is not set
```
![](images/Pasted%20image%2020251201174346.png)

R2 has no default route (0.0.0.0/0) configured.

### Root Cause

R2 is missing a default route pointing towards R1. Without this route, R2 does not know how to forward packets destined for external networks, resulting in unreachable errors.


<br>

### 2) **Fix**

Add the correct default route on R2:

```
ip route 0.0.0.0 0.0.0.0 10.32.0.1
```
![](images/Pasted%20image%2020251201174454.png)

### **Verification**

#### Verification of Routing Table on R2

```
show ip route
S* 0.0.0.0/0 [1/0] via 10.32.0.1
```
![](images/Pasted%20image%2020251201174529.png)

#### Ping Test from R2

```
ping 198.51.100.5
!!!!!
```
![](images/Pasted%20image%2020251201174614.png)

#### Ping Test from VLAN 20/30/40

```
ping 198.51.100.6
!!!!!
```
![](images/Pasted%20image%2020251201174625.png)

### **Results**

- R2 now forwards traffic correctly to R1.
    
- Devices in VLAN 20/30/40 can reach the ISP and Internet Host.
    
- NAT/PAT translations on R1 confirm traffic flow from these VLANs.

<br>

## **11.4 Wrong IP Address on VLAN 99 (Management VLAN)**


This troubleshooting case focuses on a misconfigured IP address on the management SVI for VLAN 99 on SW1. The switch has an incorrect IP address in the management network, so it cannot reach the default gateway on R1 or respond to management traffic. Layer 2 switching and trunking work normally, but all management connectivity fails.

##### **Symptoms:**

- SW1 cannot ping the default gateway 10.32.99.1.
    
- R1 and Admin VLAN cannot ping SW1.
    
- Management VLAN 99 appears active, but has no Layer 3 connectivity.
    

### **1) Ping Tests**

#### Device: Xubuntu-Admin (VLAN 20)

Test connection to SW1.

```prikaz
ping 10.32.99.11    # Expected: fail
```
![](images/Pasted%20image%2020251201201926.png)

Result: Ping fails.

#### Device: R1

Test connection to SW1.

```prikaz
ping 10.32.99.11    # Expected: fail
```
![](images/Pasted%20image%2020251201202501.png)

Result: Ping fails.

#### Device: SW1

Test connection to R1 gateway.

```prikaz
ping 10.32.99.1    # Expected: fail
```
![](images/Pasted%20image%2020251201202555.png)

Result: Ping fails.

### **2) Diagnostics**

#### Devices for Testing

- **SW1** – SVI configuration
    
- **R1** – default gateway for VLAN 99
    
- **Xubuntu-Admin** – external verification
    

#### 2.1 Check VLAN 99 SVI on SW1

```prikaz
show running-config interface vlan 99
```
![](images/Pasted%20image%2020251201202629.png)

**!!!SW1 uses an incorrect IP address (10.32.90.11 instead of 10.32.99.11)!!!**


#### 2.2 Check VLAN 99 Gateway on R1

```prikaz
R1#show ip int br
```
![](images/Pasted%20image%2020251201203640.png)

## **Results**

The output confirms that SW1 is using the wrong SVI address for VLAN 99 (**10.32.90.11** instead of **10.32.99.11**). On R1, the gateway for VLAN 99 (**10.32.99.1**) is configured correctly and the subinterface is up. Because the two IP addresses are in different subnets, L3 connectivity cannot work, which explains the failed pings and unreachable management VLAN.

This verifies that the issue is caused by the incorrect IP address on SW1.

<br>

### **3) Fix**

Correct the IP address on SW1.

```prikaz
configure terminal
int vlan 99
no ip address 10.32.90.11 255.255.255.0
ip address 10.32.99.11 255.255.255.0
end
write memory
```
![](images/Pasted%20image%2020251201204123.png)



<br>

### **4) Verification**


#### 4.1 – SW1 to R1

```prikaz
ping 10.32.99.1
```
![](images/Pasted%20image%2020251201204146.png)

Expected: success.

#### 4.2 – R1 to SW1

```prikaz
ping 10.32.99.11
```
![](images/Pasted%20image%2020251201204256.png)

Expected: success.

#### 4.3 – Admin VLAN to SW1

```prikaz
ping 10.32.99.11
```
![](images/Pasted%20image%2020251201204346.png)

Expected: success.

### **Results**

- SW1 now uses the correct IP for VLAN 99.
    
- Management connectivity between SW1, R1, and Admin VLAN works normally.
    
- VLAN 99 routing is fully restored.


<br>


## **11.5 Troubleshooting DHCP Relay (Development VLAN)**


Development VLAN (40) shows signs of abnormal behavior. The workstation in VLAN 40 is unable to obtain a correct IP configuration from DHCP. Other VLANs operate normally and receive valid DHCP leases. The issue appears isolated.

### 1) Diagnostics

<br>

### 1. Verify IP configuration on PC2 (Development VLAN)

```prikaz
ip dhcp
```
![](images/Pasted%20image%2020251202012907.png)

Result: The device assigns a 169.254.x.x address, indicating that no DHCP offer is received.

### 2. Check VLAN presence on SW3

```prikaz
show vlan brief
```
![](images/Pasted%20image%2020251202022154.png)

Result: VLAN 40 is present in the VLAN database.

### 3. Inspect subinterface configuration on R2

```prikaz
show running-config interface gi0/3.40
```
![](images/Pasted%20image%2020251202022230.png)

Result: Subinterface Gi0/3.40 is active and configured with the correct IP address and encapsulation.

### 4. Verify routing table on R2

```prikaz
show ip route 10.32.10.0
```
![](images/Pasted%20image%2020251202022259.png)

Result: Route to the Server VLAN is present.

### 6. Verify DHCP relay configuration on R2

```
show running-config interface gi0/3.40
```
![](images/Pasted%20image%2020251202022825.png)

Result: The interface contains a helper-address, but the value does not match the DHCP server IP. The configured address points to 10.32.100.10, which is not reachable and does not host a DHCP service.

<br>

### **2) Fix**

Replace the incorrect helper-address with the correct DHCP server IP.

```prikaz
configure terminal
interface Gi0/3.40
no ip helper-address 10.32.100.10
ip helper-address 10.32.10.10
exit
end
write memory
```
![](images/Pasted%20image%2020251202022844.png)

<br>

### 3) **Verification**

<br>

### 1. Renew DHCP on PC2

```prikaz
ip dhcp
```
![](images/Pasted%20image%2020251202022915.png)

Result: PC2 receives a correct lease in the 10.32.40.0/24 range.

### 2. Confirm DHCP operation in running configuration

```prikaz
show running-config interface gi0/3.40
```
![](images/Pasted%20image%2020251202022954.png)

### **Results**

Development VLAN clients now receive valid DHCP leases. Inter-VLAN traffic operates normally, and relay functionality is restored after adding the missing helper address.


<br>


## **11.6 Conclusion**

The troubleshooting scenarios demonstrate that configuration faults on Layer 3 devices, management SVIs, and DHCP relay pathways can significantly affect network operation. By applying structured diagnostics and verifying routing, VLAN assignment, and service parameters, each issue is isolated and resolved. After correcting all three faults, the network operates with full inter-VLAN routing, working management access, and functional DHCP delivery.


---

**Final chapter:** [Conclusion and Summary](12-conclusion-and-summary.md)
