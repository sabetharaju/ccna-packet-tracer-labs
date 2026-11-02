# VLAN & Access Port Configuration Lab

## Objective

Create VLANs on a Cisco switch and assign access ports to them, demonstrating logical traffic segmentation on a single physical switch.

## Topology

*(screenshot: topology.png)*

A single switch (Switch0) connects five PCs split across two departments, plus an uplink to Router0 (which connects out to a WAN cloud):

- **Engineering (VLAN 10)**: ENG PC1, ENG PC2, ENG PC3
- **Sales (VLAN 20)**: SALES PC1, SALES PC2

## Key Concepts Demonstrated

- VLANs as broadcast domain segmentation at Layer 2
- Access ports vs. trunk ports
- Devices on different VLANs cannot communicate without a Layer 3 device (router-on-a-stick or L3 switch)

## Actual Configuration (from `show running-config`)

```
interface FastEthernet0/1
 switchport access vlan 10
 switchport mode access
interface FastEthernet0/2
 switchport access vlan 20
 switchport mode access
interface FastEthernet0/3
 switchport access vlan 10
 switchport mode access
interface FastEthernet0/4
 switchport access vlan 10
 switchport mode access
interface FastEthernet0/5
 switchport access vlan 10
 switchport mode access
interface FastEthernet0/6
 switchport access vlan 20
 switchport mode access
interface FastEthernet0/7
 switchport access vlan 20
 switchport mode access
```

- **VLAN 10 ("Eng")** — ports Fa0/1, Fa0/3, Fa0/4, Fa0/5
- **VLAN 20 ("sales")** — ports Fa0/2, Fa0/6, Fa0/7

## Verification

```
Switch#show vlan brief

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Fa0/8, Fa0/9, Fa0/10, Fa0/11
                                                 Fa0/12, Fa0/13, Fa0/14, Fa0/15
                                                 Fa0/16, Fa0/17, Fa0/18, Fa0/19
                                                 Fa0/20, Fa0/21, Fa0/22, Fa0/23
                                                 Fa0/24, Gig0/1, Gig0/2
10   Eng                              active    Fa0/1, Fa0/3, Fa0/4, Fa0/5
20   sales                            active    Fa0/2, Fa0/6, Fa0/7
```

*(screenshot: vlan-brief-output.png)*

## What This Demonstrates

This lab shows how a single physical switch can be logically divided into isolated broadcast domains — every unused port defaults to VLAN 1, while Engineering and Sales devices are explicitly segmented into VLAN 10 and VLAN 20. Hosts on VLAN 10 and VLAN 20 cannot communicate directly since no trunk link or Layer 3 routing between the VLANs was configured in this lab — confirming that VLAN segmentation alone isolates traffic at Layer 2.
