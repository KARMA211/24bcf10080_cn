# Computer Networks — Unit 2 Notes

**Routing Algorithms • IP Addressing (IPv4/IPv6) • Subnetting & Supernetting • DHCP • VLAN • Congestion Control • Tunnelling • Fragmentation**

*Mid-semester exam prep — based on your syllabus*

---

# Chapter 2.1 — Routing Algorithm, IP Addressing and Network Layer Protocols

## 1. Network Layer Design Issues

The network layer delivers packets from the source host to the destination host, possibly across multiple heterogeneous networks. Its main design issues are:

- **Routing:** choosing the best path (route) from source to destination. Done by routing algorithms, which fill routing tables.
- **Forwarding:** moving a packet from the router's input port to the correct output port using the routing table. (Routing = filling the table, forwarding = using it.)
- **Logical addressing:** every host gets a unique IP address, independent of the underlying hardware (MAC) address.
- **Fragmentation and reassembly:** if a packet is larger than the network's MTU, it is split into fragments and reassembled at the destination.
- **Congestion control:** preventing too many packets from flooding the subnet (covered in Chapter 2.3).

**Services provided to the transport layer should be:**

- Independent of the subnet topology (router count, layout).
- Independent of the technology of the routers (no transport layer changes if routers are replaced).
- Uniform addressing scheme — a transport entity should be able to talk to any other, anywhere.

### Connectionless (datagram) vs Connection-oriented (virtual circuit) subnets

| Issue | Datagram Subnet | Virtual Circuit Subnet |
|---|---|---|
| Setup phase | None — packets sent anytime | Connection (VC) established first; all packets follow it |
| Address carried | Full source + destination address in every packet | Short VC identifier in each packet |
| State in routers | None — stateless | Each VC occupies a table entry |
| Routing | Each packet routed independently (may take different paths) | All packets follow the same fixed route |
| Failure handling | Reroute around failed router easily | VC breaks; connection must be re-established |
| Congestion control | Difficult (no reserved resources) | Easier — resources can be reserved in advance |
| Examples | IP (the Internet) | ATM, MPLS, X.25, frame relay |

## 2. IP Addressing — IPv4

### Why do we need IP addresses when MAC addresses already exist?

- The **MAC address** is a Layer 2 (Data Link) address: switches use it for delivery inside one LAN, and it never changes.
- Routers work at Layer 3 and must move packets **between different networks**, so they need an address that identifies both the *network* and the *host* — that is the IP address.
- IP is called a **logical address** because it can change with location: the same laptop may be `192.168.10.25` at university and `192.168.1.15` at home, while its MAC stays the same. A router can also carry a different IP on each interface (e.g. G0/0 = 192.168.1.1, G0/1 = 192.168.2.1).

An IPv4 address is **32 bits**, written as four decimal octets separated by dots (dotted-decimal notation), e.g. `192.168.10.25`. Every address has a **network part** and a **host part**.

### 2.1 Classful Addressing (original scheme)

| Class | Leading bits | 1st octet range | Default mask | Max hosts per network | Use |
|---|---|---|---|---|---|
| A | 0 | 1 – 126 | 255.0.0.0 (/8) | 16,777,214 | Very large networks |
| B | 10 | 128 – 191 | 255.255.0.0 (/16) | 65,534 | Medium networks |
| C | 110 | 192 – 223 | 255.255.255.0 (/24) | 254 | Small networks |
| D | 1110 | 224 – 239 | — | — | Multicast groups |
| E | 1111 | 240 – 255 | — | — | Experimental / reserved |

- **Class A:** first octet = network, last 3 octets = host. Only 126 networks but huge host count — wasteful for small users.
- **Class B:** first two octets = network, last two = host.
- **Class C:** first three octets = network, last octet = host — max 254 hosts per network.
- **Special addresses:** `127.x.x.x` = loopback (test local stack); host part all 0s = network address; host part all 1s = directed broadcast; `0.0.0.0` = "this host" (source at boot); `255.255.255.255` = limited broadcast.
- **Private addresses (RFC 1918, not routable on the Internet):** `10.0.0.0 – 10.255.255.255`, `172.16.0.0 – 172.31.255.255`, `192.168.0.0 – 192.168.255.255`; `169.254.x.x` = APIPA (self-assigned when DHCP fails).

**Why classful addressing had to go (the 500-device problem):** a company needing 500 hosts gets only 254 from Class C (not enough), so it must take a whole Class B with 65,534 hosts — over 65,000 addresses wasted. This massive wastage motivated CIDR.

**Classful vs Classless (CIDR):**

| Feature | Classful | Classless / CIDR |
|---|---|---|
| Network size | Fixed by class | Flexible — any prefix length |
| Allowed prefixes | Only /8, /16, /24 | Any length, e.g. /17, /23, /26, /27 |
| Address allocation | Very wasteful | Allocated exactly as needed |
| Network size known from | The class alone | The prefix length — it must always be stated with the address |
| Modern usage | Historical / legacy | Widely used today |

**Classful practice questions:**

- **Q:** Host `192.168.17.9` (Class C) — how many addresses, first and last? → 256 (2^8); first = 192.168.17.0 (network); last = 192.168.17.255 (broadcast).
- **Q:** Address `185.28.17.9` (Class B /16)? → 65,536 addresses (65,534 usable); first = 185.28.0.0; last = 185.28.255.255.
- **Q:** Class A host `10.200.240.4` (/8)? → 16,777,216 addresses (16,777,214 usable); first = 10.0.0.0; last = 10.255.255.255.

### 2.2 Classless Addressing (CIDR)

Classful addressing wasted addresses and made routing tables explode. CIDR removed the classes.

- Written as **`a.b.c.d/n`** where *n* = number of network bits (the prefix length), e.g. `192.168.4.0/24` means the first 24 bits identify the network.
- The mask has *n* ones followed by (32 − *n*) zeros, e.g. `/26` = `255.255.255.192`.
- **Advantage:** address blocks can be allocated in the exact size needed, and many small networks can be aggregated (summarized) into one route — this reduces router table size.

### 2.3 Subnetting

**Idea:** "borrow" bits from the host part of an address to create smaller sub-networks inside one network. Subnetting reduces broadcast traffic, improves security and stops address waste.

- **Steps:** (1) find how many subnet/host bits are needed; (2) write the new mask; (3) the block size = 256 − mask's interesting octet value; (4) enumerate network addresses, host ranges and broadcast addresses.
- **Number of subnets** = 2^(borrowed bits); **usable hosts per subnet** = 2^(remaining host bits) − 2 (network and broadcast are reserved).

**Worked Example 1: Subnetting `192.168.1.0/24` into 4 subnets**

Need 4 subnets: 2² = 4, so borrow 2 bits → prefix `/26`, mask `255.255.255.192`, block size 64, 62 usable hosts each.

| Subnet | Network address | Host range | Broadcast address |
|---|---|---|---|
| 1 | 192.168.1.0 | 192.168.1.1 – 192.168.1.62 | 192.168.1.63 |
| 2 | 192.168.1.64 | 192.168.1.65 – 192.168.1.126 | 192.168.1.127 |
| 3 | 192.168.1.128 | 192.168.1.129 – 192.168.1.190 | 192.168.1.191 |
| 4 | 192.168.1.192 | 192.168.1.193 – 192.168.1.254 | 192.168.1.255 |

**Worked Example 2: Subnetting for hosts**

Given `172.16.0.0/16`, a department needs 500 hosts per subnet. 2⁹ − 2 = 510 (works), so 9 host bits → prefix `/23`, mask `255.255.254.0`, block size 2 in the third octet. Subnets: `172.16.0.0/23`, `172.16.2.0/23`, `172.16.4.0/23`, ...

**Quick subnetting reference**

| Prefix | Mask | Usable hosts | Prefix | Mask | Usable hosts |
|---|---|---|---|---|---|
| /24 | 255.255.255.0 | 254 | /27 | 255.255.255.224 | 30 |
| /25 | 255.255.255.128 | 126 | /28 | 255.255.255.240 | 14 |
| /26 | 255.255.255.192 | 62 | /30 | 255.255.255.252 | 2 (links) |

**More subnetting practice (exam favourites):**

- **Which subnet is `192.168.10.77/26` in?** Block size = 64; boundaries 0, 64, 128, 192. 77 lies in the 64-block → network 192.168.10.64, hosts .65–.126, broadcast .127.
- **Are `192.168.1.65` and `192.168.1.126` in the same subnet (mask 255.255.255.192)?** Block size 64 → ranges 0–63, 64–127, 128–191, 192–255. Both fall in 64–127 → **yes, same subnet (192.168.1.64/26).**
- **`172.16.0.0/20` — how many hosts?** H = 32 − 20 = 12 → 4096 addresses, **4094 usable**.
- **Smallest subnet for 50 computers?** Need 2^H − 2 ≥ 50: H = 6 gives 62 → **/26 (255.255.255.192).**

**`216.21.5.0` subnetted into 30-host subnets:** need 2^H − 2 ≥ 30 → H = 5 exactly → /27, mask 255.255.255.224, block 32, 8 subnets:

| Subnet | Network | Usable range | Broadcast |
|---|---|---|---|
| 1 | 216.21.5.0 | .1 – .30 | .31 |
| 2 | 216.21.5.32 | .33 – .62 | .63 |
| 3 | 216.21.5.64 | .65 – .94 | .95 |
| 4 | 216.21.5.96 | .97 – .126 | .127 |
| 5 | 216.21.5.128 | .129 – .158 | .159 |
| 6 | 216.21.5.160 | .161 – .190 | .191 |
| 7 | 216.21.5.192 | .193 – .222 | .223 |
| 8 | 216.21.5.224 | .225 – .254 | .255 |

**`150.15.0.0` into 500-host subnets:** need 2^H − 2 ≥ 500 → H = 9 gives 510 → **/23**, mask 255.255.254.0, block size 2 in the third octet. Subnets: `150.15.0.0/23` (hosts 150.15.0.1 – 150.15.1.254, broadcast 150.15.1.255), `150.15.2.0/23`, `150.15.4.0/23`, … up to `150.15.254.0/23` (128 subnets).

### 2.4 Supernetting (Route Aggregation)

- **Idea:** the reverse of subnetting — combine several consecutive classful networks into one larger block, using a shorter prefix.
- **Example:** `192.168.0.0/24`, `192.168.1.0/24`, `192.168.2.0/24` and `192.168.3.0/24` (four Class C networks) can be advertised together as **`192.168.0.0/22`** — one entry instead of four, because the first 22 bits are common.
- Reduces routing table size on the Internet (this summarization is a key benefit of CIDR).

## 3. Forwarding of IP Packets

- Each router keeps a **routing table** with entries: destination network/prefix, subnet mask, next-hop router (or "direct"), outgoing interface, and optionally a metric.
- When a packet arrives, the router **ANDs** the destination IP with each entry's mask; if it matches the entry's network, the packet is forwarded to that entry's next hop.
- **Longest prefix match:** if several entries match, the one with the longest (most specific) prefix wins — this is how CIDR and supernetting work together.
- **Default route:** `0.0.0.0/0` — used when no other entry matches (gateway of last resort).

> **Exam tip:** a typical numerical gives you a routing table and a few destination IPs — AND each IP with each mask, apply longest prefix match, and state the next hop.

## 4. IPv4 Header

The IPv4 header is **20 bytes minimum, up to 60 bytes** with options. Fields (in order):

| Field | Size | Purpose |
|---|---|---|
| Version | 4 bits | IP version in use (4) |
| IHL (Header Length) | 4 bits | Header length in 32-bit words: 5 to 15, i.e. 20 to 60 bytes |
| DSCP + ECN | 8 bits | 6-bit DSCP: priority/QoS treatment; 2-bit ECN: congestion notification from routers |
| Total Length | 16 bits | Header + data length in bytes (max 65,535) |
| Identification | 16 bits | Same value in all fragments of one datagram (for reassembly) |
| Flags | 3 bits | DF = Do not Fragment; MF = More Fragments follow |
| Fragment Offset | 13 bits | Position of this fragment's data, in units of 8 bytes |
| TTL (Time To Live) | 8 bits | Hop counter; decremented each router; packet discarded at 0 (prevents loops) |
| Protocol | 8 bits | Upper-layer protocol: 6 = TCP, 17 = UDP, 1 = ICMP |
| Header Checksum | 16 bits | Integrity check on the header only; recomputed at every hop (TTL changes) |
| Source Address | 32 bits | Sender's IP address |
| Destination Address | 32 bits | Receiver's IP address |
| Options | 0 – 40 bytes | Rarely used: record route, timestamp, strict/loose source route |
| Padding | 0 – 3 bytes | Fills the header up to a 32-bit boundary |

- **IHL examples:** IHL = 5 → 5 × 4 = 20 bytes (no options); IHL = 7 → 28 bytes (8 bytes of options).

## 5. IPv6 Addressing

- IPv6 addresses are **128 bits**, written as 8 groups of 4 hexadecimal digits separated by colons, e.g. `2001:0db8:0000:0000:0000:ff00:0042:8329`.
- Leading zeros in a group may be omitted; **one** run of consecutive zero groups may be replaced by `::`. Example above → `2001:db8::ff00:42:8329`.

### IPv6 header (fixed 40 bytes, no options field)

| Field | Size | Purpose |
|---|---|---|
| Version | 4 bits | 6 |
| Traffic Class | 8 bits | QoS / priority (like DSCP) |
| Flow Label | 20 bits | Marks packets of one flow for special handling |
| Payload Length | 16 bits | Length of data after the 40-byte header |
| Next Header | 8 bits | Type of the next header: an extension header or upper protocol (6 = TCP, 17 = UDP, 58 = ICMPv6) |
| Hop Limit | 8 bits | Same job as IPv4 TTL |
| Source Address | 128 bits | Sender |
| Destination Address | 128 bits | Receiver |

### IPv4 vs IPv6 — key differences

| Feature | IPv4 | IPv6 |
|---|---|---|
| Address size | 32 bits | 128 bits (huge space; restores end-to-end addressing) |
| Header | 20 – 60 bytes, 14 fields | Fixed 40 bytes, 8 fields — faster router processing |
| Header checksum | Present; recomputed every hop | Removed (lower layers already check integrity) |
| Fragmentation | Routers and source can fragment; reassembly at destination | Only the SOURCE fragments; routers never fragment (simpler routers) |
| Options | Variable part of header | Extension headers placed between header and payload |
| Broadcast | Supported | Not supported — replaced by multicast and anycast |
| Configuration | Manual or DHCP | Autoconfiguration (SLAAC) + DHCPv6 |
| Address types | Unicast, broadcast, multicast | Unicast (global, link-local FE80::/10), multicast (FF00::/8), anycast |

- **Anycast (new in IPv6):** one address assigned to several hosts; a packet to it is delivered to the NEAREST one — used for load balancing and finding the closest server.

## 6. Network Layer Protocols (working and purpose)

| Protocol | Full name | Working / purpose |
|---|---|---|
| IP | Internet Protocol | Delivers datagrams between hosts; connectionless, best-effort, no guarantees |
| ICMP | Internet Control Message Protocol | Error reporting and diagnostics between hosts/routers (destination unreachable, time exceeded); ping uses Echo Request / Echo Reply |
| ARP | Address Resolution Protocol | Finds the MAC address of a host on the same LAN, given its IP (request broadcast, reply unicast) |
| RARP | Reverse ARP | Finds the IP given the MAC — superseded by BOOTP and DHCP |
| IGMP | Internet Group Management Protocol | Lets hosts join / leave multicast groups; routers use it to track group members on each interface |
| DHCP | Dynamic Host Configuration Protocol | Automatically assigns IP address, mask, gateway and DNS to hosts at boot time (detailed in Chapter 2.2) |
| NAT | Network Address Translation | Maps many private addresses to one (or few) public IP(s) at the router — conserves IPv4 addresses || ND | Neighbor Discovery (IPv6) | The IPv6 replacement for ARP: resolves IPv6 addresses to MAC addresses, plus router discovery and prefix information (uses ICMPv6) |

---

# Chapter 2.2 — Routing Algorithms, DHCP and VLAN

## Routing Fundamentals

### Routing vs Forwarding

| Routing | Forwarding |
|---|---|
| Finds/learns the path | Sends the actual packet |
| Builds and maintains the routing table | Uses the table — one lookup per packet |
| Concerned with path selection | Concerned with packet movement |
| "Use R2 to reach network B" | "Send this packet to R2" |

### Metric
A value used by a routing protocol to compare routes and pick the preferred path. **Shortest path = path with the lowest metric**, not necessarily the shortest physical distance.

| Metric | Meaning |
|---|---|
| Hop count | Number of routers/hops to the destination |
| Cost | An assigned value for a link or path |
| Bandwidth | Capacity/speed of the link |
| Delay | Time required to send data over the link |
| Reliability | How stable the link is |
| Load | How busy the link currently is |

### How a router learns routes
1. **Connected routes** — networks directly attached to the router's interfaces.
2. **Static routes** — manually configured by the administrator.
3. **Dynamic routes** — learned automatically via routing protocols (RIP, OSPF, EIGRP, BGP).

### Convergence
**Convergence** is the process by which, after a network change, all routers exchange updates, recalculate routes and reach a consistent view of the network again: detect failure → exchange updates → select new routes → tables stabilize. Fast convergence is a key quality of any routing protocol.

## 1. Shortest Path Algorithm (Dijkstra)

- **Goal:** find the least-cost path between a source and every other node in the network, where cost can be hops, distance, delay or a money measure.
- **Method:** each node is labelled with its best-known distance from the source; the node with the smallest tentative label is made permanent, and its neighbours are relaxed (updated). Repeat until all nodes are permanent.

**Worked example**

Topology and link costs: A–B = 2, A–C = 1, B–C = 2, B–D = 1, C–D = 4, C–E = 3, D–E = 1. Find shortest paths from A.

| Step | Permanent set N' | B | C | D | E |
|---|---|---|---|---|---|
| 0 | A | 2 (via A) | 1 (via A) | ∞ | ∞ |
| 1 | A, C | 2 (via A) | — | 5 (via C) | 4 (via C) |
| 2 | A, B, C | — | — | 3 (via B) | 4 (via C) |
| 3 | A, B, C, D | — | — | — | 4 (tie: via C or via D) |
| 4 | all | — | — | — | — |

Result: A→C = 1, A→B = 2, A→B→D = 3, A→C→E = 4 (or A→B→D→E = 4).
**Relaxation rule used at every step:** `D(v) = min[ D(v), D(u) + c(u,v) ]` — if reaching v through the newly permanent node u is cheaper than v's current label, update it.


> **Complexity:** O(n²) naively; can be reduced with heap data structures. Used inside link state routing (OSPF).

## 2. Distance Vector Routing (DVR)

- Used by **RIP**. Each router keeps a table with the **best known distance to every destination and the next hop** — a vector of distances.
- Routers periodically exchange their entire tables with **directly connected neighbours only**.
- On receiving a neighbour's table, a router applies the **Bellman-Ford update**: D_x(y) = min over all neighbours v of [ c(x,v) + D_v(y) ] — if a cheaper route via that neighbour is found, update the entry.

**Worked example: line network A — B — C — D (each link cost 1)**

| Router A learns (final) | Distance | Next hop | How it was learned |
|---|---|---|---|
| A to B | 1 | B | direct neighbour |
| A to C | 2 | B | C is 1 from B (1 + 1) |
| A to D | 3 | B | D is 2 from B (2 + 1) |

**Main rule of DV:** cost to a destination through a neighbour = (cost to that neighbour) + (neighbour's advertised cost to the destination). Keep the minimum over all neighbours.

**Initial knowledge of every router:** (1) distance to itself = 0; (2) distance to directly connected neighbours = the link cost; (3) distance to all unknown destinations = infinity.

**Advantages of Distance Vector:**

| Advantage | Explanation |
|---|---|
| Simple concept | Easy to understand and implement |
| Low memory and CPU | No complete topology map; simple calculations |
| Automatic route learning | Dynamically learns and updates remote routes |
| Neighbour-based exchange | Only talks to directly connected neighbours |
| Suitable for small networks | Works well when network size is limited |

**Disadvantages of Distance Vector:**

| Disadvantage | Explanation |
|---|---|
| Slow convergence | Long time for information to stabilize after a change |
| Count-to-infinity | Bad news (failures) propagates slowly; metric inflates to infinity (16 = infinity in RIP) |
| Routing loops | More prone to loops during convergence |
| Periodic update overhead | Exchanges full tables at fixed intervals even when nothing changed |
| Limited scalability | Not suitable for very large networks |
| No topology view | Router knows distances, not the actual map |

**Count-to-infinity problem**

- If a link goes down, two neighbours can keep telling each other "I can reach X through you" — bad news travels slowly; the estimated distance rises to infinity (16 = infinity in RIP).
- **Fixes:** split horizon (never advertise a route back to the neighbour you learned it from), split horizon with poison reverse (advertise it back with cost infinity), and hold-down timers (ignore updates about a failed route for a while).

## 3. Link State Routing (LSR)

- Used by **OSPF** and IS-IS. Every router learns the **complete map** of the network and runs Dijkstra on it.

**Five steps**

1. **Discover neighbours:** send HELLO packets on each link; record who replies.
2. **Measure cost:** measure delay / bandwidth to each neighbour (echo packets with timestamps).
3. **Build Link State Packet (LSP):** containing (router ID, neighbours, costs, sequence number, age).
4. **Distribute LSPs by flooding:** each router forwards the LSP on all links except the one it arrived on, using sequence numbers to reject duplicates/loops (acknowledged for reliability).
5. **Compute shortest paths:** once every router has all LSPs, it reconstructs the full topology and runs Dijkstra to build its routing table.

**Advantages of Link State:**

| Advantage | Explanation |
|---|---|
| Fast convergence | Topology changes are flooded immediately |
| Complete topology knowledge | Every router holds the full map (LSDB) |
| No count-to-infinity | Not affected by the classic DV problem |
| Better scalability | Areas/hierarchy support very large networks (OSPF) |
| Efficient path calculation | Dijkstra gives provably least-cost paths |
| Robust | One router's bad information harms only itself |

**Disadvantages of Link State:**

| Disadvantage | Explanation |
|---|---|
| More complex | Harder to understand and configure |
| Memory-hungry | Stores the entire link-state database |
| CPU-intensive | Runs Dijkstra on every significant change |
| Flooding overhead | LSAs consume bandwidth during distribution |
| Heavy initial sync | Large topology exchange when routers boot |
| Needs stronger routers | More resources required than DV |

**Distance Vector vs Link State**

| Point | Distance Vector | Link State |
|---|---|---|
| Knowledge | Only distance to destinations, via neighbours' summaries | Full topology map of the whole network |
| Information shared | Entire routing table with neighbours only | LSPs flooded to all routers |
| Algorithm | Bellman-Ford (distributed) | Dijkstra (run locally on complete map) |
| Update trigger | Periodic (every 30 s in RIP) plus triggered updates | On change only (faster convergence) |
| Bandwidth use | Small messages but frequent | Heavy at start / on changes only |
| Convergence | Slow; count-to-infinity problem | Fast; loops less likely |
| Robustness | Bad information propagates | One router's error affects only itself |
| Examples | RIP | OSPF, IS-IS |

## 4. Hierarchical Routing

- As networks grow, routing tables become huge. **Idea:** divide routers into **regions**; a router knows full detail about its own region but treats other regions as a single destination.
- Trade-off: smaller tables and less update traffic, at the cost of slightly longer routes than optimal.
- For *n* routers, the optimal number of hierarchy levels is approximately **ln n**, giving an average table size of about e · ln n entries.
- This is exactly how the Internet works: networks, then areas, then autonomous systems (domains).

**Flat vs Hierarchical Routing:**

| Feature | Flat | Hierarchical |
|---|---|---|
| Network organization | One broad structure | Divided into regions/areas |
| Best for | Small networks | Large networks |
| Routing information | Grows large | Summarized / reduced |
| Scalability | Low | High |
| Example | Single-area network | Multi-area OSPF, Internet AS hierarchy |

**Analogy:** a postal address works hierarchically — Country → State → City → Sector → House; your local post office does not need every house number in the country.

**Advantages:** better scalability, smaller routing tables (summarization), changes stay local, less memory/CPU, easier management.
**Disadvantages:** more complex design and configuration, may produce non-optimal paths, harder troubleshooting, poor area design hurts performance.

## 5. Broadcast Routing

Delivering a packet from one source to **all** hosts. Four techniques:

- **Flooding:** send on every outgoing link except the one it arrived on. Simple and robust but generates duplicate packets — controlled by a **sequence number per source**: a router forwards a packet only if it has not seen that sequence number before.
- **Multi-destination routing:** the packet carries a list of destinations; each router picks out the destinations reachable through each outgoing line, splits the list and forwards — one copy per needed link.
- **Reverse Path Forwarding (RPF):** a broadcast packet is forwarded only if it arrives on the link that lies on the shortest path back to the source — eliminates most duplicates cheaply, one copy per LAN.
- **Spanning-tree routing:** build a spanning tree covering all LANs; each packet is forwarded only along tree branches — the most efficient scheme, one copy per link (basis of multicast routing).

**Unicast vs Broadcast vs Multicast:**

| Feature | Unicast | Broadcast | Multicast |
|---|---|---|---|
| Communication | One-to-one | One-to-all in scope | One-to-selected group |
| Receiver | One specific host | Every host in the broadcast domain | Only group members |
| Example | PC → web server | ARP request on a LAN | Video stream to a group |
| IPv4 | Yes | Yes | Yes |
| IPv6 | Yes | Not supported | Yes (replaces broadcast) |

**Advantages of broadcast:** simple one-to-all delivery, essential for discovery (ARP), robust (flooding finds a path even with failures).
**Disadvantages:** high bandwidth consumption, duplicate packets, routing loops if uncontrolled, processing overhead on every host, poor scalability, security concerns.

## 6. Unicast Routing

- Delivery of a packet from **one source to one destination** — the normal case handled by all the algorithms above (DVR, LSR, Dijkstra).
- Destination-based forwarding: the router looks only at the destination address in the routing table — one table lookup per packet, independent of the source.

**Information a unicast routing/forwarding table entry contains:**

| Information | Meaning |
|---|---|
| Destination prefix | The network to be reached |
| Prefix length / mask | Defines that network |
| Next hop | The next router toward the destination |
| Outgoing interface | The interface used to send the packet |
| Metric / preference | Chooses among multiple possible routes |

**Advantages:** precise one-to-one delivery, the common Internet model, flexible routing, scalable destination-based forwarding.
**Disadvantages:** inefficient when the same data must reach many receivers — one separate copy per receiver is needed.

## 7. DHCP — Dynamic Host Configuration Protocol

- **Purpose:** automates IP configuration. A host boots with no IP and gets one automatically: address, subnet mask, default gateway, DNS server(s) and the lease time — all in one transaction.
- **Transport:** UDP; server port 67, client port 68. Works across routers via a DHCP relay agent.

**DORA — the four-step exchange**

| Step | Message | Who sends | What happens |
|---|---|---|---|
| 1 | DHCP Discover | Client → broadcast | "I need an IP" (to 255.255.255.255, since client has no address) |
| 2 | DHCP Offer | Server → client | One (or more) servers offer an address + lease parameters |
| 3 | DHCP Request | Client → broadcast | Client accepts one offer (broadcast so other servers withdraw theirs) |
| 4 | DHCP Acknowledge | Server → client | Lease confirmed; client configures the address and starts the timer |

- **Allocation methods:** automatic (permanent), dynamic (leased, returned when host leaves — reuses scarce addresses), manual (admin fixed).

**DHCP terminology:**

| Term | Meaning |
|---|---|
| DHCP pool | The set of configuration the server may hand out for a network (address range + mask + gateway + DNS) |
| Excluded addresses | Addresses the admin reserves so DHCP never assigns them (e.g. servers, gateways) |
| Lease | The time an address is granted for; the client must renew before expiry |

**Advantages of DHCP:** automatic configuration, less manual work and fewer mistakes, no IP conflicts, centralized management, leases recycle scarce addresses, scales to large networks.
**Disadvantages:** hosts depend on the DHCP service being up, rogue DHCP servers are a security risk, the address can change between leases, separate VLANs need proper DHCP relay design.

## 8. VLAN — Virtual LAN

- **Definition (IEEE 802.1Q):** a group of devices that communicate as if on the same LAN, regardless of their physical location — membership is configured in the switch, not by cabling.
- Switches tag frames with a **4-byte VLAN ID** (inserted after the source address); switches strip or add tags between trunk links.

**Benefits**

- Each VLAN is a separate **broadcast domain** — broadcast traffic stays inside the VLAN (better performance).
- **Security and management:** departments are isolated; users can only reach services of their own group.
- **Flexibility:** moving a user to another department = a configuration change, no rewiring.
- **Types:** port-based (switch port membership — most common), MAC-based (by device address), and protocol-based.

---

**Access Port vs Trunk Port:**

| Access Port | Trunk Port |
|---|---|
| Carries traffic for **one** VLAN | Carries traffic for **many** VLANs |
| Connects end devices (PC, printer) | Connects switches (or switch ↔ router) |
| Frames are untagged | Frames carry an 802.1Q VLAN tag |

**Disadvantages of VLANs:** requires configuration and management; devices in different VLANs need a Layer-3 device (router / L3 switch) to communicate; a wrong VLAN assignment isolates the device; troubleshooting is more complex than in a single flat LAN.

---

# Chapter 2.3 — Congestion Control, Tunnelling and Fragmentation

## 1. Congestion — what and why

- **Congestion:** too many packets in (part of) the subnet — demand exceeds capacity, queues grow, delay rises sharply, and routers start dropping packets.
- **Causes:** sudden bursts of traffic, too many hosts sending at full speed, slow processors/links, insufficient memory (buffer overflow), retransmissions of lost packets making it worse (congestion collapse).
- Congestion is a **global** phenomenon involving many hosts and routers; flow control (one sender, one receiver) cannot solve it.

## 2. Congestion Prevention Policies

**Open-loop (prevention):** act before congestion happens.

- **Traffic shaping:** regulate the rate/pattern at which traffic is injected into the network — leaky bucket and token bucket (below).
- **Admission control:** for connection-oriented networks — a new virtual circuit is admitted only if the network can carry it without degrading existing ones (negotiated at setup).

**Open-loop policies in detail:**

| Policy | Meaning | Example |
|---|---|---|
| Retransmission policy | Avoid unnecessary retransmissions, which add load | Do not resend a packet too quickly |
| Window policy | Limit how many packets may be unacknowledged | Allow only 5 unacked packets |
| Acknowledgement policy | Reduce acknowledgement traffic | Use cumulative ACKs for several packets |
| Discarding policy | Drop packets early when necessary | Discard a low-priority packet during heavy load |
| Admission policy | Refuse new traffic when resources are insufficient | Reject a new connection when overloaded |

**Closed-loop (control):** monitor → detect → convey → adjust. Routers watch queue lengths / utilization; when a threshold is crossed they inform sources; sources then slow down (covered below and in TCP).

### Router Buffers and Queue Growth

- Router buffer = temporary memory holding packets waiting to be forwarded.
- **Queue growth rate = R_in − R_out** (arrival rate minus outgoing rate).
- **Time to fill the buffer:** T = (Buffer capacity − Current queue) / (R_in − R_out)

**Worked examples:**
- Buffer = 1000 packets, queue empty, R_in = 1200 pkt/s, R_out = 1000 pkt/s → growth 200 pkt/s → T = 1000/200 = **5 seconds**.
- Buffer = 2000, current queue = 500 (free space 1500), R_in = 1400, R_out = 900 → growth 500 pkt/s → T = 1500/500 = **3 seconds**.

| Condition | What happens |
|---|---|
| R_in < R_out | Router keeps up; the queue shrinks |
| R_in = R_out | No sustained queue growth |
| R_in > R_out | Queue grows steadily |
| Buffer full | **Buffer overflow** — arriving packets must be dropped (**tail drop**: the packet at the tail of the full queue is discarded) |

Effects of overflow: packet loss → retransmissions → even more traffic → higher delay → worse congestion.

## 3. Congestion Control in Datagram Subnets

- **Warning bits:** routers set a special bit in the packet header when congestion is building; the destination echoes the warning back to the source in its reply, and the source slows down.
- **Choke packets:** the congested router sends an explicit control packet directly to the source ("slow down to X rate"). Sources that ignore repeated choke packets get dropped.
- **Hop-by-hop choke packets:** each router on the path applies the relief immediately (relieves congestion faster along the path, at the cost of more control packets).

**Load shedding**

- When buffers are full, routers must drop packets — choosing **which** to drop is load shedding.
- **Wine vs milk policy:** drop old packets (wine gets better with age) or fresh ones (old milk goes bad first — VoIP prefers dropping old audio).
- **RED (Random Early Detection):** drop packets randomly **before** buffers are completely full, so all sources slow down together gently instead of one flow losing everything.

**Traffic discarding vs Load shedding:**

| Traffic Discarding / Buffer Overflow | Load Shedding |
|---|---|
| Packet dropped because no buffer space is available | Packets **deliberately** discarded to relieve severe congestion |
| A consequence of a full queue | A congestion-control strategy |
| Storage/resource limitation is the problem | Reducing load is the objective |

**Load shedding strategies:**
1. **Random drop** — discard arriving packets with a fixed probability (e.g. 10%).
2. **Tail drop** — drop the newest packet when the queue is full.
3. **Weighted drop** — lower-priority packets are dropped first.
4. **Early drop** — drop before the buffer is completely full so congestion eases before collapse (the idea behind RED).

**Jitter control**

- **Jitter** = variation in end-to-end packet delay. Even with low average delay, high jitter ruins real-time traffic (VoIP, video).
- Fix: **buffer packets at the receiver** and play them out at constant intervals — smooths delay variation; trade-off is added fixed delay.

## 4. Leaky Bucket Algorithm

- **Analogy:** a bucket with a small hole at the bottom. Water (packets) can be poured in at any rate, but it leaks out at a constant rate; if the bucket overflows, the excess is lost.
- **Working:** a host sends packets into a queue (the bucket); a scheduler releases exactly one packet every fixed time interval (constant output rate), regardless of how bursty the input is.
- **Effect:** converts a **bursty stream into a perfectly even stream** at the leak rate.
- **Properties:** output rate is always constant; idle periods cannot be "saved up" — bursts are lost; simple queue with a timer; may waste capacity when the network is idle.

- **Maximum burst duration (leaky bucket):** T_b = B / (C − R), where B = bucket capacity, C = peak arrival rate, R = leak rate. Note: the leaky bucket *limits* bursts by overflowing — it does not allow them.
- **Worked example:** B = 30 MB, C = 10 MB/s arrival, R = 4 MB/s leak → net fill 6 MB/s → T_b = 30/6 = **5 seconds** before overflow.
- **Unit reminder:** 1 MB = 8 Mbit — data volume and transmission rate must be in consistent units.
- **Drawbacks:** no bursts allowed (fixed output), drops packets on overflow, adds queuing delay, and idle-time capacity is wasted — the output never exceeds the leak rate even when the link is completely free.

## 5. Token Bucket Algorithm

- **Working:** tokens are generated at a constant rate (ρ) into a bucket of capacity *C*. To send a packet of size *S*, the host must remove *S* tokens; if not enough tokens are available, the packet waits (or is dropped).
- **Effect:** long-term rate is limited to the token rate, but **bursts up to bucket capacity are allowed** — unlike leaky bucket.
- **Idle hosts accumulate tokens** (up to *C*), so saved capacity can be used in a later burst.
- **Maximum burst length:** *S* = *C* / (*M* − ρ), where *M* is the maximum output rate of the link.

**Worked example**

Bucket capacity *C* = 500 KB, tokens generated at ρ = 10 MB/s, maximum output rate *M* = 50 MB/s. Maximum burst length *S* = *C* / (*M* − ρ) = 0.5 MB / 40 MB/s = **12.5 ms**.

**Idle-time examples (how tokens accumulate):**
- B = 20 tokens, r = 4 tokens/s, idle for 3 s → 12 tokens generated (< capacity) → up to **12 packets** may burst out immediately.
- B = 20 tokens, r = 5 tokens/s, idle for 6 s → 30 tokens generated, but the bucket holds only 20 → **10 tokens are discarded**; max burst = 20 packets.
- When the token bucket is full, the excess **tokens** are discarded — packets are not dropped just because the token bucket is full.

**Leaky Bucket vs Token Bucket**

| Point | Leaky Bucket | Token Bucket |
|---|---|---|
| Bucket holds | Packets (queue) | Tokens |
| Output rate | Strictly constant | Average = token rate; short bursts up to capacity allowed |
| Idle sender | Gains nothing; input bursts are lost | Accumulates tokens (up to capacity) for later |
| Network capacity | Wasted when idle | Used efficiently when idle |
| Typical use | Hard rate limiting | Traffic shaping allowing controlled bursts |

**Why jitter occurs:**

| Cause | What happens |
|---|---|
| Network congestion | Packets wait longer in queues |
| Variable queuing delay | Queue lengths change over time |
| Different routes | Packets may take paths with different delays |
| Processing variation | Routers/hosts take different times to handle packets |
| Traffic bursts | Sudden surges grow the queues unpredictably |

**Jitter buffer:** a small buffer at the receiver where packets wait before playback: irregular arrival → jitter buffer → regular playback. It adds a little fixed delay but greatly improves voice/video quality.

## 6. Tunnelling

- **Definition:** to move a packet through a network that speaks a different protocol, the whole packet is placed as **data** inside the payload of another packet; at the far end it is extracted (decapsulated) and forwarded normally.
- **Uses:** carrying IPv6 packets across IPv4 networks (6to4); VPNs — private IP packets wrapped in public Internet packets, encrypted; multicast through unicast-only networks; connecting dissimilar WANs.
- Like putting a letter inside a bigger envelope so a foreign postal system can carry it.

**Advantages of tunnelling:** connects dissimilar or physically separated networks, supports IPv4↔IPv6 transition, enables VPNs, can add security.
**Disadvantages:** extra header overhead, encapsulation/decapsulation processing, higher delay, MTU problems (the wrapped packet may exceed the MTU), harder troubleshooting.

**Second fragmentation example: MTU = 1000 bytes, header = 20 bytes, original data = 2980 bytes**

Max data per full fragment = 976 bytes (largest multiple of 8 that fits in 1000 − 20 = 980).

| Fragment | Data | Header | Total | Offset (bytes/8) | MF |
|---|---|---|---|---|---|
| F1 | 976 | 20 | 996 | 0 | 1 |
| F2 | 976 | 20 | 996 | 122 | 1 |
| F3 | 976 | 20 | 996 | 244 | 1 |
| F4 | 52 | 20 | 72 | 366 | 0 |

Check: 976 × 3 + 52 = 2980. Every fragment must carry its own 20-byte header.

## 7. Internetworking and Internetwork Routing

- **Internetwork (internet):** a collection of networks (LANs, WANs) connected by routers/gateways, appearing as one network to hosts.
- **Problems when routing between different networks:** different addressing schemes, different MTUs (needs fragmentation), different protocols/quality of service, security and administration — the network layer hides all this from the transport layer.
- Routers forward packets hop-by-hop across network boundaries; routing protocols (RIP, OSPF, BGP) exchange reachability so every router knows where to send traffic for distant networks.
- Gateways translate between completely different protocols/architectures (stronger than a router, which connects networks sharing the same protocol family).

## 8. Fragmentation

- Each network (link) has a **Maximum Transmission Unit (MTU)** — the largest frame payload it can carry. A datagram bigger than the MTU must be split.
- IPv4: routers along the path fragment when needed; **reassembly only at the final destination** (so fragments can take different routes).
- IPv6: routers never fragment — if a packet is too big, the router sends back an ICMPv6 "packet too big" message and the SOURCE must fragment.
- **Fields used:** Identification (all fragments of one datagram carry the same value), MF (More Fragments — 1 on all but the last), DF (Don't Fragment — if set and packet too big, it's dropped + ICMP error), Fragment Offset (position of this fragment in the original data, measured in **units of 8 bytes**).

**Worked example**

A 4000-byte IP datagram (20-byte header + 3980 bytes data) must cross a network with MTU 1500 bytes. Max data per fragment = 1480 (must be a multiple of 8).

| Fragment | Total length | Data bytes | MF | Fragment offset (bytes / 8) |
|---|---|---|---|---|
| 1 | 1500 | 1480 | 1 | 0 (0 / 8) |
| 2 | 1500 | 1480 | 1 | 185 (1480 / 8) |
| 3 | 1040 | 1020 | 0 | 370 (2960 / 8) |

- Check: 1480 + 1480 + 1020 = 3980 data bytes. Offsets are 0, 185, 370 (each offset = previous offset + previous data size / 8).
- The destination uses Identification + MF + offset to reassemble in order; the timer discards fragments if some never arrive.

---

# Last-Minute Revision — Numbers and One-Liners

- IPv4 = 32 bits; IPv6 = 128 bits; IPv4 header 20–60 B; IPv6 header fixed 40 B; max IPv4 datagram = 65,535 B.
- Default masks: A /8 (255.0.0.0), B /16 (255.255.0.0), C /24 (255.255.255.0).
- /26 = 62 hosts, /27 = 30 hosts, /30 = 2 hosts; block size = 256 − mask octet.
- RIP = distance vector (metric = hop count, max 15); OSPF = link state (Dijkstra).
- Count-to-infinity fixed by: split horizon, poison reverse, hold-down timers.
- Optimal hierarchy levels = ln *n* routers.
- DHCP DORA: Discover, Offer, Request, Acknowledge; UDP ports 67 (server) / 68 (client).
- VLAN = IEEE 802.1Q tag = 4 bytes; each VLAN = separate broadcast domain.
- Leaky bucket = constant rate; token bucket = average rate + bursts (*S* = *C* / (*M* − ρ)).
- Fragment offset units = 8 bytes; IPv4 reassembly at destination only; IPv6 source fragments only.
- Protocol field: 6 = TCP, 17 = UDP, 1 = ICMP.

## Key Formulas (quick reference)

| Formula | Meaning |
|---|---|
| H = 32 − P | Host bits from prefix length |
| 2^H / 2^H − 2 | Total addresses / usable hosts |
| 2^n | Number of subnets (n = borrowed bits) |
| New prefix = old prefix + borrowed bits | Subnetting prefix |
| Block size = 256 − mask octet | Subnet increment |
| Queue growth rate = R_in − R_out | Router queue growth |
| T = (Capacity − Current queue) / (R_in − R_out) | Time to fill buffer |
| T_b = B / (C − R) | Max burst duration — Leaky Bucket |
| T_b = B / (C − r) | Max burst duration — Token Bucket |
| D_x(y) = min_v { c(x,v) + D_v(y) } | Bellman-Ford (Distance Vector) |
| D(v) = min[ D(v), D(u) + c(u,v) ] | Dijkstra relaxation step |

## Key Algorithms (quick reference)

| Algorithm | Used in | Purpose |
|---|---|---|
| Bellman-Ford | Distance Vector routing | Shortest paths from neighbours' information (RIP) |
| Dijkstra | Link State routing | Shortest paths from the topology map (OSPF) |
| Leaky Bucket | Traffic shaping | Smooth bursts into a constant rate |
| Token Bucket | Traffic shaping | Allow controlled bursts up to bucket capacity |
| DORA | DHCP | Four-step IP address assignment |

## Most likely exam questions from this unit

1. Explain the IPv4 header field by field (draw the diagram).
2. Subnet a given network into N subnets / for N hosts — full table with ranges and broadcast.
3. Differentiate IPv4 and IPv6; explain IPv6 addressing notation and header.
4. Explain distance vector routing; illustrate count-to-infinity and its remedies.
5. Explain link state routing and its five steps; compare with distance vector.
6. Run Dijkstra's algorithm on a given graph (labelled steps).
7. Explain DHCP with the DORA sequence; explain VLAN and its advantages.
8. Compare leaky bucket and token bucket (with the burst-length formula).
9. Explain fragmentation with a worked example (MTU calculation).
10. What is congestion? Explain congestion control in datagram subnets: choke packets, load shedding, jitter control; explain tunnelling.

**All the best for your mid-sems!**
