# **9 Wireshark NAT/PAT and Packet Analysis**

<br>

## **9.1 Introduction**

This chapter provides a practical packet-level view of how the network behaves across internal and external paths. The goal is to observe key protocols directly in Wireshark and confirm that each part of the network operates as designed. The analysis includes internal ICMP and ARP communication, local DNS and HTTP traffic, the TCP handshake, and a comparison of packet flow before and after NAT/PAT. The chapter also includes two OSPF captures to verify routing protocol behavior on both the routed link and the VLAN 99 transit link.

<br>

## **9.2 Topology Diagram**

![](images/Pasted%20image%2020251127023510.png)


<br>

## **9.3 Objectives** 

- Capture and analyse internal client traffic (ICMP, ARP, DNS, TCP, HTTP).
    
- Observe NAT/PAT translation by capturing packets on the R1–ISP link.
    
- Compare how packets look before translation on the client interface and after translation on the WAN interface.
    
- Verify OSPF Hello packet exchanges on the R1–R2 routed link.
    
- Confirm that VLAN 99 correctly forwards OSPF packets across SW1–SW2 as a Layer 2 transit path.


<br>

## **9.4 Installing Wireshark on Xubuntu-admin**

Wireshark is installed on Xubuntu-Admin to allow packet inspection on the active Ethernet interface.

### Commands

```
sudo apt update
sudo apt install wireshark
sudo usermod -aG wireshark xubuntu-admin
sudo systemctl restart NetworkManager
wireshark
```
![](images/Pasted%20image%2020251126235420.png)

### Wireshark Main Interface View (No Filter)

![](images/Pasted%20image%2020251127004335.png)
<br>

## **9.5 ICMP Echo Request to Internet Host (Before NAT)**

This section captures the moment when Xubuntu-Admin sends an ICMP Echo Request toward an external destination. At this stage, the packet still carries its original private source address because NAT has not yet been applied. This allows us to clearly observe the internal identity of the client before translation and understand how the packet looks when leaving the VLAN.

### Commands

```
ping 198.51.100.6
```
![](images/Pasted%20image%2020251127003404.png)

### Wireshark – ICMP Echo Request Header Filter

```
icmp.type == 8
```
![](images/Pasted%20image%2020251127003447.png)

### **Results**

The capture shows a complete ICMP exchange between Xubuntu-Admin and the Internet Host (198.51.100.6). The Echo Request packets leave the client with the private source address **10.32.20.10**, and the replies return from the external host through NAT/PAT. The sequence numbers and timestamps confirm that every outbound request has a correct matching reply, demonstrating successful end-to-end connectivity through R1.

<br>

## **9.6 ICMP Echo Request to Internal Server**

This part focuses on ICMP communication that remains fully inside the internal network. Since the traffic never leaves the LAN, NAT is not involved. Capturing this packet makes it easy to compare internal traffic with traffic that later goes through R1 for translation, highlighting the difference between internal and external ICMP flows.

### Commands

```
ping 10.32.10.10
```
![](images/Pasted%20image%2020251127003759.png)

### Wireshark – ICMP Internal Communication Filter

```
icmp
```
![](images/Pasted%20image%2020251127003832.png)

The capture shows a standard internal ICMP exchange. The source and destination addresses remain within the internal network, so no NAT translation occurs. Each Echo Request is followed by the correct Echo Reply, with matching IDs and sequence numbers. This confirms that internal ICMP communication between Xubuntu-Admin and the internal server is functioning normally and stays entirely inside the LAN.

<br>
## **9.7 TCP Handshake and HTTP Request Analysis**



This part focuses on analysing a basic TCP connection between Xubuntu-Admin and the internal Xubuntu-Server. The objective is to observe how a client initiates a TCP session and how the server responds. TCP uses a three-step process called the **Three-Way Handshake**, which establishes a reliable communication channel before any application data is exchanged. Capturing this traffic allows us to see the SYN, SYN/ACK and ACK packets and understand how a TCP session begins.

##### Commands

 ```
ping 10.32.10.10  
http://10.32.10.10
 ```
![](images/Pasted%20image%2020251127004758.png)

#### Wireshark – TCP Handshake Filter

 ```
 tcp.flags.syn == 1
 ```
![](images/Pasted%20image%2020251127005112.png)

### **Results**  

The capture clearly shows the 3-way TCP handshake between Xubuntu-Admin and the internal server. We can see the initial **SYN**, the server’s **SYN/ACK**, and the client’s final **ACK**, which confirms that the TCP session is successfully established. This demonstrates normal TCP behaviour inside the internal LAN without NAT involvement.


#### Wireshark – HTTP Request Filter

 ```
 tcp.port == 80
 ```
![](images/Pasted%20image%2020251127005249.png)

### **Results** 

The capture shows a complete HTTP communication flow between Xubuntu-Admin and the internal web server. You can see the full TCP three-way handshake (**SYN → SYN/ACK → ACK**), followed by the client’s **HTTP GET** request and the server’s **HTTP/1.1 200 OK** response containing the webpage data. This confirms that the web server is running correctly and that internal TCP/HTTP communication works as expected.



<br>

## **9.8 DNS Query and Response (company.local)**

This section examines DNS communication generated when Xubuntu-Admin accesses the internal domain company.local. The DNS query leaves the client with its private address, and the response returns with the resolved IP. Capturing this traffic shows how name resolution works in a NAT environment and how the query appears before any external translation.

### Commands

```
firefox company.local
```
![](images/Pasted%20image%2020251127012608.png)


### Wireshark – DNS Query/Response Filter – DNS Query/Response Filter

 ```
 dns.flags.rcode == 0
 ```
![](images/Pasted%20image%2020251127014227.png)

### **Results**  

The DNS capture shows a successful query from Xubuntu-Admin (10.32.10.10) to the internal DNS server (10.32.20.10) for `www.company.local`. The first highlighted response returns an A record: `www.company.local` is a CNAME for `xubuntu-server.company.local`, which resolves to `10.32.10.10`. The second highlighted response returns the AAAA record for the same name, also pointing via the CNAME to `xubuntu-server.company.local`. Both responses have `rcode = 0`, so the name is resolved correctly and the browser can load the web page from the Xubuntu server.

<br>

## **9.9 ARP Resolution on the Local Network**

This part focuses on ARP activity inside the local broadcast domain. ARP is responsible for discovering MAC addresses before any IP-based communication can occur. Capturing ARP frames provides a clear view of how the client identifies its default gateway or another local host and prepares the path for Layer 3 communication.

### Commands

```
ping 10.32.10.10
```
![](images/Pasted%20image%2020251127015201.png)

### Wireshark – ARP Packet Filter

```
arp
```
![](images/Pasted%20image%2020251127015114.png)

### **Results**

Xubuntu-Admin sends an ARP request to discover the MAC address of its default gateway (10.32.20.1). The server in VLAN 10 is located in a different Layer 2 segment, so its MAC address never appears in the ARP table. The gateway replies with its MAC address, and the communication continues as routed traffic. The ARP capture correctly shows only gateway-related ARP frames, confirming expected Layer 2 and Layer 3 behavior.

<br>

## **9.10 NAT/PAT External Translation View**

This section demonstrates how outbound traffic is translated by NAT/PAT on **R1** once it leaves the internal network.  
The ping command is executed **from Xubuntu-Admin**, but the packet capture is performed **directly on the link between R1 and the ISP router**, because NAT translation happens _after_ the traffic leaves the internal LAN.

Traffic captured on Xubuntu-Admin still shows the private source IP (**10.32.20.10**).  
To observe the translated packets using the public address **203.0.113.5**, the capture must be started on the WAN interface.

Therefore, **Start Capture** is activated on the GNS3 link:

**R1 Gi0/5 ↔ ISP Gi0/5**

This provides the correct external NAT/PAT view.

## **Commands**
 ```
 ping 198.51.100.6
 ```
![](images/Pasted%20image%2020251127020854.png)

Executed from **Xubuntu-Admin**.

## **Wireshark – External NAT Translation Filter**
 ```
 ip.src == 203.0.113.5
 ```
![](images/Pasted%20image%2020251127020939.png)

This filter displays only packets that have already been translated by NAT on R1.

## **Results**

The capture on the R1–ISP link shows ICMP Echo Requests with their source rewritten by NAT:
 ```
 203.0.113.5 → 198.51.100.6   ICMP Echo (ping)
 ```
![](images/Pasted%20image%2020251127020953.png)

This confirms that:

- The internal host (10.32.20.10) sends the ping normally
    
- R1 performs PAT on its WAN interface
    
- The packet leaving toward the ISP uses the public IP 203.0.113.5
    

These translated packets **cannot be seen** when capturing on Xubuntu-Admin, because NAT occurs only when the packet exits R1’s WAN interface.
Capturing on the link between R1 and the ISP provides the accurate external representation of the translated traffic.

<br>

## **9.11 OSPF Traffic Capture Summary**


This part combines two separate OSPF packet captures taken in GNS3. The goal is to compare OSPF behavior on two different links:

- The routed link between **R1 and R2**
    
- The switched link between **SW1 and SW2**, which operates through **VLAN 99** (the management and OSPF transit VLAN)
    

![](images/Pasted%20image%2020251127022306.png)


Both captures help illustrate how OSPF exchanges Hello packets across router-to-router and switch-to-switch paths.

### **Capture 1: R1 ↔ R2 (Direct Routed Link)**

**Display Filter:** 

 ```
 ospf
 ```
![](images/Pasted%20image%2020251127022158.png)

On the routed link between R1 (10.32.0.1) and R2 (10.32.0.2), OSPF Hello packets are exchanged at regular intervals. The communication uses the standard multicast destination **224.0.0.5**, which is used by OSPF for sending Hello packets to all OSPF routers.

**Key observations:**

- Packets originate from **10.32.0.1** and **10.32.0.2**.
    
- Protocol is **OSPF**, specifically **Hello Packet**.
    
- Destination is always **224.0.0.5**.
    
- This confirms stable adjacency formation between R1 and R2.
    

This capture shows normal OSPF behavior on a routed point-to-point segment.

### **Capture 2: SW1 ↔ SW2 (VLAN 99 Transit Link)**

**Display Filter:** 

 ```
 ospf
 ```
![](images/Pasted%20image%2020251127022233.png)


On the link between SW1 and SW2, the switches forward OSPF packets from the routers because this trunk carries **VLAN 99**, which serves as the Layer 2 transit segment for OSPF.

**In the capture:**

- Packets originate from **10.32.10.1** and **10.32.99.2** depending on which router the packet belongs to.
    
- The destination is again **224.0.0.5**.
    
- SW1 and SW2 do not run OSPF themselves, but since the trunk passes VLAN 99, OSPF packets traverse this link.
    

This confirms that VLAN 99 is functioning as the correct transit VLAN for OSPF communication. The backup path carries OSPF traffic as expected.

## **Summary**

Across both captures, OSPF behaves consistently using Hello packets to maintain neighbor relationships.

- On the **R1–R2 routed link**, we see direct router-internal communication.
    
- On the **SW1–SW2 VLAN 99 link**, we see forwarded OSPF traffic moving through the Layer 2 transit network.
    

These captures verify correct OSPF operation and confirm that both the primary and backup paths forward routing protocol traffic as designed.


<br>

## **9.12 Conclusion**

The Wireshark captures confirm that every major network function operates correctly. Internal communication behaves as expected, including ICMP, DNS, ARP, TCP, and HTTP flows. Traffic leaving the network is translated by NAT/PAT on R1, which is visible only when capturing on the WAN interface. The comparison between internal and external views clearly shows how packet headers change during translation.

Both OSPF captures demonstrate stable neighbor relationships on the routed link and on the VLAN 99 transit link, proving that the primary and backup paths forward routing protocol traffic correctly. Overall, the captures provide a complete packet-level validation of the network design.

---

<br>

**Next part:** [Network Security](10-network-security.md)
