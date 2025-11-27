# 📘 Enterprise Office Network Design – GNS3

Enterprise-grade office network design using GNS3, featuring **VLANs, OSPF, HSRP, PortChannel, DHCP, ACLs, and NAT**. Includes full topology, configs, and verification outputs.

---

## ✅ 1️⃣ Project Overview

This project simulates a fully functional **enterprise office network** using GNS3.  
It follows real-world corporate design standards with **core & access layers**, **redundancy**, **segmentation**, **dynamic routing**, and **secure internet access**.

This project demonstrates practical skills in designing, configuring, and troubleshooting medium-scale enterprise networks.

---

## ✅ 2️⃣ Objectives

- ✅ Build a multi-layer hierarchical network (Edge → Core → Access)  
- ✅ Segment departments using VLANs  
- ✅ Enable inter-VLAN routing using Layer 3 switches  
- ✅ Configure OSPF for dynamic routing  
- ✅ Provide internet access using NAT  
- ✅ Apply ACLs for secure access control  
- ✅ Implement HSRP for gateway redundancy  
- ✅ Deploy PortChannel for high availability  

---

## ✅ 3️⃣ Logical Network Topology

```text
                 ┌──────────────────────┐
                 │      ISP / Cloud     │
                 └────────────┬─────────┘
                              │
                        (NAT Outside)
                              │
                       ┌───────────┐
                       │  EDGE R1  │
                       └─────┬─────┘
                           OSPF
                        Area 0 Link
       ┌────────────────┴───────────────┐
       │                                │
 ┌───────────────┐                ┌───────────────┐
 │ Core Switch C1│====PortChannel====│ Core Switch C2│
 └───────┬───────┘                └───────┬───────┘
     HSRP│ VIP                        HSRP│ VIP
─────────┼────────────────────────────────┼───────────────
         │                │              │              │              │
     VLAN 10           VLAN 20        VLAN 30        VLAN 40        VLAN 50
 IT & Mgmt Dept    Finance & HR    Sales & Mktg    Voice Services   Guest Services

### 🖼️ Topology Diagram (GNS3)
![Office Network Topology](Topology/OfficeNetworkTopology.png)


✅ 4️⃣ Technologies Used

VLANs & Inter-VLAN Routing (SVIs on Core)
OSPF (Dynamic Routing, Area 0)
HSRP (Gateway Redundancy)
EtherChannel / PortChannel (LACP)
DHCP (Per-VLAN scopes)
NAT Overload (on Edge router)
ACLs (Inter-VLAN & management security)
GNS3 simulator

✅ 5️⃣ VLAN & IP Addressing Plan
| VLAN | Department        | Subnet          | Gateway (HSRP VIP) |
| ---- | ----------------- | --------------- | ------------------ |
| 10   | IT & Management   | 192.168.10.0/24 | 192.168.10.1       |
| 20   | Finance & HR      | 192.168.20.0/24 | 192.168.20.1       |
| 30   | Sales & Marketing | 192.168.30.0/24 | 192.168.30.1       |
| 40   | Voice Services    | 192.168.40.0/24 | 192.168.40.1       |
| 50   | Guest Services    | 192.168.50.0/24 | 192.168.50.1       |

✅ 6️⃣ Core Routing – OSPF Configuration
OSPF Area 0 was used between CORE1, CORE2, and EDGE for dynamic routing.

router ospf 1
 network 192.168.10.0 0.0.0.255 area 0
 network 192.168.20.0 0.0.0.255 area 0
 network 192.168.30.0 0.0.0.255 area 0
 network 192.168.40.0 0.0.0.255 area 0
 network 192.168.50.0 0.0.0.255 area 0
 network 192.168.255.0 0.0.0.3 area 0

✅ 7️⃣ Redundancy & High Availability Design
🔹 7.1 HSRP – Gateway Redundancy
CORE1 is active gateway for VLAN 10 & 20
CORE2 is active gateway for VLAN 30, 40 & 50
If any core fails, the other becomes gateway for all VLANs using HSRP failover

🔹 7.2 PortChannel – Link Redundancy
Multiple physical uplinks were bundled using LACP PortChannels
Increases available bandwidth
Prevents single-link failures from impacting connectivity
Result: End-user traffic remains online even during link or core failures.

✅ 8️⃣ DHCP Implementation
Dynamic IP addressing configured on CORE1:
ip dhcp excluded-address 192.168.10.1 192.168.10.9
ip dhcp excluded-address 192.168.20.1 192.168.20.9
ip dhcp excluded-address 192.168.30.1 192.168.30.9
ip dhcp excluded-address 192.168.40.1 192.168.40.9
ip dhcp excluded-address 192.168.50.1 192.168.50.9

ip dhcp pool IT_VLAN10
 network 192.168.10.0 255.255.255.0
 default-router 192.168.10.1
 dns-server 8.8.8.8 1.1.1.1

ip dhcp pool FINANCE_VLAN20
 network 192.168.20.0 255.255.255.0
 default-router 192.168.20.1
 dns-server 8.8.8.8 1.1.1.1

✅ 9️⃣ NAT & Internet Access (EDGE Router)
NAT overload on the EDGE router provides secure internet access for internal users.
ip access-list standard NAT_ACL
 permit 192.168.0.0 0.0.255.255

interface Ethernet0/0
 ip address dhcp
 ip nat outside

interface Ethernet0/1
 description Link to CORE1
 ip address 192.168.255.1 255.255.255.252
 ip nat inside

interface Ethernet0/2
 description Link to CORE2
 ip address 192.168.255.5 255.255.255.252
 ip nat inside

ip nat inside source list NAT_ACL interface Ethernet0/0 overload

✅ 🔟 ACL Security Policies
Three key security policies were implemented on the Core switches:

🔹 10.1 Protect Finance & HR (VLAN 20)
Only IT (VLAN 10) can reach Finance & HR; Sales, Voice and Guest are blocked.\
ip access-list extended HR-FIN-PROTECT
 remark Block unauthorized access to Finance & HR
 deny   ip 192.168.30.0 0.0.0.255 192.168.20.0 0.0.0.255   ! Sales
 deny   ip 192.168.40.0 0.0.0.255 192.168.20.0 0.0.0.255   ! Voice
 deny   ip 192.168.50.0 0.0.0.255 192.168.20.0 0.0.0.255   ! Guest
 permit ip any any

interface Vlan20
 ip access-group HR-FIN-PROTECT in

🔹 10.2 Guest VLAN – Internet Only
Guest VLAN 50 is blocked from internal subnets but allowed to the internet via NAT.
ip access-list extended GUEST-INTERNET-ONLY
 remark Deny Guest access to internal subnets, allow internet
 deny   ip 192.168.50.0 0.0.0.255 192.168.10.0 0.0.0.255
 deny   ip 192.168.50.0 0.0.0.255 192.168.20.0 0.0.0.255
 deny   ip 192.168.50.0 0.0.0.255 192.168.30.0 0.0.0.255
 deny   ip 192.168.50.0 0.0.0.255 192.168.40.0 0.0.0.255
 permit ip 192.168.50.0 0.0.0.255 any

interface Vlan50
 ip access-group GUEST-INTERNET-ONLY in

🔹 10.3 Secure Device Management (SSH)
Only IT & Management (VLAN 10) is allowed to manage CORE and EDGE devices via SSH.

ip access-list standard IT-MGMT-SSH
 permit 192.168.10.0 0.0.0.255
 deny   any

line vty 0 4
 access-class IT-MGMT-SSH in
 transport input ssh

ACLs and ip access-group are configured on both core switches so that security is preserved even during HSRP failover.

✅ 1️⃣1️⃣ Verification & Testing
Key verification commands:
show standby
show etherchannel summary
show ip ospf neighbor
show ip dhcp binding
show ip route
ping 8.8.8.8
traceroute

These were used to confirm:

- HSRP state and failover behavior
- PortChannel status and member links
- OSPF neighbor relationships and routes
- DHCP bindings per VLAN
- End-to-end connectivity to the internet

✅ 1️⃣2️⃣ Results
✅ Successful inter-VLAN routing for all departments
✅ Seamless gateway failover during core switch failure
✅ Reliable internet access via NAT on EDGE
✅ Finance & HR traffic protected from non-authorized VLANs
✅ Guest VLAN isolated from internal networks
✅ Device management restricted to IT subnet

✅ Conclusion
This project successfully demonstrates the design and deployment of an enterprise-grade network with VLAN segmentation, dynamic routing, high availability, secure internet access, and inter-department security controls.
The implementation of OSPF, HSRP, PortChannel, NAT, DHCP, and ACLs reflects real-world networking practices and showcases hands-on skills required for Network Engineer Intern / Junior Network Engineer roles.
The network remained functional during failover testing and enforced all security policies as intended
