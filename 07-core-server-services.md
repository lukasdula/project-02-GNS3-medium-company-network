
# **7 - Core Server Services**

<br>

## **7.1 Introduction**

This chapter defines the core server services that support the internal network. The Xubuntu Server in VLAN 10 provides DHCP for dynamic hosts, Apache for internal web access, and Bind9 for local DNS resolution. These services operate together across all routed VLANs and ensure that clients receive configuration automatically and can reach internal resources using hostnames rather than IP addresses.

<br>

## **7.2 Topology Diagram**

![](images/Pasted%20image%2020251125221236.png)

<br>

## **7.3 Objectives**

1. Configure DHCP on the Xubuntu Server for all dynamic VLANs.
    
2. Deploy a basic Apache HTTP intranet page.
    
3. Configure Bind9 for internal DNS resolution.
    
4. Verify service reachability from multiple VLANs.
    
5. Ensure that all services start automatically and integrate with routing.

<br>

## **7.4 DHCP Service**

This section defines the DHCP infrastructure used in the network. The Xubuntu Server in VLAN 10 distributes IPv4 addresses to all dynamic VLANs. The server and admin systems keep static IP addresses. All VPCS clients must be set to **ip dhcp** to receive their configuration.

<br>

### **7.4.1 DHCP Scope Design**

The DHCP service assigns addresses to dynamic VLANs. Each VLAN uses gateway _.1_, DNS server _10.32.10.10_, and a dynamic range from _.100_ to _.199_.

### DHCP Scope Table

| **VLAN**           | **Subnet**        | **Gateway**    | **DHCP Range**                  | **Notes**         |
| -------------- | ------------- | ---------- | --------------------------- | ------------- |
| 30 Executive   | 10.32.30.0/24 | 10.32.30.1 | 10.32.30.100 – 10.32.30.199 | Dynamic hosts |
| 40 Development | 10.32.40.0/24 | 10.32.40.1 | 10.32.40.100 – 10.32.40.199 | Dynamic hosts |
| 50 Finance     | 10.32.50.0/24 | 10.32.50.1 | 10.32.50.100 – 10.32.50.199 | Dynamic hosts |
| 60 Logistic    | 10.32.60.0/24 | 10.32.60.1 | 10.32.60.100 – 10.32.60.199 | Dynamic hosts |
| 70 Warehouse   | 10.32.70.0/24 | 10.32.70.1 | 10.32.70.100 – 10.32.70.199 | Dynamic hosts |
| 80 Sales       | 10.32.80.0/24 | 10.32.80.1 | 10.32.80.100 – 10.32.80.199 | Dynamic hosts |
| 90 Printer     | 10.32.90.0/24 | 10.32.90.1 | 10.32.90.100 – 10.32.90.199 | Dynamic hosts |



<br>

### **7.4.2 DHCP Server Installation**

The DHCP service installs and enables on Xubuntu Server.

#### Commands

```plaintext
sudo apt update
sudo apt install isc-dhcp-server -y
```
![](images/Pasted%20image%2020251125140632.png)

#### Configuration

Set the interface line:

```plaintext
sudo nano /etc/default/isc-dhcp-server
INTERFACESv4="ens3"
```
![](images/Pasted%20image%2020251125141200.png)

#### Enable the service

```plaintext
sudo systemctl enable isc-dhcp-server
sudo systemctl restart isc-dhcp-server
sudo systemctl status isc-dhcp-server
```
![](images/Pasted%20image%2020251125141648.png)

The service is enabled but cannot start until all required subnets are defined.


<br>

### **Default route on DHCP server**

The DHCP server needs a default route to reach client networks in other VLANs. The default route sends all traffic to the router gateway in VLAN 10.

```plaintext
sudo ip route add default via 10.32.10.1
```
![](images/Pasted%20image%2020251125141817.png)

### **Results**

The DHCP daemon starts and listens on the server interface. The service runs without errors.

<br>

### **7.4.3 DHCP Pool Configuration**

The DHCP pools define dynamic addressing for each user VLAN.

#### Commands

```plaintext
sudo nano /etc/dhcp/dhcpd.conf
```


#### Configuration



```plaintext
authoritative;
default-lease-time 3600;
max-lease-time 86400;
option domain-name "company.local";
option domain-name-servers 10.32.10.10;

# VLAN 10 - Server subnet (no DHCP pool)
subnet 10.32.10.0 netmask 255.255.255.0 {
}

# VLAN 30 - Executive
subnet 10.32.30.0 netmask 255.255.255.0 {
  option routers 10.32.30.1;
  range 10.32.30.100 10.32.30.199;
}

# VLAN 40 - Development
subnet 10.32.40.0 netmask 255.255.255.0 {
  option routers 10.32.40.1;
  range 10.32.40.100 10.32.40.199;
}

# VLAN 50 - Finance
subnet 10.32.50.0 netmask 255.255.255.0 {
  option routers 10.32.50.1;
  range 10.32.50.100 10.32.50.199;
}

# VLAN 60 - Logistic
subnet 10.32.60.0 netmask 255.255.255.0 {
  option routers 10.32.60.1;
  range 10.32.60.100 10.32.60.199;
}

# VLAN 70 - Warehouse
subnet 10.32.70.0 netmask 255.255.255.0 {
  option routers 10.32.70.1;
  range 10.32.70.100 10.32.70.199;
}

# VLAN 80 - Sales
subnet 10.32.80.0 netmask 255.255.255.0 {
  option routers 10.32.80.1;
  range 10.32.80.100 10.32.80.199;
}

# VLAN 90 - Printer
subnet 10.32.90.0 netmask 255.255.255.0 {
  option routers 10.32.90.1;
  range 10.32.90.100 10.32.90.199;
}
```
![](images/Pasted%20image%2020251125143929.png)
![](images/Pasted%20image%2020251125143947.png)


Restart the service:

```
sudo systemctl restart isc-dhcp-server
sudo systemctl status isc-dhcp-server
```
![](images/Pasted%20image%2020251125144101.png)

### Results

The DHCP service is enabled and running. The server loads the configuration without errors and listens on interface _ens3_ for all DHCP requests.

<br>

### **7.4.4 IP Helper Configuration on Routers**

Routers forward DHCP broadcasts to the server using helper addresses. All dynamic VLANs point to the Xubuntu Server at 10.32.10.10 in VLAN 10.

### R2 Configuration

```plaintext
enable
configure terminal
interface Gi0/3.30
ip helper-address 10.32.10.10
exit
interface Gi0/3.40
ip helper-address 10.32.10.10
end
write memory
```
![](images/Pasted%20image%2020251125144458.png)

### R1 Configuration

```plaintext
enable
configure terminal
interface Gi0/4.50
ip helper-address 10.32.10.10
exit
interface Gi0/4.60
ip helper-address 10.32.10.10
exit
interface Gi0/4.70
ip helper-address 10.32.10.10
exit
interface Gi0/4.80
ip helper-address 10.32.10.10
exit
interface Gi0/4.90
ip helper-address 10.32.10.10
end
write memory
```
![](images/Pasted%20image%2020251125144420.png)

### **Verification – DHCP leases on VPCS clients**

For example:

PC3 (Development)

 ```
 ip dhcp
 show ip
 ``` 
![](images/Pasted%20image%2020251125145349.png)

PC4 (Finance)

 ```
 ip dhcp
 show ip
 ``` 
![](images/Pasted%20image%2020251125145359.png)


### **Results**

All dynamic VLANs send DHCP requests to the Xubuntu Server. Clients in VLANs 30–80 and the printer VLAN 90 receive valid IPv4 leases with the correct gateway and DNS settings.

<br>

## **7.5 HTTP Service (Apache2)**

This section configures the Apache2 web server on the Xubuntu Server in VLAN 10. The goal is to provide a simple internal web page that is reachable from all routed VLANs using IPv4.

<br>

### **Instal apache web server**

```
sudo apt update
sudo apt install apache2 -y
```
![](images/Pasted%20image%2020251125150350.png)

<br>

### **7.5.1 Service Status and Local Test**

The Apache2 service runs on the Xubuntu Server and listens on TCP port 80.

##### Commands

```plaintext
sudo systemctl status apache2 --no-pager -l
sudo systemctl is-enabled apache2
```
![](images/Pasted%20image%2020251125150742.png)

### **Results**

Apache2 is installed, enabled and running on the server.
### **Quick local test**

```plaintext
http://10.32.10.10
```
![](images/Pasted%20image%2020251125151136.png)

### **Results**

The default Apache page loads locally. The server responds to HTTP requests on its IP address.

<br>

### **7.5.2 Replace the Default Page**

The default Apache page is replaced with a simple custom HTML file. This makes it easy to confirm that clients reach the correct server.

##### Commands

```plaintext
sudo nano /var/www/html/index.html
```
![](images/Pasted%20image%2020251125161548.png)

### Configuration

```plaintext
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8">
    <title>GNS3 Project - Web Server</title>
    <style>
      body {
        background-color: #ccffcc;   /* světle zelená */
        font-family: Arial, sans-serif;
        margin: 40px;
      }
      h1 {
        color: #003300;
      }
      p {
        font-size: 18px;
      }
    </style>
  </head>

  <body>
    <h1>GNS3 Project – Medium Company Network</h1>
    <p>This page is served from the Xubuntu Server in VLAN 10 (10.32.10.10).</p>
  </body>
</html>

```
![](images/Pasted%20image%2020251125162812.png)


Check file permissions and reload Apache:

```plaintext
ls -l /var/www/html/index.html
sudo systemctl reload apache2
```
![](images/Pasted%20image%2020251125163005.png)

### **Results**

The custom intranet page replaces the default Apache page and is ready for client testing.

<br>

### **7.5.3 Client Tests (curl)**

Clients from different VLANs verify IP connectivity and HTTP access to the server.

##### Example tests from Xubuntu-Admin

```plaintext
http://10.32.10.10
```
![](images/Pasted%20image%2020251125163107.png)


##### Example tests from Admin-PC

```plaintext
ping 10.32.10.10
http://10.32.10.10
```
![](images/Pasted%20image%2020251125164022.png)


### **Results**

The custom intranet page loads from multiple VLANs. Ping and curl confirm IP reachability and HTTP service availability.



<br>

## **7.6 DNS Service (Bind9)**

This section configures the Bind9 DNS service on the Xubuntu Server in VLAN 10. The server provides internal name resolution for the company domain and allows clients to reach resources using hostnames instead of IP addresses.

<br>

### **7.6.1 DNS Role and Overview**

The DNS server hosts an internal zone for the company network. It resolves names for the Xubuntu Server, Xubuntu-Admin, routers and the intranet web page.

Main goals:

- Provide a forward lookup zone for the domain `company.local`.
    
- Provide a reverse lookup zone for selected server addresses.
    
- Allow clients in all VLANs to use 10.32.10.10 as their DNS server.
    

<br>

### **7.6.2 Bind9 Installation and Service Check**

Bind9 runs on the Xubuntu Server and listens on UDP/TCP port 53.

##### Commands

```plaintext
sudo apt update
sudo apt install bind9 bind9utils -y
sudo systemctl enable bind9
sudo systemctl status bind9 --no-pager -l
```
![](images/Pasted%20image%2020251125170750.png)

### **Results**

Bind9 installs, enables and runs on the server.

<br>

### **7.6.3 Forward Zone Configuration**

The forward zone maps hostnames to IPv4 addresses inside the domain `company.local`.

##### Commands

```plaintext
sudo nano /etc/bind/named.conf.local
```


### Configuration

Add the forward zone definition:

```plaintext
zone "company.local" {
  type master;
  file "/etc/bind/db.company.local";
};
```
![](images/Pasted%20image%2020251125171207.png)

### **Verification**

```plaintext
sudo systemctl status bind9 --no-pager -l
```
![](images/Pasted%20image%2020251125171319.png)


Create the zone file based on the default template:

Copy the default template:

```plaintext
sudo cp /etc/bind/db.local /etc/bind/db.company.local
```


Open the new zone file:

```plaintext
sudo nano /etc/bind/db.company.local
```


Example zone content:

```plaintext
$TTL    86400
@   IN  SOA ns1.company.local. admin.company.local. (
            1       ; Serial
            3600    ; Refresh
            1800    ; Retry
            604800  ; Expire
            86400 ) ; Minimum

    IN  NS  ns1.company.local.

ns1             IN  A   10.32.10.10
xubuntu-server  IN  A   10.32.10.10
xubuntu-admin   IN  A   10.32.20.10
r1              IN  A   10.32.0.1
r2              IN  A   10.32.0.2
intranet        IN  A   10.32.10.10
@               IN  A   10.32.10.10
www             IN  CNAME xubuntu-server
```
![](images/Pasted%20image%2020251125212708.png)

### **Results**

The forward zone defines hostnames for core devices and the intranet page.

<br>

### **7.6.4 Reverse Zone Configuration**

The reverse zone maps IPv4 addresses back to hostnames. This step is optional but helps with diagnostics.

##### Commands

```plaintext
sudo nano /etc/bind/named.conf.local
```


### Configuration

Add the reverse zone definition under the forward zone:

```plaintext
zone "company.local" {
  type master;
  file "/etc/bind/db.company.local";
};

zone "10.32.10.in-addr.arpa" { 
   type master;
   file "/etc/bind/db.10.32.10";
}   
```
![](images/Pasted%20image%2020251125211714.png)

Create the reverse zone file:

```plaintext
sudo cp /etc/bind/db.127 /etc/bind/db.10.32.10
sudo nano /etc/bind/db.10.32.10
```


Example content:

```plaintext
$TTL    86400
@   IN  SOA ns1.company.local. admin.company.local. (
            1       ; Serial
            3600    ; Refresh
            1800    ; Retry
            604800  ; Expire
            86400 ) ; Minimum

    IN  NS  ns1.company.local.

10  IN PTR xubuntu-server.company.local.
```
![](images/Pasted%20image%2020251125174406.png)

### **Results**

The reverse zone allows basic PTR lookups for the server address.

<br>

### **7.6.5 Configuration Check and Service Restart**

Bind9 configuration is checked before restart to avoid syntax errors.

##### Commands

```plaintext
sudo named-checkconf
sudo named-checkzone company.local /etc/bind/db.company.local
sudo named-checkzone 10.32.10.in-addr.arpa /etc/bind/db.10.32.10
sudo systemctl restart bind9
sudo systemctl status bind9 --no-pager -l
```
![](images/Pasted%20image%2020251125175231.png)

### **Results**

The configuration loads without errors and Bind9 restarts successfully.

<br>

### **7.6.6 Client DNS Tests**

Clients in different VLANs use the DNS server at 10.32.10.10. For VPCS, the DNS address comes from DHCP. For Xubuntu-Admin, the DNS server is set in the network settings.

### **Example tests from Xubuntu-Admin**

```plaintext
ping intranet.company.local
ping xubuntu-server.company.local
www.company.local
```
![](images/Pasted%20image%2020251125213157.png)

```
www.company.local
```
![](images/Pasted%20image%2020251125213118.png)


### **Example tests from Xubuntu Admin and VPCS client**

Admin-xubuntu

```plaintext
ping intranet.company.local
wwww.company.local
```
![](images/Pasted%20image%2020251125214116.png)


```plaintext
ping intranet.company.local
```
![](images/Pasted%20image%2020251125214452.png)


### **Results**

DNS resolving works correctly on the Xubuntu-Server and Xubuntu-Admin machines. Name resolution does **not work on VPCS**, because VPCS has a very limited built-in DNS client. This is a limitation of VPCS itself, not a configuration issue in the network.

<br>

## **7.8 VLAN Connectivity Diagnostics**

This part verifies end-to-end connectivity between all internal VLANs after enabling DHCP, routing and core server services. Each endpoint sends ICMP echo requests to three devices in different VLANs. These tests confirm that VLAN segmentation, router-on-a-stick subinterfaces and DHCP relay operate correctly across the whole topology.

### **VLAN Endpoint Address Table**

| **Device**          | **VLAN** | **Role**        | **IPv4 address** | **Addressing** |
| --------------- | ---- | ----------- | ------------ | ---------- |
| Xubuntu-Server  | 10   | Server      | 10.32.10.10  | Static     |
| Xubuntu-Admin   | 20   | Admin       | 10.32.20.10  | Static     |
| PC1-Executive   | 30   | Executive   | 10.32.30.100 | DHCP       |
| PC2-Development | 40   | Development | 10.32.40.103 | DHCP       |
| PC3-Development | 40   | Development | 10.32.40.100 | DHCP       |
| PC4-Finance     | 50   | Finance     | 10.32.50.100 | DHCP       |
| PC5-Logistic    | 60   | Logistic    | 10.32.60.100 | DHCP       |
| PC6-Warehouse   | 70   | Warehouse   | 10.32.70.100 | DHCP       |
| PC7-Sales       | 80   | Sales       | 10.32.80.100 | DHCP       |

#### **Xubuntu-Server tests**

- Xubuntu-Admin (VLAN 20)
    
- PC1-Executive (VLAN 30)
    
- PC7-Sales (VLAN 80)
    

```plaintext
ping 10.32.20.10      
ping 10.32.30.100     
ping 10.32.80.100     
```
![](images/Pasted%20image%2020251125225751.png)


#### **Xubuntu-Admin tests**

- Xubuntu-Server (VLAN 10)
    
- PC3-Development (VLAN 40)
    
- PC6-Warehouse (VLAN 70)
    

```plaintext
ping 10.32.10.10      
ping 10.32.40.100     
ping 10.32.70.100     
```
![](images/Pasted%20image%2020251125225902.png)


#### **PC1-Executive tests**

- PC4-Finance (VLAN 50)
    
- PC5-Logistic (VLAN 60)
    
- Xubuntu-Admin (VLAN 20)
    

```plaintext
ping 10.32.50.100     
ping 10.32.60.100     
ping 10.32.20.10     
```
![](images/Pasted%20image%2020251125225953.png)


#### **PC2-Development tests**

- Xubuntu-Server (VLAN 10)
    
- PC4-Finance (VLAN 50)
    
- PC7-Sales (VLAN 80)
    

```plaintext
ping 10.32.10.10      
ping 10.32.50.100     
ping 10.32.80.100     
```
![](images/Pasted%20image%2020251125230038.png)


#### **PC3-Development tests**

- Xubuntu-Server (VLAN 10)
    
- PC6-Warehouse (VLAN 70)
    
- PC7-Sales (VLAN 80)
    

```plaintext
ping 10.32.10.10      
ping 10.32.70.100     
ping 10.32.80.100     
```
![](images/Pasted%20image%2020251125230114.png)


#### **PC4-Finance tests**

- Xubuntu-Server (VLAN 10)
    
- PC1-Executive (VLAN 30)
    
- PC5-Logistic (VLAN 60)
    

```plaintext
ping 10.32.10.10      
ping 10.32.30.100     
ping 10.32.60.100     
```
![](images/Pasted%20image%2020251125230153.png)


#### **PC5-Logistic tests**

- Xubuntu-Server (VLAN 10)
    
- Xubuntu-Admin (VLAN 20)
    
- PC6-Warehouse (VLAN 70)
    

```plaintext
ping 10.32.10.10      
ping 10.32.20.10      
ping 10.32.70.100     
```
![](images/Pasted%20image%2020251125230229.png)

#### **PC6-Warehouse tests**

- Xubuntu-Server (VLAN 10)
    
- PC2-Development (VLAN 40)
    
- PC7-Sales (VLAN 80)
    

```plaintext
ping 10.32.10.10      
ping 10.32.40.103     
ping 10.32.80.100     
```
![](images/Pasted%20image%2020251125230326.png)

#### **PC7-Sales tests**

- Xubuntu-Server (VLAN 10)
    
- PC1-Executive (VLAN 30)
    
- PC5-Logistic (VLAN 60)
    

```plaintext
ping 10.32.10.10      
ping 10.32.30.100     
ping 10.32.60.100     
```
![](images/Pasted%20image%2020251125230407.png)

### **Results**

All endpoints successfully reach devices in other VLANs using ICMP echo requests. The tests confirm that router-on-a-stick subinterfaces, VLAN trunking and DHCP relay are configured correctly. Dynamic hosts receive valid IPv4 addresses from the DHCP server, and inter-VLAN routing operates as expected across the entire topology.


<br>

## **7.8 Conclusion**

The chapter establishes the essential network services on the Xubuntu Server. DHCP distributes addressing to all dynamic VLANs, Apache provides an internal web page, and Bind9 resolves local hostnames for the company domain. Connectivity tests confirm that routing, DHCP relay and VLAN segmentation operate correctly across all subnets. HTTP and DNS can be fully verified only on Xubuntu devices, because VPCS lacks complete client support, but all services themselves work as intended. The next chapter introduces NAT and PAT.
<br>

--- 


<br>


**Next chapter:** 