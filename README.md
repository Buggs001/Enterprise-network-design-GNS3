# Enterprise-network-design-GNS3
Enterprise-grade office network design using GNS3, featuring VLANs, OSPF, HSRP, PortChannel, DHCP, ACLs, and NAT. Includes full topology, configs, and verification outputs.

🏢 1️⃣ Project Overview
This project simulates a fully functional enterprise office network using GNS3.
It follows real-world corporate design standards with core & access layers, redundancy, segmentation, dynamic routing, and secure internet access.

This project demonstrates practical skills in designing, configuring, and troubleshooting medium-scale enterprise networks.

🎯 2️⃣ Objectives
✅ Build a multi-layer hierarchical network (Edge → Core → Access)
✅ Segment departments using VLANs
✅ Enable inter-VLAN routing using Layer 3 switches
✅ Configure OSPF for dynamic routing
✅ Provide internet access using NAT
✅ Apply ACLs for secure access control
✅ Implement HSRP for gateway redundancy
✅ Deploy PortChannel for high availability

🗺️ 3️⃣ Network Topology (Logical)
                 ┌──────────────────────┐
                 │      ISP / Cloud     │
                 └────────────┬─────────┘
                              |
                        (NAT Outside)
                              |
                       ┌───────────┐
                       │ Edge R1   │
                       └─────┬─────┘
                     OSPF Area 0
       ┌────────────────┴───────────────┐
       |                                |
 ┌───────────────┐                ┌───────────────┐
 │ Core Switch C1│==PortChannel===│ Core Switch C2│
 └───────┬───────┘                └────────|──────┘
         |                HSRP│VIP         |
─────────┼─────────────────────────────────|───────────────────
       │            │              │           |            |
   VLAN 10       VLAN 20        VLAN 30      VLAN 40      VLAN 50
   IT DEPT     Finance & HR     Sales Dept   Voice &     Guest services
                                            Services 


🧩 4️⃣ Technologies Used

VLANs & Inter-VLAN Routing
OSPF (Dynamic Routing)
DHCP
NAT Overload
ACLs
HSRP
EtherChannel (LACP)
GNS3 Simulation

✅ 5️⃣ VLAN & IP Addressing Plan
| VLAN | Department        | Subnet          | Gateway (HSRP VIP) |
| ---- | ----------------- | --------------- | ------------------ |
| 10   | IT & Management   | 192.168.10.0/24 | 192.168.10.1       |
| 20   | Finance & HR      | 192.168.20.0/24 | 192.168.20.1       |
| 30   | Sales & Marketing | 192.168.30.0/24 | 192.168.30.1       |
| 40   | Voice Services    | 192.168.40.0/24 | 192.168.40.1       |
| 50   | Guest Services    | 192.168.50.0/24 | 192.168.50.1       |

✅ 6️⃣ Core Routing – OSPF Configuration
OSPF Area 0 was used to establish routing between CORE1, CORE2, and the EDGE router.
router ospf 1
network 192.168.10.0 0.0.0.255 area 0
network 192.168.20.0 0.0.0.255 area 0
network 192.168.30.0 0.0.0.255 area 0
network 192.168.40.0 0.0.0.255 area 0
network 192.168.50.0 0.0.0.255 area 0
network 192.168.255.0 0.0.0.3 area 0

✅ 7️⃣ Redundancy & High Availability
🔹 HSRP – Gateway Redundancy
CORE1 active for VLAN 10 & 20
CORE2 active for VLAN 30, 40 & 50
If any core fails, the other becomes gateway for all VLANs

🔹 PortChannel – Link Redundancy
Bundled uplinks using LACP
Higher bandwidth + zero single-link failure
* Traffic stays online during device or link failure.

✅ 8️⃣ DHCP Implementation
DHCP pools hosted on CORE1:
ip dhcp pool IT_VLAN10
 network 192.168.10.0 255.255.255.0
 default-router 192.168.10.1
 dns-server 8.8.8.8 1.1.1.1
✅ Automated IP assignment
✅ Reduced manual provisioning

✅ 9️⃣ NAT & Internet Access (EDGE Router)
NAT overload was configured to allow internal private IPs to access the internet:
ip access-list standard NAT_ACL
 permit 192.168.0.0 0.0.255.255
ip nat inside source list NAT_ACL interface Ethernet0/0 overload
- Secure outbound internet access

✅ 🔟 ACL Security Policies
Three real-world policies were enforced:
A) Protect Finance & HR (VLAN 20)
Only IT VLAN can access VLAN 20; others blocked.
B) Guest VLAN – Internet Only
Guest VLAN blocked from internal networks.
C) Device Management Restricted
Only VLAN 10 can SSH into CORE/EDGE devices.
- ACLs applied on both core switches to maintain security during HSRP failover.

✅ 1️⃣1️⃣ Verification & Testing
Commands used:
show standby
show etherchannel summary
show ip ospf neighbor
show ip dhcp binding
ping 8.8.8.8
traceroute

✅ Verified routing
✅ Verified redundancy
✅ Verified security policies

✅ 1️⃣2️⃣ Results
✅ Successful inter-VLAN routing
✅ Seamless failover during CORE failure
✅ Internet access via NAT
✅ ACLs enforced departmental security
✅ DHCP auto-assigned IPs across VLANs

### ✅ Conclusion
This project successfully demonstrated the design and deployment of an enterprise-grade network with VLAN segmentation, dynamic routing, high availability, secure internet access, and inter-department security controls. The implementation of OSPF, HSRP, PortChannel, NAT, DHCP, and ACLs reflects real-world networking practices and showcases hands-on skills required for Network Engineer roles. The network remained functional during failover testing and enforced all security policies as intended.

