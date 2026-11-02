# NAT Configuration Lab

## Objective

Configure and verify Network Address Translation (NAT) on a Cisco router so internal (private) addressed devices can communicate with external networks using a public IP address, including both static NAT and Port Address Translation (PAT).

## Topology

*(screenshot: topology.png)*

PC1 and PC2 connect through switch SW2 to router R1 (inside network). R1 connects through router SP1 to switch SW3 and a server (EXT-S1), representing the outside/internet-facing network.

- **R1 Fa0/0** — `203.0.113.2/28` — `ip nat outside`
- **R1 Fa0/1** — `10.0.1.1/24` — inside LAN segment (PC1 subnet)
- **R1 Fa1/0** — `10.0.2.1/24` — `ip nat inside` (PC2 subnet)

## Key Concepts Demonstrated

- **Inside Local** — the private IP address assigned to a host inside the network
- **Inside Global** — the public IP address that represents an inside host to the outside world
- **Outside Local / Outside Global** — how an external host's address appears from inside vs. outside
- **Static NAT** — fixed one-to-one mapping between an inside local and inside global address
- **PAT (NAT Overload)** — many inside local addresses sharing a single inside global address, differentiated by port number

## Actual Configuration (from `show running-config`)

```
interface FastEthernet0/0
 ip address 203.0.113.2 255.255.255.240
 ip nat outside

interface FastEthernet0/1
 ip address 10.0.1.1 255.255.255.0

interface FastEthernet1/0
 ip address 10.0.2.1 255.255.255.0
 ip nat inside

! Pool defined for dynamic NAT (available but not referenced by the active overload statement below)
ip nat pool ccna 203.0.113.4 203.0.113.14 netmask 255.255.255.240

! PAT - overloads outbound traffic from the 10.0.2.0/24 subnet onto R1's outside interface address
ip nat inside source list 1 interface FastEthernet0/0 overload

! Static NAT - permanent one-to-one mapping for a specific host (10.0.1.10)
ip nat inside source static 10.0.1.10 203.0.113.3

access-list 1 permit 10.0.2.0 0.0.0.255

ip route 0.0.0.0 0.0.0.0 203.0.113.1
```

## Verification

```
R1#show ip nat translations
Pro  Inside global     Inside local       Outside local      Outside global
---  203.0.113.3       10.0.1.10          ---                ---
```

*(screenshot: nat-translations-output.png)*

This confirms the static NAT entry: host `10.0.1.10` is permanently translated to public address `203.0.113.3` with no port multiplexing (no protocol/port shown), which is the distinguishing signature of static NAT versus PAT.

## What This Demonstrates

This lab combines two NAT types on a single router: **PAT** dynamically translates an entire subnet (10.0.2.0/24) through the router's single outside IP address using port-level multiplexing, while **Static NAT** gives one specific host (10.0.1.10) a dedicated, always-on public address — a common real-world pattern where most hosts share bandwidth via PAT but a server or specific device needs a consistent, reachable public IP.
