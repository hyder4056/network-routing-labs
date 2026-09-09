<p align="center">
  <img width="700" height="478" alt="image" src="https://github.com/user-attachments/assets/afce666e-a74e-4d2a-9c0a-07ae814bd5b0" />
</p>
# MikroTik OSPF Multi-Area Lab (IPv4 + IPv6)

This lab demonstrates a complete **OSPF Multi-Area** design using **MikroTik RouterOS v7**, running both **OSPFv2 (IPv4)** and **OSPFv3 (IPv6)** simultaneously in an ISP-style topology.

**Lab Goal:**  
Configure OSPF across multiple areas so that all Loopback and point-to-point networks (both IPv4 and IPv6) are fully reachable from every router.

---

## Topology

| Device         | Role              | OSPF Areas          | IPv4 Loopback    | IPv6 Loopback       |
|----------------|-------------------|---------------------|------------------|---------------------|
| Core-Router-1  | Backbone          | Area 0              | 172.25.0.1/32    | fc02:49c0::1/128    |
| Core-Router-2  | Backbone          | Area 0              | 172.25.0.2/32    | fc02:49c0::2/128    |
| AGG-Router-1   | ABR               | Area 0 + 1 + 2      | 172.25.0.3/32    | fc02:49c0::3/128    |
| AGG-Router-2   | ABR               | Area 0 + 1 + 2      | 172.25.0.4/32    | fc02:49c0::4/128    |
| PoP-Router-1   | Internal Router   | Area 1 only         | 172.25.0.5/32    | fc02:49c0::5/128    |
| PoP-Router-2   | Internal Router   | Area 2 only         | 172.25.0.6/32    | fc02:49c0::6/128    |

---

## Addressing Plan

### IPv4 Point-to-Point Links

| Link                        | Network          | Side A              | Side B              |
|----------------------------|------------------|---------------------|---------------------|
| Core-1 ↔ Core-2            | 172.25.1.0/30    | 172.25.1.1          | 172.25.1.2          |
| Core-1 ↔ AGG-1             | 172.25.1.4/30    | 172.25.1.5          | 172.25.1.6          |
| Core-1 ↔ AGG-2             | 172.25.1.12/30   | 172.25.1.13         | 172.25.1.14         |
| Core-2 ↔ AGG-1             | 172.25.1.16/30   | 172.25.1.17         | 172.25.1.18         |
| Core-2 ↔ AGG-2             | 172.25.1.8/30    | 172.25.1.9          | 172.25.1.10         |
| AGG-1 ↔ AGG-2              | 172.25.1.20/30   | 172.25.1.21         | 172.25.1.22         |
| AGG-1 ↔ PoP-1              | 172.25.1.24/30   | 172.25.1.25         | 172.25.1.26         |
| AGG-1 ↔ PoP-2              | 172.25.1.32/30   | 172.25.1.33         | 172.25.1.34         |
| AGG-2 ↔ PoP-1              | 172.25.1.36/30   | 172.25.1.37         | 172.25.1.38         |
| AGG-2 ↔ PoP-2              | 172.25.1.28/30   | 172.25.1.29         | 172.25.1.30         |

### IPv6 Addressing
- Loopbacks: `fc02:49c0::x/128`
- Point-to-point links use `fc02:49c0::/126`

---

## OSPF Design

### Instances
- `DEFAULT_V2` → OSPFv2 (IPv4)
- `DEFAULT_V3` → OSPFv3 (IPv6)

## Router ID Configuration Method

In this lab we used the **Router ID** object method (as shown in the video):

1. Go to **Routing → Router ID**
2. Create a named Router ID (example: `MAIN-RID` = `172.25.0.1`)
3. Then select this named Router ID inside the OSPF Instance

### Why this method instead of putting the ID directly in the Instance?

| Method                        | Advantage                                      | Disadvantage                          |
|-------------------------------|------------------------------------------------|---------------------------------------|
| Directly in OSPF Instance     | Faster for small labs                          | Hard to manage when many instances exist |
| Separate Router ID object     | Can be reused in OSPF, BGP, etc.               | One extra step                        |
|                               | Change ID in one place → updates everywhere    |                                       |
|                               | Cleaner and professional approach              |                                       |

### Areas
- **Area 0** → Backbone (Core + AGG)
- **Area 1** → PoP-1
- **Area 2** → PoP-2

### Interface Templates
| Interface Type       | Network Type | Passive |
|----------------------|--------------|---------|
| Loopback (`lo`)      | broadcast    | Yes     |
| Router-to-Router     | ptp          | No      |

---

---

## 🧪 Verification & Test Results

> **Status:** `ALL TESTS PASSED` (100% Reachability)

### 📡 Ping & Traceroute Verification

* **IPv4 End-to-End Connectivity**
  * `Core-Router-1` ➔ `PoP-Router-2`
    ```routeros
    ping 172.25.0.6 src-address=172.25.0.1 count=5
    # Result: sent=5 received=5 packet-loss=0%
    ```
  * `PoP-Router-2` ➔ `Core-Router-1`
    ```routeros
    ping 172.25.0.1 src-address=172.25.0.6 count=5
    # Result: sent=5 received=5 packet-loss=0%
    ```
  * **Traceroute:** Successful bi-directional trace complete within **2 hops**.

* **IPv6 Dual-Stack Reachability**
  * `Core-Router-1` ➔ `PoP-Router-2`: `ping fc02:49c0::6` *(5/5 received, 0% loss)*
  * `PoP-Router-2` ➔ `Core-Router-1`: `ping fc02:49c0::1` *(5/5 received, 0% loss)*

### 📊 Routing & Adjacency Status

| Parameter | Observed Status | Verification Summary |
| :--- | :---: | :--- |
| **Loopback Routes** | `172.25.0.1` – `172.25.0.6` | Active in Routing Table (OSPF Distance: `110`) |
| **OSPFv2 Neighbors** | `FULL` | All IPv4 adjacencies established |
| **OSPFv3 Neighbors** | `FULL` | All IPv6 adjacencies established |

---

## 📌 Lab Overview & Details

| Attribute | Configuration Details |
| :--- | :--- |
| **Lab Name** | Multi-Area OSPF Network Architecture |
| **Platform** | EVE-NG Emulation |
| **OSPF Areas** | `Area 0` (Backbone) • `Area 1` • `Area 2` (Stub/Edge) |
| **Addressing** | Dual-Stack IPv4 & IPv6 Architecture |

---

## 🔗 Credits & Reference

> **Acknowledgment:** This laboratory framework was constructed for technical research and practical implementation based on guidance by **Mohammad Belal Hossain**.
