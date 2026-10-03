# CCNA-Labs - Enterprise Network Projects
by Muhammad Faizan | Telecom / Network Engineer

### 🚀 Latest Lab: Enterprise Network Lab (Oct 2026)
Designed a real-world ISP style topology in Cisco Packet Tracer 9.0.1 - 18 PCs, 6 Routers, 4 Switches with Central FTP Server.

**Topology Details:**
- Core Router: Router0 connecting 5 branch networks (192.168.2.0 - 192.168.8.0)
- Central Services: FTP Server 192.168.100.2 & SSH Remote Access

**What I Tested:**
1. Static Routing between all branches - Full connectivity
2. FTP Service - File `telecom.txt` (14 bytes) downloaded successfully on PC8
3. SSH Secure Login - `ssh -l faiz 192.168.3.1`
4. Next: ACL to block FTP for specific network & allow only via VPN

**Files in this Repo:**
- Enterprise-Network-lab-Faiz.pkt - Main Enterprise Lab (Latest)
- OSPF_4_Area_Lab.md - OSPF Multi-Area Configuration

**Tools:** Cisco Packet Tracer 9.0.1, CLI, FTP, SSH, ACL, Static Routing

🔗 Open to feedback and collaboration!

---
### 🚀 Latest Lab: Multi-ISP Load Balancing & Failover (Oct 2026)

**Goal:** Ensure network stays up even if one ISP fails - ISP Redundancy

**Setup:**
- 3 ISP Routers (Router3, Router9, Router10) + ISP Switch7
- Border Router8(1) as Load Balancer (max-paths 3)
- Servers: Web/DNS (192.168.100.100) + FTP Server (192.168.100.19)
- DHCP: Router4 for Branch 192.168.1.0/24

**Routing:** OSPF Area 0 - ECMP Load Balancing

**Result:**
- All Links UP: Ping 192.168.100.100 = 0% loss TTL=123
- One Link DOWN: Ping = 0% loss TTL=124 - Failover Successful!

**File:** `Multi-isp-load-balancing.pkt`
