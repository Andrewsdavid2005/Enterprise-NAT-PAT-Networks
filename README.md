# Enterprise NAT and PAT Configuration

<p align="center">
  <strong>Cisco Packet Tracer | Network Engineering | CCNA Practical</strong>
</p>

## 1. Project Overview

The Enterprise NAT and PAT Configuration project demonstrates how an enterprise network connects private internal devices to an external network using Network Address Translation (NAT).

This project is developed using Cisco Packet Tracer and Cisco 2911 routers. It implements Static NAT, Dynamic NAT, and Port Address Translation (PAT) to manage communication between internal users, an internal web server, and an external network.

The project simulates a real-world enterprise environment where multiple employees share public IP connectivity while an internal web server is accessible through a dedicated public IP address.

## 2. Project Objectives

- Understand Network Address Translation (NAT).
- Configure Static NAT for internal server access.
- Implement Dynamic NAT using a public IP address pool.
- Configure PAT (NAT Overload) for multiple internal users.
- Configure NAT inside and outside interfaces.
- Implement standard Access Control Lists (ACLs).
- Configure static routing between enterprise and ISP routers.
- Verify NAT translations and troubleshoot network connectivity.

## 3. Network Architecture

### Network Topology

The network consists of an enterprise LAN connected to an ISP router and an external test network.

**Network architecture flow:**

```text
             INTERNAL ENTERPRISE NETWORK
                         |
        +----------------+----------------+
        |                |                |
      PC-HR         PC-FINANCE          PC-IT
        |                |                |
        +----------------+----------------+
                         |
                    SW1-LAN
                   Cisco 2960
                         |
                         |
                R1-ENTERPRISE
                  Cisco 2911
                   NAT Router
                         |
                   WAN Connection
                         |
                     R2-ISP
                   Cisco 2911
                         |
                EXTERNAL NETWORK
                         |
                 INTERNET-SERVER

        WEB-SERVER is connected to SW1-LAN
```

### Devices Used

| Device | Quantity | Model |
|---|---:|---|
| Enterprise Router | 1 | Cisco 2911 |
| ISP Router | 1 | Cisco 2911 |
| LAN Switch | 1 | Cisco 2960 |
| HR PC | 1 | PC-PT |
| Finance PC | 1 | PC-PT |
| IT PC | 1 | PC-PT |
| Internal Web Server | 1 | Server-PT |
| External Server | 1 | Server-PT |

## 4. IP Addressing Plan

### 4.1 Internal LAN Network

Network Address: `192.168.10.0/24`

| Device | Interface | IP Address | Default Gateway |
|---|---|---|---|
| R1-ENTERPRISE | G0/0 | 192.168.10.1 | — |
| PC-HR | Fa0 | 192.168.10.10 | 192.168.10.1 |
| PC-FINANCE | Fa0 | 192.168.10.11 | 192.168.10.1 |
| PC-IT | Fa0 | 192.168.10.12 | 192.168.10.1 |
| WEB-SERVER | Fa0 | 192.168.10.100 | 192.168.10.1 |

Subnet Mask: `255.255.255.0`

### 4.2 WAN Network

Network Address: `203.0.113.0/29`

| Device | Interface | IP Address |
|---|---|---|
| R1-ENTERPRISE | G0/1 | 203.0.113.1 |
| R2-ISP | G0/0 | 203.0.113.2 |
| Static NAT Global IP | — | 203.0.113.3 |
| Dynamic NAT Pool | — | 203.0.113.4–203.0.113.6 |

Subnet Mask: `255.255.255.248`

### 4.3 External Network

Network Address: `198.51.100.0/24`

| Device | Interface | IP Address |
|---|---|---|
| R2-ISP | G0/1 | 198.51.100.1 |
| INTERNET-SERVER | Fa0 | 198.51.100.10 |

Subnet Mask: `255.255.255.0`

## 5. NAT Implementation

This project implements three types of Network Address Translation.

### 5.1 Static NAT

Static NAT establishes a permanent one-to-one mapping between a private IP address and a public IP address.

**Configuration:**

```cisco
ip nat inside source static 192.168.10.100 203.0.113.3
```

**Mapping:**

```text
Private IP: 192.168.10.100
Public IP:  203.0.113.3
```

**Purpose:** Allows external users to access the internal web server through its public IP address.

### 5.2 Dynamic NAT

Dynamic NAT translates private IP addresses into available public addresses from a configured NAT pool.

**Configuration:**

```cisco
access-list 11 permit host 192.168.10.12

ip nat pool DYNAMIC_POOL 203.0.113.4 203.0.113.6 netmask 255.255.255.248

ip nat inside source list 11 pool DYNAMIC_POOL
```

**Mapping:**

```text
Internal IP: 192.168.10.12

Public NAT Pool:
203.0.113.4
203.0.113.5
203.0.113.6
```

**Purpose:** Provides public IP translation for the IT department's PC using an available address from the configured pool.

### 5.3 PAT (Port Address Translation)

PAT, also known as NAT Overload, allows multiple internal devices to share one public IP address by using different transport-layer port numbers or identifiers.

**Configuration:**

```cisco
access-list 10 permit host 192.168.10.10
access-list 10 permit host 192.168.10.11

ip nat inside source list 10 interface gigabitEthernet 0/1 overload
```

**Mapping:**

```text
PC-HR       192.168.10.10
PC-FINANCE  192.168.10.11

Shared Public IP:
203.0.113.1
```

**Purpose:** Allows HR and Finance users to access the external network through the same public IP address.

## 6. Router Configuration

### 6.1 R1-ENTERPRISE Configuration

**Configure the internal LAN interface:**

```cisco
enable
configure terminal

hostname R1-ENTERPRISE

interface gigabitEthernet 0/0
description INSIDE-LAN
ip address 192.168.10.1 255.255.255.0
ip nat inside
no shutdown
exit
```

**Configure the WAN interface:**

```cisco
interface gigabitEthernet 0/1
description OUTSIDE-WAN
ip address 203.0.113.1 255.255.255.248
ip nat outside
no shutdown
exit
```

**Configure the default route:**

```cisco
ip route 0.0.0.0 0.0.0.0 203.0.113.2
```

**Save the configuration:**

```cisco
end
write memory
```

### 6.2 R2-ISP Configuration

```cisco
enable
configure terminal

hostname R2-ISP

interface gigabitEthernet 0/0
description CONNECTION-TO-ENTERPRISE
ip address 203.0.113.2 255.255.255.248
no shutdown
exit

interface gigabitEthernet 0/1
description EXTERNAL-NETWORK
ip address 198.51.100.1 255.255.255.0
no shutdown
exit

end
write memory
```

## 7. Access Control List Configuration

Standard ACLs are used to identify which internal source addresses are eligible for NAT translation.

**PAT ACL:**

```cisco
access-list 10 permit host 192.168.10.10
access-list 10 permit host 192.168.10.11
```

**Dynamic NAT ACL:**

```cisco
access-list 11 permit host 192.168.10.12
```

### Optional Outside Interface Security ACL

The following example permits HTTP access to the published web server, denies other traffic addressed to the internal LAN, and permits other traffic.

```cisco
ip access-list extended OUTSIDE-FILTER
permit tcp any host 203.0.113.3 eq 80
deny ip any 192.168.10.0 0.0.0.255
permit ip any any
exit

interface gigabitEthernet 0/1
ip access-group OUTSIDE-FILTER in
exit
```

This is an optional lab security exercise. Verify the HTTP and other traffic behavior in Packet Tracer after applying it.

## 8. Server Configuration

### Internal Web Server

| Parameter | Value |
|---|---|
| IP Address | 192.168.10.100 |
| Subnet Mask | 255.255.255.0 |
| Default Gateway | 192.168.10.1 |

Enable HTTP under:

`WEB-SERVER → Services → HTTP → On`

### External Internet Server

| Parameter | Value |
|---|---|
| IP Address | 198.51.100.10 |
| Subnet Mask | 255.255.255.0 |
| Default Gateway | 198.51.100.1 |

Enable HTTP under:

`INTERNET-SERVER → Services → HTTP → On`

The addresses `203.0.113.0/24` and `198.51.100.0/24` are documentation networks used for this educational simulation.

## 9. Verification Commands

The following Cisco IOS commands are used to verify the network configuration.

**Interface verification:**

```cisco
show ip interface brief
```

**Routing table verification:**

```cisco
show ip route
```

**NAT translation verification:**

```cisco
show ip nat translations
```

**NAT statistics:**

```cisco
show ip nat statistics
```

**ACL verification:**

```cisco
show access-lists
```

**Complete configuration:**

```cisco
show running-config
```

## 10. Network Connectivity Testing

### Test 1: HR PC Connectivity

From PC-HR:

```text
ping 192.168.10.1
ping 198.51.100.10
```

Expected: Successful replies when LAN and PAT are correctly configured.

### Test 2: Finance PC Connectivity

From PC-FINANCE:

```text
ping 192.168.10.1
ping 198.51.100.10
```

Expected: Successful replies using PAT.

### Test 3: IT PC Connectivity

From PC-IT:

```text
ping 192.168.10.1
ping 198.51.100.10
```

Expected: Successful replies using Dynamic NAT.

### Test 4: Static NAT Web Server Access

From INTERNET-SERVER, open Desktop → Web Browser.

Enter:

```text
http://203.0.113.3
```

Expected: The internal web server's HTTP page should be displayed.

### Test 5: NAT Translation Verification

On R1-ENTERPRISE:

```cisco
show ip nat translations
```

Expected translation behavior:

| NAT Type | Inside Local | Inside Global |
|---|---|---|
| PAT | 192.168.10.10 | 203.0.113.1 |
| PAT | 192.168.10.11 | 203.0.113.1 |
| Dynamic NAT | 192.168.10.12 | 203.0.113.4–203.0.113.6 |
| Static NAT | 192.168.10.100 | 203.0.113.3 |

The exact displayed entries depend on generated traffic and Packet Tracer's simulation behavior.

## 11. Troubleshooting

| Problem | Troubleshooting Solution |
|---|---|
| PC cannot reach gateway | Check PC IP address, subnet mask, gateway and switch connection |
| HR cannot reach external server | Check PAT ACL, NAT interface roles and default route |
| Finance cannot reach external server | Verify ACL 10 and PAT configuration |
| IT cannot reach external server | Verify ACL 11 and dynamic NAT pool |
| Static NAT web access fails | Check static mapping, server HTTP service and routing |
| WAN interface is down | Check cable, interface configuration and `no shutdown` |
| NAT table is empty | Generate traffic and inspect ACL matches |
| External server cannot return traffic | Verify external server gateway and ISP connectivity |

## 12. Key Networking Concepts Learned

- Network Address Translation
- Static NAT
- Dynamic NAT
- Port Address Translation
- NAT Overload
- Private and public IPv4 addressing
- Standard and Extended ACLs
- Inside and outside NAT interfaces
- Static routing
- Enterprise-to-ISP connectivity
- Cisco IOS troubleshooting

## 13. Future Enhancements

The project can be extended with:

- Dual ISP connectivity and NAT failover
- OSPF dynamic routing
- Firewall security policies
- DHCP and DNS integration
- VLAN-based enterprise segmentation
- Network monitoring and alerting
- Network performance monitoring
- Automated configuration backups

## 14. Project Files

```text
Enterprise-NAT-PAT-Network/
│
├── README.md
├── Enterprise-NAT-PAT-Network.pkt
├── topology.png
│
└── configs/
    ├── R1-ENTERPRISE.txt
    └── R2-ISP.txt
```

## 15. Learning Outcomes

This project provided practical experience in configuring and troubleshooting NAT in an enterprise network environment.

It strengthened my understanding of public and private IP address translation, router configuration, access control lists, static routing, and network connectivity testing.

## 16. Author

**Andrews GnanaSelvin D**

Computer Science and Engineering Student

**Areas of Interest:**
- Network Engineering
- Cisco Networking
- CCNA
- Linux Administration
- Network Automation

GitHub: https://github.com/Andrewsdavid2005

LinkedIn: https://www.linkedin.com/in/andrews-gnana-selvin-d

---

**Project:** Enterprise NAT and PAT Configuration  
**Platform:** Cisco Packet Tracer  
**Difficulty:** Medium  
**Category:** Network Engineering / Enterprise Networking

*Developed as a practical networking project to strengthen Cisco IOS configuration, NAT implementation, and troubleshooting skills.*
