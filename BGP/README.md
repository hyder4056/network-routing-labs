**3-Router eBGP Route Propagation Lab**
**A Cisco networking lab demonstrating eBGP route propagation across three Autonomous Systems (AS) using three routers.**

The main objective of this lab is to understand how BGP can propagate routes through a transit Autonomous System and provide end-to-end connectivity between Loopback interfaces without using Static Routing, OSPF, EIGRP, or RIP.

🗺️ Topology
                eBGP                  eBGP
        AS 1             AS 2             AS 3

      +------+          +------+          +------+
      |  R1  |----------|  R0  |----------|  R2  |
      +------+          +------+          +------+
         |                                    |
      Lo0                                  Lo0

---

## 🗺️ Network Topology Overview

```text
 1.1.1.1/24                                                            2.2.2.2/24
┌───────────┐            10.0.0.0/24            ┌───────────┐            20.0.0.0/24            ┌───────────┐
│ R1 (AS 1) ├───────────────────────────────────┤ R0 (AS 2) ├───────────────────────────────────┤ R2 (AS 3) │
└───────────┘                                   └───────────┘                                   └───────────┘
📋 IP Addressing TableDeviceInterfaceIP AddressSubnet MaskNetwork AddressAutonomous SystemR1Fa0/010.0.0.1255.255.255.010.0.0.0/24AS 1R0Fa0/010.0.0.2255.255.255.010.0.0.0/24AS 2R0Fa1/020.0.0.1255.255.255.020.0.0.0/24AS 2R2Fa0/020.0.0.2255.255.255.020.0.0.0/24AS 3R1Loopback01.1.1.1255.255.255.01.1.1.0/24AS 1R2Loopback02.2.2.2255.255.255.02.2.2.0/24AS 3🎯 Lab Objectives[x] Configure three Cisco routers across three distinct Autonomous Systems (AS 1, AS 2, AS 3).[x] Establish external BGP (eBGP) peering sessions between R1-R0 and R0-R2.[x] Advertise R1's Loopback network (1.1.1.0/24) into BGP.[x] Advertise R2's Loopback network (2.2.2.0/24) into BGP.[x] Understand BGP route propagation through a Transit Autonomous System (AS 2).[x] Observe AS_PATH changes as routes travel between Autonomous Systems.[x] Verify end-to-end connectivity using BGP exclusively (without Static Routes or IGPs).⚙️ BGP Design & ArchitecturePlaintextR1 (AS 1) <---- eBGP Peering ----> R0 (AS 2) <---- eBGP Peering ----> R2 (AS 3)
Architecture Notes:No Direct Peering: R1 and R2 are not directly connected and do not form a BGP peering session with each other.Transit Role: R0 acts as the transit router between AS 1 and AS 3.🔧 Router Configurations🔹 R1 — AS 1Cisco CLIinterface FastEthernet0/0
 ip address 10.0.0.1 255.255.255.0
 no shutdown

interface Loopback0
 ip address 1.1.1.1 255.255.255.0

router bgp 1
 neighbor 10.0.0.2 remote-as 2
 network 1.1.1.0 mask 255.255.255.0
🔹 R0 — AS 2 (Transit Router)Cisco CLIinterface FastEthernet0/0
 ip address 10.0.0.2 255.255.255.0
 no shutdown

interface FastEthernet1/0
 ip address 20.0.0.1 255.255.255.0
 no shutdown

router bgp 2
 neighbor 10.0.0.1 remote-as 1
 neighbor 20.0.0.2 remote-as 3
Configuration Note: R0 does not need to originate a Loopback network in this lab. Its primary role is to learn routes from both eBGP neighbors and propagate them between AS 1 and AS 3.🔹 R2 — AS 3Cisco CLIinterface FastEthernet0/0
 ip address 20.0.0.2 255.255.255.0
 no shutdown

interface Loopback0
 ip address 2.2.2.2 255.255.255.0

router bgp 3
 neighbor 20.0.0.1 remote-as 2
 network 2.2.2.0 mask 255.255.255.0
🔍 Verification & Route Analysis1. Verify BGP NeighborsRun on all routers:Bashshow ip bgp summary
Expected BGP relationships:PlaintextR1 (AS 1) ◄── eBGP ──► R0 (AS 2) ◄── eBGP ──► R2 (AS 3)
Note: The State/PfxRcd column should show a number rather than states like Idle, Active, or Connect.2. Verify BGP Routing TableRun on R0:Bashshow ip bgp
R0 should learn both networks (1.1.1.0/24 and 2.2.2.0/24):Plaintext   Network          Next Hop            Metric LocPrf Weight Path
*> 1.1.1.0/24       10.0.0.1                               0 1 i
*> 2.2.2.0/24       20.0.0.2                               0 3 i
3. R2 Learning R1's NetworkLearned Network: 1.1.1.0/24Observed AS_PATH: 2 1Path Sequence: AS 1 → AS 2 → AS 3Path Traversal: From R2's perspective, the path back toward the originating AS is 2 1.4. R1 Learning R2's NetworkLearned Network: 2.2.2.0/24Observed AS_PATH: 2 3Path Sequence: AS 3 → AS 2 → AS 1🛣️ Route Propagation Flow📤 R1 Loopback → R2 (1.1.1.0/24)Plaintext  1.1.1.0/24 (Originated in AS 1)
      │
      ▼
  R1 (AS 1)
      │  eBGP
      ▼
  R0 (AS 2)
      │  eBGP
      ▼
  R2 (AS 3) ──► Learns 1.1.1.0/24 via 20.0.0.1 (AS_PATH: 2 1)
📥 R2 Loopback → R1 (2.2.2.0/24)Plaintext  2.2.2.0/24 (Originated in AS 3)
      │
      ▼
  R2 (AS 3)
      │  eBGP
      ▼
  R0 (AS 2)
      │  eBGP
      ▼
  R1 (AS 1) ──► Learns 2.2.2.0/24 via 10.0.0.2 (AS_PATH: 2 3)
🧪 Connectivity TestsFrom R1:Bashping 2.2.2.2
Expected Output:Plaintext!!!!! Success rate is 100 percent
From R2:Bashping 1.1.1.1
Expected Output:Plaintext!!!!! Success rate is 100 percent
🔥 Key Learning PointsRoute Propagation across AS Boundaries:BGP can propagate routes between different ASes: AS 1 → AS 2 → AS 3.R0 acts as a transit AS router.No Direct Connection Required:R1 and R2 don't need a direct physical connection.They can exchange routing information through R0: R1 ←→ R0 ←→ R2.No IGP Required:No IGP is required for the end-to-end route exchange in this lab.Excluded Protocols: ❌ Static Routing | ❌ OSPF | ❌ EIGRP | ❌ RIPUsed Protocol: ✅ eBGP (for Loopback prefixes)AS_PATH Tracking:AS_PATH records the ASes a route has traversed.Example: R2 receives 1.1.1.0/24 with AS_PATH = 2 1.This tells R2 that the prefix originated in AS 1 and was advertised through AS 2.Neighbor Establishment vs. Prefix Exchange:BGP neighbor establishment and route exchange are different.A BGP session can be in the Established state while PfxRcd = 0.This means the BGP session is working, but no prefixes have been received yet.📚 Commands UsedBashshow ip bgp summary
show ip bgp
show ip route
ping
✅ Final ResultPlaintext      eBGP                  eBGP
  AS 1        AS 2          AS 3

  R1 --------- R0 --------- R2
   |                         |
1.1.1.1                   2.2.2.2
   |                         |
   +-- BGP Reachability -----+
R1 and R2 can reach each other's Loopback networks through the BGP transit router R0, without using Static Routing or an IGP.💡 What This Lab DemonstratesBGP is not only used to establish neighbor relationships; it can also propagate prefixes across multiple Autonomous Systems through intermediate BGP routers.
