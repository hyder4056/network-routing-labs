# MikroTik OSPF Multi-Area Lab (ISP Style)

This lab shows OSPF Multi-Area configuration using MikroTik RouterOS v7.

## Topology Overview

- Core Routers → Area 0 (Backbone)
- Aggregation Routers → ABR (Area 0 + Area 1/2)
- PoP Routers → Only their own Area

## Devices

| Device        | Role     | Area        | Loopback      |
|---------------|----------|-------------|---------------|
| Core-Router-1 | Backbone | Area 0      | 172.25.0.1/32 |
| Core-Router-2 | Backbone | Area 0      | 172.25.0.2/32 |
| Agg-Router-1  | ABR      | Area 0 + 1  | 172.25.0.3/32 |
| Agg-Router-2  | ABR      | Area 0 + 2  | 172.25.0.4/32 |
| PoP-Router-1  | Internal | Area 1 only | 172.25.0.5/32 |
| PoP-Router-2  | Internal | Area 2 only | 172.25.0.6/32 |

## Important Lesson

PoP routers must have **only their own Area**.

- PoP-1 → Only Area 1
- PoP-2 → Only Area 2

If you create both Area 1 and Area 2 on PoP routers, Neighbor becomes Full but routes do not install.

## How to Verify

```bash
/routing ospf neighbor print
/ip route print where protocol=ospf
