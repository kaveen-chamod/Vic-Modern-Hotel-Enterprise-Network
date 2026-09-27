# Vic Modern Hotel - Enterprise Network Design & Implementation

This repository contains the complete network design and implementation for the Vic Modern Hotel, deployed using Cisco Packet Tracer. The architecture spans three floors, supporting eight distinct departments with strict departmental isolation, inter-VLAN routing, dynamic IP allocation, and robust network security protocols.

## Network Architecture & Topology
The core of the network consists of three edge routers housed in the 3rd-floor IT Server Room, interconnected via Serial DCE cables forming a robust core backbone. Each floor is equipped with a dedicated switch and wireless access points to support both wired workstations and mobile devices (laptops and smartphones).

### Floor Breakdown & VLAN Allocation
| Floor | Department | VLAN ID | Network Address |
| :--- | :--- | :--- | :--- |
| **1st Floor** | Reception | VLAN 80 | `192.168.8.0/24` |
| | Store | VLAN 70 | `192.168.7.0/24` |
| | Logistics | VLAN 60 | `192.168.6.0/24` |
| **2nd Floor** | Finance | VLAN 50 | `192.168.5.0/24` |
| | HR | VLAN 40 | `192.168.4.0/24` |
| | Sales/Marketing | VLAN 30 | `192.168.3.0/24` |
| **3rd Floor** | Admin | VLAN 20 | `192.168.2.0/24` |
| | IT | VLAN 10 | `192.168.1.0/24` |

### Core Router Networks (Serial Connections)
* `10.10.10.0/30`
* `10.10.10.4/30`
* `10.10.10.8/30`

## Key Technologies & Protocols Implemented

* **OSPF Routing:** Configured Single-Area OSPF across all three routers to seamlessly advertise routes and ensure full end-to-end communication across the entire hotel network.
* **Router-on-a-Stick (Inter-VLAN Routing):** Implemented 802.1Q trunking and sub-interfaces on the routers to route traffic securely between the isolated departmental VLANs.
* **DHCP Services:** Each router acts as a DHCP server for its respective floor, dynamically allocating IP addresses to PCs, laptops, and smartphones.
* **Secure Remote Access (SSH):** Secure Shell (SSH) is configured on all routers with local authentication and RSA crypto keys to allow secure remote administration.
* **Layer 2 Port Security:** Applied strict MAC-address sticky port security on the IT department switch (`fa0/1`). Only the designated **Test-PC** is permitted access; any unauthorized device triggers a port shutdown violation.
* **Wireless Networking:** WAPs configured per floor to provide untethered access for modern hotel operations.

## Testing & Verification
1. **Dynamic IP Assignment:** Verified that all devices successfully obtain IPv4 addresses, subnet masks, and default gateways from their respective router's DHCP pools.
2. **End-to-End Connectivity:** ICMP Echo requests (Pings) are successful across all VLANs and floors.
3. **SSH Verification:** Successfully accessed the remote routers via the CLI of the *Test-PC* using configured credentials.
4. **Security Audit:** Connecting a rogue laptop to port `fa0/1` immediately placed the port into an `err-disable` state, confirming the violation policy.

### Username - gtech  
### Password - getech

### DHCP Verification (Reception PC - VLAN 80)
C:\>ipconfig

FastEthernet0 Connection:(default port)
   IPv4 Address....................: 192.168.8.2
   Subnet Mask.....................: 255.255.255.0
   Default Gateway.................: 192.168.8.1


### Router Configuration

enable
configure terminal
hostname R3-Floor3

! SSH Configuration
ip domain-name vicmodern.local
crypto key generate rsa modulus 2048
username admin privilege 15 secret Admin@123
line vty 0 4
transport input ssh
login local
exit

! VLAN 10 (IT) Subinterface
interface GigabitEthernet0/0.10
encapsulation dot1Q 10
ip address 192.168.1.1 255.255.255.0
exit

! VLAN 20 (Admin) Subinterface
interface GigabitEthernet0/0.20
encapsulation dot1Q 20
ip address 192.168.2.1 255.255.255.0
exit

! DHCP Configuration for dynamic IPs
ip dhcp pool IT_VLAN10
network 192.168.1.0 255.255.255.0
default-router 192.168.1.1
ip dhcp pool ADMIN_VLAN20
network 192.168.2.0 255.255.255.0
default-router 192.168.2.1
exit

! OSPF Routing Configuration
router ospf 1
network 192.168.1.0 0.0.0.255 area 0
network 192.168.2.0 0.0.0.255 area 0
network 10.10.10.0 0.0.0.3 area 0
network 10.10.10.8 0.0.0.3 area 0
exit


## Switch Configuration:

enable
configure terminal
hostname SW3-Floor3

vlan 10
name IT
vlan 20
name Admin
exit

! Trunk port connecting to Router
interface GigabitEthernet0/1
switchport mode trunk
exit

! Port Security for Test-PC on IT Department
interface FastEthernet0/1
switchport mode access
switchport access vlan 10
switchport port-security
switchport port-security maximum 1
switchport port-security mac-address sticky
switchport port-security violation shutdown
exit
