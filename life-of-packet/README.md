# Life of a Packet Lab

## Objective

Trace a single packet's full journey from source host to destination host across multiple network devices, observing encapsulation/decapsulation and forwarding decisions at each hop.

## Topology

*(screenshot: topology.png)*

| Device | IP Address | Subnet Mask | Default Gateway |
|---|---|---|---|
| Host A | 11.11.11.10 | 255.255.255.0 | 11.11.11.1 |
| Host B | 11.11.11.20 | 255.255.255.0 | 11.11.11.1 |
| Host C | 22.22.22.30 | 255.255.255.0 | 22.22.22.1 |
| Host D | 22.22.22.40 | 255.255.255.0 | 22.22.22.1 |

Host A and Host B connect via SwitchX to Router0's Gi0/0 (`11.11.11.1/24`). Router0's Gi0/1 (`22.22.22.1/24`) connects via SwitchY to Host C and Host D. Both subnets are directly connected on Router0 — no static or dynamic routing was required for Host A/B to reach Host C/D.

```
Router0#show ip interface brief
Interface              IP-Address      OK? Method Status                Protocol
GigabitEthernet0/0      11.11.11.1     YES manual up                    up
GigabitEthernet0/1      22.22.22.1     YES manual up                    up

Router0#show ip route
     11.0.0.0/8 is variably subnetted, 2 subnets, 2 masks
C    11.11.11.0/24 is directly connected, GigabitEthernet0/0
L    11.11.11.1/32 is directly connected, GigabitEthernet0/0
     22.0.0.0/8 is variably subnetted, 2 subnets, 2 masks
C    22.22.22.0/24 is directly connected, GigabitEthernet0/1
L    22.22.22.1/32 is directly connected, GigabitEthernet0/1
```

## Key Concepts Demonstrated

- Encapsulation as data moves down the OSI stack on the sending host (Layer 7 → Layer 1)
- ARP resolution for next-hop MAC address, and the real delay it introduces on a packet's first attempt to reach a new destination
- Layer 2 switching decisions (MAC address table) vs. Layer 3 routing decisions (routing table)
- De-encapsulation as the packet moves up the stack on the receiving host

## Walkthrough

1. Host B (11.11.11.20) needs to reach Host C (22.22.22.30) — a different subnet
2. Host B compares the destination IP against its own subnet mask, determines Host C is not local, and sends the packet to its default gateway (11.11.11.1)
3. Host B ARPs for the gateway's MAC address (if not cached), encapsulates the packet with its own MAC as source and the gateway's MAC as destination, and sends it
4. Router0 receives the frame on Gi0/0, strips the Layer 2 header, reads the destination IP (22.22.22.30), and finds a directly connected route out Gi0/1
5. Router0 ARPs for Host C's MAC address (if not cached), re-encapsulates the packet with a new Layer 2 header, and forwards it out Gi0/1
6. SwitchY forwards the frame to Host C based on its MAC address table
7. Host C de-encapsulates the frame up through the stack and replies

## Verification

```
C:\>ping 22.22.22.30

Pinging 22.22.22.30 with 32 bytes of data:

Request timed out.
Reply from 22.22.22.30: bytes=32 time<1ms TTL=127
Reply from 22.22.22.30: bytes=32 time<1ms TTL=127
Reply from 22.22.22.30: bytes=32 time<1ms TTL=127

Ping statistics for 22.22.22.30:
    Packets: Sent = 4, Received = 3, Lost = 1 (25% loss),
```

```
C:\>ping 22.22.22.1

Pinging 22.22.22.1 with 32 bytes of data:

Reply from 22.22.22.1: bytes=32 time<1ms TTL=255
Reply from 22.22.22.1: bytes=32 time<1ms TTL=255
Reply from 22.22.22.1: bytes=32 time=12ms TTL=255
Reply from 22.22.22.1: bytes=32 time<1ms TTL=255

Ping statistics for 22.22.22.1:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
```

*(screenshot: ping-arp-resolution.png)*

### Key Insight: Why the First Packet Timed Out

Both routing (`show ip route`) and interfaces (`show ip interface brief`) were confirmed correctly configured — there was no misconfiguration. The first ICMP echo request to a newly-contacted destination (22.22.22.30) timed out because **ARP resolution for the destination's MAC address had to complete before the packet could be delivered**, and that resolution took slightly longer than the first ping's timeout window. Once the ARP cache was populated, every subsequent packet — to that same destination, and even to a different destination on the same subnet (22.22.22.1) — succeeded with 0% loss. This is a very common real-world pattern: **"first ping fails, retries succeed"** isn't a fault, it's ARP doing its job before the data plane is ready for a brand-new destination.

## What This Demonstrates

This lab builds intuition for how MAC addresses change at every hop while source/destination IP addresses stay the same end-to-end, and — more importantly — shows concretely why ARP resolution timing is a routine part of a packet's real "life," not an error condition. This is exactly the kind of detail that separates surface-level CCNA knowledge from genuinely understanding what's happening on the wire.
