# BGP tunnel: What It Really Is, Why the Tunnel Comes Up Before the Session Does, and Which SixtyNet Plan Gives You a BGP Peer (Full Price Table)

BGP does not build tunnels. A BGP UPDATE can carry a Tunnel Encapsulation attribute — type code 23 in the IANA path-attribute registry, defined in RFC 9012 — and that attribute tells the receiving router which encapsulation to use and where the far end of the tunnel sits. The forwarding itself is done by the data plane, and that split explains most of the confusion around the phrase "BGP tunnel."

Two kinds of readers land on this page. One is debugging: the IPsec tunnel shows UP and traffic still dies. The other is buying: you want a provider that will actually run a BGP session with your router or VPS, and most VPS hosts won't. SixtyNet lists a provider-side BGP session on exactly one plan, at $2,500.00 a year, so the rest of this article is mostly about deciding whether you are that buyer.

> **Quick pick.** If you need a provider-side BGP session plus an AS9929 path into China, take **Los Angeles China Unicom-Premium at $2,500.00 USD per year** (list $3,000.00): free BGP session setup, 100M base network with 3× burst, 95th-percentile billing, quarterly and annual terms. If you only need a box to terminate a tunnel, **LAX-STARTER at $200.00 USD per year** covers 2 cores, 2 GB RAM, 40 GB NVMe and 5 TB of traffic. 👉 [Check both plans and current prices on SixtyNet](https://sixtynet.net/aff.php?aff=39)

## SixtyNet plans, prices and what each one is for

Every price below is billed annually and shown next to the list price the site displays. SixtyNet advertises a saving of 17% on most tiers and 13% on Residential-STARTER.

| Plan | Compute | Network & traffic | Price (annual) | Order |
| --- | --- | --- | --- | --- |
| Los Angeles China Unicom-Premium | Not listed on the pricing page | China Unicom Premium AS9929, 100M base with 3× burst, 95th-percentile billing, free BGP session setup, limited BGP support | $2,500.00 (list $3,000.00) | [Order Unicom-Premium](https://sixtynet.net/aff.php?aff=39) |
| LAX-STARTER | 2 core / 2 GB / 40 GB NVMe / 12,500 IOPS | 2G premium network, 5 TB traffic, DDoS protection | $200.00 (list $240.00) | [Order LAX-STARTER](https://sixtynet.net/aff.php?aff=39) |
| LAX-MICRO | 4 core / 4 GB / 80 GB NVMe / 12,500 IOPS | 4G premium network, 10 TB traffic, DDoS protection | $400.00 (list $480.00) | [Order LAX-MICRO](https://sixtynet.net/aff.php?aff=39) |
| LAX-MEDIUM | 8 core / 8 GB / 160 GB NVMe / 25,000 IOPS | 10G premium network, 20 TB traffic, DDoS protection | $800.00 (list $960.00) | [Order LAX-MEDIUM](https://sixtynet.net/aff.php?aff=39) |
| LAX-LARGE | 8 core / 16 GB / 320 GB NVMe / 25,000 IOPS | 10G premium network, 40 TB traffic, DDoS protection | $1,600.00 (list $1,920.00) | [Order LAX-LARGE](https://sixtynet.net/aff.php?aff=39) |
| Residential-STARTER | 2 core / 2 GB / 40 GB NVMe / 12,500 IOPS | 2G premium network, 5 TB traffic, residential IP | $260.00 (list $300.00) | [Order Residential-STARTER](https://sixtynet.net/aff.php?aff=39) |
| IRVINE-PRO-STARTER | 2 core / 2 GB / 40 GB NVMe / 12,500 IOPS | 2G premium network, AS9929 optimised for China, 5 TB traffic, residential IP | $350.00 (list $420.00) | [Order IRVINE-PRO-STARTER](https://sixtynet.net/aff.php?aff=39) |

The page also lists configurations without headline pricing in the version I checked: residential tiers at 4, 8 and 16 cores with 10 TB, 20 TB and 40 TB of traffic, plus two RAID 10 storage tiers (4 core with 300 GB + 200 GB bonus, 8 core with 700 GB + 300 GB bonus, 10 TB and 15 TB traffic respectively). Confirm those on the order page rather than assuming they scale linearly from the table above.

Three limitations worth knowing before you pay:

| Plan group | Restriction |
| --- | --- |
| Residential-STARTER and the larger residential tiers | Ports 80 and 443 are closed, stated as an IP risk-control measure |
| IRVINE-PRO (all tiers) | Inbound probes and scans are rejected; access only through configured ports or a whitelist. Refunds are issued as website balance only |
| Los Angeles China Unicom-Premium | BGP support is described as "limited"; the free part covers session setup |

That last row matters most for this article. "Set up BGP session for free" plus "limited BGP technical support" leaves the scope undefined: how many sessions, whether SixtyNet announces your prefix or expects you to, whether they'll accept a 32-bit ASN, whether they filter prefixes longer than /24. Ask before ordering, because a session that comes up and then filters everything you send is worse than no session.

## What a BGP session does for a tunnel, and what it doesn't

A BGP session is a TCP/179 peering that exchanges reachability information. The tunnel is a separate encapsulation agreed by both ends. Pairing them buys you three things: automatic withdrawal when a path dies, prefix-level policy over which traffic uses which tunnel, and the ability to advertise networks you own into someone else's routing table.

An old Reddit thread from 2015 put the distinction bluntly: "BGP doesn't get you a tunnel. There is no such thing as a 'BGP tunnel.'" Literally, that sentence is still correct. What changed is that RFC 9012 (February 2021) standardised how BGP carries the parameters needed to build a tunnel, which is why the shorthand stuck even among people who know better.

The practical consequence: if a provider sells you "BGP," ask what the session is for. A session that only exchanges loopback addresses so you can reach a looking glass is not the same product as a session that lets you announce a /24 and steer it into an IP-IP or VXLAN tunnel.

## What RFC 9012 actually specifies

The Tunnel Encapsulation attribute is an optional transitive BGP path attribute with type code 23. It carries a set of TLVs; each Tunnel TLV has a 2-octet Tunnel Type, a 2-octet Length and a variable Value made of sub-TLVs. Each sub-TLV has a 1-octet Type, a Length of 1 or 2 octets, and a Value.

The length rule is the detail most summaries skip. Sub-TLV types 0 through 127 use a 1-octet Length field; types 128 through 255 use 2 octets. That is one of five defects RFC 9012 fixed in RFC 5512, which it obsoletes. RFC 5512 was never deployed in production, and because RFC 5566 depended on it, RFC 5566 is obsoleted too.

The other fixes: the Encapsulation SAFI is deprecated (it was never used, and it forced every session that might carry tunnel data to be reconfigured); a new Tunnel Egress Endpoint sub-TLV lets you name a tunnel's remote endpoint directly instead of encoding it in SAFI NLRI; the attribute can ride on more AFI/SAFI combinations; and new sub-TLVs describe outer IP and UDP header fields. Encapsulation sub-TLVs are now defined for VXLAN, NVGRE and MPLS-in-GRE. MPLS-in-UDP is supported without one, since it has no extra parameters to signal.

Two more pieces worth knowing. The color and protocol sub-TLVs let a router decide whether a given packet belongs in the tunnel described by that TLV. And the Tunnel Egress Endpoint Value length must be the address-family length plus 6, so 10 for IPv4, 22 for IPv6 and 6 when the address family is zero; an address inside a special-purpose block flagged as destination or non-forwardable is an error unless explicitly permitted.

If you are reading older material: code points 3, 4, 5 and 6 (Transmit tunnel endpoint, IPsec in Tunnel-mode, IP in IP with IPsec Transport Mode, MPLS-in-IP with IPsec Transport Mode) are deprecated, along with sub-TLV 3, IPsec Tunnel Authenticator. Don't build to them.

## Three places this runs in production

**Juniper, MPLS to IP fabric.** IP fabric cores don't carry MPLS VPN labels, so the egress PE has no label to identify the VRF. Junos supports BGP L3VPN over IP-in-IP tunnels that terminate straight into the service-layer VRF, which removes the need for a service label. When an IP-IP tunnel is selected, the application service label is ignored and a firewall filter demultiplexes traffic; L3VPN traffic is decapsulated and looked up in the VRF, while IPv4 and IPv6 traffic goes to inet.0 and inet6.0. From Junos OS 20.3R1 you can choose MPLS transport or IP-IP based on the tunnel attribute. A receiver on 20.2 or earlier ignores the attribute and falls back to MPLS, so mixed-version networks need that in the migration plan.

**AWS Site-to-Site VPN.** Every connection gives you two tunnels, and the design intent is availability, not a spare sitting idle. A typical setup pairs an on-premises firewall at ASN 65010 with a Transit Gateway at ASN 64512; mismatched ASNs keep the session down while the IPsec tunnel stays up. That is the sentence to remember: VPN UP does not mean BGP is UP.

**Azure VPN Gateway.** BGP is optional on route-based gateways, and without it you are running static routes. The minimum you can advertise to a peer is a host prefix, /32. You can advertise something large like 10.0.0.0/8, but a prefix identical to any VNet prefix is rejected. Multiple tunnels between a VNet and one site run active-active, and when one drops the routes are withdrawn so traffic shifts.

## Why the tunnel is up and traffic still isn't flowing

Work the layers in order and stop at the first one that fails: tunnel, then BGP session, then routing, then security, then application.

Tunnel up and BGP down is usually ASN mismatch, an access list blocking TCP/179, or the wrong peer address on one side. BGP up but no prefixes is usually a filter or a missing network statement; an established session tells you nothing about what was exchanged.

Two routing mistakes account for most of the rest. On AWS, a Transit Gateway route table can look perfect while the workload subnet's own route table has no path back to your network, so the request arrives and the reply doesn't. And on any hybrid setup, people check the forward path and skip the return path. Security is separate again: a security group that doesn't permit the port and source range will drop traffic that routing delivered correctly, and network ACLs are stateless, so both directions need an allow rule.

## Buying a BGP-capable VPS: what to confirm first

The plan matters less than the session terms. Before you order anywhere, get answers to these:

1. Is the session eBGP with the provider, and which ASN do they expect from you? Private ranges are 64512–65534 (16-bit) and 4200000000–4294967294 (32-bit).
2. Does the provider announce your prefix, or do you announce it and they pass it upstream? Prefixes longer than /24 are commonly filtered in the global table.
3. How many IPv4 addresses come with the plan, and can you get more for tunnel endpoints?
4. What is the billing model under load? On the Unicom-Premium plan, 95th-percentile billing means the top 5% of samples are discarded, so short bursts to 3× the 100M base don't dominate the bill; sustained traffic does.
5. Does the network announce IPv6 and publish RPKI ROAs?

On that last point, SixtyNet's own network is AS60699 (SixtyNet LLC, RIPE record assigned mid-2022). A third-party ASN record last updated in early 2025 shows six announced IPv4 prefixes, two upstreams visible in BGP paths, no IPv6 announcements and no RPKI ROAs on any of the six. Two upstreams is the useful part if you're planning failover. The missing ROAs matter if your filters are RPKI-strict, because those routes validate as "not found" rather than "invalid" and behaviour then depends on your policy.

If you don't need a provider-side session at all and just want a stable endpoint to terminate a tunnel into, the LAX tiers do that for a fraction of the price — LAX-MICRO at $400.00 a year gives 4 cores, 4 GB RAM, 80 GB NVMe, 10 TB of traffic and DDoS protection, which is more than a tunnel endpoint needs. 👉 [Compare the LAX tier pricing](https://sixtynet.net/aff.php?aff=39)

## Picking by situation

- **Troubleshooting only, no purchase yet:** read the five-layer order above and check ASN and TCP/179 first. Nothing to buy.
- **You need a tunnel endpoint and nothing else:** LAX-STARTER at $200.00/year. Skip the residential plans: closed 80/443 makes them useless for anything serving web traffic.
- **You need an AS9929 path into China with a residential IP:** IRVINE-PRO-STARTER at $350.00/year, but plan around the blocked probes and the balance-only refund.
- **You need a BGP session with the provider:** Los Angeles China Unicom-Premium at $2,500.00/year is the only plan that lists one. At that price it's a commitment, so get the session scope in writing before paying.

Two things to settle before checkout on the Unicom-Premium plan: how many BGP sessions and prefixes the free setup covers, and whether quarterly billing changes the effective annual cost, since the page mentions a quarterly option and an annual discount without listing the quarterly rate. 👉 [Open the SixtyNet plan page and confirm the BGP terms](https://sixtynet.net/aff.php?aff=39)
