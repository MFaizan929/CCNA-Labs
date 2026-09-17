# OSPF Multi-Area Lab (4 Areas, No Subnetting, Serial Links)

## 1. Why a Virtual Link Is Needed

OSPF's core rule: **every area must have a direct connection to Area 0 (the backbone)**.

Your design:
- Area 1 → connects to Area 0 ✅ (direct)
- Area 4 → connects to Area 0 ✅ (direct)
- Area 2 → connects to Area 1 ❌ (NOT directly connected to Area 0)

Because Area 2 only touches Area 1, it violates the backbone rule. To fix this without redesigning the physical topology, we configure a **virtual link** between the two ABRs (R2 and R3) using Area 1 as the "transit area." This logically extends the backbone through Area 1 so Area 2 gets backbone connectivity.

## 2. Topology

```
        Area 4                Area 0                Area 1                Area 2
 R5 ───────────── R1 ───────────── R2 ───────────── R3 ───────────── R4
 (Se0/0/0)   (Se0/0/1)(Se0/0/0)   (Se0/0/1)(Se0/0/0)  (Se0/0/1)(Se0/0/0)

 Virtual Link: R2 <---(across Area 1)---> R3
```

- **R1** = ABR between Area 0 and Area 4
- **R2** = ABR between Area 0 and Area 1 (virtual-link endpoint)
- **R3** = ABR between Area 1 and Area 2 (virtual-link endpoint)
- **R4** = internal router, Area 2
- **R5** = internal router, Area 4

## 3. IP Addressing (No Subnetting — Full Classful Networks per Link)

| Link | Area | Network | R-side A | R-side B |
|---|---|---|---|---|
| R1 – R2 | 0 | 10.0.0.0 /8 | R1: 10.0.0.1 | R2: 10.0.0.2 |
| R2 – R3 | 1 | 20.0.0.0 /8 | R2: 20.0.0.1 | R3: 20.0.0.2 |
| R3 – R4 | 2 | 30.0.0.0 /8 | R3: 30.0.0.1 | R4: 30.0.0.2 |
| R1 – R5 | 4 | 40.0.0.0 /8 | R1: 40.0.0.1 | R5: 40.0.0.2 |

Loopbacks (used as OSPF Router-ID, mask 255.255.255.255):

| Router | Loopback |
|---|---|
| R1 | 1.1.1.1 |
| R2 | 2.2.2.2 |
| R3 | 3.3.3.3 |
| R4 | 4.4.4.4 |
| R5 | 5.5.5.5 |

> **DCE/DTE note:** On each serial link, whichever end has the DCE cable end needs `clock rate 64000`. Check with `show controllers serial X` on each router. In the configs below I've assumed R1, R2, R3 hold the DCE side of their second link outward — adjust to match your actual cabling.

---

## 4. Router Configurations

### R1 (ABR: Area 0 / Area 4)
```
hostname R1
!
interface Loopback0
 ip address 1.1.1.1 255.255.255.255
!
interface Serial0/0/0
 description Link to R2 - Area 0
 ip address 10.0.0.1 255.0.0.0
 no shutdown
!
interface Serial0/0/1
 description Link to R5 - Area 4
 ip address 40.0.0.1 255.0.0.0
 clock rate 64000
 no shutdown
!
router ospf 1
 router-id 1.1.1.1
 network 1.1.1.1 0.0.0.0 area 0
 network 10.0.0.0 0.255.255.255 area 0
 network 40.0.0.0 0.255.255.255 area 4
```

### R2 (ABR: Area 0 / Area 1 — Virtual Link endpoint)
```
hostname R2
!
interface Loopback0
 ip address 2.2.2.2 255.255.255.255
!
interface Serial0/0/0
 description Link to R1 - Area 0
 ip address 10.0.0.2 255.0.0.0
 clock rate 64000
 no shutdown
!
interface Serial0/0/1
 description Link to R3 - Area 1
 ip address 20.0.0.1 255.0.0.0
 clock rate 64000
 no shutdown
!
router ospf 1
 router-id 2.2.2.2
 network 2.2.2.2 0.0.0.0 area 0
 network 10.0.0.0 0.255.255.255 area 0
 network 20.0.0.0 0.255.255.255 area 1
 area 1 virtual-link 3.3.3.3
```

### R3 (ABR: Area 1 / Area 2 — Virtual Link endpoint)
```
hostname R3
!
interface Loopback0
 ip address 3.3.3.3 255.255.255.255
!
interface Serial0/0/0
 description Link to R2 - Area 1
 ip address 20.0.0.2 255.0.0.0
 no shutdown
!
interface Serial0/0/1
 description Link to R4 - Area 2
 ip address 30.0.0.1 255.0.0.0
 clock rate 64000
 no shutdown
!
router ospf 1
 router-id 3.3.3.3
 network 3.3.3.3 0.0.0.0 area 1
 network 20.0.0.0 0.255.255.255 area 1
 network 30.0.0.0 0.255.255.255 area 2
 area 1 virtual-link 2.2.2.2
```

### R4 (Internal router, Area 2)
```
hostname R4
!
interface Loopback0
 ip address 4.4.4.4 255.255.255.255
!
interface Serial0/0/0
 description Link to R3 - Area 2
 ip address 30.0.0.2 255.0.0.0
 no shutdown
!
router ospf 1
 router-id 4.4.4.4
 network 4.4.4.4 0.0.0.0 area 2
 network 30.0.0.0 0.255.255.255 area 2
```

### R5 (Internal router, Area 4)
```
hostname R5
!
interface Loopback0
 ip address 5.5.5.5 255.255.255.255
!
interface Serial0/0/0
 description Link to R1 - Area 4
 ip address 40.0.0.2 255.0.0.0
 no shutdown
!
router ospf 1
 router-id 5.5.5.5
 network 5.5.5.5 0.0.0.0 area 4
 network 40.0.0.0 0.255.255.255 area 4
```

---

## 5. Verification Commands (run on any/all routers)

```
show ip ospf neighbor
show ip protocols
show ip ospf interface brief
show ip route ospf
show ip ospf virtual-links        ! run on R2 and R3
show ip ospf border-routers
show ip ospf database
```

Expected results once everything is up:
- `show ip ospf neighbor` on each router shows FULL state with its directly connected neighbor(s).
- `show ip ospf virtual-links` on **R2** and **R3** shows the virtual link state as **UP**.
- `show ip route ospf` on R4 and R5 should each show routes to *every other* network/loopback in the topology (proof that Area 2 has full backbone reachability via the virtual link).
- End-to-end ping test: `ping 5.5.5.5` from R4 (Area 2 → Area 4, crossing Area 0) should succeed.

## 6. Common Pitfalls to Check

- If `show ip ospf virtual-links` shows **DOWN**, the transit area (Area 1) itself isn't fully adjacent yet — check R2↔R3 OSPF neighbor state first.
- Router-IDs in the `area 1 virtual-link X.X.X.X` command must match the **OSPF router-id** of the far-end router, not its interface IP.
- No `clock rate` on the DCE end = interface stays down/down — always confirm with `show controllers`.
- Since there's no subnetting, double-check no two links accidentally reuse the same major network (each link above uses a distinct /8 network to avoid overlap).
