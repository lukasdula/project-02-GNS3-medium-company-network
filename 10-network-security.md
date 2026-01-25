# **10 - Network Security**

<br>

## **10.1 Introduction**

This chapter defines the main security controls that protect the medium-sized company network. The goal is to create a secure baseline on all devices, limit access on switch ports, enable safe remote management, and control communication between VLANs. The security configuration builds on the earlier topology, routing and services. It improves the network with passwords, encrypted access, SSH management, port security and Access Control Lists.

<br>

## **10.2 Topology Diagram**

![](images/Pasted%20image%2020260125065511.png)

<br>

## **10.3 Objectives**

1. Define basic access security on all routers and switches, including passwords, encrypted credentials and a message-of-the-day banner.
    
2. Configure an administrator account with local authentication.
    
3. Apply port security on all host-facing switch ports.
    
4. Enable SSH management from the Admin PC and verify remote access to a selected switch.
    
5. Deploy Access Control Lists to control inter-VLAN communication.


<br>

## **10.4 Basic Access Security**

This section defines the baseline protection for all routers and switches. It secures privileged access, enables local authentication, protects console and VTY access, and applies a security banner. The configuration is demonstrated on one device (R1), but the same logic must applies to all routers and switches.

<br>

### **10.4.1 Enable Secret and Administrator Account**

This part secures privileged EXEC mode and creates a local admin account for authentication.

**Device:** R1

```plaintext
enable
configure terminal
enable secret Netcore1994
username admin secret adminpass1994
end
write memory
```
![](images/Pasted%20image%2020251129135245.png)

### **Results:**  

Privileged access is protected with an encrypted enable secret. A local admin account is created and ready for console or SSH authentication.

### Notes

Simple passwords are used for demonstration. Real networks require long, complex passwords with multiple character types.


<br>

### **10.4.2 Console and VTY Line Security**

This part protects physical console access and remote VTY access. Console uses a direct password, while VTY lines require local authentication.

**Device:** R1

```plaintext
enable
configure terminal
line console 0
password project2
login local
exec-timeout 5 0
exit
line vty 0 4
login local
transport input ssh
exec-timeout 10 0
end
write memory
```
![](images/Pasted%20image%2020251129135601.png)

### **Results:**  

Console sessions require authentication and automatically close after inactivity. Remote access over VTY lines is limited to SSH and protected with local login.

<br>

### **10.4.3 Message-of-the-Day Banner**

This banner provides a simple legal warning before login.

**Device:** R1

```plaintext
enable
configure terminal
banner motd # Access restricted to authorized personnel. #
end
write memory
```
![](images/Pasted%20image%2020251129140010.png)

### **Results:**  

A warning banner is displayed before any login attempt, creating a consistent security notice across the network.

<br>

### **10.4.1 Password Encryption**

This part enables service password encryption so that all plain-text passwords in the configuration are stored in an obscured form.

**Device:** R1

```plaintext
enable
configure terminal
service password-encryption
end
write memory
```
![](images/Pasted%20image%2020251129140414.png)

### **Results**

All plain-text passwords (such as console or VTY passwords) are encrypted in the running configuration.

### **Diagnostics**

To verify that password encryption is active, check the running configuration.

**Device:** R1

```plaintext
show running-config | include password
```
![](images/Pasted%20image%2020251129140456.png)
### **Results**

Passwords appear in encrypted form instead of plain text, confirming that password encryption is enabled.

<br>

## **10.5 Port Security**

This section defines port-level protection on access interfaces. Port security limits the number of allowed MAC addresses and prevents unauthorized devices from connecting. The configuration is demonstrated on one switch (SW4), but the same logic applies to all access switches in the network.

### **Configuration Static MAC Binding**

This part assigns one allowed MAC address to the access port that connects to the printer. Only a single known device is permitted.

**Device:** SW4

```plaintext
enable
configure terminal
interface gi0/1
switchport mode access
switchport port-security
switchport port-security mac-address sticky
switchport port-security maximum 1
switchport port-security violation restrict
end
write memory
```
![](images/Pasted%20image%2020251129143648.png)

### **Results**

The port accepts only one dynamically learned MAC address. Any additional device triggers a restrict action.

### **Diagnostics**

This diagnostics section verifies how port security behaves when a new device is connected to the protected port. The test demonstrates how the switch learns a single MAC address and enforces the configured security policy.


**Device:** SW4

```plaintext
show port-security interface Gi0/1
show port-security address
```
![](images/Pasted%20image%2020251129150110.png)

The port learned the MAC address **0050.7966.6889** as a _SecureSticky_ entry. The interface remains _secure-up_ and accepts only this single device. No violations were detected, confirming that port security works as expected.

<br>
## **10.6 SSH configuration for SW1**


SSH is prepared on SW1 to allow secure remote management from the Xubuntu-admin PC. Both switches require correct IP routing in order to reach their gateways. The command "ip default-gateway" is configured on each switch so they can communicate with devices outside their local VLAN.

**SW1 configuration** 

 ```
enable
configure terminal
ip domain-name company.local
crypto key generate rsa
1024
ip ssh version 2
transport input ssh
exec-timeout 10
end
write memory
 ```



**Xubuntu-admin connection**  

 ```
ip a  
ip route  
ssh admin@10.32.99.11  
 ```
![](images/Pasted%20image%2020251129165009.png)

**Diagnostics**  

 ```
show ip interface brief  
ping 10.32.99.1  
 ```
 ![](images/Pasted%20image%2020251129165055.png)
### **Results**  

The SSH connection establishes successfully. The administrator can access SW1, verify interface states, and confirm connectivity to the default gateway R1 (10.32.0.1).

<br>

## **10.7 Advanced ACL Policy (ACL)**


This chapter applies Access Control Lists to control how every VLAN communicates inside the network. The goal of the ACL policy is to isolate user VLANs, protect the core infrastructure, and allow only the services that are required for normal operation.  

Each ACL is placed on the router subinterfaces, so the filtering happens directly at the default gateways. The design blocks unnecessary inter-VLAN traffic, limits access to privileged VLANs, and keeps important services reachable, such as DNS, HTTP, printing, and management tools. Diagnostics and internet connectivity remain available so the network can operate smoothly and be troubleshooted without restrictions.


## **10.7.1 Steps**

1. Isolate all user VLANs (30, 40, 50, 60, 70, 80) from each other.
2. Permit user VLANs to access Server VLAN (10) for DNS, HTTP, HTTPS, and DHCP only.
3. Allow printing services (IPP, LPD) to Printer VLAN (90) from all user VLANs.
4. Block user access to Admin VLAN (20) and Management VLAN (99).
5. Enable ICMP diagnostics (echo-reply, time-exceeded, unreachable).
6. Maintain internet access for all user VLANs via NAT/PAT.
7. Grant full access to Admin (20), Server (10), and Management (99) VLANs.


## **10.7.2 ACL Policy Overview**

| **Source VLAN**      | **Server (10) Access** | **Admin (20) Access** | **Printer (90) Access** | **Inter-VLAN** | **ICMP**       | **Internet** |
| -------------------- | ---------------------- | --------------------- | ----------------------- | -------------- | -------------- | ------------ |
| **10 - Server**      | N/A                    | Full                  | Full                    | Full           | Send echo      | Yes          |
| **20 - Admin**       | Full                   | N/A                   | Full                    | Full           | Send echo      | Yes          |
| **30 - Executive**   | DNS/HTTP/HTTPS/DHCP    | Echo-reply only       | IPP/LPD/Echo            | Deny all       | Reply to 10,20 | Yes          |
| **40 - Development** | DNS/HTTP/HTTPS/DHCP    | Echo-reply only       | IPP/LPD/Echo            | Deny all       | Reply to 10,20 | Yes          |
| **50 - Finance**     | DNS/HTTP/HTTPS/DHCP    | Echo-reply only       | IPP/LPD/Echo            | Deny all       | Reply to 10,20 | Yes          |
| **60 - Logistic**    | DNS/HTTP/HTTPS/DHCP    | Echo-reply only       | IPP/LPD/Echo            | Deny all       | Reply to 10,20 | Yes          |
| **70 - Warehouse**   | DNS/HTTP/HTTPS/DHCP    | Echo-reply only       | IPP/LPD/Echo            | Deny all       | Reply to 10,20 | Yes          |
| **80 - Sales**       | DNS/HTTP/HTTPS/DHCP    | Echo-reply only       | IPP/LPD/Echo            | Deny all       | Reply to 10,20 | Yes          |
| **90 - Printer**     | Echo-reply only        | Echo-reply only       | N/A                     | Deny all       | Reply only     | No           |
| **99 - Management**  | Full                   | Full                  | Full                    | Full           | Send echo      | Yes          |
|                      |                        |                       |                         |                |                |              |

**Protocol Legend:**

- **DNS** = UDP/TCP 53
- **HTTP** = TCP 80
- **HTTPS** = TCP 443
- **DHCP** = UDP 67/68
- **IPP** = TCP 631
- **LPD** = TCP 515



## **10.7.3 ACL Configuration on R1**

R1 provides default gateways for VLANs 10, 50, 60, 70, 80, 90, and 99.

### VLAN 10 - Server (Full Access)

```
enable
configure terminal
ip access-list extended ACL-VLAN10-OUT
permit ip any 10.32.10.0 0.0.0.255
permit ip any 10.32.99.0 0.0.0.255
exit
interface Gi0/1.10
ip access-group ACL-VLAN10-OUT out
exit
end
write memory
```
![](images/Pasted%20image%2020251129230154.png)

### VLAN 50 - Finance (Restricted)

```
enable
configure terminal
ip access-list extended ACL-VLAN50-OUT
permit udp any eq 68 any eq 67
permit udp 10.32.50.0 0.0.0.255 eq 68 10.32.10.0 0.0.0.255 eq 67
permit udp 10.32.50.0 0.0.0.255 eq 67 10.32.10.0 0.0.0.255 eq 68
permit udp 10.32.50.0 0.0.0.255 10.32.10.0 0.0.0.255 eq 53
permit tcp 10.32.50.0 0.0.0.255 10.32.10.0 0.0.0.255 eq 53
permit tcp 10.32.50.0 0.0.0.255 10.32.10.0 0.0.0.255 eq 80
permit tcp 10.32.50.0 0.0.0.255 10.32.10.0 0.0.0.255 eq 443
permit tcp 10.32.50.0 0.0.0.255 10.32.90.0 0.0.0.255 eq 631
permit tcp 10.32.50.0 0.0.0.255 10.32.90.0 0.0.0.255 eq 515
permit icmp 10.32.50.0 0.0.0.255 10.32.10.0 0.0.0.255 echo-reply
permit icmp 10.32.50.0 0.0.0.255 10.32.20.0 0.0.0.255 echo-reply
permit icmp 10.32.50.0 0.0.0.255 any time-exceeded
permit icmp 10.32.50.0 0.0.0.255 any unreachable
permit icmp 10.32.50.0 0.0.0.255 10.32.90.0 0.0.0.255 echo
deny ip 10.32.50.0 0.0.0.255 10.32.10.0 0.0.0.255 log
deny ip 10.32.50.0 0.0.0.255 10.32.20.0 0.0.0.255 log
deny ip 10.32.50.0 0.0.0.255 10.32.99.0 0.0.0.255 log
deny ip 10.32.50.0 0.0.0.255 10.32.90.0 0.0.0.255 log
deny ip 10.32.50.0 0.0.0.255 10.32.30.0 0.0.0.255 log
deny ip 10.32.50.0 0.0.0.255 10.32.40.0 0.0.0.255 log
deny ip 10.32.50.0 0.0.0.255 10.32.60.0 0.0.0.255 log
deny ip 10.32.50.0 0.0.0.255 10.32.70.0 0.0.0.255 log
deny ip 10.32.50.0 0.0.0.255 10.32.80.0 0.0.0.255 log
permit ip 10.32.50.0 0.0.0.255 any
exit
interface Gi0/4.50
ip access-group ACL-VLAN50-OUT in
exit
end
write memory
```
![](images/Pasted%20image%2020251129230216.png)


### VLAN 60 - Logistic (Restricted)

```
enable
configure terminal
ip access-list extended ACL-VLAN60-OUT
permit udp any eq 68 any eq 67
permit udp 10.32.60.0 0.0.0.255 eq 68 10.32.10.0 0.0.0.255 eq 67
permit udp 10.32.60.0 0.0.0.255 eq 67 10.32.10.0 0.0.0.255 eq 68
permit udp 10.32.60.0 0.0.0.255 10.32.10.0 0.0.0.255 eq 53
permit tcp 10.32.60.0 0.0.0.255 10.32.10.0 0.0.0.255 eq 53
permit tcp 10.32.60.0 0.0.0.255 10.32.10.0 0.0.0.255 eq 80
permit tcp 10.32.60.0 0.0.0.255 10.32.10.0 0.0.0.255 eq 443
permit tcp 10.32.60.0 0.0.0.255 10.32.90.0 0.0.0.255 eq 631
permit tcp 10.32.60.0 0.0.0.255 10.32.90.0 0.0.0.255 eq 515
permit icmp 10.32.60.0 0.0.0.255 10.32.10.0 0.0.0.255 echo-reply
permit icmp 10.32.60.0 0.0.0.255 10.32.20.0 0.0.0.255 echo-reply
permit icmp 10.32.60.0 0.0.0.255 any time-exceeded
permit icmp 10.32.60.0 0.0.0.255 any unreachable
permit icmp 10.32.60.0 0.0.0.255 10.32.90.0 0.0.0.255 echo
deny ip 10.32.60.0 0.0.0.255 10.32.10.0 0.0.0.255 log
deny ip 10.32.60.0 0.0.0.255 10.32.20.0 0.0.0.255 log
deny ip 10.32.60.0 0.0.0.255 10.32.99.0 0.0.0.255 log
deny ip 10.32.60.0 0.0.0.255 10.32.90.0 0.0.0.255 log
deny ip 10.32.60.0 0.0.0.255 10.32.30.0 0.0.0.255 log
deny ip 10.32.60.0 0.0.0.255 10.32.40.0 0.0.0.255 log
deny ip 10.32.60.0 0.0.0.255 10.32.50.0 0.0.0.255 log
deny ip 10.32.60.0 0.0.0.255 10.32.70.0 0.0.0.255 log
deny ip 10.32.60.0 0.0.0.255 10.32.80.0 0.0.0.255 log
permit ip 10.32.60.0 0.0.0.255 any
exit
interface Gi0/4.60
ip access-group ACL-VLAN60-OUT in
exit
end
write memory
```
![](images/Pasted%20image%2020251129230235.png)

### VLAN 70 - Warehouse (Restricted)

```
enable
configure terminal
ip access-list extended ACL-VLAN70-OUT
permit udp any eq 68 any eq 67
permit udp 10.32.70.0 0.0.0.255 eq 68 10.32.10.0 0.0.0.255 eq 67
permit udp 10.32.70.0 0.0.0.255 eq 67 10.32.10.0 0.0.0.255 eq 68
permit udp 10.32.70.0 0.0.0.255 10.32.10.0 0.0.0.255 eq 53
permit tcp 10.32.70.0 0.0.0.255 10.32.10.0 0.0.0.255 eq 53
permit tcp 10.32.70.0 0.0.0.255 10.32.10.0 0.0.0.255 eq 80
permit tcp 10.32.70.0 0.0.0.255 10.32.10.0 0.0.0.255 eq 443
permit tcp 10.32.70.0 0.0.0.255 10.32.90.0 0.0.0.255 eq 631
permit tcp 10.32.70.0 0.0.0.255 10.32.90.0 0.0.0.255 eq 515
permit icmp 10.32.70.0 0.0.0.255 10.32.10.0 0.0.0.255 echo-reply
permit icmp 10.32.70.0 0.0.0.255 10.32.20.0 0.0.0.255 echo-reply
permit icmp 10.32.70.0 0.0.0.255 any time-exceeded
permit icmp 10.32.70.0 0.0.0.255 any unreachable
permit icmp 10.32.70.0 0.0.0.255 10.32.90.0 0.0.0.255 echo
deny ip 10.32.70.0 0.0.0.255 10.32.10.0 0.0.0.255 log
deny ip 10.32.70.0 0.0.0.255 10.32.20.0 0.0.0.255 log
deny ip 10.32.70.0 0.0.0.255 10.32.99.0 0.0.0.255 log
deny ip 10.32.70.0 0.0.0.255 10.32.90.0 0.0.0.255 log
deny ip 10.32.70.0 0.0.0.255 10.32.30.0 0.0.0.255 log
deny ip 10.32.70.0 0.0.0.255 10.32.40.0 0.0.0.255 log
deny ip 10.32.70.0 0.0.0.255 10.32.50.0 0.0.0.255 log
deny ip 10.32.70.0 0.0.0.255 10.32.60.0 0.0.0.255 log
deny ip 10.32.70.0 0.0.0.255 10.32.80.0 0.0.0.255 log
permit ip 10.32.70.0 0.0.0.255 any
exit
interface Gi0/4.70
ip access-group ACL-VLAN70-OUT in
exit
end
write memory
```
![](images/Pasted%20image%2020251129230258.png)

### VLAN 80 - Sales (Restricted)

```
enable
configure terminal
ip access-list extended ACL-VLAN80-OUT
permit udp any eq 68 any eq 67
permit udp 10.32.80.0 0.0.0.255 eq 68 10.32.10.0 0.0.0.255 eq 67
permit udp 10.32.80.0 0.0.0.255 eq 67 10.32.10.0 0.0.0.255 eq 68
permit udp 10.32.80.0 0.0.0.255 10.32.10.0 0.0.0.255 eq 53
permit tcp 10.32.80.0 0.0.0.255 10.32.10.0 0.0.0.255 eq 53
permit tcp 10.32.80.0 0.0.0.255 10.32.10.0 0.0.0.255 eq 80
permit tcp 10.32.80.0 0.0.0.255 10.32.10.0 0.0.0.255 eq 443
permit tcp 10.32.80.0 0.0.0.255 10.32.90.0 0.0.0.255 eq 631
permit tcp 10.32.80.0 0.0.0.255 10.32.90.0 0.0.0.255 eq 515
permit icmp 10.32.80.0 0.0.0.255 10.32.10.0 0.0.0.255 echo-reply
permit icmp 10.32.80.0 0.0.0.255 10.32.20.0 0.0.0.255 echo-reply
permit icmp 10.32.80.0 0.0.0.255 any time-exceeded
permit icmp 10.32.80.0 0.0.0.255 any unreachable
permit icmp 10.32.80.0 0.0.0.255 10.32.90.0 0.0.0.255 echo
deny ip 10.32.80.0 0.0.0.255 10.32.10.0 0.0.0.255 log
deny ip 10.32.80.0 0.0.0.255 10.32.20.0 0.0.0.255 log
deny ip 10.32.80.0 0.0.0.255 10.32.99.0 0.0.0.255 log
deny ip 10.32.80.0 0.0.0.255 10.32.90.0 0.0.0.255 log
deny ip 10.32.80.0 0.0.0.255 10.32.30.0 0.0.0.255 log
deny ip 10.32.80.0 0.0.0.255 10.32.40.0 0.0.0.255 log
deny ip 10.32.80.0 0.0.0.255 10.32.50.0 0.0.0.255 log
deny ip 10.32.80.0 0.0.0.255 10.32.60.0 0.0.0.255 log
deny ip 10.32.80.0 0.0.0.255 10.32.70.0 0.0.0.255 log
permit ip 10.32.80.0 0.0.0.255 any
exit
interface Gi0/4.80
ip access-group ACL-VLAN80-OUT in
exit
end
write memory
```
![](images/Pasted%20image%2020251129230321.png)


### VLAN 90 - Printer (Receive-Only)

```
enable
configure terminal
ip access-list extended ACL-VLAN90-OUT
permit udp any eq 68 any eq 67
permit udp 10.32.90.0 0.0.0.255 eq 68 10.32.10.0 0.0.0.255 eq 67
permit udp 10.32.90.0 0.0.0.255 eq 67 10.32.10.0 0.0.0.255 eq 68
permit icmp 10.32.90.0 0.0.0.255 10.32.10.0 0.0.0.255 echo-reply
permit icmp 10.32.90.0 0.0.0.255 10.32.20.0 0.0.0.255 echo-reply
deny ip 10.32.90.0 0.0.0.255 any log
exit
interface Gi0/4.90
ip access-group ACL-VLAN90-OUT in
exit
end
write memory
```
![](images/Pasted%20image%2020251129230341.png)



### VLAN 99 - Management (Full Access)

```
enable
configure terminal
ip access-list extended ACL-VLAN99-OUT
permit ip any 10.32.99.0 0.0.0.255
exit
interface Gi0/1.99
ip access-group ACL-VLAN99-OUT out
exit
end
write memory
```
![](images/Pasted%20image%2020251129230358.png)

<br>

## **10.7.4 ACL Configuration on R2**

R2 provides default gateways for VLANs 20, 30, and 40.

### VLAN 20 - Admin (Full Access)

```
enable
configure terminal
ip access-list extended ACL-VLAN20-OUT
permit ip any 10.32.20.0 0.0.0.255
permit ip any 10.32.99.0 0.0.0.255
exit
interface Gi0/2.20
ip access-group ACL-VLAN20-OUT out
exit
end
write memory
```
![](images/Pasted%20image%2020251129230441.png)

### VLAN 30 - Executive (Restricted)

```
enable
configure terminal
ip access-list extended ACL-VLAN30-OUT
permit udp any eq 68 any eq 67
permit udp 10.32.30.0 0.0.0.255 eq 68 10.32.10.0 0.0.0.255 eq 67
permit udp 10.32.30.0 0.0.0.255 eq 67 10.32.10.0 0.0.0.255 eq 68
permit udp 10.32.30.0 0.0.0.255 10.32.10.0 0.0.0.255 eq 53
permit tcp 10.32.30.0 0.0.0.255 10.32.10.0 0.0.0.255 eq 53
permit tcp 10.32.30.0 0.0.0.255 10.32.10.0 0.0.0.255 eq 80
permit tcp 10.32.30.0 0.0.0.255 10.32.10.0 0.0.0.255 eq 443
permit tcp 10.32.30.0 0.0.0.255 10.32.90.0 0.0.0.255 eq 631
permit tcp 10.32.30.0 0.0.0.255 10.32.90.0 0.0.0.255 eq 515
permit icmp 10.32.30.0 0.0.0.255 10.32.10.0 0.0.0.255 echo-reply
permit icmp 10.32.30.0 0.0.0.255 10.32.20.0 0.0.0.255 echo-reply
permit icmp 10.32.30.0 0.0.0.255 any time-exceeded
permit icmp 10.32.30.0 0.0.0.255 any unreachable
permit icmp 10.32.30.0 0.0.0.255 10.32.90.0 0.0.0.255 echo
deny ip 10.32.30.0 0.0.0.255 10.32.10.0 0.0.0.255 log
deny ip 10.32.30.0 0.0.0.255 10.32.20.0 0.0.0.255 log
deny ip 10.32.30.0 0.0.0.255 10.32.99.0 0.0.0.255 log
deny ip 10.32.30.0 0.0.0.255 10.32.90.0 0.0.0.255 log
deny ip 10.32.30.0 0.0.0.255 10.32.40.0 0.0.0.255 log
deny ip 10.32.30.0 0.0.0.255 10.32.50.0 0.0.0.255 log
deny ip 10.32.30.0 0.0.0.255 10.32.60.0 0.0.0.255 log
deny ip 10.32.30.0 0.0.0.255 10.32.70.0 0.0.0.255 log
deny ip 10.32.30.0 0.0.0.255 10.32.80.0 0.0.0.255 log
permit ip 10.32.30.0 0.0.0.255 any
exit
interface Gi0/3.30
ip access-group ACL-VLAN30-OUT in
exit
end
write memory
```
![](images/Pasted%20image%2020251129230506.png)


### VLAN 40 - Development (Restricted)

```
enable
configure terminal
ip access-list extended ACL-VLAN40-OUT
permit udp any eq 68 any eq 67
permit udp 10.32.40.0 0.0.0.255 eq 68 10.32.10.0 0.0.0.255 eq 67
permit udp 10.32.40.0 0.0.0.255 eq 67 10.32.10.0 0.0.0.255 eq 68
permit udp 10.32.40.0 0.0.0.255 10.32.10.0 0.0.0.255 eq 53
permit tcp 10.32.40.0 0.0.0.255 10.32.10.0 0.0.0.255 eq 53
permit tcp 10.32.40.0 0.0.0.255 10.32.10.0 0.0.0.255 eq 80
permit tcp 10.32.40.0 0.0.0.255 10.32.10.0 0.0.0.255 eq 443
permit tcp 10.32.40.0 0.0.0.255 10.32.90.0 0.0.0.255 eq 631
permit tcp 10.32.40.0 0.0.0.255 10.32.90.0 0.0.0.255 eq 515
permit icmp 10.32.40.0 0.0.0.255 10.32.10.0 0.0.0.255 echo-reply
permit icmp 10.32.40.0 0.0.0.255 10.32.20.0 0.0.0.255 echo-reply
permit icmp 10.32.40.0 0.0.0.255 any time-exceeded
permit icmp 10.32.40.0 0.0.0.255 any unreachable
permit icmp 10.32.40.0 0.0.0.255 10.32.90.0 0.0.0.255 echo
deny ip 10.32.40.0 0.0.0.255 10.32.10.0 0.0.0.255 log
deny ip 10.32.40.0 0.0.0.255 10.32.20.0 0.0.0.255 log
deny ip 10.32.40.0 0.0.0.255 10.32.99.0 0.0.0.255 log
deny ip 10.32.40.0 0.0.0.255 10.32.90.0 0.0.0.255 log
deny ip 10.32.40.0 0.0.0.255 10.32.30.0 0.0.0.255 log
deny ip 10.32.40.0 0.0.0.255 10.32.50.0 0.0.0.255 log
deny ip 10.32.40.0 0.0.0.255 10.32.60.0 0.0.0.255 log
deny ip 10.32.40.0 0.0.0.255 10.32.70.0 0.0.0.255 log
deny ip 10.32.40.0 0.0.0.255 10.32.80.0 0.0.0.255 log
permit ip 10.32.40.0 0.0.0.255 any
exit
interface Gi0/3.40
ip access-group ACL-VLAN40-OUT in
exit
end
write memory
```
![](images/Pasted%20image%2020251129230530.png)

### **VLAN 99 - Management (Full Access)**


```
enable
configure terminal
ip access-list extended ACL-VLAN99-OUT
permit ip any 10.32.99.0 0.0.0.255
exit
interface Gi0/3.99
ip access-group ACL-VLAN99-OUT out
exit
end
write memory
```
![](images/Pasted%20image%2020251129230547.png)

<br>

## **10.7.5 Results**

All ACLs are applied correctly on R1 and R2 subinterfaces. User VLANs (30, 40, 50, 60, 70, 80) are isolated from each other and can only access specific services on Server VLAN (DNS, HTTP, HTTPS, DHCP) and printing services on Printer VLAN (IPP, LPD). Admin VLAN (20), Server VLAN (10), and Management VLAN (99) maintain full access to all internal networks. ICMP diagnostics work as intended with echo-reply enabled from user VLANs to privileged VLANs. Internet access through NAT/PAT remains functional for all user VLANs. Logging captures unauthorized access attempts for security auditing. The network now operates according to the designed security policy with proper inter-VLAN communication control.


<br>

## **10.8 ACL Communication Testing**

This last section verifies that the ACL policy applied on R1 and R2 works as designed. Each selected device performs communication tests toward other VLANs to confirm allowed traffic (server access, echo‑reply, Internet) and blocked traffic (admin protection, user VLAN isolation). Router diagnostics using show access‑lists confirm correct placement and behavior of all ACL entries.

All tests are based on the addressing plan of this project.

### **VLAN Endpoint Address Table**

|Device|VLAN|Role|IPv4 address|Addressing|
|---|---|---|---|---|
|Xubuntu-Server|10|Server|10.32.10.10|Static|
|Xubuntu-Admin|20|Admin|10.32.20.10|Static|
|PC1-Executive|30|Executive|10.32.30.100|DHCP|
|PC2-Development|40|Development|10.32.40.103|DHCP|
|PC3-Development|40|Development|10.32.40.100|DHCP|
|PC4-Finance|50|Finance|10.32.50.100|DHCP|
|PC5-Logistic|60|Logistic|10.32.60.100|DHCP|
|PC6-Warehouse|70|Warehouse|10.32.70.100|DHCP|
|PC7-Sales|80|Sales|10.32.80.100|DHCP|

### **Testing Steps**

1. Verify ACL diagnostics on R1
    
2. Test echo‑reply behavior from user VLANs toward Server VLAN.
    
3. Test server reachability from selected user VLANs.
    
4. Test user VLAN isolation.
    
5. Test access restrictions toward Admin VLAN and Management VLAN.
    
6. Test Internet connectivity through NAT/PAT.
    

### **Router Diagnostics (R1)**

To confirm correct ACL operation on R1, the following command is used:

```plaintext
show access-lists
```
![](images/Pasted%20image%2020251129230855.png)


>**Note:** The ACL output shown in the diagnostics section is intentionally shortened. Complete ACL listings are extremely long, so only a partial example is included for demonstration purposes.

---

### **Device Testing**

Below are the essential tests performed from selected devices. Each block contains tests only from the specific device.

### **Xubuntu-Admin (VLAN 20)**


#### Tests (Admin → others)

- Ping Server VLAN.
    
- Ping selected User VLAN.
    
- Ping Printer VLAN.
    



```plaintext
ping 10.32.10.10   # server (allow)
ping 10.32.30.100  # pc1 (allow)
ping 10.32.90.100  # printer (allow)
```
![](images/Pasted%20image%2020251129231046.png)

### PC1-Executive (VLAN 30)


#### Tests (PC1 → others)

- Ping Server VLAN (blocked).
    
- Ping Development VLAN (blocked).
    
- Ping Admin VLAN (blocked).
    



```plaintext
ping 10.32.10.10   # server (fail)
ping 10.32.40.100  # other user vlan (fail)
ping 10.32.20.10   # admin (fail)
```
![](images/Pasted%20image%2020251129231150.png)

### PC4-Finance (VLAN 50)


#### Tests

- Ping Server VLAN. (blocked)
    
- Ping Logistic VLAN (blocked).
    


```plaintext
ping 10.32.10.10   # server (fail)
ping 10.32.60.100  # user vlan (fail)
```
![](images/Pasted%20image%2020251129231322.png)


### Printer VLAN Test (VLAN 90)


#### Tests

- Ping Server VLAN. (blocked)
    
- Ping User VLAN (blocked).
    


```plaintext
ping 10.32.10.10   # server (fail)
ping 10.32.30.100  # user (fail)
```
![](images/Pasted%20image%2020251129231459.png)

<br>
## Internet Reachability (NAT/PAT)


### Tests (PC2‑Development)

- Ping to "Internet"

```plaintext
ping 198.51.100.5
```
![](images/Pasted%20image%2020251129231619.png)

<br>

### **Results**

The tests confirm that the ACL policy works as intended.  
User VLANs are isolated, server access works correctly, and admin VLANs retain unrestricted connectivity.  
Blocked traffic is handled as expected, diagnostics respond properly, and NAT/PAT provides working internet reachability.  
Overall, the ACL rules enforce the required security controls across the network.


<br>


## **10.9 Conclusion**

The chapter defines a complete security baseline for the company network. It protects all devices with encrypted access, local authentication, console and VTY security, and a unified warning banner. Port security restricts unauthorized devices on user-facing switch ports. SSH management is enabled and works from the admin workstation.

Access Control Lists enforce the communication policy between VLANs. User VLANs are isolated, can reach only required services on the server VLAN, and can print through the printer VLAN. Admin, Server, and Management VLANs keep full access. ICMP diagnostics stay available for reply traffic. DHCP, DNS, HTTP, and HTTPS are permitted as required. NAT/PAT continues to provide Internet access.

All tests confirm that the ACL rules behave correctly and match the intended design. The network now applies the defined security policy and keeps controlled communication across all VLANs.

---

<br>

**Next part** [Troubleshooting](11-troubleshooting.md)
