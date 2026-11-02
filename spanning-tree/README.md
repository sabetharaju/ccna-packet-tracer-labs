# Spanning Tree Protocol (STP) Lab

## Objective

Configure and verify Spanning Tree Protocol on a redundant switched topology, including manually influencing root bridge election.

## Topology

*(screenshot: topology.png — redundant switch links)*

A layered topology with two routers (R1, R2) connected to two distribution-layer switches (CD1, CD2), which are fully cross-connected (each distribution switch links to both access switches) down to two access-layer switches (Acc3, Acc4). Acc3 connects to Eng1 and Sales1; Acc4 connects to Eng2 and Sales2. The cross-linking between CD1 and CD2 to both access switches creates the redundant (looped) paths that STP must block.

## Key Concepts Demonstrated

- Root bridge election based on lowest Bridge ID (priority + MAC address)
- Port roles: Root Port, Designated Port, Blocking Port
- **Per-VLAN load balancing** — different switches act as root bridge for different VLANs, spreading traffic across both distribution switches instead of funneling everything through one
- **Root Guard** — protects the intended root bridge's position by blocking any port that would otherwise accept a superior BPDU
- Why manually setting bridge priority on the intended root switch is a best practice rather than relying on default election

## Actual Configuration (from CD2's `show running-config`)

```
spanning-tree mode pvst
spanning-tree extend system-id
spanning-tree vlan 11 priority 24576
spanning-tree vlan 10 priority 28672
 spanning-tree guard root
 spanning-tree guard root
```

- **VLAN 11 (SALES)** — CD2 set to priority `24576`, making it root bridge for this VLAN
- **VLAN 10** — CD2 set to priority `28672` (higher priority value = less preferred), deliberately leaving root bridge duty to CD1 for this VLAN
- **VLAN 1** — left at default priority, so root bridge is decided by lowest MAC address (CD1 won this tiebreaker)
- `spanning-tree guard root` applied on uplink-facing ports, preventing a rogue or incorrectly configured switch from taking over as root

This is a **per-VLAN root bridge load-balancing design**: CD2 handles root duties for VLAN 11 (SALES) while CD1 handles VLAN 1 and VLAN 10, spreading the distribution load across both switches instead of concentrating it on one.

## Verification

```
CD2#show spanning-tree summary
Switch is in pvst mode
Root bridge for: SALES
...
Name                   Blocking Listening Learning Forwarding STP Active
---------------------- -------- --------- -------- ---------- ----------
VLAN0001                     0         0        0          2          2
VLAN0010                     0         0        0          4          4
VLAN0011                     0         0        0          4          4
```

*(screenshot: stp-summary-output.png)*

```
CD2#show spanning-tree
VLAN0011
  Spanning tree enabled protocol ieee
  Root ID    Priority    24587
             Address     0060.3EE1.4C2A
             This bridge is the root
```

*(screenshot: stp-vlan11-root-output.png)*

### Observed Detail: PVID Inconsistency Flag

While reviewing the per-VLAN output for VLAN 1, interface `Fa0/2` showed a `BKN*19 ... *PVID_Inc` flag — indicating Packet Tracer detected a **Port VLAN ID (PVID) inconsistency** between the two ends of that trunk link, in addition to the port being blocked by STP. This is a useful real-world reminder that STP troubleshooting output often surfaces other misconfigurations (like native VLAN mismatches) alongside the blocking/forwarding state itself.

## What This Demonstrates

This lab shows how STP prevents Layer 2 loops in a fully cross-connected, redundant topology by electing a root bridge per VLAN and blocking redundant paths — and goes a step further than a single-root design by deliberately splitting root bridge responsibility across two switches (CD1 and CD2) per VLAN, which balances traffic load across the distribution layer rather than forcing all VLANs through a single root switch. Root Guard was also configured to lock in this design and prevent an unintended topology change if another switch with a lower priority were introduced.
