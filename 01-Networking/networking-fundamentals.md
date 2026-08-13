# Networking Fundamentals

Foundational networking concepts relevant to cloud infrastructure work — the basis for understanding AWS VPCs, subnets, routing, and security groups later in this repository.

## What is a Network?

A network is a collection of interconnected devices that communicate and share resources. In a cloud context, this extends to virtual networks (like AWS VPCs) that replicate physical networking concepts in software.

## Types of Networks

| Type | Full Name | Scope |
|---|---|---|
| LAN | Local Area Network | A single building or campus |
| WAN | Wide Area Network | Spans cities, countries, or continents |
| MAN | Metropolitan Area Network | A city or large campus |

The Internet itself is essentially the largest WAN — a global network connecting millions of interconnected devices and smaller networks.

## The OSI Model

A 7-layer conceptual framework describing how data moves across a network:

| Layer | Name | Function | Example |
|---|---|---|---|
| 7 | Application | User-facing services | HTTP, DNS, FTP |
| 6 | Presentation | Data formatting/encryption | SSL/TLS |
| 5 | Session | Manages connections | Session tokens |
| 4 | Transport | End-to-end delivery | TCP, UDP |
| 3 | Network | Routing | IP |
| 2 | Data Link | Local delivery | Ethernet, MAC addresses |
| 1 | Physical | Raw transmission | Cables, radio signals |

## TCP/IP Basics

TCP/IP is the practical protocol suite the Internet actually runs on:

- **TCP (Transmission Control Protocol):** Connection-oriented, reliable, ordered delivery — used for HTTP, SSH, database connections.
- **UDP (User Datagram Protocol):** Connectionless, faster but no delivery guarantee — used for DNS lookups, video streaming.

## IP Addressing

An IP address uniquely identifies a device on a network.

- **IPv4 example:** `192.168.1.2` (32-bit, ~4.3 billion addresses)
- **IPv6 example:** `2001:db8::1` (128-bit, designed to solve IPv4 exhaustion)

**Private vs. Public IP ranges (IPv4):**
- `10.0.0.0 – 10.255.255.255`
- `172.16.0.0 – 172.31.255.255`
- `192.168.0.0 – 192.168.255.255`

These private ranges matter directly when designing AWS VPC CIDR blocks.

## DNS (Domain Name System)

DNS translates human-readable domain names into IP addresses.

```
google.com → 142.250.xxx.xxx
```

**Key record types:**
- **A record:** Maps a domain to an IPv4 address
- **CNAME:** Maps a domain to another domain name
- **MX:** Mail server routing
- **NS:** Nameserver delegation

This is directly relevant to AWS Route 53 later in this repository.

## Ports & Common Protocols

| Port | Protocol | Use |
|---|---|---|
| 22 | SSH | Secure remote login |
| 80 | HTTP | Unencrypted web traffic |
| 443 | HTTPS | Encrypted web traffic |
| 53 | DNS | Domain resolution |
| 3306 | MySQL | Database connections |

## Key Learnings

- Difference between LAN and WAN, and where the Internet fits in
- How the OSI model maps to real protocols
- TCP vs. UDP and when each is used
- What an IP address is, and public vs. private ranges
- How DNS resolution works
- Common ports relevant to cloud and web infrastructure

## Interview Prep

**Q: What's the difference between TCP and UDP?**
TCP is connection-oriented and guarantees ordered, reliable delivery (used where accuracy matters, like file transfers or database queries). UDP is connectionless and faster, but doesn't guarantee delivery — used where speed matters more than perfection, like video streaming or DNS lookups.

**Q: Walk me through what happens when you type a URL into a browser.**
DNS resolves the domain to an IP address → the browser opens a TCP connection to that IP (typically over port 443 for HTTPS) → a TLS handshake is negotiated → the browser sends an HTTP request → the server responds with the requested content → the browser renders it.

**Q: Why does private IP addressing matter in AWS?**
When you design a VPC, you choose a CIDR block from the private IP ranges (e.g., `10.0.0.0/16`). Resources inside the VPC get private IPs from that range, and subnets divide it further for public-facing vs. internal resources.
