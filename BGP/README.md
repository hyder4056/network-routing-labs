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
   1.1.1.1/24                            2.2.2.2/24
Network Overview
R1 (AS 1) -------- R0 (AS 2) -------- R2 (AS 3)
10.0.0.0/24                         20.0.0.0/24

R1 Loopback0                         R2 Loopback0
1.1.1.1/24                           2.2.2.2/24
📋 IP Addressing
Router	Interface	IP Address	Network	AS
R1	Fa0/0	10.0.0.1/24	10.0.0.0/24	1
R0	Fa0/0	10.0.0.2/24	10.0.0.0/24	2
R0	Fa1/0	20.0.0.1/24	20.0.0.0/24	2
R2	Fa0/0	20.0.0.2/24	20.0.0.0/24	3
R1	Loopback0	1.1.1.1/24	1.1.1.0/24	1
R2	Loopback0	2.2.2.2/24	2.2.2.0/24	3
🎯 Lab Objectives
Configure three Cisco routers with three different Autonomous Systems.
Establish eBGP peering between R1-R0 and R0-R2.
Advertise R1's Loopback network through BGP.
Advertise R2's Loopback network through BGP.
Understand BGP route propagation through a transit AS.
Observe AS_PATH changes as routes travel between Autonomous Systems.
Verify end-to-end connectivity using only BGP.
⚙️ BGP Design

The BGP relationships are:

R1 (AS 1) <---- eBGP ----> R0 (AS 2)
R0 (AS 2) <---- eBGP ----> R2 (AS 3)

R1 and R2 are not directly connected and do not form a BGP peering session with each other.

R0 acts as the transit router between AS 1 and AS 3.

🔧 Configuration
R1 — AS 1
interface FastEthernet0/0
 ip address 10.0.0.1 255.255.255.0
 no shutdown

interface Loopback0
 ip address 1.1.1.1 255.255.255.0

router bgp 1
 neighbor 10.0.0.2 remote-as 2
 network 1.1.1.0 mask 255.255.255.0
R0 — AS 2
interface FastEthernet0/0
 ip address 10.0.0.2 255.255.255.0
 no shutdown

interface FastEthernet1/0
 ip address 20.0.0.1 255.255.255.0
 no shutdown

router bgp 2
 neighbor 10.0.0.1 remote-as 1
 neighbor 20.0.0.2 remote-as 3

R0 does not need to originate a Loopback network in this lab.

Its primary role is to learn routes from both eBGP neighbors and propagate them between AS 1 and AS 3.

R2 — AS 3
interface FastEthernet0/0
 ip address 20.0.0.2 255.255.255.0
 no shutdown

interface Loopback0
 ip address 2.2.2.2 255.255.255.0

router bgp 3
 neighbor 20.0.0.1 remote-as 2
 network 2.2.2.0 mask 255.255.255.0
🔍 Verification
1. Verify BGP Neighbors

Run on all routers:

show ip bgp summary

Expected BGP relationships:

R1 ---- AS 2 ---- R0
R0 ------------- AS 3 ---- R2

The State/PfxRcd column should show a number rather than Idle, Active, Connect, etc.

2. Verify BGP Routing Table
show ip bgp
R0 should learn both networks:
1.1.1.0/24
2.2.2.0/24

**Example:**

Network          Next Hop       Path

*> 1.1.1.0/24    10.0.0.1       1 i
*> 2.2.2.0/24    20.0.0.2       3 i
3. R2 Learning R1's Network

R2 should learn:

1.1.1.0/24

with an AS_PATH similar to:

2 1

This represents:

AS 1 → AS 2 → AS 3

From R2's perspective, the path back toward the originating AS is:

2 1
4. R1 Learning R2's Network

R1 should learn:

2.2.2.0/24

with an AS_PATH similar to:

2 3

This represents:

AS 3 → AS 2 → AS 1
🛣️ Route Propagation
R1 Loopback → R2
1.1.1.0/24
      |
      v
    R1
   AS 1
      |
     eBGP
      |
      v
    R0
   AS 2
      |
     eBGP
      |
      v
    R2
   AS 3

R2 eventually learns:

1.1.1.0/24 via 20.0.0.1
R2 Loopback → R1
2.2.2.0/24
      |
      v
    R2
   AS 3
      |
     eBGP
      |
      v
    R0
   AS 2
      |
     eBGP
      |
      v
    R1
   AS 1

R1 eventually learns:

2.2.2.0/24 via 10.0.0.2
🧪 Connectivity Test

From R1:

ping 2.2.2.2

Expected:

!!!!!
Success rate is 100 percent

From R2:

ping 1.1.1.1

Expected:

!!!!!
Success rate is 100 percent
🔥 Key Learning Points
1. BGP can propagate routes between different ASes
AS 1 → AS 2 → AS 3

R0 acts as a transit AS router.

2. R1 and R2 don't need a direct physical connection

They can exchange routing information through R0:

R1 ←→ R0 ←→ R2
3. No IGP is required for the end-to-end route exchange in this lab

This lab intentionally avoids:

❌ Static Routing
❌ OSPF
❌ EIGRP
❌ RIP

and uses:

✅ eBGP

for the Loopback prefixes.

4. AS_PATH records the ASes a route has traversed

For example:

R2 receives:

1.1.1.0/24
AS_PATH = 2 1

This tells R2 that the prefix originated in AS 1 and was advertised through AS 2.

5. BGP neighbor establishment and route exchange are different

A BGP session can be:

Established

while:

PfxRcd = 0

This means the BGP session is working but no prefixes have been received yet.

📚 Commands Used
show ip bgp summary
show ip bgp
show ip route
ping <destination>
✅ Final Result

The lab successfully demonstrates:

          eBGP                eBGP
     AS 1       AS 2          AS 3

     R1 -------- R0 -------- R2
      |                        |
  1.1.1.1                  2.2.2.2
      |                        |
      +------ BGP Reachability +

R1 and R2 can reach each other's Loopback networks through the BGP transit router R0, without using Static Routing or an IGP.

💡 What This Lab Demonstrates

BGP is not only used to establish neighbor relationships; it can also propagate prefixes across multiple Autonomous Systems through intermediate BGP routers.
