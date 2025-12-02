# **8 - NAT PAT Configuration**


<br>

## **8.1 Introduction**  

This chapter defines how the company internal IPv4 networks use Network Address Translation (NAT) and Port Address Translation (PAT) to reach external hosts. R1 functions as the edge router between the private VLAN infrastructure and the ISP network. NAT replaces internal private addresses with the public IPv4 address assigned to R1 on its ISP-facing interface. VLANs 10, 50, 60, 70, 80 and 90 connect directly to R1, while VLANs 20, 30 and 40 reach R1 through R2 and the transit link. All of these networks are translated on R1 when they send traffic towards the Internet Host.


<br>

## **8.2 Topology Diagram**

![](images/Pasted%20image%2020251127024214.png)


<br>

## **8.3 Objectives**

- Define NAT inside and outside interfaces on R1.
    
- Create NAT ACL for internal networks (10.32.0.0/16).
    
- Enable PAT overload on the ISP-facing interface.
    
- Verify connectivity from VLAN 20 and VLAN 70 to the external host.
    
- Check NAT translations and statistics directly on R1.
    
- Confirm that all VLANs use one public IP (203.0.113.5) when accessing external networks.

<br>

## **8.4 NAT Inside and Outside Interfaces on R1**  

This section marks the R1 interfaces that belong to the internal side of the network and the ISP-facing external side. All routed VLAN gateways on R1 and the transit link towards R2 are configured as NAT inside. The interface towards the ISP is configured as NAT outside.

#### Commands

```plaintext
enable
configure terminal
interface Gi0/1.10
ip nat inside
exit
interface Gi0/4.50
ip nat inside
exit
interface Gi0/4.60
ip nat inside
exit
interface Gi0/4.70
ip nat inside
exit
interface Gi0/4.80
ip nat inside
exit
interface Gi0/4.90
ip nat inside
exit
interface Gi0/1.99
ip nat inside
exit
interface Gi0/0
ip nat inside
exit
interface Gi0/5
ip nat outside
end
write memory
```
![](images/Pasted%20image%2020251126000816.png)

### **Results**  

R1 treats all internal VLAN gateways and the transit link towards R2 as NAT inside interfaces. Traffic from any internal VLAN arrives on an inside interface and leaves the router through the outside interface Gi0/5 when it is sent to external networks.

<br>

## **8.5 NAT Access List on R1**  

This section creates an access list that defines which internal IPv4 networks are allowed to be translated. The list includes the complete 10.32.0.0/16 range that covers all internal VLANs in the company network.

#### Commands

```plaintext
access-list 1 permit 10.32.0.0 0.0.255.255
```
![](images/Pasted%20image%2020251126000907.png)

### **Results**  

R1 permits address translation for all internal IPv4 networks in the 10.32.0.0/16 range. Any packet that matches this access list and enters the router on an inside interface is eligible for NAT and PAT.

>**Notes:** The **/16 value does not define VLAN subnets.** All VLANs use **/24**, but NAT needs one **summary range** for the entire internal network.**10.32.0.0/16** simply means _“all company addresses starting with 10.32”_. Wildcard **0.0.255.255** matches every internal VLAN without listing them individually.



<br>

## **8.6 PAT Overload on R1**  

This section enables PAT overload on R1. All internal connections share the public IPv4 address that is configured on the ISP-facing interface Gi0/5. Different internal sessions are separated by unique TCP and UDP port numbers.

#### Commands

```plaintext
ip nat inside source list 1 interface Gi0/5 overload
```
![](images/Pasted%20image%2020251126001309.png)

### **Results**  

All internal clients use the public IPv4 address on R1 interface Gi0/5 when they contact external hosts. The router creates PAT entries for each active session so that return traffic from the ISP can be mapped back to the correct internal hosts.

<br>

## **8.7 Verification**  

This section verifies that internal hosts can reach the Internet Host and that R1 creates active NAT translations. Tests are performed from Xubuntu-Admin, from a VPCS client in the Warehouse VLAN and directly on R1.

<br>

### **8.7.1 Connectivity Test from Xubuntu-Admin (VLAN 20)**  

Xubuntu-Admin is placed in VLAN 20 and uses R2 as its default gateway. Traffic is routed from R2 to R1 over the transit link and is translated on R1 before it reaches the Internet Host.

#### Commands

```plaintext
ping 198.51.100.6
```
![](images/Pasted%20image%2020251126004125.png)

### Results  

The ping from Xubuntu-Admin to 198.51.100.6 is successful. This confirms that traffic from VLAN 20 passes through R2, is translated on R1 and reaches the external host.

<br>

### **8.7.2 Connectivity Test from PC6-Warehouse (VLAN 70)**  

PC6-Warehouse is located in VLAN 70 and receives its IPv4 address dynamically from the DHCP server. It uses R1 as its default gateway. Traffic goes directly from R1 to the ISP and is translated on the edge router.

#### Commands

```plaintext
ping 198.51.100.6
```
![](images/Pasted%20image%2020251126004147.png)

### **Results**  

The ping from PC6-Warehouse to 198.51.100.6 is successful. This confirms that DHCP clients in the user VLANs can reach the external host and that NAT and PAT operate correctly for traffic that uses R1 as the default gateway.

<br>

## **8.7.3 NAT Verification on R1**  

This part verifies the NAT operation directly on R1. The router displays the active translation table and NAT statistics.

#### Commands

```plaintext
show ip nat translations
show ip nat statistics
```
![](images/Pasted%20image%2020251126004331.png)

### **Results**  

Internal VLAN addresses are successfully translated to the public IP 203.0.113.5. The translation table shows inside local addresses changing to one public inside global address. R1 creates new dynamic PAT entries when devices send traffic to the Internet. Both PC6 and Xubuntu-Admin can reach the ISP and the Internet Host through NAT/PAT.

<br>

## **8.8 Final NAT/PAT Address Translation Overview**  

This section summarizes how internal devices are represented on the external network through PAT. It shows the relationship between private internal addresses, their public translated form and the external host that is accessed by internal VLAN clients.

| **Term**           | **Example Value** | **Meaning**                                  | **Explanation**                                                                                                                                                     |
| -------------- | ------------- | ---------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Inside Local   | 10.32.70.100  | Internal private address                 | The actual IPv4 address of a device inside the company network. It exists only within the internal LAN and is never visible on the Internet.                    |
| Inside Global  | 203.0.113.5   | Public address for inside                | The public IPv4 address assigned to R1 on interface Gi0/5. All internal devices appear on the external network under this address when PAT overload is active.  |
| Outside Local  | 198.51.100.6  | External host as seen from inside        | The IPv4 address of the Internet Host as it appears from the company network. In this design it is identical to the outside global value.                       |
| Outside Global | 198.51.100.6  | Real public address of the external host | The actual public IPv4 address of the Internet Host on the Internet. It is globally reachable and represents the external server that internal clients contact. |

<br>

## **8.9 Conclusion** 

This chapter explains how the internal network uses NAT and PAT to reach external resources. R1 translates all private addresses to one public identity and provides full outbound connectivity for every VLAN. Routing between R1 and R2 ensures that all internal networks reach the edge router. Verification confirms that clients can communicate with the external network and that dynamic translations are created correctly. The next chapter focuses on analysing this behaviour in Wireshark.

---

<br>

**Next chapter:** [Wireshark Monitoring](09-wireshark-monitoring.md)
