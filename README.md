# Static Routing + EIGRP Case Study — Cisco Packet Tracer

## Student Details

**Student Name:** Akhila Anish Das
**Roll No:** 150096725016
**Platform:** Cisco Packet Tracer
**Routing Protocol:** EIGRP
**EIGRP Autonomous System:** 100

---

# TechNova University — Static Routing & EIGRP Case Study

## 1. Project Overview

This project implements and troubleshoots a university network using a combination of **EIGRP and static routing** in Cisco Packet Tracer.

The network consists of:

* **R1 — Main Campus**
* **R2 — Research Center**
* **R3 — Remote Training Center**
* **ISP**
* Multiple LANs, switches, and end devices

The objective was to reconstruct the network, configure the required routing, verify connectivity, troubleshoot failures, and explain how packets travel through the network.

The final network uses **EIGRP AS 100 for internal routing** and **static routing for the ISP/default route and management network**, as required by the case study. 

---

# 2. Network Topology

The topology contains three routers connected through serial links:

```text
                 ISP
            203.0.113.1/30
                   |
                   |
                  R1
             Main Campus
              /       \
             /         \
     192.168.10.0   192.168.11.0
             |
          Serial
        10.0.12.0/30
             |
             R2
       Research Center
          /       \
         /         \
192.168.20.0   192.168.21.0
             |
          Serial
        10.0.23.0/30
             |
             R3
      Training Center
          /       \
         /         \
192.168.30.0   192.168.31.0
```

A management network **192.168.99.0/24** was additionally connected behind R3 during Mission 5.

---

# 3. IP Addressing Plan

| Device | Interface       | IP Address      |
| ------ | --------------- | --------------- |
| R1     | G0/0            | 192.168.10.1/24 |
| R1     | G0/1            | 192.168.11.1/24 |
| R1     | S0/0/0          | 10.0.12.1/30    |
| R1     | S0/0/1          | 203.0.113.2/30  |
| R2     | G0/0            | 192.168.20.1/24 |
| R2     | G0/1            | 192.168.21.1/24 |
| R2     | S0/0/0          | 10.0.12.2/30    |
| R2     | S0/0/1          | 10.0.23.1/30    |
| R3     | G0/0            | 192.168.30.1/24 |
| R3     | G0/1            | 192.168.31.1/24 |
| R3     | S0/0/0          | 10.0.23.2/30    |
| ISP    | Interface to R1 | 203.0.113.1/30  |
| R3     | G0/2            | 192.168.99.1/24 |

The addressing plan follows the assignment's specified networks and interfaces. 

---

# 4. Mission 1 — Build the Network

## Objective

* Build the complete topology.
* Configure router interfaces and end devices.
* Verify directly connected networks.
* Use `show ip route`.
* Understand why remote LANs are initially unreachable.

## Verification

The command used was:

```text
show ip route
```

Before routing was configured, each router knew only its directly connected networks.

For example, R1 knew:

```text
192.168.10.0/24
192.168.11.0/24
10.0.12.0/30
203.0.113.0/30
```

but did not initially know the remote LANs behind R2 and R3.

## Investigation Answer

**Can a PC in 192.168.10.0/24 communicate with a PC in 192.168.20.0/24 before routing is configured?**

**Answer:** No.

The two PCs belong to different networks. R1 initially has no route to `192.168.20.0/24`, so it cannot forward the packet toward R2.

A routing protocol or static route is required before communication between the remote networks can occur.

---

# 5. Mission 2 — Configure EIGRP

## Objective

EIGRP was configured on R1, R2, and R3 using:

```text
EIGRP AS 100
```

The internal LANs and inter-router networks were advertised through EIGRP. The assignment specifically requires EIGRP AS 100 and verification using EIGRP neighbors, routing tables, and routing protocols. 

## Verification Commands

```text
show ip eigrp neighbors
```

```text
show ip route
```

```text
show ip protocols
```

## EIGRP Neighbor Verification

R1 successfully formed an EIGRP adjacency with R2:

```text
IP-EIGRP neighbors for process 100

Address       Interface
10.0.12.2     Se0/0/0
```

This confirmed that EIGRP was operating correctly between the routers.

## Learned Routes

The routing table displayed EIGRP routes using the `D` code.

Example:

```text
D 192.168.20.0/24 via 10.0.12.2
D 192.168.21.0/24 via 10.0.12.2
D 192.168.30.0/24 via 10.0.12.2
D 192.168.31.0/24 via 10.0.12.2
```

This confirmed that the routers were successfully learning remote networks through EIGRP.

---

# 6. Mission 3 — The Missing Internet Route

## Problem

The ISP does not participate in EIGRP.

Therefore, R1 needed a static default route toward the ISP:

```text
S* 0.0.0.0/0 via 203.0.113.1
```

The assignment asks whether R2 automatically learns the ISP path simply because R1 knows it. 

## Answer

No.

R1 knowing the route does not automatically mean R2 and R3 know it.

The default route must be propagated through EIGRP so that the other routers can use it.

---

# 7. Mission 4 — The Mystery Failure

## Problem

A Training Center user could access the Research Center but could not access the Internet.

The troubleshooting process was performed before changing configurations.

## Commands Used

```text
show ip route
```

```text
show ip eigrp neighbors
```

```text
show ip protocols
```

```text
ping
```

```text
traceroute
```

These are the troubleshooting tools specified by the assignment. 

## Diagnosis

Internal communication was working because EIGRP was successfully exchanging the internal routes.

The problem was the Internet/default route.

R1 had:

```text
S* 0.0.0.0/0 via 203.0.113.1
```

The static default route was then redistributed into EIGRP.

## Configuration

```text
router eigrp 100
redistribute static metric 10000 100 255 1 1500
```

After redistribution, the other routers learned the default route as an EIGRP external route.

Example:

```text
D*EX 0.0.0.0/0 [170/2707456] via 10.0.23.1
```

## Traceroute Verification

A traceroute from the Training Center showed the internal path:

```text
1  192.168.30.1
2  10.0.23.1
3  10.0.12.1
```

This helped identify the point where Internet connectivity stopped.

## Conclusion

The missing Internet route was identified and the static default route was successfully propagated through EIGRP.

---

# 8. Mission 5 — Static Route Challenge

## Objective

A backup management network was added behind R3:

```text
192.168.99.0/24
```

The administrator required that this network **not be advertised through EIGRP**.

Therefore, R1 needed a static route to reach the management network. This requirement is explicitly specified in the assignment. 

## Management Network

R3 interface:

```text
192.168.99.1/24
```

The interface was configured and verified as operational.

Verification:

```text
show ip interface brief
```

Result:

```text
GigabitEthernet0/2
192.168.99.1
up
up
```

## Local Connectivity Test

A PC successfully tested the R3 management interface:

```text
ping 192.168.99.1
```

Result:

```text
Packets: Sent = 4
Received = 4
Lost = 0
```

This confirmed connectivity to the management network.

## Remote Verification

Connectivity to the existing LANs was also tested.

Successful examples included:

```text
ping 192.168.20.1
```

```text
ping 192.168.21.1
```

```text
ping 192.168.30.1
```

```text
ping 192.168.31.1
```

These tests confirmed that the EIGRP-learned internal routes remained functional.

---

# 9. Mission 6 — Routing Table Detective

The routing table was analyzed to understand the meaning of different route codes and values.

## What does `D` mean?

`D` represents an **EIGRP route**.

Example:

```text
D 192.168.30.0/24
```

## What does `[90/...]` represent?

The first value, `90`, is the **administrative distance of an internal EIGRP route**.

The second value is the EIGRP metric.

## Why is the default route marked `S*`?

```text
S* 0.0.0.0/0
```

`S` means static and `*` indicates that it is a candidate default route.

## What is the next hop for `192.168.30.0/24`?

From R1:

```text
via 10.0.12.2
```

Therefore, R2 is the next hop from R1.

## Why does R1 not need a direct connection to R3?

R1 can forward the packet to R2, and R2 can forward it toward R3 using the EIGRP routing information.

## Which route is used for `192.168.30.50`?

R1 matches the destination with:

```text
192.168.30.0/24
```

and forwards the packet through the EIGRP-learned route via:

```text
10.0.12.2
```

The assignment specifically asks these routing-table questions as part of Mission 6. 

---

# 10. Mission 7 — Break It and Troubleshoot

After the network was working, faults were intentionally introduced and investigated.

The assignment requires three injected faults, with examples including:

* Shutting down an interface
* Changing an EIGRP network statement
* Configuring an incorrect next hop in a static route

The purpose is to identify and repair the faults using troubleshooting commands. 

## Troubleshooting Approach

The following commands were useful:

```text
show ip interface brief
```

```text
show ip route
```

```text
show ip eigrp neighbors
```

```text
show ip protocols
```

```text
ping
```

```text
traceroute
```

After identifying the faults, the affected configurations were corrected and connectivity was verified again.

---

# 11. Mission 8 — Final Verification

The final network was verified using routing tables, EIGRP information, interface status, and connectivity tests.

## Interface Verification

```text
show ip interface brief
```

The required interfaces were verified as:

```text
up/up
```

including the management interface:

```text
GigabitEthernet0/2
192.168.99.1
up
up
```

## EIGRP Verification

```text
show ip eigrp neighbors
```

The EIGRP neighbor relationship was successfully established.

Example:

```text
10.0.12.2
Se0/0/0
```

## Routing Table Verification

```text
show ip route
```

The routing table contained:

* `C` — Connected routes
* `L` — Local routes
* `D` — EIGRP routes
* `S*` — Static default route
* `D*EX` — EIGRP external default route

## Connectivity Verification

Successful tests included:

```text
ping 192.168.20.1
```

```text
ping 192.168.21.1
```

```text
ping 192.168.30.1
```

```text
ping 192.168.31.1
```

and:

```text
ping 192.168.99.1
```

The successful ping results demonstrated that the required internal and management connectivity was operational.

---

# 12. Packet Journey — 192.168.10.10 to 192.168.30.10

When PC `192.168.10.10` sends a packet to PC `192.168.30.10`, the packet first uses its default gateway, **R1 at 192.168.10.1**.

R1 performs a routing-table lookup for the destination:

```text
192.168.30.10
```

R1 has an EIGRP-learned route for:

```text
192.168.30.0/24
```

with next hop:

```text
10.0.12.2
```

Therefore, R1 forwards the packet to R2.

R2 uses its EIGRP routing information to determine the path toward R3 through:

```text
10.0.23.2
```

R3 receives the packet and sees that:

```text
192.168.30.0/24
```

is directly connected to its `GigabitEthernet0/0`.

R3 then delivers the packet to:

```text
192.168.30.10
```

The assignment specifically requires the packet journey explanation to include the default gateway, routing-table lookup, EIGRP routes, next hops, and final delivery by R3. 

---

# 13. What Happens If the R2–R3 EIGRP Adjacency Fails?

If the EIGRP adjacency between R2 and R3 fails, R2 will eventually remove the EIGRP-learned routes that depend on R3.

For example, routes toward:

```text
192.168.30.0/24
192.168.31.0/24
```

may disappear from R2's routing table if no alternative route exists.

As a result, traffic from the Main Campus or Research Center toward the Training Center will no longer be successfully forwarded.

The assignment specifically asks students to explain this scenario. 

---

# 14. Important EIGRP Route Codes

| Code | Meaning                      |
| ---- | ---------------------------- |
| C    | Connected                    |
| L    | Local                        |
| D    | EIGRP                        |
| D*EX | EIGRP external default route |
| S    | Static                       |
| S*   | Static default route         |

---

# 15. Important Verification Commands

```text
show ip interface brief
```

Used to verify interface IP addresses and whether interfaces are `up/up`.

```text
show ip route
```

Used to examine connected, static, and EIGRP routes.

```text
show ip eigrp neighbors
```

Used to verify EIGRP neighbor relationships.

```text
show ip protocols
```

Used to verify EIGRP configuration and advertised networks.

```text
ping <destination-ip>
```

Used to test end-to-end connectivity.

```text
traceroute <destination-ip>
```

Used to identify the path taken by packets and locate possible points of failure.

---

# 16. Final Network Design

The completed design uses two routing methods:

### EIGRP

Used for:

* Internal LANs
* R1–R2 routing
* R2–R3 routing
* Dynamic exchange of internal routes

### Static Routing

Used for:

* R1's default route toward the ISP
* Management network routing where explicitly required

This combination follows the case-study requirement that EIGRP should not be configured everywhere and that the final network should use both EIGRP and static routing. 

---

# 17. Learning Outcomes

Through this activity, I learned:

* How to build a multi-router network in Cisco Packet Tracer.
* How to configure router interfaces and IP addresses.
* How directly connected routes appear in a routing table.
* Why remote networks are unreachable without routing information.
* How to configure EIGRP using AS 100.
* How to verify EIGRP neighbors.
* How to interpret EIGRP routes in `show ip route`.
* How static default routes work.
* How to redistribute a static route into EIGRP.
* How to configure a static route for a network that should not be advertised through EIGRP.
* How to use `ping` and `traceroute` for troubleshooting.
* How to analyze routing-table entries.
* How to troubleshoot network failures.
* How packets travel across multiple routers using next-hop information.

---

# 18. Student Deliverables

The completed project contains the items required by the case study:

* Completed Cisco Packet Tracer topology
* Router configurations for R1, R2, and R3
* Interface verification
* Routing-table verification
* EIGRP neighbor verification
* Routing-protocol verification
* Troubleshooting evidence
* Packet-journey explanation
* Exploration-question answers

These correspond to the assignment's listed student deliverables. 

---

# 19. Conclusion

The **TechNova University Static Routing & EIGRP Case Study** was successfully implemented in Cisco Packet Tracer.

The network was first constructed and verified using directly connected routes. EIGRP **AS 100** was then configured to provide dynamic routing between the internal networks. The ISP was handled using a static default route, which was redistributed into EIGRP so that the other routers could learn a usable default path.

A separate management network was also configured behind R3 and handled using static routing as required. Finally, routing tables, EIGRP neighbors, interfaces, `ping`, and `traceroute` were used to verify and troubleshoot the network.

The completed activity demonstrates the practical use of **connected routes, EIGRP, static routes, default routes, route redistribution, next-hop routing, and network troubleshooting**.

**Project Status: COMPLETED** ✅

---
