☕ Coffee Shop Network Topology | Cisco Packet Tracer

📌 Project Overview

This project demonstrates the design and configuration of a small coffee shop network using Cisco Packet Tracer. The network separates office devices, Point of Sale (POS) devices, guest Wi-Fi users, and network management traffic using VLANs.
The project uses a Cisco router for inter-VLAN routing, DHCP services, and access control, while a switch connects wired devices and a wireless access point.
The objective is to build a segmented, manageable, and secure network suitable for a small business environment.

🎯 Project Objectives

•	Design a small business network using Cisco Packet Tracer.
•	Configure VLANs to separate different departments.
•	Implement router-on-a-stick inter-VLAN routing.
•	Configure DHCP for office, POS, and guest Wi-Fi networks.
•	Configure static IP addresses for printers and network management.
•	Configure a wireless access point with WPA2-PSK security.
•	Implement an extended ACL to restrict guest access to internal networks.
•	Configure SSH for secure remote switch management.
•	Test network connectivity and troubleshoot configuration issues.

🛠️ Tools and Technologies

•	Cisco Packet Tracer
•	Cisco IOS CLI
•	VLANs and 802.1Q trunking
•	Inter-VLAN routing
•	DHCP
•	IPv4 addressing and subnetting
•	Extended Access Control Lists (ACLs)
•	Wireless LAN and WPA2-PSK
•	SSH
•	Network troubleshooting commands

🖥️ Network Topology

The network contains the following devices:
Device	Purpose
CoffeeShop-RTR	Routing, DHCP, and guest traffic filtering
CoffeeShop-SW	VLAN creation, switching, and trunk connectivity
Wireless Access Point	Provides guest Wi-Fi connectivity
Manager PC	Office administration
Office Printer	Office printing
POS Terminal	Point of Sale operations
Receipt Printer	POS receipt printing
Guest Laptop 1	Guest wireless access
Guest Laptop 2	Guest wireless access
ISP Cloud	Represents the external network
Topology image: Save your Cisco Packet Tracer screenshot as images/coffeeshop-topology.png and upload it to the repository. Update the filename below if you use a different name.

🗺️ Network Architecture

The router connects to the switch through an 802.1Q trunk. Router subinterfaces act as the default gateways for the different VLANs.
                      ISP Cloud
                          |
                    CoffeeShop-RTR
                          |
                     802.1Q Trunk
                          |
                     CoffeeShop-SW
                          |
          +---------------+---------------+
          |               |               |
      VLAN 10          VLAN 20         VLAN 30
    Office Devices    POS Devices     Guest Wi-Fi
                                          |
                                    Wireless AP
                                     /        \
                                Laptop 1    Laptop 2

                    VLAN 99: Management
🌐 VLAN and IP Addressing Plan

VLAN ID	VLAN Name	Network	Default Gateway
10	MANAGEMENT_OFFICE	192.168.10.0/24	192.168.10.1
20	POS	192.168.20.0/24	192.168.20.1
30	GUEST_WIFI	192.168.30.0/24	192.168.30.1
99	NETWORK_MANAGEMENT	192.168.99.0/24	192.168.99.1
IP Address Assignments
Device	IP Address	Assignment
Router VLAN 10 gateway	192.168.10.1	Static
Office Printer	192.168.10.10	Static
Manager PC	192.168.10.21 onward	DHCP
Router VLAN 20 gateway	192.168.20.1	Static
Receipt Printer	192.168.20.10	Static
POS Terminal	192.168.20.21 onward	DHCP
Router VLAN 30 gateway	192.168.30.1	Static
Guest Laptop 1	192.168.30.21 onward	DHCP
Guest Laptop 2	192.168.30.21 onward	DHCP pool, with a unique lease per device
Switch management SVI	192.168.99.2	Static
Router VLAN 99 gateway	192.168.99.1	Static
All networks use the subnet mask 255.255.255.0.

Note: The switch management IP belongs to VLAN 99. The DHCP pools exclude .1 through .20, reserving these addresses for gateways and static devices.

⚙️ Configuration 1: Multilayer Switch

The following configuration uses the switch as a Layer 2 VLAN switch with a management SVI and a trunk to the router.

1. Basic Configuration
enable
configure terminal
hostname CoffeeShop-SW
no ip domain-lookup
enable secret Cisco123!
service password-encryption
banner motd #Unauthorized access is prohibited#

line console 0
password Cisco123!
login
logging synchronous
exit
2. Create VLANs
vlan 10
name MANAGEMENT_OFFICE
exit

vlan 20
name POS
exit

vlan 30
name GUEST_WIFI
exit

vlan 99
name NETWORK_MANAGEMENT
exit

3. Assign Access Ports
   
Use the interface ranges that match your actual topology. The following are example assignments; do not apply them without checking your cable connections.

interface range fa0/2 - 3
description MANAGEMENT_OFFICE_DEVICES
switchport mode access
switchport access vlan 10
spanning-tree portfast
exit

interface range fa0/5, fa0/7
description POS_DEVICES
switchport mode access
switchport access vlan 20
spanning-tree portfast
exit

interface fa0/11
description GUEST_WIRELESS_ACCESS_POINT
switchport mode access
switchport access vlan 30
spanning-tree portfast
exit
Assign the office printer and Manager PC to VLAN 10. Assign the POS terminal and receipt printer to VLAN 20. Assign the wireless access point to VLAN 30.
Important: Confirm port numbers in Packet Tracer before using these commands. Each access port must belong to the appropriate VLAN, and a physical port should not be assigned to conflicting roles.

4. Configure the Router Trunk
   
If the router is connected to Fa0/1:
interface fa0/1
description TO_COFFESHOP_RTR
switchport mode trunk
switchport trunk allowed vlan 10,20,30,99
no shutdown
exit

If your router is connected to a GigabitEthernet port, configure the actual connected interface instead.
Some switch models require switchport trunk encapsulation dot1q before enabling trunk mode. Use this command only if your switch supports it.

6. Configure Switch Management
   
interface vlan 99
description SWITCH_MANAGEMENT
ip address 192.168.99.2 255.255.255.0
no shutdown
exit

ip default-gateway 192.168.99.1

The management SVI requires VLAN 99 to exist and have an active Layer 2 path.

7. Configure SSH
   
ip domain-name ittechstudies.local
username admin privilege 15 secret Cisco123!
crypto key generate rsa
When prompted for the key size, select 2048 bits if supported by your Packet Tracer device.
ip ssh version 2

line vty 0 15
login local
transport input ssh
exec-timeout 10 0
exit

end
copy running-config startup-config

⚙️ Configuration 2: Router

1. Basic Router Configuration
   
enable
configure terminal
hostname CoffeeShop-RTR
no ip domain-lookup
enable secret Cisco123!
service password-encryption
banner motd #Unauthorized access is prohibited#

line console 0
password Cisco123!
login
logging synchronous
exit

2. Configure Physical Interfaces
   
interface gigabitEthernet0/0
description TO_ISP
no shutdown
exit

interface gigabitEthernet0/1
description TO_COFFESHOP_SW
no shutdown
exit

The ISP interface requires the appropriate ISP-provided IP addressing and routing configuration before Internet connectivity can work.

3. Configure Router-on-a-Stick Subinterfaces
   
VLAN 10 — Management Office

interface gigabitEthernet0/1.10
description MANAGEMENT_OFFICE_GATEWAY
encapsulation dot1Q 10
ip address 192.168.10.1 255.255.255.0
exit

VLAN 20 — POS

interface gigabitEthernet0/1.20
description POS_GATEWAY
encapsulation dot1Q 20
ip address 192.168.20.1 255.255.255.0
exit

VLAN 30 — Guest Wi-Fi

interface gigabitEthernet0/1.30
description GUEST_WIFI_GATEWAY
encapsulation dot1Q 30
ip address 192.168.30.1 255.255.255.0
exit

VLAN 99 — Network Management

interface gigabitEthernet0/1.99
description NETWORK_MANAGEMENT_GATEWAY
encapsulation dot1Q 99
ip address 192.168.99.1 255.255.255.0
exit

end

⚙️ Configuration 3: DHCP

DHCP automatically assigns IP addresses, subnet masks, default gateways, and DNS server addresses to client devices.

1. Exclude Static Addresses
   
configure terminal
ip dhcp excluded-address 192.168.10.1 192.168.10.20
ip dhcp excluded-address 192.168.20.1 192.168.20.20
ip dhcp excluded-address 192.168.30.1 192.168.30.20

2. Create the Office DHCP Pool
   
ip dhcp pool MANAGEMENT_OFFICE
network 192.168.10.0 255.255.255.0
default-router 192.168.10.1
dns-server 8.8.8.8
exit

4. Create the POS DHCP Pool
   
ip dhcp pool POS
network 192.168.20.0 255.255.255.0
default-router 192.168.20.1
dns-server 8.8.8.8
exit

6. Create the Guest Wi-Fi DHCP Pool
   
ip dhcp pool GUEST_WIFI
network 192.168.30.0 255.255.255.0
default-router 192.168.30.1
dns-server 8.8.8.8
exit

end
copy running-config startup-config

The DNS address 8.8.8.8 is a public DNS resolver. It will only be reachable if the external network and routing are configured appropriately.

🔐 Configuration 4: Guest Network Security ACL

The guest network should not access the office, POS, or management subnets. The following extended ACL blocks those destinations while permitting DHCP requests and other permitted guest traffic.
configure terminal

ip access-list extended GUEST_RESTRICTIONS
permit udp any eq 68 any eq 67
deny ip 192.168.30.0 0.0.0.255 192.168.10.0 0.0.0.255
deny ip 192.168.30.0 0.0.0.255 192.168.20.0 0.0.0.255
deny ip 192.168.30.0 0.0.0.255 192.168.99.0 0.0.0.255
permit ip 192.168.30.0 0.0.0.255 any
exit

interface gigabitEthernet0/1.30
ip access-group GUEST_RESTRICTIONS in
exit

end
copy running-config startup-config

Security Notes
•	The DHCP permit is placed before the deny statements to allow guest clients to request an address.
•	The ACL blocks guest traffic to the three listed internal subnets.
•	Other internal subnets, if added later, must be considered in the ACL.
•	The final permit allows guest traffic to other destinations but does not create Internet connectivity.
•	Internet access also requires appropriate ISP addressing, routing, and potentially NAT.

📶 Configuration 5: Wireless Access Point

Configure the wireless access point in Packet Tracer using the following settings:
Setting	Value
SSID	Coffeeshop-Guest
Security	WPA2-PSK
Authentication	Pre-shared key
Password	Use a strong, unique lab password
Connected switch VLAN	VLAN 30
Connect the Guest Laptops
1.	Open each laptop in Packet Tracer.
2.	If required, install a compatible wireless network adapter in the Physical tab.
3.	Open the wireless utility or configuration page.
4.	Connect to Coffeeshop-Guest.
5.	Enter the configured WPA2-PSK password.
6.	Open Desktop → IP Configuration.
7.	Select DHCP.
Expected guest addressing:
•	IP address: 192.168.30.x
•	Subnet mask: 255.255.255.0
•	Default gateway: 192.168.30.1
🖨️ Configuration 6: Printers and End Devices
Office Printer
Configure a static IP address:
IP address:      192.168.10.10
Subnet mask:     255.255.255.0
Default gateway: 192.168.10.1
DNS server:      8.8.8.8
Receipt Printer
Configure a static IP address:
IP address:      192.168.20.10
Subnet mask:     255.255.255.0
Default gateway: 192.168.20.1
DNS server:      8.8.8.8
Manager PC and POS Terminal
Select DHCP in the IP Configuration window.
•	Manager PC: VLAN 10
•	POS Terminal: VLAN 20
🧪 Configuration 7: Verification and Testing
Switch Verification
show vlan brief
show interfaces trunk
show ip interface brief
show running-config
Check that the access ports are assigned to the correct VLANs and the trunk permits VLANs 10, 20, 30, and 99.
Router Verification
show ip interface brief
show ip dhcp pool
show ip dhcp binding
show ip access-lists
show running-config
Check that the router subinterfaces are operational, DHCP pools exist, clients have received leases, and ACL rules are installed.
Connectivity Testing
From a guest laptop:
ipconfig
ping 192.168.30.1
From the Manager PC, test:
ping 192.168.10.1
ping 192.168.10.10
From the POS Terminal, test:
ping 192.168.20.1
ping 192.168.20.10
From a guest laptop, test access to internal networks:
ping 192.168.10.10
ping 192.168.20.10
ping 192.168.99.2
The guest-to-internal tests should be blocked by the ACL. A failed ping alone is not definitive proof of ACL operation; check ACL counters and ensure the destination is online.
🛠️ Troubleshooting
Problem	Checks and possible solution
Guest laptop does not receive an IP address	Verify DHCP pool, wireless association, VLAN 30 assignment, trunk configuration, and ACL DHCP permit
Laptop cannot ping 192.168.30.1	Check laptop IP configuration, router subinterface status, VLAN 30, and trunk
Office PC cannot reach its gateway	Check VLAN 10 port assignment and router subinterface
POS terminal cannot reach its gateway	Check VLAN 20 port assignment and router subinterface
Guest can reach internal networks	Verify ACL entries, inbound application, and rule counters
SSH does not work	Verify RSA keys, local username, VTY configuration, and management IP reachability
Internet is unavailable	Verify ISP interface addressing, routing, and NAT where required
💾 Saving the Configuration
On the router and switch:
copy running-config startup-config
Save the Packet Tracer project using File → Save As and include the .pkt file in the GitHub repository if you want others to inspect and reproduce the topology.
📁 Suggested GitHub Repository Structure
coffee-shop-network-topology/
│
├── README.md
├── coffeeshop-topology.pkt
├── images/
│   └── coffeeshop-topology.png
└── configurations/
    ├── router-config.txt
    └── switch-config.txt
🚀 Learning Outcomes
Through this project, I practised:
•	Creating and managing VLANs.
•	Configuring 802.1Q trunks.
•	Implementing inter-VLAN routing with router subinterfaces.
•	Configuring DHCP pools and address exclusions.
•	Setting up wireless connectivity with WPA2-PSK.
•	Applying extended ACLs to isolate guest traffic.
•	Configuring switch management and SSH.
•	Testing connectivity and troubleshooting network configuration issues.
🔮 Future Improvements
•	Configure NAT and default routing for Internet access.
•	Add a dedicated firewall or stronger guest isolation.
•	Implement a dedicated management VLAN access policy.
•	Configure logging, monitoring, and device backups.
•	Add redundant links and improve network availability.
