# AMG Enterprise Network Lab
### Aureon Manufacturing Group — Multi-Site Network Simulation

**Author:** Daniel Vargas Carpeta | [@Dani08-art](https://github.com/Dani08-art)  
**Environment:** Cisco Packet Tracer  
**Level:** CCNA / Junior Network Engineer  
**Status:** Version 1 Complete ✅ | Version 2 In Progress 🔧

---

## Overview

Full enterprise network simulation for **Aureon Manufacturing Group (AMG)**, a fictional mid-large manufacturing and technology distribution company. The lab reproduces a realistic corporate campus environment with a headquarters in Bogotá and a branch office in Medellín, covering the complete CCNA stack from physical topology to security policies.

This project was built from scratch as a hands-on learning tool to consolidate practical skills in network design, implementation, validation, and structured troubleshooting — targeting Junior Network Engineer and NOC Engineer Tier 2 roles.

---

## Network Architecture

```
                        [R-ISP]
                           |
                      Serial WAN
                           |
                     [R-HQ-EDGE]
                    /            \
              Gi0/0              Gi0/1
                |                  |
           [DSW1-HQ] ══ Po1 ══ [DSW2-HQ]
          (L3 Switch)  EtherChannel  (L3 Switch)
         /    |    \              /    |    \
      ASW1  ASW2  ASW3          ASW1 ASW2  ASW3
       |      |     |
    ADMIN   ENG   OPS
     FIN   SALES   IT
     RRHH         GUEST
                           [ASW4-HQ]
                               |
                           [SRV-HQ]
                        DHCP/DNS/NTP/Syslog

                     [R-MED] ── [SW-MED] ── Users
```

---

## Technologies Implemented

| Category | Technologies |
|---|---|
| **Switching** | VLANs, 802.1Q Trunking, Inter-VLAN Routing, STP Rapid PVST+, EtherChannel LACP |
| **Routing** | OSPF Multi-Area (Areas 0/1), Static routes, Layer 3 Switching |
| **High Availability** | HSRP v2 with load balancing across VLANs |
| **IP Infrastructure** | IPv4, VLSM, Subnetting, NAT/PAT |
| **Network Services** | DHCP (centralized), DNS (simulated), NTP, Syslog |
| **Security** | Extended ACLs, SSH v2, Port-Security (sticky), BPDU Guard, PortFast |
| **WAN** | Serial point-to-point links, simulated ISP connectivity |

---

## Site Information

### HQ — Bogotá
- Hierarchical campus design: Distribution (L3) + Access (L2)
- Dual distribution switches (DSW1/DSW2) with EtherChannel backbone
- HSRP active/standby load-balanced across 10 VLANs
- Centralized services server (DHCP, DNS, NTP, Syslog)
- Edge router with NAT/PAT for Internet access

### Branch — Medellín
- Single router + switch topology
- OSPF Area 1 connected to HQ backbone (Area 0)
- DHCP served centrally from HQ via ip helper-address
- WAN serial point-to-point link to HQ edge router

---

## VLAN Design

| VLAN | Name | Subnet | Mask | Gateway |
|---|---|---|---|---|
| 10 | ADMIN | 10.10.10.0 | /24 | 10.10.10.1 |
| 20 | FIN | 10.10.20.0 | /25 | 10.10.20.1 |
| 30 | HR | 10.10.30.0 | /26 | 10.10.30.1 |
| 40 | ENG | 10.10.40.0 | /23 | 10.10.40.1 |
| 50 | SALES | 10.10.50.0 | /25 | 10.10.50.1 |
| 60 | OPS | 10.10.60.0 | /24 | 10.10.60.1 |
| 70 | IT | 10.10.70.0 | /26 | 10.10.70.1 |
| 80 | GUEST | 10.10.80.0 | /24 | 10.10.80.1 |
| 90 | SERVER | 10.10.90.0 | /27 | 10.10.90.1 |
| 99 | MGMT | 10.10.99.0 | /27 | 10.10.99.1 |
| 999 | NATIVE | — | — | — |

---

## OSPF Design

| Area | Devices | Networks |
|---|---|---|
| Area 0 (backbone) | DSW1-HQ, DSW2-HQ, R-HQ-EDGE | 10.10.0.0/16, point-to-point /30 links |
| Area 1 (Medellín) | R-HQ-EDGE (ABR), R-MED | 10.20.0.0/16 |

- `passive-interface default` on all devices
- `default-information originate` on R-HQ-EDGE
- Router-IDs via loopback interfaces (1.1.1.1 – 4.4.4.4)

---

## HSRP Load Balancing

| VLANs | Active | Standby | Priority Active |
|---|---|---|---|
| 10, 20, 40, 60, 80 | DSW1-HQ | DSW2-HQ | 150 |
| 30, 50, 70, 90, 99 | DSW2-HQ | DSW1-HQ | 150 |

- HSRP version 2
- Preempt enabled on all groups
- Virtual IP = .1 of each subnet

---

## Security Policies

| Policy | Implementation |
|---|---|
| GUEST → internal networks | DENY (Extended ACL on VLAN 80) |
| GUEST → Internet | PERMIT |
| Any → MGMT VLAN | DENY (except IT) |
| IT → all segments | PERMIT |
| SSH access | Restricted to VLAN IT only (ACL on VTY lines) |
| Unauthorized switches | BPDU Guard on all access ports |
| MAC address control | Port-security sticky, max 2 MACs, violation shutdown |

---

## Device Inventory

| Device | Model | Role |
|---|---|---|
| DSW1-HQ | Cisco 3560-24PS | Distribution L3, HSRP Active (even VLANs) |
| DSW2-HQ | Cisco 3560-24PS | Distribution L3, HSRP Active (odd VLANs) |
| ASW1-HQ | Cisco 2960-24TT | Access — ADMIN, FIN, HR |
| ASW2-HQ | Cisco 2960-24TT | Access — ENG, SALES |
| ASW3-HQ | Cisco 2960-24TT | Access — OPS, IT, GUEST |
| ASW4-HQ | Cisco 2960-24TT | Access — SERVERS |
| R-HQ-EDGE | Cisco 2911 | Edge router, NAT/PAT, OSPF ABR |
| SRV-HQ | Server-PT | DHCP, DNS, NTP, Syslog |
| R-MED | Cisco 2911 | Branch gateway, OSPF Area 1 |
| SW-MED | Cisco 2960-24TT | Branch access switch |
| R-ISP | Cisco 2911 | Simulated ISP |

---

## Repository Structure

```
amg-network-lab/
│
├── README.md
├── v1-final/
│   ├── AMG-V1-FINAL-COMPLETO.pkt
│   └── configs/
│       ├── DSW1-HQ.txt
│       ├── DSW2-HQ.txt
│       ├── ASW1-HQ.txt
│       ├── ASW2-HQ.txt
│       ├── ASW3-HQ.txt
│       ├── ASW4-HQ.txt
│       ├── R-HQ-EDGE.txt
│       ├── R-MED.txt
│       ├── SW-MED.txt
│       └── R-ISP.txt
│
├── docs/
│   ├── ip-addressing-table.md
│   └── vlan-table.md
│
└── screenshots/
    ├── topology.png
    ├── ospf-neighbors.png
    ├── hsrp-standby.png
    ├── etherchannel-summary.png
    ├── nat-translations.png
    └── ping-tests.png
```

---

## Validation Results

| Test | Result |
|---|---|
| Inter-VLAN ping (ADMIN → FIN) | ✅ Success |
| HQ → Medellín ping | ✅ Success |
| Medellín → HQ servers | ✅ Success |
| Internet access (NAT/PAT) | ✅ Success |
| GUEST → internal network | ✅ Blocked by ACL |
| GUEST → Internet | ✅ Success |
| DHCP assignment all VLANs | ✅ Success |
| DHCP assignment Medellín | ✅ Success (via helper-address over WAN) |
| OSPF neighbors FULL state | ✅ 3 neighbors on R-HQ-EDGE |
| EtherChannel LACP | ✅ Po1(SU) Gi0/1(P) Gi0/2(P) |
| HSRP failover | ✅ Preempt active |
| SSH access from IT VLAN | ✅ Success |
| SSH blocked from other VLANs | ✅ Blocked by ACL |
| Port-security sticky | ✅ MAC learned and locked |

---

## Key Learnings

- Designed and implemented a **hierarchical campus network** from scratch
- Configured **OSPF multi-area** with ABR and inter-area route propagation (O IA routes)
- Implemented **HSRP load balancing** distributing active roles across VLANs
- Built **centralized DHCP** serving multiple VLANs and a remote branch via `ip helper-address` over WAN
- Applied **structured troubleshooting** methodology — diagnosing EtherChannel mismatches, OSPF adjacency failures, NAT interface conflicts, and ACL placement issues
- Understood the difference between **operational experience** (MPLS, service provisioning from Colvatel) and **hands-on CCNA configuration** (CLI-level routing and switching)

---

## Roadmap

- [x] Version 1 — HQ Bogotá + Medellín branch
- [ ] Version 2 — Add Cali branch (OSPF Area 2) + IPv6 dual stack + NTP/Syslog
- [ ] Version 3 — Full documentation, structured troubleshooting scenarios, production-change simulation

---

## Author

**Daniel Vargas Carpeta**  
Electronic Engineer | Junior Network Engineer  
2+ years operational experience in corporate telecommunications (MPLS, Carrier Network, Metro Network)  
Currently completing CCNA training — routing, switching, WAN, security  

📧 Daniel.vargasc02@outlook.com  
🔗 [GitHub](https://github.com/Dani08-art)  
🔗 LinkedIn: [add your LinkedIn URL]
