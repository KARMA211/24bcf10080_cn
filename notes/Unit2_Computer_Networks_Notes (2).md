# Computer Networks – Unit 2 Complete Notes

**Routing Algorithms • IP Addressing (IPv4 and IPv6) • Subnetting and Supernetting • DHCP • VLAN • Congestion Control • Tunnelling • Fragmentation**

| Chapter | Syllabus Topic                                                                                                                                                                                            | Included |
| ------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------- |
| 2.1     | Routing Algorithm – Network layer design issues, classful and classless IP addressing, subnetting, supernetting, forwarding of IP packets, IPv4 header, IPv4 and IPv6 addressing, network layer protocols | Yes      |
| 2.2     | Distance vector, link state, hierarchical and broadcast routing, shortest path algorithm, unicast routing, DHCP, VLAN                                                                                     | Yes      |
| 2.3     | Congestion prevention policies, congestion control in datagram subnets, load shedding, jitter control, leaky bucket and token bucket algorithms, tunnelling, internetwork routing, fragmentation          | Yes      |

> **How to use these notes:** theory first, then the worked examples – subnetting, Dijkstra, fragmentation and token bucket are the most frequently asked numericals from this unit. Tables and comparison charts are written for last-minute revision.

---

# Chapter 2.1 – Routing Algorithm, IP Addressing and Network Layer Protocols

## 1. Network Layer Design Issues

The network layer is responsible for delivering packets from the source host to the destination host, possibly across multiple heterogeneous networks. Its main design issues are:

- **Routing:** choosing the best path (route) from source to destination. Done by routing algorithms, which fill routing tables.
- **Forwarding:** moving a packet from the router's input port to the correct output port using the routing table. (Routing = filling the table, forwarding = using it.)
- **Logical addressing:** every host gets a unique IP address, independent of the underlying hardware (MAC) address.
- **Fragmentation and reassembly:** if a packet is larger than the network's MTU, it is split into fragments and reassembled at the destination.
- **Congestion control:** preventing too many packets from flooding the subnet (covered in Chapter 2.3).

Services provided to the transport layer should be:

- Independent of the subnet topology (router count, layout).
- Independent of the technology of the routers (no transport layer changes if routers are replaced).
- Uniform addressing scheme – a transport entity should be able to talk to any other, anywhere.

### Connectionless (datagram) vs Connection-oriented (virtual circuit) subnets

| Issue              | Datagram Subnet                                             | Virtual Circuit Subnet                                   |
| ------------------ | ----------------------------------------------------------- | -------------------------------------------------------- |
| Setup phase        | None – packets sent anytime                                 | Connection (VC) established first; all packets follow it |
| Address carried    | Full source + destination address in every packet           | Short VC identifier in each packet                       |
| State in routers   | None – stateless                                            | Each VC occupies a table entry                           |
| Routing            | Each packet routed independently (may take different paths) | All packets follow the same fixed route                  |
| Failure handling   | Reroute around failed router easily                         | VC breaks; connection must be re-established             |
| Congestion control | Difficult (no reserved resources)                           | Easier – resources can be reserved in advance            |
| Examples           | IP (the Internet)                                           | ATM, MPLS, X.25, frame relay                             |

---

## 2. IP Addressing – IPv4

An IPv4 address is **32 bits**, written as four decimal octets separated by dots (dotted-decimal notation), e.g. `192.168.10.25`. Every address has a network part and a host part.

### 2.1 Classful Addressing (original scheme)

| Class | Leading bits | 1st octet range | Default mask        | Max hosts per network | Use                     |
| ----- | ------------ | --------------- | ------------------- | --------------------- | ----------------------- |
| A     | 0            | 1 – 126         | 255.0.0.0 (/8)      | 16,777,214            | Very large networks     |
| B     | 10           | 128 – 191       | 255.255.0.0 (/16)   | 65,534                | Medium networks         |
| C     | 110          | 192 – 223       | 255.255.255.0 (/24) | 254                   | Small networks          |
| D     | 1110         | 224 – 239       | –                   | –                     | Multicast groups        |
| E     | 1111         | 240 – 255       | –                   | –                     | Experimental / reserved |


 
0    0   0   0   0  0  0  0
1    0   0   0   0  0  0  0 -> 128 - 191 
1    1   0   0   0  0  0  0 -> 192 - 223
1    1   1   0   0  0  0  0 -> 224 - 239 
1    1   1   1   0  0  0  0 -> 240 - 255 

128  64  32  16  8  4  2  1



- **Class A:** first octet = network, last 3 octets = host. Only 126 networks but huge host count – wasteful for small users.
- **Class B:** first two octets = network, last two = host.
- **Class C:** first three octets = network, last octet = host – max 254 hosts per network.
- **Special addresses:**
  - `127.x.x.x` = loopback (test local stack)
  - host part all 0s = network address
  - host part all 1s = directed broadcast
  - `0.0.0.0` = 'this host' (source at boot)
  - `255.255.255.255` = limited broadcast
- **Private addresses (RFC 1918, not routable on Internet):**
  - `10.0.0.0` – `10.255.255.255`
  - `172.16.0.0` – `172.31.255.255`
  - `192.168.0.0` – `192.168.255.255`
  - `169.254.x.x` = APIPA (self-assigned when DHCP fails)

### 2.2 Classless Addressing (CIDR – Classless Inter-Domain Routing)

- Classful addressing wasted addresses and made routing tables explode. CIDR removed the classes.
- Written as `a.b.c.d/n` where *n* = number of network bits (the prefix length), e.g. `192.168.4.0/24` means the first 24 bits identify the network.
- The mask has *n* ones followed by (32 − *n*) zeros, e.g. /26 = 255.255.255.192.
- **Advantage:** address blocks can be allocated in exact sizes needed, and many small networks can be aggregated (summarized) into one route – this reduces router table size.

### 2.3 Subnetting

**Idea:** 'borrow' bits from the host part of an address to create smaller sub-networks inside one network. Subnetting reduces broadcast traffic, improves security and stops address waste.

- **Steps:**
  1. Find how many subnet/host bits are needed.
  2. Write the new mask.
  3. The block size = 256 − mask's interesting octet value.
  4. Enumerate network addresses, host ranges and broadcast addresses.
- Number of subnets = 2^(borrowed bits); usable hosts per subnet = 2^(remaining host bits) − 2 (network and broadcast are reserved).

#### Worked Example 1: Subnetting 192.168.1.0/24 into 4 subnets

Need 4 subnets: 2² = 4, so borrow 2 bits → prefix **/26**, mask **255.255.255.192**, block size **64**, **62** usable hosts each.

| Subnet | Network address | Host range                    | Broadcast address |
| ------ | --------------- | ----------------------------- | ----------------- |
| 1      | 192.168.1.0     | 192.168.1.1 – 192.168.1.62    | 192.168.1.63      |
| 2      | 192.168.1.64    | 192.168.1.65 – 192.168.1.126  | 192.168.1.127     |
| 3      | 192.168.1.128   | 192.168.1.129 – 192.168.1.190 | 192.168.1.191     |
| 4      | 192.168.1.192   | 192.168.1.193 – 192.168.1.254 | 192.168.1.255     |

#### Worked Example 2: Subnetting for hosts

Given 172.16.0.0/16, a department needs 500 hosts per subnet. 2⁹ − 2 = 510 (works), so 9 host bits → prefix **/23**, mask **255.255.254.0**, block size 2 in the third octet.

Subnets: `172.16.0.0/23`, `172.16.2.0/23`, `172.16.4.0/23`, ...

#### Quick subnetting reference

| Prefix | Mask            | Usable hosts |
| ------ | --------------- | ------------ |
| /24    | 255.255.255.0   | 254          |
| /25    | 255.255.255.128 | 126          |
| /26    | 255.255.255.192 | 62           |
| /27    | 255.255.255.224 | 30           |
| /28    | 255.255.255.240 | 14           |
| /30    | 255.255.255.252 | 2 (links)    |

### 2.4 Supernetting (Route Aggregation)

- **Idea:** the reverse of subnetting – combine several consecutive classful networks into one larger block, using a shorter prefix.
- **Example:** `192.168.0.0/24`, `192.168.1.0/24`, `192.168.2.0/24` and `192.168.3.0/24` (four Class C networks) can be advertised together as `192.168.0.0/22` – one entry instead of four, because the first 22 bits are common.
- Reduces routing table size on the Internet (this summarization is a key benefit of CIDR).

---

## 3. Forwarding of IP Packets

- Each router keeps a routing table with entries: destination network/prefix, subnet mask, next-hop router (or 'direct'), outgoing interface, and optionally a metric.
- When a packet arrives, the router ANDs the destination IP with each entry's mask; if it matches the entry's network, the packet is forwarded to that entry's next hop.
- **Longest prefix match:** if several entries match, the one with the longest (most specific) prefix wins – this is how CIDR and supernetting work together.
- **Default route:** `0.0.0.0/0` – used when no other entry matches (gateway of last resort).

> **Exam tip:** a typical numerical gives you a routing table and a few destination IPs – AND each IP with each mask, apply longest prefix match, and state the next hop.

---

## 4. IPv4 Header

The IPv4 header is **20 bytes minimum, up to 60 bytes** with options. Fields (in order):

| Field                  | Size         | Purpose                                                                      |
| ---------------------- | ------------ | ---------------------------------------------------------------------------- |
| Version                | 4 bits       | IP version in use (4)                                                        |
| IHL (Header Length)    | 4 bits       | Header length in 32-bit words: 5 to 15, i.e. 20 to 60 bytes                  |
| DSCP / Type of Service | 8 bits       | Priority / QoS treatment for the packet                                      |
| Total Length           | 16 bits      | Header + data length in bytes (max 65,535)                                   |
| Identification         | 16 bits      | Same value in all fragments of one datagram (for reassembly)                 |
| Flags                  | 3 bits       | DF = Do not Fragment; MF = More Fragments follow                             |
| Fragment Offset        | 13 bits      | Position of this fragment's data, in units of 8 bytes                        |
| TTL (Time To Live)     | 8 bits       | Hop counter; decremented each router; packet discarded at 0 (prevents loops) |
| Protocol               | 8 bits       | Upper-layer protocol: 6 = TCP, 17 = UDP, 1 = ICMP                            |
| Header Checksum        | 16 bits      | Integrity check on the header only; recomputed at every hop (TTL changes)    |
| Source Address         | 32 bits      | Sender's IP address                                                          |
| Destination Address    | 32 bits      | Receiver's IP address                                                        |
| Options                | 0 – 40 bytes | Rarely used: record route, timestamp, strict/loose source route              |

---

## 5. IPv6 Addressing

- IPv6 addresses are **128 bits**, written as 8 groups of 4 hexadecimal digits separated by colons, e.g. `2001:0db8:0000:0000:0000:ff00:0042:8329`.
- Leading zeros in a group may be omitted; one run of consecutive zero groups may be replaced by `::`. Example above → `2001:db8::ff00:42:8329`.

### IPv6 header (fixed 40 bytes, no options field)

| Field               | Size     | Purpose                                                                                         |
| ------------------- | -------- | ----------------------------------------------------------------------------------------------- |
| Version             | 4 bits   | 6                                                                                               |
| Traffic Class       | 8 bits   | QoS / priority (like DSCP)                                                                      |
| Flow Label          | 20 bits  | Marks packets of one flow for special handling                                                  |
| Payload Length      | 16 bits  | Length of data after the 40-byte header                                                         |
| Next Header         | 8 bits   | Type of the next header: an extension header or upper protocol (6 = TCP, 17 = UDP, 58 = ICMPv6) |
| Hop Limit           | 8 bits   | Same job as IPv4 TTL                                                                            |
| Source Address      | 128 bits | Sender                                                                                          |
| Destination Address | 128 bits | Receiver                                                                                        |

### IPv4 vs IPv6 – key differences

| Feature         | IPv4                                                       | IPv6                                                                  |
| --------------- | ---------------------------------------------------------- | --------------------------------------------------------------------- |
| Address size    | 32 bits                                                    | 128 bits (huge space; restores end-to-end addressing)                 |
| Header          | 20 – 60 bytes, 14 fields                                   | Fixed 40 bytes, 8 fields – faster router processing                   |
| Header checksum | Present; recomputed every hop                              | Removed (lower layers already check integrity)                        |
| Fragmentation   | Routers and source can fragment; reassembly at destination | Only the SOURCE fragments; routers never fragment (simpler routers)   |
| Options         | Variable part of header                                    | Extension headers placed between header and payload                   |
| Broadcast       | Supported                                                  | Not supported – replaced by multicast and anycast                     |
| Configuration   | Manual or DHCP                                             | Autoconfiguration (SLAAC) + DHCPv6                                    |
| Address types   | Unicast, broadcast, multicast                              | Unicast (global, link-local FE80::/10), multicast (FF00::/8), anycast |

- **Anycast** (new in IPv6): one address assigned to several hosts; a packet to it is delivered to the NEAREST one – used for load balancing and finding the closest server.

---

## 6. Network Layer Protocols (working and purpose)

| Protocol | Full name                           | Working / purpose                                                                                                                   |
| -------- | ----------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------- |
| IP       | Internet Protocol                   | Delivers datagrams between hosts; connectionless, best-effort, no guarantees                                                        |
| ICMP     | Internet Control Message Protocol   | Error reporting and diagnostics between hosts/routers (destination unreachable, time exceeded); ping uses Echo Request / Echo Reply |
| ARP      | Address Resolution Protocol         | Finds the MAC address of a host on the same LAN, given its IP (request broadcast, reply unicast)                                    |
| RARP     | Reverse ARP                         | Finds the IP given the MAC – superseded by BOOTP and DHCP                                                                           |
| IGMP     | Internet Group Management Protocol  | Lets hosts join / leave multicast groups; routers use it to track group members on each interface                                   |
| DHCP     | Dynamic Host Configuration Protocol | Automatically assigns IP address, mask, gateway and DNS to hosts at boot time (detailed in Chapter 2.2)                             |
| NAT      | Network Address Translation         | Maps many private addresses to one (or few) public IP(s) at the router – conserves IPv4 addresses                                   |

---

# Chapter 2.2 – Routing Algorithms, DHCP and VLAN

## 1. Shortest Path Algorithm (Dijkstra)

- **Goal:** find the least-cost path between a source and every other node in the network, where cost can be hops, distance, delay or a money measure.
- **Method:** each node is labelled with its best-known distance from the source; the node with the smallest tentative label is made permanent, and its neighbours are relaxed (updated). Repeat until all nodes are permanent.

### Worked example

Topology and link costs: A–B = 2, A–C = 1, B–C = 2, B–D = 1, C–D = 4, C–E = 3, D–E = 1. Find shortest paths from A.

| Step | Permanent set N' | B         | C         | D         | E                       |
| ---- | ---------------- | --------- | --------- | --------- | ----------------------- |
| 0    | A                | 2 (via A) | 1 (via A) | ∞         | ∞                       |
| 1    | A, C             | 2 (via A) | –         | 5 (via C) | 4 (via C)               |
| 2    | A, B, C          | –         | –         | 3 (via B) | 4 (via C)               |
| 3    | A, B, C, D       | –         | –         | –         | 4 (tie: via C or via D) |
| 4    | all              | –         | –         | –         | –                       |

**Result:** A→C = 1, A→B = 2, A→B→D = 3, A→C→E = 4 (or A→B→D→E = 4).

**Complexity:** O(n²) naively; can be reduced with heap data structures. Used inside link state routing (OSPF).

---

## 2. Distance Vector Routing (DVR)

- Used by **RIP**. Each router keeps a table with the best known distance to every destination and the next hop – a vector of distances.
- Routers periodically exchange their entire tables with directly connected neighbours only.
- On receiving a neighbour's table, a router applies the **Bellman-Ford** update: `D_x(y) = min over all neighbours v of [ c(x,v) + D_v(y) ]` – if a cheaper route via that neighbour is found, update the entry.

### Worked example: line network A – B – C – D (each link cost 1)

| Router A learns (final) | Distance | Next hop | How it was learned    |
| ----------------------- | -------- | -------- | --------------------- |
| A to B                  | 1        | B        | direct neighbour      |
| A to C                  | 2        | B        | C is 1 from B (1 + 1) |
| A to D                  | 3        | B        | D is 2 from B (2 + 1) |

### Count-to-infinity problem

- If a link goes down, two neighbours can keep telling each other 'I can reach X through you' – bad news travels slowly; the estimated distance rises to infinity (16 = infinity in RIP).
- **Fixes:**
  - **Split horizon** (never advertise a route back to the neighbour you learned it from)
  - **Split horizon with poison reverse** (advertise it back with cost infinity)
  - **Hold-down timers** (ignore updates about a failed route for a while)

---

## 3. Link State Routing (LSR)

- Used by **OSPF** and **IS-IS**. Every router learns the complete map of the network and runs Dijkstra on it.

### Five steps

1. **Discover neighbours:** send HELLO packets on each link; record who replies.
2. **Measure cost:** measure delay / bandwidth to each neighbour (echo packets with timestamps).
3. **Build Link State Packet (LSP):** containing (router ID, neighbours, costs, sequence number, age).
4. **Distribute LSPs by flooding:** each router forwards the LSP on all links except the one it arrived on, using sequence numbers to reject duplicates/loops (acknowledged for reliability).
5. **Compute shortest paths:** once every router has all LSPs, it reconstructs the full topology and runs Dijkstra to build its routing table.

### Distance Vector vs Link State

| Point              | Distance Vector                                          | Link State                             |
| ------------------ | -------------------------------------------------------- | -------------------------------------- |
| Knowledge          | Only distance to destinations, via neighbours' summaries | Full topology map of the whole network |
| Information shared | Entire routing table with neighbours only                | LSPs flooded to all routers            |
| Algorithm          | Bellman-Ford (distributed)                               | Dijkstra (run locally on complete map) |
| Update trigger     | Periodic (every 30 s in RIP) plus triggered updates      | On change only (faster convergence)    |
| Bandwidth use      | Small messages but frequent                              | Heavy at start / on changes only       |
| Convergence        | Slow; count-to-infinity problem                          | Fast; loops less likely                |
| Robustness         | Bad information propagates                               | One router's error affects only itself |
| Examples           | RIP                                                      | OSPF, IS-IS                            |

---

## 4. Hierarchical Routing

- As networks grow, routing tables become huge. Idea: divide routers into regions; a router knows full detail about its own region but treats other regions as a single destination.
- **Trade-off:** smaller tables and less update traffic, at the cost of slightly longer routes than optimal.
- For *n* routers, the optimal number of hierarchy levels is approximately **ln n**, giving average table size of about **e · ln n** entries.
- This is exactly how the Internet works: networks, then areas, then autonomous systems (domains).

---

## 5. Broadcast Routing

Delivering a packet from one source to all hosts. Four techniques:

- **Flooding:** send on every outgoing link except the one it arrived on. Simple and robust but generates duplicate packets – controlled by a sequence number per source: a router forwards a packet only if it has not seen that sequence number before.
- **Multi-destination routing:** the packet carries a list of destinations; each router picks out the destinations reachable through each outgoing line, splits the list and forwards – one copy per needed link.
- **Reverse Path Forwarding (RPF):** a broadcast packet is forwarded only if it arrives on the link that lies on the shortest path back to the source – eliminates most duplicates cheaply, one copy per LAN.
- **Spanning-tree routing:** build a spanning tree covering all LANs; each packet is forwarded only along tree branches – the most efficient scheme, one copy per link (basis of multicast routing).

---

## 6. Unicast Routing

- Delivery of a packet from one source to one destination – the normal case handled by all the algorithms above (DVR, LSR, Dijkstra).
- **Destination-based forwarding:** the router looks only at the destination address in the routing table – one table lookup per packet, independent of the source.

---

## 7. DHCP – Dynamic Host Configuration Protocol

- **Purpose:** automates IP configuration. A host boots with no IP and gets one automatically: address, subnet mask, default gateway, DNS server(s) and the lease time – all in one transaction.
- **Transport:** UDP; server port **67**, client port **68**. Works across routers via a DHCP relay agent.

### DORA – the four-step exchange

| Step | Message          | Who sends          | What happens                                                          |
| ---- | ---------------- | ------------------ | --------------------------------------------------------------------- |
| 1    | DHCP Discover    | Client → broadcast | 'I need an IP' (to 255.255.255.255, since client has no address)      |
| 2    | DHCP Offer       | Server → client    | One (or more) servers offer an address + lease parameters             |
| 3    | DHCP Request     | Client → broadcast | Client accepts one offer (broadcast so other servers withdraw theirs) |
| 4    | DHCP Acknowledge | Server → client    | Lease confirmed; client configures the address and starts the timer   |

- **Allocation methods:** automatic (permanent), dynamic (leased, returned when host leaves – reuses scarce addresses), manual (admin fixed).

---

## 8. VLAN – Virtual LAN

- **Definition (IEEE 802.1Q):** a group of devices that communicate as if on the same LAN, regardless of their physical location – membership is configured in the switch, not by cabling.
- Switches tag frames with a **4-byte** VLAN ID (inserted after the source address); switches strip or add tags between trunk links.

### Benefits

- Each VLAN is a separate **broadcast domain** – broadcast traffic stays inside the VLAN (better performance).
- **Security and management:** departments are isolated; users can only reach services of their own group.
- **Flexibility:** moving a user to another department = a configuration change, no rewiring.

### Types

- **Port-based** (switch port membership – most common)
- **MAC-based** (by device address)
- **Protocol-based**

---

# Chapter 2.3 – Congestion Control, Tunnelling and Fragmentation

## 1. Congestion – what and why

- **Congestion:** too many packets in (part of) the subnet – demand exceeds capacity, queues grow, delay rises sharply, and routers start dropping packets.
- **Causes:** sudden bursts of traffic, too many hosts sending at full speed, slow processors/links, insufficient memory (buffer overflow), retransmissions of lost packets making it worse (congestion collapse).
- Congestion is a **global** phenomenon involving many hosts and routers; flow control (one sender, one receiver) cannot solve it.

---

## 2. Congestion Prevention Policies

**Open-loop (prevention):** act before congestion happens.

- **Traffic shaping:** regulate the rate/pattern at which traffic is injected into the network – leaky bucket and token bucket (below).
- **Admission control:** for connection-oriented networks – a new virtual circuit is admitted only if the network can carry it without degrading existing ones (negotiated at setup).

**Closed-loop (control):** monitor → detect → convey → adjust. Routers watch queue lengths / utilization; when a threshold is crossed they inform sources; sources then slow down (covered below and in TCP).

---

## 3. Congestion Control in Datagram Subnets

- **Warning bits:** routers set a special bit in the packet header when congestion is building; the destination echoes the warning back to the source in its reply, and the source slows down.
- **Choke packets:** the congested router sends an explicit control packet directly to the source ('slow down to X rate'). Sources that ignore repeated choke packets get dropped.
- **Hop-by-hop choke packets:** each router on the path applies the relief immediately (relieves congestion faster along the path, at the cost of more control packets).

### Load shedding

- When buffers are full, routers must drop packets – choosing which to drop is load shedding.
- **Wine vs milk policy:** drop old packets (wine gets better with age) or fresh ones (old milk goes bad first – VoIP prefers dropping old audio).
- **RED (Random Early Detection):** drop packets randomly before buffers are completely full, so all sources slow down together gently instead of one flow losing everything.

### Jitter control

- **Jitter** = variation in end-to-end packet delay. Even with low average delay, high jitter ruins real-time traffic (VoIP, video).
- **Fix:** buffer packets at the receiver and play them out at constant intervals – smooths delay variation; trade-off is added fixed delay.

---

## 4. Leaky Bucket Algorithm

- **Analogy:** a bucket with a small hole at the bottom. Water (packets) can be poured in at any rate, but it leaks out at a constant rate; if the bucket overflows, the excess is lost.
- **Working:** a host sends packets into a queue (the bucket); a scheduler releases exactly one packet every fixed time interval (constant output rate), regardless of how bursty the input is.
- **Effect:** converts a bursty stream into a perfectly even stream at the leak rate.
- **Properties:** output rate is always constant; idle periods cannot be 'saved up' – bursts are lost; simple queue with a timer; may waste capacity when the network is idle.

---

## 5. Token Bucket Algorithm

- **Working:** tokens are generated at a constant rate (ρ) into a bucket of capacity C. To send a packet of size S, the host must remove S tokens; if not enough tokens are available, the packet waits (or is dropped).
- **Effect:** long-term rate is limited to the token rate, but bursts up to bucket capacity are allowed – unlike leaky bucket.
- Idle hosts accumulate tokens (up to C), so saved capacity can be used in a later burst.
- **Maximum burst length:** `S = C / (M − ρ)`, where M is the maximum output rate of the link.

### Worked example

Bucket capacity C = 500 KB, tokens generated at ρ = 10 MB/s, maximum output rate M = 50 MB/s.

Maximum burst length S = C / (M − ρ) = 0.5 MB / 40 MB/s = **12.5 ms**.

### Leaky Bucket vs Token Bucket

| Point            | Leaky Bucket                         | Token Bucket                                              |
| ---------------- | ------------------------------------ | --------------------------------------------------------- |
| Bucket holds     | Packets (queue)                      | Tokens                                                    |
| Output rate      | Strictly constant                    | Average = token rate; short bursts up to capacity allowed |
| Idle sender      | Gains nothing; input bursts are lost | Accumulates tokens (up to capacity) for later             |
| Network capacity | Wasted when idle                     | Used efficiently when idle                                |
| Typical use      | Hard rate limiting                   | Traffic shaping allowing controlled bursts                |

---

## 6. Tunnelling

- **Definition:** to move a packet through a network that speaks a different protocol, the whole packet is placed as data inside the payload of another packet; at the far end it is extracted (decapsulated) and forwarded normally.
- **Uses:** carrying IPv6 packets across IPv4 networks (6to4); VPNs – private IP packets wrapped in public Internet packets, encrypted; multicast through unicast-only networks; connecting dissimilar WANs.
- Like putting a letter inside a bigger envelope so a foreign postal system can carry it.

---

## 7. Internetworking and Internetwork Routing

- **Internetwork (internet):** a collection of networks (LANs, WANs) connected by routers/gateways, appearing as one network to hosts.
- **Problems when routing between different networks:** different addressing schemes, different MTUs (needs fragmentation), different protocols/quality of service, security and administration – the network layer hides all this from the transport layer.
- Routers forward packets hop-by-hop across network boundaries; routing protocols (RIP, OSPF, BGP) exchange reachability so every router knows where to send traffic for distant networks.
- **Gateways** translate between completely different protocols/architectures (stronger than a router, which connects networks sharing the same protocol family).

---

## 8. Fragmentation

- Each network (link) has a **Maximum Transmission Unit (MTU)** – the largest frame payload it can carry. A datagram bigger than the MTU must be split.
- **IPv4:** routers along the path fragment when needed; reassembly only at the final destination (so fragments can take different routes).
- **IPv6:** routers never fragment – if a packet is too big, the router sends back an ICMPv6 'packet too big' message and the SOURCE must fragment.
- **Fields used:**
  - **Identification** – all fragments of one datagram carry the same value
  - **MF (More Fragments)** – 1 on all but the last
  - **DF (Don't Fragment)** – if set and packet too big, it's dropped + ICMP error
  - **Fragment Offset** – position of this fragment in the original data, measured in units of 8 bytes

### Worked example

A 4000-byte IP datagram (20-byte header + 3980 bytes data) must cross a network with MTU 1500 bytes. Max data per fragment = 1480 (must be a multiple of 8).

| Fragment | Total length | Data bytes | MF  | Fragment offset (bytes / 8) |
| -------- | ------------ | ---------- | --- | --------------------------- |
| 1        | 1500         | 1480       | 1   | 0 (0 / 8)                   |
| 2        | 1500         | 1480       | 1   | 185 (1480 / 8)              |
| 3        | 1040         | 1020       | 0   | 370 (2960 / 8)              |

- **Check:** 1480 + 1480 + 1020 = 3980 data bytes. Offsets are 0, 185, 370 (each offset = previous offset + previous data size / 8).
- The destination uses Identification + MF + offset to reassemble in order; the timer discards fragments if some never arrive.

---

# Last-Minute Revision – Numbers and One-Liners

- IPv4 = 32 bits; IPv6 = 128 bits; IPv4 header 20–60 B; IPv6 header fixed 40 B; max IPv4 datagram = 65,535 B.
- Default masks: A /8 (255.0.0.0), B /16 (255.255.0.0), C /24 (255.255.255.0).
- /26 = 62 hosts, /27 = 30 hosts, /30 = 2 hosts; block size = 256 − mask octet.
- RIP = distance vector (metric = hop count, max 15); OSPF = link state (Dijkstra).
- Count-to-infinity fixed by: split horizon, poison reverse, hold-down timers.
- Optimal hierarchy levels = ln n routers.
- DHCP DORA: Discover, Offer, Request, Acknowledge; UDP ports 67 (server) / 68 (client).
- VLAN = IEEE 802.1Q tag = 4 bytes; each VLAN = separate broadcast domain.
- Leaky bucket = constant rate; token bucket = average rate + bursts (S = C / (M − ρ)).
- Fragment offset units = 8 bytes; IPv4 reassembly at destination only; IPv6 source fragments only.
- Protocol field: 6 = TCP, 17 = UDP, 1 = ICMP.

## Most likely exam questions from this unit

- Explain the IPv4 header field by field (draw the diagram).
- Subnet a given network into N subnets / for N hosts – full table with ranges and broadcast.
- Differentiate IPv4 and IPv6; explain IPv6 addressing notation and header.
- Explain distance vector routing; illustrate count-to-infinity and its remedies.
- Explain link state routing and its five steps; compare with distance vector.
- Run Dijkstra's algorithm on a given graph (labelled steps).
- Explain DHCP with the DORA sequence; explain VLAN and its advantages.
- Compare leaky bucket and token bucket (with the burst-length formula).
- Explain fragmentation with a worked example (MTU calculation).
- What is congestion? Explain congestion control in datagram subnets: choke packets, load shedding, jitter control; explain tunnelling.

---

*All the best for your mid-sems! If you want, I can also make a one-page quick-revision sheet or a practice question set with answers for Unit 2.*
