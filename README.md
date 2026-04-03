# 🌐 Internet & Cloud Networking — Comprehensive Reference Card v2

> Deep-dive reference covering network fundamentals, IP addressing, DNS, virtual networks, load balancing, VPN, private endpoints, firewalls, and more — with explanations, abbreviations, real-world examples, and implementation details for **Azure**, **AWS**, and **Google Cloud (GCP)**.

-----

## Table of Contents

1. [Network Basics](#1-network-basics)
1. [IP Addressing](#2-ip-addressing)
1. [Subnets](#3-subnets)
1. [Virtual Networks (VNet / VPC)](#4-virtual-networks-vnet--vpc)
1. [DNS — Domain Name System](#5-dns--domain-name-system)
1. [Load Balancing](#6-load-balancing)
1. [NAT — Network Address Translation](#7-nat--network-address-translation)
1. [VPN — Virtual Private Network](#8-vpn--virtual-private-network)
1. [Private Endpoints & Private Link](#9-private-endpoints--private-link)
1. [NSG & Firewalls](#10-nsg--firewalls)
1. [IP Whitelisting & Access Control](#11-ip-whitelisting--access-control)
1. [Network Routing](#12-network-routing)
1. [CDN — Content Delivery Network](#13-cdn--content-delivery-network)
1. [Network Monitoring & Diagnostics](#14-network-monitoring--diagnostics)
1. [Zero Trust Networking](#15-zero-trust-networking)
1. [Encryption in Transit](#16-encryption-in-transit)
1. [DDoS Protection](#17-ddos-protection)
1. [Cloud Networking Architectures](#18-cloud-networking-architectures)
1. [Quick Reference — Port Numbers](#19-quick-reference--port-numbers)
1. [Abbreviations Glossary](#20-abbreviations-glossary)

-----

## 1. Network Basics

### Why It Matters

Understanding how data moves across networks is the foundation for everything in cloud architecture — from designing secure VNets to debugging connectivity failures. Whether you’re configuring a firewall rule, choosing between TCP and UDP for a messaging system, or troubleshooting why a private endpoint doesn’t resolve, you need to know which layer of the network stack is involved.

-----

### OSI Model (7 Layers)

**OSI** = Open Systems Interconnection. A conceptual framework that standardizes how different network protocols interact. Created by ISO in 1984. In practice, the TCP/IP model (4 layers) is used, but OSI is the universal reference language for networking.

|Layer|Name        |Abbreviation|Role                             |Real-World Examples                   |
|-----|------------|------------|---------------------------------|--------------------------------------|
|7    |Application |L7          |User-facing services             |HTTP, HTTPS, DNS, FTP, SMTP, REST APIs|
|6    |Presentation|L6          |Encoding, encryption, compression|TLS/SSL, JPEG, UTF-8, Base64          |
|5    |Session     |L5          |Session management, auth         |NetBIOS, RPC, SQL sessions            |
|4    |Transport   |L4          |End-to-end delivery, ports       |TCP, UDP, QUIC                        |
|3    |Network     |L3          |IP routing between networks      |IP, ICMP, ARP, BGP, OSPF              |
|2    |Data Link   |L2          |Node-to-node on same network     |Ethernet, MAC, VLAN, Wi-Fi (802.11)   |
|1    |Physical    |L1          |Raw bits over cable/wire         |RJ45, Fiber optic, Radio waves        |

**Why it matters in cloud:** Load balancers, firewalls, and WAFs are distinguished by which OSI layer they operate at. An L4 load balancer only sees IP and port. An L7 load balancer can inspect HTTP headers, URLs, and cookies. An L3 firewall blocks IPs; an L7 firewall can block specific domains (FQDNs) or malicious request patterns.

**Mnemonic (top-down):** *All People Seem To Need Data Processing*

-----

### TCP vs UDP

**TCP** = Transmission Control Protocol | **UDP** = User Datagram Protocol

Both are L4 transport protocols. The key trade-off is **reliability vs. speed**.

|Feature       |TCP                                  |UDP                               |
|--------------|-------------------------------------|----------------------------------|
|Connection    |Connection-oriented (3-way handshake)|Connectionless                    |
|Reliability   |Guaranteed delivery, ordering        |No guarantee, no ordering         |
|Speed         |Slower (overhead for ACKs)           |Faster (no handshake, no ACKs)    |
|Error Handling|Retransmit on loss                   |Drop on loss                      |
|Use Cases     |HTTP/S, SSH, FTP, databases, email   |DNS, video streaming, VoIP, gaming|
|Header Size   |20–60 bytes                          |8 bytes                           |
|Flow Control  |Yes (sliding window)                 |No                                |

**TCP 3-Way Handshake:**

```
Client  ──── SYN ────►  Server      (I want to connect)
Client  ◄── SYN-ACK ─── Server      (OK, I'm listening)
Client  ──── ACK ────►  Server      (Great, let's go)
         [Connection established]
```

**Real-world examples:**

- A REST API call over HTTPS uses TCP — a dropped packet means wrong data, so we retransmit.
- A video stream uses UDP — a dropped frame just causes a brief glitch, retransmitting it would cause worse lag.
- DNS queries use UDP (fast lookups), but fall back to TCP if the response is too large (>512 bytes).

**QUIC** — modern protocol (used by HTTP/3) built on UDP with reliability layered on top. Used by Google, Cloudflare, and increasingly in cloud CDNs.

-----

### Key Protocols & Abbreviations

|Protocol|Full Name                            |Port(s)|Layer|Notes                                                            |
|--------|-------------------------------------|-------|-----|-----------------------------------------------------------------|
|HTTP    |HyperText Transfer Protocol          |80     |7    |Unencrypted web traffic — never use for sensitive data           |
|HTTPS   |HTTP Secure                          |443    |7    |HTTP over TLS — always use in production                         |
|SSH     |Secure Shell                         |22     |7    |Encrypted remote terminal access to Linux/Unix                   |
|FTP     |File Transfer Protocol               |20/21  |7    |Unencrypted file transfer — use SFTP or FTPS instead             |
|SFTP    |SSH File Transfer Protocol           |22     |7    |Secure file transfer over SSH                                    |
|SMTP    |Simple Mail Transfer Protocol        |25/587 |7    |Email delivery (25 = server-to-server, 587 = client submission)  |
|DNS     |Domain Name System                   |53     |7    |Name → IP resolution (UDP for queries, TCP for zone transfers)   |
|DHCP    |Dynamic Host Configuration Protocol  |67/68  |7    |Automatically assigns IPs, gateway, DNS to clients               |
|RDP     |Remote Desktop Protocol              |3389   |7    |Windows GUI remote access — high-risk if exposed publicly        |
|LDAP    |Lightweight Directory Access Protocol|389    |7    |Directory services (Active Directory, OpenLDAP)                  |
|LDAPS   |LDAP over SSL                        |636    |7    |Encrypted LDAP                                                   |
|ICMP    |Internet Control Message Protocol    |—      |3    |Ping, traceroute, unreachable messages                           |
|BGP     |Border Gateway Protocol              |179    |3/4  |Internet’s routing protocol — used between ISPs and in cloud VPNs|
|ARP     |Address Resolution Protocol          |—      |2/3  |Maps IP address to MAC address on local network                  |
|NTP     |Network Time Protocol                |123    |7    |Time synchronization — critical for TLS certs, Kerberos          |
|SNMP    |Simple Network Management Protocol   |161/162|7    |Network device monitoring                                        |

-----

## 2. IP Addressing

### Why It Matters

Every device and service on a network needs an address. IP addressing determines how traffic is routed between networks. Misunderstanding CIDR notation or using overlapping address spaces is one of the most common causes of VNet peering failures and on-prem connectivity issues.

-----

### IPv4 vs IPv6

**IPv4** = Internet Protocol version 4 | **IPv6** = Internet Protocol version 6

The internet ran out of IPv4 addresses around 2011. IPv6 was designed to replace IPv4 but adoption has been gradual. Most cloud infrastructure still runs dual-stack (both).

|Feature       |IPv4                                |IPv6                                           |
|--------------|------------------------------------|-----------------------------------------------|
|Length        |32-bit (4 octets)                   |128-bit (8 groups of 4 hex digits)             |
|Format        |`192.168.1.1`                       |`2001:0db8:85a3::8a2e:0370:7334`               |
|Separator     |Dot `.`                             |Colon `:`                                      |
|Address space |~4.3 billion                        |~340 undecillion (3.4 × 10³⁸)                  |
|NAT required? |Yes (shortage workaround)           |No — every device can have a unique IP         |
|Header size   |20 bytes (variable)                 |40 bytes (fixed, simpler)                      |
|Configuration |Manual or DHCP                      |SLAAC (auto), DHCPv6, or manual                |
|Private ranges|RFC 1918 (10.x, 172.16.x, 192.168.x)|Unique Local (fc00::/7), Link-local (fe80::/10)|

**IPv6 shorthand rules:**

- Leading zeros in a group can be omitted: `0db8` → `db8`
- One consecutive sequence of all-zero groups can be replaced with `::`:
  `2001:0000:0000:0000:0000:0000:0000:0001` → `2001::1`

**Cloud IPv6 support:**

- **Azure:** Dual-stack VNets supported; IPv6 for VMs, Load Balancers, Public IPs
- **AWS:** IPv6 CIDR blocks can be added to VPCs; supported in subnets, EC2, ALB
- **GCP:** IPv6 supported in VPC subnets (External IPv6 and ULA for internal)

-----

### CIDR Notation

**CIDR** = Classless Inter-Domain Routing. Introduced in 1993 to replace the rigid Class A/B/C system and slow IPv4 exhaustion.

Format: `<network address>/<prefix length>` — prefix length = number of fixed (network) bits.

|CIDR|Subnet Mask    |Total IPs |Usable Hosts|Typical Use                   |
|----|---------------|----------|------------|------------------------------|
|/8  |255.0.0.0      |16,777,216|16,777,214  |Large corporate / ISP backbone|
|/16 |255.255.0.0    |65,536    |65,534      |VNet address space            |
|/20 |255.255.240.0  |4,096     |4,094       |Large subnet                  |
|/24 |255.255.255.0  |256       |251*        |Standard subnet (cloud)       |
|/26 |255.255.255.192|64        |59*         |Small subnet                  |
|/27 |255.255.255.224|32        |27*         |Gateway subnets               |
|/28 |255.255.255.240|16        |11*         |Very small / management subnet|
|/29 |255.255.255.248|8         |3*          |Minimal subnet                |
|/30 |255.255.255.252|4         |2           |Point-to-point links          |
|/32 |255.255.255.255|1         |1           |Single host route             |


> *Cloud providers reserve 5 IPs per subnet (see Section 3). Formula: `Usable = 2^(32 - prefix) - 5`

**How to read CIDR:** `10.0.1.0/24` means the first 24 bits (`10.0.1`) are fixed (the network), and the last 8 bits are free for hosts (`.0` to `.255`).

**Example:** Planning a VNet at `10.10.0.0/16`:

```
VNet: 10.10.0.0/16  →  contains 10.10.0.0 through 10.10.255.255

Subnet 1: 10.10.1.0/24  →  256 IPs, 251 usable
Subnet 2: 10.10.2.0/24  →  256 IPs, 251 usable
Subnet 3: 10.10.3.0/28  →  16  IPs, 11 usable
... no overlap, all fit within /16
```

**Critical rule:** Subnets within the same VNet/VPC must NOT overlap. VNets being peered must also have non-overlapping address spaces — plan ahead!

-----

### Private IP Ranges (RFC 1918)

These ranges are not routable on the public internet. Routers on the internet will drop packets destined for these ranges. They are reserved for private/internal use only.

|Range                        |CIDR          |Class|Typical Use                    |
|-----------------------------|--------------|-----|-------------------------------|
|10.0.0.0 – 10.255.255.255    |10.0.0.0/8    |A    |Large cloud VNets, data centers|
|172.16.0.0 – 172.31.255.255  |172.16.0.0/12 |B    |Medium enterprise networks     |
|192.168.0.0 – 192.168.255.255|192.168.0.0/16|C    |Home routers, small office     |

**Why it matters:** When designing VNet address spaces for hybrid (on-prem + cloud) environments, you must ensure your cloud VNet CIDR does not overlap with your corporate network CIDR — otherwise VPN/ExpressRoute routing becomes ambiguous or broken.

-----

### Reserved / Special Addresses

|Address          |Purpose                                                                                                                                            |
|-----------------|---------------------------------------------------------------------------------------------------------------------------------------------------|
|`0.0.0.0/0`      |Default route — matches ALL destinations (used in routing)                                                                                         |
|`127.0.0.1`      |Loopback / localhost — traffic stays on the same machine                                                                                           |
|`169.254.x.x`    |APIPA (Automatic Private IP Addressing) — assigned when DHCP fails. Also used by cloud metadata services (e.g., `169.254.169.254` on AWS/Azure/GCP)|
|`255.255.255.255`|Limited broadcast — sent to all hosts on local segment                                                                                             |
|`x.x.x.0`        |Network address — identifies the subnet (cannot be assigned to a host)                                                                             |
|`x.x.x.255`      |Directed broadcast — sent to all hosts in subnet (usually blocked)                                                                                 |

**Cloud metadata service:** `169.254.169.254` is a special address that cloud VMs use to fetch instance metadata (hostname, region, attached IAM role, etc.) without leaving the host. This is how Managed Identities and EC2 instance profiles deliver credentials to VMs.

-----

### Static vs Dynamic IPs

|Feature      |Static IP                              |Dynamic IP (DHCP)                     |
|-------------|---------------------------------------|--------------------------------------|
|Assignment   |Pre-configured or reserved             |Automatically assigned by DHCP server |
|Persistence  |Never changes (even after reboot)      |Can change on reconnect / lease expiry|
|Use Cases    |Servers, VPNs, firewalls, DNS A records|End-user devices, ephemeral workloads |
|Cost (cloud) |Usually charged extra                  |Free / included                       |
|DNS-friendly?|Yes — stable A record                  |Requires DDNS (Dynamic DNS)           |
|Security risk|Predictable — target for scanners      |Slightly less predictable             |

**DHCP** = Dynamic Host Configuration Protocol. When a device joins a network, it broadcasts a request; a DHCP server responds with an IP lease (address + gateway + DNS + lease time).

**DDNS** = Dynamic DNS. A service that automatically updates a DNS record when a dynamic IP changes. Used by home servers, IoT devices.

**Cloud implementations:**

|Platform|Static Public IP          |Dynamic Public IP      |Static Private IP                      |
|--------|--------------------------|-----------------------|---------------------------------------|
|Azure   |Public IP (Static SKU)    |Public IP (Dynamic SKU)|Static private IP via NIC config       |
|AWS     |Elastic IP (EIP)          |Default EC2 public IP  |Fixed private IP at ENI/instance launch|
|GCP     |External Static IP Address|Ephemeral External IP  |Static internal IP via interface config|

**Azure example — assign static public IP to a VM:**

```bash
# Create static public IP
az network public-ip create \
  --name myStaticIP \
  --resource-group myRG \
  --allocation-method Static \
  --sku Standard

# Attach to VM's NIC
az network nic ip-config update \
  --name ipconfig1 \
  --nic-name myVMNic \
  --resource-group myRG \
  --public-ip-address myStaticIP
```

**AWS example — allocate and associate Elastic IP:**

```bash
# Allocate EIP
aws ec2 allocate-address --domain vpc

# Associate with EC2 instance
aws ec2 associate-address \
  --instance-id i-0abc123def456 \
  --allocation-id eipalloc-0abc123456
```

**GCP example — reserve static external IP:**

```bash
gcloud compute addresses create my-static-ip \
  --region=europe-west1

gcloud compute instances add-access-config my-vm \
  --access-config-name="External NAT" \
  --address=$(gcloud compute addresses describe my-static-ip \
    --region=europe-west1 --format='get(address)')
```

-----

## 3. Subnets

### What Is a Subnet?

A **subnet** (sub-network) is a logical subdivision of a larger IP network. Subnetting allows you to divide a single large address space into smaller, isolated segments. Each subnet gets its own CIDR range and can have its own security rules, routing configuration, and access controls.

**Why subnets matter:**

- **Security isolation:** Separate subnets for web, app, and database tiers means a compromised web server cannot directly reach the database subnet.
- **Traffic control:** NSGs, route tables, and firewall rules are applied per subnet.
- **Compliance:** Regulatory frameworks (PCI-DSS, HIPAA) often require network segmentation.
- **Availability Zones:** In AWS and Azure, subnets are associated with specific Availability Zones (AZs) — deploying replicas across subnets in different AZs gives fault tolerance.

-----

### Cloud-Reserved IPs per Subnet

Cloud providers reserve a fixed set of IP addresses in every subnet for internal use. You cannot assign these to your resources.

**Azure (5 reserved per subnet):**

|Offset|Address Example|Purpose          |
|------|---------------|-----------------|
|+0    |10.0.1.0       |Network address  |
|+1    |10.0.1.1       |Default gateway  |
|+2    |10.0.1.2       |Azure DNS mapping|
|+3    |10.0.1.3       |Azure DNS mapping|
|last  |10.0.1.255     |Broadcast address|

**AWS (5 reserved per subnet):**

|Offset|Purpose                |
|------|-----------------------|
|+0    |Network address        |
|+1    |VPC router             |
|+2    |Amazon-provided DNS    |
|+3    |Reserved for future use|
|last  |Broadcast address      |

**GCP:** Reserves the first and last IP of the subnet range. GCP subnets are regional (not AZ-bound), which is unique.

-----

### Availability Zones (AZ)

**AZ** = Availability Zone. Physically separate data centers within the same region, connected by low-latency links. Deploying resources across multiple AZs gives high availability.

|Concept      |Azure            |AWS                |GCP                      |
|-------------|-----------------|-------------------|-------------------------|
|AZ equivalent|Availability Zone|Availability Zone  |Zone (e.g., us-east1-b)  |
|Count/region |Typically 3      |Typically 3        |Typically 3              |
|Subnet scope |Subnet spans AZs |One subnet = one AZ|Regional (span all zones)|

**Azure note:** Azure subnets span all AZs in a region. AZ pinning happens at the VM/resource level, not the subnet level.

**AWS note:** Each AWS subnet lives in exactly one AZ. To achieve HA, create one subnet per AZ per tier.

-----

### Example: 3-Tier Architecture Subnetting (`10.0.0.0/16`)

```
VNet/VPC: 10.0.0.0/16
│
├── GatewaySubnet / TransitSubnet      10.0.0.0/27    (32 IPs)   VPN / ExpressRoute gateway
├── BastionSubnet                       10.0.1.0/26    (64 IPs)   Secure admin access
├── web-subnet-az1                      10.0.2.0/24    (256 IPs)  Public-facing, AZ1
├── web-subnet-az2                      10.0.3.0/24    (256 IPs)  Public-facing, AZ2
├── app-subnet-az1                      10.0.4.0/24    (256 IPs)  Application tier, AZ1
├── app-subnet-az2                      10.0.5.0/24    (256 IPs)  Application tier, AZ2
├── data-subnet-az1                     10.0.6.0/24    (256 IPs)  Database / PEs, AZ1
├── data-subnet-az2                     10.0.7.0/24    (256 IPs)  Database / PEs, AZ2
└── mgmt-subnet                         10.0.8.0/28    (16 IPs)   Admin, monitoring
```

**Why GatewaySubnet needs /27 minimum:** Azure requires at least /27 for gateway subnets to accommodate redundant gateway instances.

-----

## 4. Virtual Networks (VNet / VPC)

### What Is a Virtual Network?

A **Virtual Network** is a software-defined, logically isolated network that you provision in the cloud. It behaves like a private data center network — resources inside it can communicate with each other via private IPs, and you fully control inbound/outbound traffic with firewalls and routing rules.

**Without a VNet/VPC:** Cloud resources would communicate over the public internet, exposing them to interception and attack.

**With a VNet/VPC:** All traffic between your resources stays within the provider’s backbone network, never touching the public internet.

-----

### Cross-Cloud Comparison

|Concept           |Azure                       |AWS                        |GCP                                     |
|------------------|----------------------------|---------------------------|----------------------------------------|
|Network           |VNet (Virtual Network)      |VPC (Virtual Private Cloud)|VPC (Virtual Private Cloud)             |
|Subnet            |Subnet                      |Subnet                     |Subnet                                  |
|Region-scoped     |Yes                         |Yes                        |**Global** (VPC spans regions)          |
|AZ-scoped subnets |No (spans AZs)              |Yes (one AZ per subnet)    |No (regional subnets)                   |
|Internet Gateway  |Internet Gateway (automatic)|IGW (must attach)          |Default internet access                 |
|Private routing   |Managed via route tables    |Route tables               |Route tables                            |
|Peering           |VNet Peering                |VPC Peering                |VPC Peering                             |
|Hub-and-spoke     |Azure Virtual WAN / Hub VNet|Transit Gateway            |Network Connectivity Center / Shared VPC|
|Terraform resource|`azurerm_virtual_network`   |`aws_vpc`                  |`google_compute_network`                |

**GCP key difference:** GCP VPCs are global — a single VPC can have subnets in `us-east1`, `europe-west1`, and `asia-east1` simultaneously. VMs in different regions in the same VPC can communicate without peering. Azure and AWS VNets/VPCs are region-scoped.

-----

### VNet / VPC Peering

Peering connects two VNets/VPCs so that resources can communicate over private IPs using the cloud provider’s backbone network — no public internet, no VPN, no extra latency from encryption overhead.

**Key rules:**

- Address spaces must NOT overlap (critical — plan CIDR carefully).
- Peering is **non-transitive**: A↔B and B↔C does NOT allow A↔C. Use a Transit Gateway (AWS) or Hub VNet / Virtual WAN (Azure) for transitive routing.
- Peering can cross subscriptions/accounts and regions (Global Peering).

**Azure example — create VNet peering:**

```bash
# Peer VNet-A to VNet-B
az network vnet peering create \
  --name VNetA-to-VNetB \
  --vnet-name VNetA \
  --resource-group myRG \
  --remote-vnet /subscriptions/.../VNetB \
  --allow-vnet-access true

# Peer VNet-B to VNet-A (peering is bidirectional but must be created on both sides)
az network vnet peering create \
  --name VNetB-to-VNetA \
  --vnet-name VNetB \
  --resource-group myRG \
  --remote-vnet /subscriptions/.../VNetA \
  --allow-vnet-access true
```

**AWS example — create VPC peering:**

```bash
# Request peering
aws ec2 create-vpc-peering-connection \
  --vpc-id vpc-111aaa \
  --peer-vpc-id vpc-222bbb \
  --peer-region eu-west-1

# Accept the peering (from the peer account/region)
aws ec2 accept-vpc-peering-connection \
  --vpc-peering-connection-id pcx-0abc123456789

# Update route tables on both sides to point to pcx-xxxx for the peer CIDR
```

**GCP example — VPC peering:**

```bash
gcloud compute networks peerings create peer-a-to-b \
  --network=network-a \
  --peer-project=my-project \
  --peer-network=network-b \
  --auto-create-routes
```

-----

### Hub-and-Spoke Topology

The most common enterprise pattern for multi-VNet/VPC environments:

```
               ┌──────────────┐
               │   Hub VNet   │
               │  (Firewall,  │
               │  VPN GW,     │
               │  Bastion)    │
               └──────┬───────┘
          ┌───────────┼───────────┐
     ┌────▼────┐  ┌───▼────┐  ┌──▼─────┐
     │ Spoke 1 │  │Spoke 2 │  │Spoke 3 │
     │(Dev env)│  │(Prod)  │  │(Data)  │
     └─────────┘  └────────┘  └────────┘
```

All inter-spoke traffic is routed through the hub (where you centralize security inspection). Spokes are peered only to the hub, not to each other.

|Implementation     |Azure                      |AWS                |GCP                        |
|-------------------|---------------------------|-------------------|---------------------------|
|Hub routing        |Azure Firewall / NVA in hub|Transit Gateway    |Network Connectivity Center|
|Managed service    |Azure Virtual WAN          |AWS Transit Gateway|Cloud Router + HA VPN      |
|Shared services VPC|Hub VNet                   |Shared VPC / TGW   |Shared VPC (XPN)           |

-----

### Network Interface Card (NIC)

**NIC** = Network Interface Card. In cloud, a virtual NIC (vNIC) connects a VM to a subnet. Each NIC has a private IP (always) and optionally a public IP.

- A VM can have multiple NICs for traffic isolation (e.g., management NIC on a separate subnet).
- **Azure:** Network Interface resource, associated with a subnet and optionally an NSG.
- **AWS:** Elastic Network Interface (ENI), same concept.
- **GCP:** Network Interface on a VM instance.

-----

## 5. DNS — Domain Name System

### What Is DNS and Why Does It Matter?

**DNS** = Domain Name System. The “phone book of the internet” — it translates human-readable names (`api.example.com`) into machine-readable IP addresses (`52.13.44.7`).

Without DNS, you’d need to memorize IP addresses for every service you access. In cloud, DNS is critical for:

- Service discovery within VNets (internal DNS names).
- Routing traffic to the correct endpoint (public DNS for internet-facing services).
- Making Private Endpoints work — they rely on DNS overrides to redirect public FQDN lookups to private IPs.

-----

### DNS Record Types

|Record|Full Name                   |Purpose                                       |Example                                         |
|------|----------------------------|----------------------------------------------|------------------------------------------------|
|A     |Address                     |Hostname → IPv4 address                       |`api.example.com.  → 52.13.44.7`                |
|AAAA  |Quad-A                      |Hostname → IPv6 address                       |`api.example.com.  → 2001:db8::1`               |
|CNAME |Canonical Name              |Alias → another hostname                      |`www.example.com.  → app.example.com.`          |
|MX    |Mail Exchanger              |Mail server(s) for a domain                   |`example.com.  10  mail.example.com.`           |
|TXT   |Text                        |Arbitrary text (SPF, DKIM, domain ownership)  |`"v=spf1 include:amazonses.com ~all"`           |
|NS    |Name Server                 |Authoritative DNS servers for a zone          |`example.com.  →  ns1.azure-dns.com.`           |
|SOA   |Start of Authority          |Zone metadata: primary NS, admin, serial no.  |Required in every DNS zone                      |
|PTR   |Pointer                     |Reverse: IP → hostname                        |`7.44.13.52.in-addr.arpa. → api.example.com.`   |
|SRV   |Service                     |Service location: priority, weight, port, host|`_sip._tcp.example.com. → sip.example.com.:5060`|
|CAA   |Certification Authority Auth|Which CAs can issue certs for a domain        |`example.com. CAA 0 issue "letsencrypt.org"`    |

**CNAME vs A record:**

- Use `CNAME` when pointing to another service hostname that may change its IP (e.g., an Azure App Service or S3 bucket endpoint). You can’t use CNAME on the zone apex (`@` / `example.com` itself — use ALIAS/ANAME instead).
- Use `A` when you know the exact IP and it’s stable.

-----

### DNS Resolution Flow (Step by Step)

```
Browser: "What is the IP of api.example.com?"

1. Check local OS cache → not found
2. Ask Recursive Resolver (e.g., 8.8.8.8 / ISP DNS)
   └── Check resolver cache → not found
3. Resolver asks Root Name Server (.)
   └── "I don't know api.example.com, but here are the .com TLD servers"
4. Resolver asks .com TLD Name Server
   └── "I don't know api.example.com, but here are example.com's NS records"
5. Resolver asks example.com's Authoritative Name Server
   └── "api.example.com → 52.13.44.7 (TTL: 300)"
6. Resolver caches the answer and returns it to the browser
7. Browser connects to 52.13.44.7
```

**TTL** = Time To Live (in seconds). Controls how long a DNS record is cached by resolvers and clients.

|TTL Value  |Effect                                 |When to Use                    |
|-----------|---------------------------------------|-------------------------------|
|60–300s    |Fast propagation, high DNS query volume|Before/during migrations       |
|3600s (1h) |Balanced                               |Normal production services     |
|86400s (1d)|Slow propagation, low query volume     |Stable, rarely-changing records|

**Migration tip:** Lower TTL to 60s at least 24–48h before a migration. After cutover and validation, restore to 3600s+.

-----

### Private DNS Zones

A **Private DNS Zone** is a DNS zone that is only resolvable from within your VNet/VPC — not from the public internet.

Used for: internal service discovery, overriding public DNS for Private Endpoints, split-horizon DNS.

**Split-horizon DNS:** The same FQDN resolves to different IPs depending on whether you’re inside or outside the network. For example, `mydb.database.windows.net` resolves to a private IP (`10.0.4.5`) inside the VNet but to a public IP from the internet.

**Implementation:**

**Azure:**

```bash
# Create private DNS zone
az network private-dns zone create \
  --resource-group myRG \
  --name "privatelink.blob.core.windows.net"

# Link zone to VNet (enables resolution from VNet)
az network private-dns link vnet create \
  --resource-group myRG \
  --zone-name "privatelink.blob.core.windows.net" \
  --name myDNSLink \
  --virtual-network myVNet \
  --registration-enabled false
```

**AWS (Route 53 Private Hosted Zone):**

```bash
# Create private hosted zone
aws route53 create-hosted-zone \
  --name internal.example.com \
  --caller-reference 2026-01-01 \
  --hosted-zone-config PrivateZone=true \
  --vpc VPCRegion=eu-west-1,VPCId=vpc-111aaa
```

**GCP (Cloud DNS private zone):**

```bash
gcloud dns managed-zones create internal-zone \
  --dns-name="internal.example.com." \
  --description="Private zone" \
  --visibility=private \
  --networks=my-vpc
```

-----

### Cloud DNS Services

|Feature              |Azure DNS          |AWS Route 53      |GCP Cloud DNS                  |
|---------------------|-------------------|------------------|-------------------------------|
|Public DNS hosting   |Yes                |Yes               |Yes                            |
|Private DNS zones    |Yes                |Yes (Hosted Zones)|Yes                            |
|Health-check routing |Via Traffic Manager|Yes (built-in)    |Via GLB                        |
|Geo-based routing    |Via Traffic Manager|Yes               |Via GFE                        |
|Latency-based routing|Via Front Door     |Yes               |Via GFE                        |
|Domain registration  |Yes                |Yes               |Via Google Domains (deprecated)|
|DNSSEC support       |Yes                |Yes               |Yes                            |
|SLA                  |100% (public zones)|100%              |100%                           |

**DNSSEC** = DNS Security Extensions. Cryptographically signs DNS records to prevent DNS spoofing/cache poisoning attacks.

-----

## 6. Load Balancing

### What Is Load Balancing and Why Does It Matter?

A **load balancer** distributes incoming traffic across multiple backend servers (instances, VMs, containers) to ensure no single server is overwhelmed. This provides:

- **High Availability (HA):** If one server fails, others handle the traffic.
- **Scalability:** Add more backends as traffic grows.
- **Performance:** Route users to the fastest/nearest server.
- **Maintenance:** Take backends offline for updates without downtime (rolling deployments).

-----

### Load Balancer Layers

|Type              |OSI Layer|Visibility                      |Examples                                     |
|------------------|---------|--------------------------------|---------------------------------------------|
|L4 (Transport)    |4        |IP + TCP/UDP port only          |Azure LB, AWS NLB, HAProxy (TCP mode)        |
|L7 (Application)  |7        |HTTP headers, URL, cookies, body|Azure App Gateway, AWS ALB, Nginx, Envoy     |
|DNS-based / Global|7 (DNS)  |DNS query origin / geolocation  |Azure Traffic Manager, AWS Global Accelerator|

**L4 vs L7:**

- L4 is faster and simpler — it just forwards packets without inspecting content. Use for raw TCP/UDP workloads (databases, IoT, gaming).
- L7 is smarter — it can route `/api/*` to one backend pool and `/static/*` to another. It can terminate TLS, add WAF rules, and make decisions based on HTTP headers. Use for HTTP/HTTPS applications.

-----

### Load Balancing Algorithms

|Algorithm           |Description                                                  |Best For                             |
|--------------------|-------------------------------------------------------------|-------------------------------------|
|Round Robin         |Requests distributed evenly in rotation                      |Stateless apps with uniform workloads|
|Weighted Round Robin|Like Round Robin but servers have weights (e.g., 70/30 split)|Canary deployments, A/B testing      |
|Least Connections   |Route to backend with fewest active connections              |Long-lived connections (WebSockets)  |
|IP Hash             |Same source IP always goes to same backend                   |Stateful apps (poor man’s sessions)  |
|Least Response Time |Route to fastest-responding backend                          |Latency-sensitive applications       |
|Random              |Random backend selection                                     |Simple, equal-capacity environments  |

**Session Affinity / Sticky Sessions:** The LB inserts a cookie (or uses IP hash) to ensure all requests from the same user go to the same backend. Required for apps that store session state in memory. Better practice: move session state to Redis/Memcached so all backends are interchangeable.

-----

### Health Probes

A health probe is a periodic check (HTTP GET, TCP connect, or HTTPS) the load balancer sends to each backend. If a backend fails the probe, the LB stops sending traffic to it.

**Common settings:**

|Setting            |Typical Value|Meaning                                          |
|-------------------|-------------|-------------------------------------------------|
|Protocol           |HTTP/HTTPS   |What kind of check to perform                    |
|Path               |`/health`    |Endpoint to call (should return 200 OK)          |
|Port               |80 / 443     |Port to check                                    |
|Interval           |15–30s       |How often to probe                               |
|Unhealthy threshold|2–3 failures |How many consecutive failures before marking down|
|Healthy threshold  |2–3 successes|How many successes to mark healthy again         |

-----

### Cloud Load Balancing Products

**Azure:**

|Product            |Layer|Scope   |Use Case                                                       |
|-------------------|-----|--------|---------------------------------------------------------------|
|Azure Load Balancer|L4   |Regional|TCP/UDP for VMs, internal (ILB) or external. No HTTP awareness.|
|Application Gateway|L7   |Regional|HTTPS, SSL termination, URL routing, WAF, cookie affinity      |
|Azure Front Door   |L7   |Global  |CDN + WAF + global HTTPS routing + failover across regions     |
|Traffic Manager    |DNS  |Global  |DNS-based routing (latency, geographic, failover, weighted)    |

**AWS:**

|Product                        |Layer|Scope   |Use Case                                                           |
|-------------------------------|-----|--------|-------------------------------------------------------------------|
|Classic Load Balancer (CLB)    |L4/L7|Regional|Legacy — avoid for new workloads                                   |
|Network Load Balancer (NLB)    |L4   |Regional|Ultra-low latency TCP/UDP, static IP per AZ, supports TLS          |
|Application Load Balancer (ALB)|L7   |Regional|HTTP/HTTPS, path/header routing, WebSockets, Lambda targets        |
|Global Accelerator             |L4/L7|Global  |Routes traffic to nearest healthy regional endpoint via Anycast IPs|

**GCP:**

|Product                      |Layer|Scope   |Use Case                                            |
|-----------------------------|-----|--------|----------------------------------------------------|
|Cloud Load Balancing (HTTP/S)|L7   |Global  |Anycast HTTP/HTTPS LB — single IP serves all regions|
|TCP/UDP Network LB           |L4   |Regional|Pass-through load balancing, preserves source IP    |
|Internal HTTP(S) LB          |L7   |Regional|L7 LB for internal services (GKE, VMs)              |
|Internal TCP/UDP LB          |L4   |Regional|Internal L4 for private services                    |

**Example — Azure Application Gateway with WAF (Terraform):**

```hcl
resource "azurerm_application_gateway" "main" {
  name                = "myAppGateway"
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location

  sku {
    name     = "WAF_v2"
    tier     = "WAF_v2"
    capacity = 2
  }

  waf_configuration {
    enabled          = true
    firewall_mode    = "Prevention"
    rule_set_version = "3.2"
  }

  # ... frontend, backend, listener, routing rules
}
```

**Example — AWS ALB with target group (AWS CLI):**

```bash
# Create ALB
aws elbv2 create-load-balancer \
  --name my-alb \
  --subnets subnet-aaa subnet-bbb \
  --security-groups sg-xxx \
  --scheme internet-facing \
  --type application

# Create target group
aws elbv2 create-target-group \
  --name my-targets \
  --protocol HTTP \
  --port 8080 \
  --vpc-id vpc-111 \
  --health-check-path /health

# Create listener
aws elbv2 create-listener \
  --load-balancer-arn arn:aws:elasticloadbalancing:... \
  --protocol HTTPS --port 443 \
  --default-actions Type=forward,TargetGroupArn=arn:...
```

-----

## 7. NAT — Network Address Translation

### What Is NAT and Why Is It Used?

**NAT** = Network Address Translation. A technique where a router/gateway rewrites the source or destination IP (and sometimes port) of packets passing through it.

**Primary reason NAT exists:** IPv4 address exhaustion. With ~4.3 billion addresses and billions of devices, NAT allows thousands of private devices to share a single public IP address.

**In cloud:** NAT is used to give private VMs/containers outbound internet access without assigning them a public IP — improving security (no inbound attack surface) and reducing cost.

-----

### NAT Types

|Type      |Full Name               |Direction|Description                                                     |Example                                  |
|----------|------------------------|---------|----------------------------------------------------------------|-----------------------------------------|
|SNAT      |Source NAT              |Outbound |Replaces source IP with a public IP                             |Private VM → NAT Gateway → internet      |
|DNAT      |Destination NAT         |Inbound  |Replaces destination IP — port forwarding                       |Public IP:3389 → Private VM 10.0.1.4:3389|
|PAT       |Port Address Translation|Outbound |Multiple private IPs share one public IP, differentiated by port|Home router, cloud NAT gateway           |
|Static NAT|1:1 NAT                 |Both     |One public IP permanently maps to one private IP                |Legacy DMZ setups                        |

**PAT** (also called NAPT = Network Address Port Translation or IP Masquerade) is what most cloud NAT gateways do — tracks connections by (source IP, source port, destination IP, destination port) 5-tuple.

-----

### NAT in Cloud

|Feature            |Azure NAT Gateway        |AWS NAT Gateway          |GCP Cloud NAT            |
|-------------------|-------------------------|-------------------------|-------------------------|
|Type               |Managed SNAT             |Managed SNAT             |Managed SNAT             |
|Static outbound IP |Yes (assign public IPs)  |Yes (Elastic IP)         |Yes (static external IPs)|
|Scalability        |Auto (up to 50k ports/IP)|Auto                     |Auto                     |
|AZ-redundant       |No (per AZ, use one/AZ)  |Yes (with AZ-independent)|Yes                      |
|Cost               |Hourly + data processed  |Hourly + data processed  |Data processed           |
|Custom port mapping|No                       |No                       |No (managed)             |

**Why you need a NAT Gateway:** VMs in private subnets (no public IP) cannot reach the internet by default. A NAT Gateway in a public subnet with a route (`0.0.0.0/0 → NAT GW`) enables outbound-only internet access.

**Azure example:**

```bash
az network nat gateway create \
  --name myNATGateway \
  --resource-group myRG \
  --location westeurope \
  --public-ip-addresses myPublicIP \
  --idle-timeout 10

# Associate with subnet
az network vnet subnet update \
  --name app-subnet \
  --vnet-name myVNet \
  --resource-group myRG \
  --nat-gateway myNATGateway
```

**AWS example:**

```bash
# NAT Gateway goes in a PUBLIC subnet
aws ec2 create-nat-gateway \
  --subnet-id subnet-public-aaa \
  --allocation-id eipalloc-0abc123456 \
  --connectivity-type public

# Update private subnet route table
aws ec2 create-route \
  --route-table-id rtb-private \
  --destination-cidr-block 0.0.0.0/0 \
  --nat-gateway-id nat-0abc123456
```

-----

## 8. VPN — Virtual Private Network

### What Is a VPN and Why Is It Used?

**VPN** = Virtual Private Network. Creates an **encrypted tunnel** between two endpoints over an untrusted network (typically the public internet). Data inside the tunnel is encrypted and cannot be read by intermediaries.

**Use cases:**

- **Hybrid cloud:** Connect on-premises data center to cloud VNet/VPC securely.
- **Remote access:** Developers/admins connect to private cloud resources from anywhere.
- **Cross-region:** Connect cloud VNets in different regions privately.

**VPN vs Direct Connectivity (ExpressRoute/Direct Connect):**

|Feature       |VPN (IPsec)                    |ExpressRoute / Direct Connect     |
|--------------|-------------------------------|----------------------------------|
|Medium        |Public internet (encrypted)    |Dedicated private circuit         |
|Encryption    |Yes (IPsec/TLS)                |Optional (MACsec layer 2)         |
|Max throughput|Up to 10 Gbps (policy limit)   |Up to 100 Gbps                    |
|Latency       |Variable (internet-dependent)  |Predictable, consistent           |
|Setup time    |Minutes (software)             |Weeks/months (physical circuit)   |
|Cost          |Low (gateway + data transfer)  |High (circuit + port + gateway)   |
|Reliability   |Internet SLA                   |Carrier-backed SLA                |
|Best for      |Dev/test, SMB, temporary hybrid|Production, financial, large-scale|

-----

### VPN Types

|Type         |Abbreviation|Description                                                    |Use Case                       |
|-------------|------------|---------------------------------------------------------------|-------------------------------|
|Site-to-Site |S2S         |IPsec tunnel between two fixed networks                        |On-prem ↔ cloud persistent link|
|Point-to-Site|P2S         |Individual device connects to VNet via VPN client              |Developer remote access        |
|VNet-to-VNet |V2V         |Two cloud VNets connected via VPN (usually use peering instead)|Cross-subscription, cross-cloud|

-----

### VPN Protocols

|Protocol     |Description                                 |Pros                         |Cons                          |
|-------------|--------------------------------------------|-----------------------------|------------------------------|
|IPsec / IKEv2|Industry-standard, hardware-compatible      |Fast, widely supported       |Complex configuration         |
|OpenVPN      |Open-source, TLS-based (port 443 or 1194)   |Firewall-friendly, flexible  |Slower than IKEv2             |
|WireGuard    |Modern, minimal codebase, UDP-based         |Fast, simple, modern crypto  |Newer, less enterprise tooling|
|SSTP         |Secure Socket Tunneling Protocol (Microsoft)|Works through firewalls (443)|Windows-only client           |
|L2TP/IPsec   |Layer 2 Tunneling Protocol over IPsec       |Widely supported             |Slower (double encapsulation) |

**IKE** = Internet Key Exchange. The protocol used to set up the Security Association (SA) — negotiating encryption keys for an IPsec tunnel. IKEv2 is the modern version (faster rekeying, MOBIKE for mobile).

**IPsec** = IP Security. Operates in two modes:

- **Transport mode:** Encrypts payload only (host-to-host).
- **Tunnel mode:** Encrypts entire IP packet and wraps in new IP header (site-to-site VPN).

-----

### Cloud VPN Implementations

**Azure VPN Gateway:**

```bash
# Create VPN Gateway (takes 30–45 min to provision)
az network vnet-gateway create \
  --name myVPNGateway \
  --resource-group myRG \
  --vnet myVNet \
  --gateway-type Vpn \
  --vpn-type RouteBased \
  --sku VpnGw2 \
  --public-ip-address myGatewayIP

# Create Local Network Gateway (represents on-prem)
az network local-gateway create \
  --name onPremGateway \
  --resource-group myRG \
  --gateway-ip-address 203.0.113.10 \   # on-prem public IP
  --local-address-prefixes 192.168.0.0/16

# Create VPN Connection
az network vpn-connection create \
  --name onPremConnection \
  --resource-group myRG \
  --vnet-gateway1 myVPNGateway \
  --local-gateway2 onPremGateway \
  --shared-key "SuperSecretPSK123!"
```

**AWS Site-to-Site VPN:**

```bash
# Create Customer Gateway (on-prem device)
aws ec2 create-customer-gateway \
  --bgp-asn 65000 \
  --public-ip 203.0.113.10 \
  --type ipsec.1

# Create Virtual Private Gateway
aws ec2 create-vpn-gateway --type ipsec.1
aws ec2 attach-vpn-gateway \
  --vpn-gateway-id vgw-xxx \
  --vpc-id vpc-yyy

# Create VPN Connection
aws ec2 create-vpn-connection \
  --type ipsec.1 \
  --customer-gateway-id cgw-xxx \
  --vpn-gateway-id vgw-xxx
```

**GCP Cloud VPN (HA VPN):**

```bash
# Create HA VPN gateway
gcloud compute vpn-gateways create my-vpn-gw \
  --network=my-vpc \
  --region=europe-west1

# Create external VPN gateway (represents on-prem)
gcloud compute external-vpn-gateways create on-prem-gw \
  --interfaces 0=203.0.113.10

# Create VPN tunnel
gcloud compute vpn-tunnels create tunnel-1 \
  --peer-external-gateway=on-prem-gw \
  --peer-external-gateway-interface=0 \
  --region=europe-west1 \
  --ike-version=2 \
  --shared-secret="SuperSecretPSK123!" \
  --router=my-cloud-router \
  --vpn-gateway=my-vpn-gw \
  --vpn-gateway-interface=0
```

-----

## 9. Private Endpoints & Private Link

### What Are They and Why Do They Matter?

By default, cloud PaaS services (Azure Blob Storage, AWS S3, GCP Cloud SQL) have public endpoints — accessible over the internet with credentials. While credentials protect the data, the attack surface includes:

- Credential theft
- Network-based attacks
- Compliance violations (data must not traverse public internet)

**Private Endpoint** = A network interface with a private IP address in your VNet/VPC that fronts a PaaS service. All traffic flows through your private network — the public endpoint can be disabled entirely.

**Private Link** = The underlying service that powers Private Endpoints (Azure terminology). Also lets you expose your own services privately.

-----

### How It Works (Step by Step)

```
1. You create a Private Endpoint for Azure SQL in your data-subnet
   → Azure creates a NIC at 10.0.4.5 in your VNet

2. Azure creates a DNS A record in your Private DNS Zone:
   myserver.database.windows.net  →  10.0.4.5

3. When your app VM (10.0.3.10) connects to myserver.database.windows.net:
   a. DNS query goes to Azure DNS (168.63.129.16)
   b. Azure DNS checks Private DNS Zone → returns 10.0.4.5
   c. Traffic flows: VM → 10.0.4.5 (Private Endpoint NIC) → Azure SQL backend
   d. Never leaves Azure network. Never hits the public internet.

4. You set "Allow public network access: Disabled" on the SQL Server
   → Direct internet access is now blocked entirely
```

-----

### Private Endpoint vs Service Endpoint vs Public Access

|Feature                 |Public Access  |Service Endpoint        |Private Endpoint                 |
|------------------------|---------------|------------------------|---------------------------------|
|Network path            |Public internet|Optimized public IP path|Private IP in your VNet          |
|DNS resolution          |Public IP      |Public IP               |Private IP (via Private DNS)     |
|Block public access?    |N/A            |No                      |Yes (can disable public endpoint)|
|Works from peered VNets?|N/A            |No                      |Yes (via peering)                |
|Works from on-prem VPN? |Via internet   |No                      |Yes (DNS + routing)              |
|Data exfiltration risk  |Higher         |Medium                  |Lowest                           |
|Cost                    |Free           |Free                    |Charged (per endpoint + data)    |

-----

### Private DNS Zones per Service

Correct DNS configuration is **critical** for Private Endpoints. Without it, DNS resolves to the public IP and traffic bypasses the private endpoint.

**Azure:**

|Service                       |Private DNS Zone                    |
|------------------------------|------------------------------------|
|Azure Blob Storage            |`privatelink.blob.core.windows.net` |
|Azure Data Lake Gen2          |`privatelink.dfs.core.windows.net`  |
|Azure SQL Database            |`privatelink.database.windows.net`  |
|Azure Key Vault               |`privatelink.vaultcore.azure.net`   |
|Azure Databricks              |`privatelink.azuredatabricks.net`   |
|Azure Container Registry (ACR)|`privatelink.azurecr.io`            |
|Azure Event Hub / Service Bus |`privatelink.servicebus.windows.net`|
|Azure Monitor                 |`privatelink.monitor.azure.com`     |
|Azure Cosmos DB               |`privatelink.documents.azure.com`   |

**AWS (PrivateLink / VPC Endpoints):**

|Service        |Endpoint Type       |DNS Pattern                                |
|---------------|--------------------|-------------------------------------------|
|S3             |Gateway or Interface|`bucket.s3.amazonaws.com` (resolves to VPC)|
|EC2            |Interface           |`ec2.region.amazonaws.com`                 |
|Secrets Manager|Interface           |`secretsmanager.region.amazonaws.com`      |
|SQS            |Interface           |`sqs.region.amazonaws.com`                 |
|SSM            |Interface           |`ssm.region.amazonaws.com`                 |


> AWS has two types of VPC endpoints: **Gateway Endpoints** (free, for S3/DynamoDB only, route-table based) and **Interface Endpoints** (PrivateLink, charged, use ENIs + DNS).

**GCP (Private Service Connect / VPC Service Controls):**

- GCP uses **Private Service Connect (PSC)** endpoints — similar concept to Azure Private Endpoints.
- **VPC Service Controls** adds a perimeter around GCP services to prevent data exfiltration, even from within your project.

```bash
# GCP: Create Private Service Connect endpoint for Cloud SQL
gcloud compute forwarding-rules create my-psc-endpoint \
  --region=europe-west1 \
  --network=my-vpc \
  --address=my-private-ip \
  --target-service-attachment=projects/PROJECT/regions/REGION/serviceAttachments/ATTACHMENT
```

-----

## 10. NSG & Firewalls

### Network Security Groups (NSG)

**NSG** = Network Security Group. A stateful, L3/L4 packet filter attached to a subnet or NIC. It evaluates rules based on source/destination IP, protocol, and port. If no rule matches, the default deny rule applies.

**Stateful** means: if you allow inbound traffic on port 443, the return traffic is automatically allowed — you don’t need a separate outbound rule for the response.

-----

### NSG Rule Anatomy

|Property   |Value Options                        |Notes                                              |
|-----------|-------------------------------------|---------------------------------------------------|
|Priority   |100–4096                             |Lower number = evaluated first; first match wins   |
|Direction  |Inbound / Outbound                   |                                                   |
|Source     |IP, CIDR, Service Tag, ASG, `*` (any)|ASG = Application Security Group (logical grouping)|
|Source Port|Port, range (80-443), or `*`         |Usually `*` for source port                        |
|Destination|IP, CIDR, Service Tag, ASG, `*`      |                                                   |
|Dest Port  |Port, range, or `*`                  |e.g., `443`, `3306`, `8080-8090`                   |
|Protocol   |TCP, UDP, ICMP, ESP, AH, Any         |                                                   |
|Action     |Allow / Deny                         |                                                   |

**Default NSG rules (un-deletable, lowest priority):**

|Priority|Name                         |Direction|Source / Dest                  |Action|
|--------|-----------------------------|---------|-------------------------------|------|
|65000   |AllowVNetInBound             |Inbound  |VirtualNetwork → VirtualNetwork|Allow |
|65001   |AllowAzureLoadBalancerInBound|Inbound  |AzureLoadBalancer → Any        |Allow |
|65500   |DenyAllInBound               |Inbound  |Any → Any                      |Deny  |
|65000   |AllowVNetOutBound            |Outbound |VirtualNetwork → VirtualNetwork|Allow |
|65001   |AllowInternetOutBound        |Outbound |Any → Internet                 |Allow |
|65500   |DenyAllOutBound              |Outbound |Any → Any                      |Deny  |

**Practical example — allow HTTPS inbound, deny all else:**

```bash
# Allow HTTPS from anywhere
az network nsg rule create \
  --nsg-name myNSG \
  --resource-group myRG \
  --name AllowHTTPS \
  --priority 100 \
  --direction Inbound \
  --access Allow \
  --protocol Tcp \
  --source-address-prefixes '*' \
  --source-port-ranges '*' \
  --destination-address-prefixes '*' \
  --destination-port-ranges 443

# The default DenyAllInBound at priority 65500 blocks everything else
```

**AWS equivalent — Security Groups:**

- AWS Security Groups are instance-level, stateful, and work like Azure NSGs.
- AWS also has **NACLs** (Network ACLs) at the subnet level — these are **stateless** (you must allow both inbound AND return traffic explicitly).
- Security Groups: Allow rules only (no explicit deny). Default: deny all inbound, allow all outbound.
- NACLs: Allow AND deny rules. Evaluated in order by rule number.

**GCP equivalent — Firewall Rules:**

- GCP firewall rules are applied to VMs via **network tags** or **service accounts** — more flexible, not tied to subnets.
- Rules can be based on tags: e.g., allow port 443 to VMs tagged `web-server`.

```bash
# GCP: Allow HTTPS to VMs with tag "web-server"
gcloud compute firewall-rules create allow-https \
  --network=my-vpc \
  --action=allow \
  --direction=ingress \
  --rules=tcp:443 \
  --source-ranges=0.0.0.0/0 \
  --target-tags=web-server
```

-----

### Azure Firewall vs NSG

|Feature            |NSG                                    |Azure Firewall                            |
|-------------------|---------------------------------------|------------------------------------------|
|Layer              |L3/L4                                  |L3–L7                                     |
|Scope              |Per subnet / NIC                       |Centralized (shared across VNets)         |
|FQDN filtering     |No                                     |Yes (e.g., allow `*.pypi.org`)            |
|Threat intelligence|No                                     |Yes (block known malicious IPs/domains)   |
|IDPS               |No                                     |Yes (Premium SKU)                         |
|TLS inspection     |No                                     |Yes (Premium SKU)                         |
|Logging            |NSG Flow Logs → Storage / Log Analytics|Azure Monitor / Firewall Logs             |
|Cost               |Free                                   |~$1.25/hr + $0.016/GB data processed      |
|Use for            |Baseline network segmentation          |Central policy enforcement, egress control|

**IDPS** = Intrusion Detection and Prevention System. Inspects traffic patterns for attack signatures (e.g., SQL injection in URLs, exploit kits, C2 communications) and blocks or alerts.

**FQDN** = Fully Qualified Domain Name. A complete domain name (e.g., `api.openai.com`). Azure Firewall can filter by FQDN, even for outbound traffic where the destination IP changes.

-----

### WAF (Web Application Firewall)

**WAF** = Web Application Firewall. A specialized L7 firewall that inspects HTTP/HTTPS traffic and blocks attacks targeting web application vulnerabilities.

**OWASP Top 10 threats a WAF addresses:**

|# |Attack                          |Description                                                |
|--|--------------------------------|-----------------------------------------------------------|
|1 |Broken Access Control           |Users accessing resources they shouldn’t                   |
|2 |Cryptographic Failures          |Weak/no encryption of sensitive data                       |
|3 |Injection (SQL, LDAP, OS)       |Malicious code injected via input fields                   |
|4 |Insecure Design                 |Architectural flaws                                        |
|5 |Security Misconfiguration       |Default creds, open ports, verbose errors                  |
|6 |Vulnerable Components           |Outdated libraries                                         |
|7 |Auth Failures                   |Weak session management, brute force                       |
|8 |Software/Data Integrity Failures|Unsigned updates, insecure CI/CD                           |
|9 |Logging Failures                |No audit trail for attacks                                 |
|10|SSRF                            |Server-Side Request Forgery — server fetches malicious URLs|

|WAF Product          |Platform|Notes                                                    |
|---------------------|--------|---------------------------------------------------------|
|Azure App Gateway WAF|Azure   |Regional; OWASP 3.2 rule sets; Detection/Prevention modes|
|Azure Front Door WAF |Azure   |Global; custom rules; rate limiting; geo-blocking        |
|AWS WAF              |AWS     |Attached to ALB, CloudFront, API Gateway; managed rules  |
|Google Cloud Armor   |GCP     |Attached to GFE/Cloud Load Balancing; adaptive protection|
|Cloudflare WAF       |Any     |Third-party; edge-based; DDoS + bot management           |

-----

## 11. IP Whitelisting & Access Control

### What Is IP Whitelisting?

**IP whitelisting** (also called allowlisting) = explicitly allowing access from specific IP addresses or CIDR ranges, and denying everyone else. It’s a coarse-grained but simple and effective access control mechanism.

**Limitations:** IP addresses can be spoofed, VPNs change IPs, and cloud services often have dynamic IP ranges. Whitelisting works best as one layer in a defense-in-depth strategy, not as the sole access control.

**Allowlist vs Denylist (Blocklist):**

- **Allowlist (whitelist):** Only listed IPs allowed. Everything else denied. More restrictive, better for sensitive services.
- **Denylist (blacklist):** Listed IPs are blocked. Everything else allowed. Used for known bad actors.

-----

### Common Whitelisting Points in Cloud

|Resource Type   |Azure                                 |AWS                                 |GCP                                     |
|----------------|--------------------------------------|------------------------------------|----------------------------------------|
|VM / Compute    |NSG inbound rules                     |Security Group inbound rules        |VPC Firewall rules (ingress)            |
|Storage         |Storage Account network firewall rules|S3 bucket policy (`aws:SourceIp`)   |Cloud Storage VPC Service Controls / IAM|
|Database        |SQL Server / PostgreSQL firewall rules|RDS Security Group                  |Cloud SQL authorized networks           |
|Key/Secret Store|Key Vault firewall + VNet rules       |Secrets Manager resource policy     |Secret Manager VPC Service Controls     |
|API / Web App   |App Service access restrictions       |API Gateway resource policy         |Cloud Endpoints + IAP                   |
|Kubernetes      |AKS authorized IP ranges              |EKS API server endpoint restrictions|GKE master authorized networks          |
|Analytics       |Databricks IP access lists            |Redshift VPC + SG                   |BigQuery VPC Service Controls           |

-----

### Azure Service Tags

Service Tags are Microsoft-managed groups of IP address ranges for Azure services. They update automatically as service IPs change — no manual maintenance needed.

|Service Tag            |Represents                                     |
|-----------------------|-----------------------------------------------|
|`AzureCloud`           |All Azure public IPs                           |
|`AzureCloud.WestEurope`|Azure public IPs in West Europe region         |
|`AzureDevOps`          |Azure DevOps service IPs                       |
|`Storage`              |Azure Storage service IPs                      |
|`Storage.WestEurope`   |Storage IPs in West Europe                     |
|`Sql`                  |Azure SQL Database IPs                         |
|`AppService`           |Azure App Service outbound IPs                 |
|`DataFactory`          |Azure Data Factory integration runtime IPs     |
|`AzureMonitor`         |Azure Monitor (Log Analytics, App Insights) IPs|
|`VirtualNetwork`       |All VNet address spaces (your subnets)         |
|`Internet`             |Any IP outside of Azure/VNet                   |

**AWS equivalent:** AWS IP ranges are published at `https://ip-ranges.amazonaws.com/ip-ranges.json`. AWS also has VPC Prefix Lists (managed lists of CIDRs for use in Security Groups and route tables). For AWS-managed ranges, use Managed Prefix Lists (e.g., `com.amazonaws.region.s3`).

**GCP equivalent:** GCP publishes IP ranges at `https://www.gstatic.com/ipranges/cloud.json`. No equivalent “Service Tags” — use VPC Service Controls and IAM conditions instead.

-----

## 12. Network Routing

### What Is Routing?

Routing determines the path a packet takes to reach its destination. A **route table** (also called a routing table) is a set of rules that maps destination CIDRs to next-hop addresses.

**Next hop types:**

|Next Hop               |Meaning                                                   |
|-----------------------|----------------------------------------------------------|
|Virtual Network / local|Destination is within the same VNet — route internally    |
|Internet               |Route traffic to the public internet via the IGW          |
|Virtual appliance      |Send to a specific IP (e.g., Azure Firewall, NVA)         |
|VNet Gateway           |Route to VPN/ExpressRoute gateway for on-prem or VNet-VNet|
|None / Blackhole       |Drop traffic to this destination                          |

-----

### UDR — User Defined Routes

**UDR** = User Defined Route. Overrides Azure/AWS system default routes to implement custom routing logic.

**Common UDR patterns:**

|Scenario                           |Route            |Next Hop                       |
|-----------------------------------|-----------------|-------------------------------|
|Force internet traffic via firewall|`0.0.0.0/0`      |Virtual appliance (Firewall IP)|
|Route on-prem traffic to VPN GW    |`10.0.0.0/8`     |Virtual Network Gateway        |
|Blackhole suspicious ranges        |`198.51.100.0/24`|None                           |
|Route between peered VNets via FW  |Peer CIDR        |Virtual appliance              |

**Azure example — force all internet egress through Azure Firewall:**

```bash
az network route-table create \
  --name ForcedEgressRT \
  --resource-group myRG \
  --location westeurope

az network route-table route create \
  --name DefaultToFirewall \
  --route-table-name ForcedEgressRT \
  --resource-group myRG \
  --address-prefix 0.0.0.0/0 \
  --next-hop-type VirtualAppliance \
  --next-hop-ip-address 10.0.0.4   # Azure Firewall private IP

# Associate with app-subnet
az network vnet subnet update \
  --name app-subnet \
  --vnet-name myVNet \
  --resource-group myRG \
  --route-table ForcedEgressRT
```

-----

### BGP — Border Gateway Protocol

**BGP** = Border Gateway Protocol. The routing protocol that makes the internet work. It’s used between ISPs (and cloud gateways) to exchange reachability information about IP prefixes.

**ASN** = Autonomous System Number. A unique number assigned to a network (ISP, cloud provider, large enterprise) that runs BGP. Azure’s ASN is 65515 (for VPN Gateway), AWS uses 64512 by default for VGW.

**In cloud:**

- Used in Site-to-Site VPNs for dynamic route exchange (vs. static routing where you hardcode CIDRs).
- With BGP: when you add a new subnet on-prem, it’s automatically advertised to the cloud gateway. Without BGP, you’d manually update static routes.
- Used in ExpressRoute/Direct Connect for enterprise-scale route exchange.

**BGP key terms:**

|Term               |Meaning                                                      |
|-------------------|-------------------------------------------------------------|
|eBGP               |External BGP — between different ASNs (internet, cross-cloud)|
|iBGP               |Internal BGP — within the same AS                            |
|Route advertisement|Publishing a CIDR prefix to BGP peers                        |
|Route filtering    |Controlling which prefixes are accepted/sent to peers        |
|AS-PATH            |List of ASNs a route has traversed — used to detect loops    |

-----

## 13. CDN — Content Delivery Network

### What Is a CDN and Why Use It?

**CDN** = Content Delivery Network. A globally distributed network of **edge servers** (called **PoPs** — Points of Presence) that cache copies of your content close to end users. Instead of every user downloading a file from your origin server in one region, they download it from an edge node in their city.

**Why it matters:**

- **Latency:** A user in Tokyo accessing a file from a European origin server has 200–300ms RTT. A cached copy in a Tokyo PoP brings that to <10ms.
- **Origin offload:** 90%+ of static traffic can be served from cache, drastically reducing load on your servers.
- **DDoS resilience:** CDN absorbs volumetric attacks at the edge before they reach your origin.
- **Availability:** If your origin is down, TTL-based cache serves recent copies.

-----

### CDN Concepts

|Concept      |Description                                                                     |
|-------------|--------------------------------------------------------------------------------|
|PoP          |Point of Presence — an edge data center in a city/region                        |
|Origin       |Your actual backend server or storage bucket                                    |
|Edge Cache   |The CDN server that serves cached copies to users                               |
|Cache-Control|HTTP response header controlling what can be cached and for how long            |
|`max-age`    |Seconds the browser/edge can cache a response                                   |
|`s-maxage`   |Overrides `max-age` for CDN caches (not browser)                                |
|`no-cache`   |Must revalidate with origin on each request                                     |
|`no-store`   |Never cache (sensitive data)                                                    |
|`private`    |Browser can cache, CDN cannot                                                   |
|ETag         |Version identifier — CDN asks origin “has this file changed?” (304 Not Modified)|
|Cache Purge  |Forcibly clear cached content before TTL expires                                |
|Cache Key    |What attributes identify a unique cacheable object (URL, query params, headers) |
|Origin Shield|A CDN tier between PoPs and origin — reduces origin requests                    |

**Example Cache-Control header:**

```
Cache-Control: public, max-age=86400, s-maxage=604800
```

→ Browser caches for 1 day, CDN edge caches for 7 days.

-----

### Cloud CDN Products

|Product              |Provider   |Key Features                                                 |
|---------------------|-----------|-------------------------------------------------------------|
|Azure CDN (Microsoft)|Azure      |Integrated with Front Door; HTTPS; custom rules              |
|Azure Front Door     |Azure      |CDN + WAF + Global LB + health probes — full edge stack      |
|AWS CloudFront       |AWS        |Deep AWS integration; Lambda@Edge; Origin Shield; signed URLs|
|GCP Cloud CDN        |GCP        |Integrated with GFE; Cache modes; signed URLs                |
|Cloudflare           |Third-party|Largest PoP network (~310 cities); CDN + WAF + DDoS + DNS    |
|Fastly               |Third-party|Real-time config push; VCL customization; popular with devs  |
|Akamai               |Third-party|Largest enterprise CDN; edge compute; IoT                    |

**AWS CloudFront example — distribution with S3 origin:**

```bash
aws cloudfront create-distribution \
  --origin-domain-name mybucket.s3.amazonaws.com \
  --default-root-object index.html
```

**GCP Cloud CDN — enable on backend service:**

```bash
gcloud compute backend-services update my-backend \
  --global \
  --enable-cdn \
  --cache-mode=CACHE_ALL_STATIC
```

-----

## 14. Network Monitoring & Diagnostics

### Why Network Monitoring Matters

Silent failures are the worst kind. A misconfigured NSG rule, a broken DNS override for a Private Endpoint, or a saturated NAT port pool can silently cause connectivity issues or security gaps. Network monitoring provides visibility and the ability to detect and diagnose these issues quickly.

-----

### Cloud-Native Tools

**Azure:**

|Tool                   |Purpose                                                                |
|-----------------------|-----------------------------------------------------------------------|
|Network Watcher        |Suite of network diagnostic tools (flow logs, topology, IP flow verify)|
|NSG Flow Logs          |Log all allowed/denied flows through NSGs → Storage or Log Analytics   |
|VNet Flow Logs         |More granular traffic analysis at VNet level (preview)                 |
|Connection Troubleshoot|Check connectivity between source and destination, show hops           |
|IP Flow Verify         |Test if a specific packet (IP, port, direction) would be allowed by NSG|
|Packet Capture         |Capture packets from a VM’s NIC to a Storage blob                      |
|Next Hop               |What route would a packet from VM X to IP Y take?                      |
|Azure Monitor          |Metrics, alerts, dashboards for all Azure resources including network  |

**AWS:**

|Tool                   |Purpose                                                                     |
|-----------------------|----------------------------------------------------------------------------|
|VPC Flow Logs          |Log IP traffic to/from ENIs → CloudWatch Logs or S3                         |
|Reachability Analyzer  |Check connectivity between two resources and explain why it’s allowed/denied|
|Network Access Analyzer|Identifies unintended access paths in your network configuration            |
|Traffic Mirroring      |Copy ENI traffic to a monitoring appliance                                  |
|CloudWatch             |Metrics and alarms for ALB, NLB, VPN, Direct Connect                        |

**GCP:**

|Tool                       |Purpose                                                             |
|---------------------------|--------------------------------------------------------------------|
|VPC Flow Logs              |Log sampled flows from VMs/GKE nodes → Cloud Logging                |
|Firewall Insights          |Analyze firewall rule effectiveness, find unused rules              |
|Network Intelligence Center|Topology visualization, connectivity tests, performance monitoring  |
|Connectivity Tests         |Verify paths between GCP resources with detailed hop-by-hop analysis|
|Cloud Monitoring           |Metrics for Load Balancers, VPN, Interconnect                       |

-----

### CLI Diagnostic Commands

|Command                               |Purpose                                     |
|--------------------------------------|--------------------------------------------|
|`ping 8.8.8.8`                        |Test ICMP reachability (L3)                 |
|`traceroute 8.8.8.8` / `tracert`      |Show hop-by-hop path to destination         |
|`nslookup mydb.database.windows.net`  |DNS resolution — what IP does a name return?|
|`dig mydb.database.windows.net +short`|DNS resolution (more detail than nslookup)  |
|`curl -v https://myapi.example.com`   |Full HTTP/HTTPS connection test with headers|
|`telnet 10.0.4.5 1433`                |Test TCP port reachability (L4)             |
|`nc -zv 10.0.4.5 5432`                |Test TCP/UDP port (netcat)                  |
|`netstat -an | grep LISTEN`           |Show all listening ports on local machine   |
|`ss -tlnp`                            |Modern replacement for netstat (Linux)      |
|`ip route show`                       |Show routing table on Linux                 |
|`route print`                         |Show routing table on Windows               |
|`arp -a`                              |Show ARP cache (IP → MAC mapping)           |
|`tcpdump -i eth0 port 443`            |Capture live packets on interface           |
|`openssl s_client -connect host:443`  |Test TLS certificate and chain              |

**NSG Flow Log format (Azure):**

```
timestamp,macAddress,category,resourceId,operationName,
version,flow=(direction,srcIP,srcPort,dstIP,dstPort,proto,action,bytesFromSrc,bytesToSrc)

Example:
2026-01-15T10:23:45Z,000D3A123456,NetworkSecurityGroupFlowEvent,...,
2,I,10.0.3.10,52341,10.0.4.5,1433,T,A,1024,2048
```

→ Inbound TCP from 10.0.3.10:52341 to 10.0.4.5:1433 — Allowed, 1KB sent, 2KB received.

-----

## 15. Zero Trust Networking

### What Is Zero Trust?

Traditional **perimeter security** model: “Hard outside, soft inside.” Once you’re inside the network (through VPN or on-prem), you’re trusted. This breaks down because:

- VPN credentials get stolen.
- Insider threats are real.
- Cloud workloads span multiple networks — there is no single perimeter.

**Zero Trust** (coined by John Kindervag at Forrester in 2010, popularized by Google’s BeyondCorp): “Never trust, always verify.” Every access request is authenticated and authorized regardless of network location.

-----

### Zero Trust Principles

|Principle                |Meaning                                                                 |Implementation                                          |
|-------------------------|------------------------------------------------------------------------|--------------------------------------------------------|
|Verify explicitly        |Always authenticate and authorize based on all available data points    |MFA, Conditional Access, device compliance checks       |
|Use least privilege      |Grant minimum permissions required, just-in-time                        |RBAC, PIM (Privileged Identity Management), JIT access  |
|Assume breach            |Minimize blast radius, segment everything, assume attacker is already in|Micro-segmentation, NSGs, Private Endpoints             |
|Inspect & log all traffic|No implicit trust even inside the network                               |Firewall inspection, flow logs, SIEM                    |
|Authenticate all services|Service-to-service calls also authenticated                             |Managed Identities, workload identity, mutual TLS (mTLS)|

**mTLS** = Mutual TLS. Both client AND server present certificates to authenticate each other — unlike standard TLS where only the server has a cert. Used in service meshes (Istio, Linkerd) for east-west traffic security.

-----

### Zero Trust in Cloud

|Layer     |Control                     |Azure                        |AWS                          |GCP                           |
|----------|----------------------------|-----------------------------|-----------------------------|------------------------------|
|Identity  |No implicit network trust   |Entra ID + Managed Identities|IAM + Instance Profiles      |Workload Identity             |
|Device    |Device must be compliant    |Intune + Conditional Access  |AWS Config + SSM             |Chrome Enterprise + BeyondCorp|
|Network   |No implicit VNet trust      |NSG + Private Endpoints + FW |Security Groups + PrivateLink|VPC FW rules + PSC            |
|App/API   |Auth per request            |APIM + Entra ID OAuth2       |API Gateway + Cognito        |Apigee + IAP                  |
|Data      |Encryption + access policies|Key Vault + RBAC + CMK       |KMS + S3 bucket policies     |Cloud KMS + IAM               |
|Monitoring|Detect anomalies            |Microsoft Defender for Cloud |AWS Security Hub + GuardDuty |Security Command Center       |

**JIT** = Just-In-Time. Access is granted for a limited time window when needed, then revoked. Reduces standing privileged access (a major attack vector).
**PIM** = Privileged Identity Management (Azure). Manages, controls, and monitors privileged access to Azure resources.

-----

## 16. Encryption in Transit

### Why Encrypt Network Traffic?

Data traveling across networks can be intercepted. Even within a cloud provider’s backbone, defense-in-depth recommends encrypting sensitive traffic. Regulatory frameworks (GDPR, PCI-DSS, HIPAA) often mandate encryption in transit.

**Threat:** A network attacker performing a **Man-in-the-Middle (MITM)** attack intercepts communication between two parties, reading or modifying data without either party knowing.

-----

### TLS (Transport Layer Security)

**TLS** = Transport Layer Security. Successor to SSL (Secure Sockets Layer — deprecated, insecure). Provides:

- **Encryption:** Data cannot be read in transit.
- **Authentication:** Server (and optionally client) proves identity via certificate.
- **Integrity:** Data cannot be tampered with undetected.

|Version|Status         |Notes                                                 |
|-------|---------------|------------------------------------------------------|
|SSL 2.0|Broken         |Never use                                             |
|SSL 3.0|Broken (POODLE)|Never use                                             |
|TLS 1.0|Deprecated     |PCI-DSS mandates disabling                            |
|TLS 1.1|Deprecated     |Disable                                               |
|TLS 1.2|Current minimum|Widely supported — require this as minimum            |
|TLS 1.3|Current best   |Faster handshake, stronger cipher suites — prefer this|

**TLS Certificate Types:**

|Type    |Full Name               |Validation Level     |Use Case                           |
|--------|------------------------|---------------------|-----------------------------------|
|DV      |Domain Validated        |Domain ownership only|Blogs, internal tools              |
|OV      |Organization Validated  |Domain + org identity|Business websites                  |
|EV      |Extended Validation     |Full org verification|Banks, e-commerce                  |
|Wildcard|`*.example.com`         |DV or OV             |All subdomains of a domain         |
|SAN     |Subject Alternative Name|Multiple domains     |Single cert covers multiple domains|

-----

### Encryption in Cloud

|Scenario                   |Azure                                |AWS                      |GCP                         |
|---------------------------|-------------------------------------|-------------------------|----------------------------|
|VM to VM (same VNet)       |Encrypted by default (Azure backbone)|Encrypted by default     |Encrypted by default        |
|App to database            |Enforce TLS (SQL: `ssl=require`)     |RDS SSL certificate      |Cloud SQL: `sslmode=require`|
|App to storage             |HTTPS only (disable HTTP)            |S3: `aws:SecureTransport`|GCS: HTTPS enforced         |
|VPN tunnel                 |IPsec (IKEv2)                        |IPsec (IKEv2/IKEv1)      |ESP/AES-256                 |
|ExpressRoute/Interconnect  |Optional MACsec (layer 2)            |Optional MACsec          |Optional MACsec             |
|Internal service-to-service|mTLS via App Gateway / Service Mesh  |ACM Private CA + mTLS    |Istio / Anthos mTLS         |

**CMK** = Customer-Managed Key. You manage the encryption key in a Key Vault/KMS rather than the provider managing it — gives you control to revoke access.

-----

## 17. DDoS Protection

### What Is DDoS?

**DDoS** = Distributed Denial of Service. An attack where thousands/millions of compromised machines (a **botnet**) flood a target with traffic, overwhelming it and making it unavailable to legitimate users.

|Attack Type     |OSI Layer|Description                                            |Example                                 |
|----------------|---------|-------------------------------------------------------|----------------------------------------|
|Volumetric      |L3/L4    |Flood the pipe with traffic (Gbps)                     |UDP flood, ICMP flood, DNS amplification|
|Protocol        |L3/L4    |Exploit protocol weaknesses to exhaust resources       |SYN flood, Ping of Death, Smurf         |
|Application (L7)|L7       |Exhaust application-level resources with valid requests|HTTP flood, Slowloris, cache bypass     |

**SYN flood:** Attacker sends millions of TCP SYN packets but never completes the 3-way handshake. Server allocates resources for each half-open connection until it runs out.

**Amplification attack:** Attacker sends small requests with forged source IP (victim’s IP) to DNS/NTP/MEMCACHED servers that respond with large replies — victim is flooded.

-----

### DDoS Protection in Cloud

|Feature         |Azure DDoS Basic|Azure DDoS Standard   |AWS Shield Standard|AWS Shield Advanced |GCP Cloud Armor         |
|----------------|----------------|----------------------|-------------------|--------------------|------------------------|
|Cost            |Free (included) |~$2,944/month per VNet|Free               |$3,000/month + DRR  |Per policy + per request|
|L3/L4 protection|Yes             |Yes                   |Yes                |Yes                 |Yes                     |
|L7 protection   |No              |Yes (with App GW WAF) |No                 |Yes (with ALB + WAF)|Yes                     |
|Auto mitigation |Basic           |Adaptive, tuned       |Basic              |Enhanced            |Configurable rules      |
|Attack telemetry|No              |Yes (metrics + alerts)|No                 |Yes                 |Yes                     |
|24/7 DRT support|No              |Yes                   |No                 |Yes                 |Via Google support      |
|Cost protection |No              |Yes (bill protection) |No                 |Yes                 |No                      |

**DRT** = DDoS Response Team. Dedicated security engineers who assist during active attacks.

-----

## 18. Cloud Networking Architectures

### Azure — Full Reference Architecture

```
Azure Subscription
└── Resource Group: rg-network-prod
    ├── VNet: vnet-prod (10.0.0.0/16)
    │   ├── GatewaySubnet (10.0.0.0/27)
    │   │   └── VPN Gateway / ExpressRoute GW
    │   ├── AzureBastionSubnet (10.0.1.0/26)
    │   │   └── Azure Bastion Host
    │   ├── AzureFirewallSubnet (10.0.2.0/26)
    │   │   └── Azure Firewall (hub)
    │   ├── web-subnet (10.0.3.0/24) + NSG
    │   │   └── App Service / VMs + Public IP / App Gateway
    │   ├── app-subnet (10.0.4.0/24) + NSG + UDR → Firewall
    │   │   └── Function Apps / AKS / VMs
    │   └── data-subnet (10.0.5.0/24) + NSG + UDR → Firewall
    │       └── Private Endpoints (SQL, Storage, KeyVault, Databricks)
    ├── Private DNS Zones
    │   ├── privatelink.database.windows.net → PE IP
    │   ├── privatelink.blob.core.windows.net → PE IP
    │   └── privatelink.vaultcore.azure.net → PE IP
    ├── NAT Gateway (for app-subnet outbound)
    ├── Azure Firewall (central egress + inspection)
    ├── Application Gateway (+ WAF) → web-subnet
    └── Azure Front Door (global entry) → App Gateway
```

**Azure networking component summary:**

|Component          |Abbreviation|Description                                          |
|-------------------|------------|-----------------------------------------------------|
|VNet               |—           |Virtual Network — isolated private cloud network     |
|NSG                |NSG         |Network Security Group — L3/L4 stateful packet filter|
|UDR                |UDR         |User Defined Route — custom routing rule             |
|NAT Gateway        |NAT GW      |Managed SNAT for private subnet outbound access      |
|VPN Gateway        |VPN GW      |IPsec VPN endpoint for S2S and P2S                   |
|ExpressRoute GW    |ER GW       |Gateway for dedicated private circuit connectivity   |
|Azure Firewall     |AzFW        |Managed L7 firewall with FQDN, IDPS, TLS inspection  |
|Application Gateway|AppGW       |Regional L7 LB + WAF                                 |
|Azure Front Door   |AFD         |Global L7 LB + CDN + WAF                             |
|Traffic Manager    |TM          |DNS-based global load balancing                      |
|Private Endpoint   |PE          |Private IP for PaaS in VNet                          |
|Private DNS Zone   |PDZ         |Internal DNS for private endpoint resolution         |
|Bastion            |—           |Secure RDP/SSH over HTTPS without public IP on VMs   |
|DDoS Protection    |—           |Standard: adaptive mitigation + telemetry            |
|Network Watcher    |NW          |Diagnostics: flow logs, packet capture, topology     |

-----

### AWS — Full Reference Architecture

```
AWS Account
└── VPC: vpc-prod (10.0.0.0/16) — Region: eu-west-1
    ├── Public Subnet AZ-a (10.0.1.0/24)
    │   └── ALB / NLB, NAT Gateway, Bastion EC2
    ├── Public Subnet AZ-b (10.0.2.0/24)
    │   └── ALB / NLB replica (Multi-AZ)
    ├── App Subnet AZ-a (10.0.3.0/24)
    │   └── EC2 / ECS / EKS nodes — Security Group: sg-app
    ├── App Subnet AZ-b (10.0.4.0/24)
    │   └── EC2 / ECS replicas
    ├── Data Subnet AZ-a (10.0.5.0/24)
    │   └── RDS, ElastiCache — Security Group: sg-data
    ├── Data Subnet AZ-b (10.0.6.0/24)
    │   └── RDS standby replica
    ├── Internet Gateway (IGW) — attached to VPC for public subnets
    ├── NAT Gateway (in public subnet) — outbound for private subnets
    ├── Route Tables
    │   ├── public-rt: 0.0.0.0/0 → IGW
    │   └── private-rt: 0.0.0.0/0 → NAT GW
    ├── VPC Endpoints (PrivateLink)
    │   ├── Interface: secretsmanager, ssm, ecr
    │   └── Gateway: S3, DynamoDB (free)
    └── VPN Gateway / Direct Connect GW → on-prem
```

**AWS networking component summary:**

|Component     |Abbreviation|Description                                                 |
|--------------|------------|------------------------------------------------------------|
|VPC           |VPC         |Virtual Private Cloud — isolated network                    |
|IGW           |IGW         |Internet Gateway — enables internet for public subnets      |
|NAT Gateway   |NAT GW      |Managed SNAT for private subnet outbound access             |
|Security Group|SG          |Stateful L4 firewall (instance-level)                       |
|NACL          |NACL        |Network ACL — stateless subnet-level firewall               |
|ENI           |ENI         |Elastic Network Interface — virtual NIC                     |
|EIP           |EIP         |Elastic IP — static public IPv4                             |
|ALB           |ALB         |Application Load Balancer (L7)                              |
|NLB           |NLB         |Network Load Balancer (L4)                                  |
|TGW           |TGW         |Transit Gateway — hub for transitive VPC routing            |
|VGW           |VGW         |Virtual Private Gateway — VPN/Direct Connect endpoint in VPC|
|CGW           |CGW         |Customer Gateway — represents on-prem device in AWS         |
|VPC Endpoint  |VPCE        |PrivateLink interface or gateway endpoint                   |
|Route 53      |R53         |DNS service                                                 |
|CloudFront    |CF          |CDN                                                         |
|WAF           |WAF         |Web Application Firewall                                    |
|Shield        |—           |DDoS protection                                             |

-----

### GCP — Full Reference Architecture

```
GCP Project
└── VPC Network: vpc-prod (GLOBAL — spans all regions)
    ├── Subnet: europe-west1-app (10.0.1.0/24) — Regional
    │   └── GKE nodes / Compute Engine VMs
    ├── Subnet: europe-west1-data (10.0.2.0/24)
    │   └── Cloud SQL Private IP, Memorystore
    ├── Subnet: us-east1-app (10.1.1.0/24) — Different region, same VPC!
    │   └── Cross-region replicas
    ├── Firewall Rules (applied via network tags)
    │   ├── allow-http: tag=web-server, port 80/443
    │   ├── allow-internal: 10.0.0.0/8 → any
    │   └── deny-all: default deny
    ├── Cloud Router + HA VPN → on-prem
    ├── Private Service Connect → Cloud SQL, Cloud Storage
    ├── GFE (Global Frontend) → HTTP(S) Load Balancer + Cloud Armor (WAF)
    └── Cloud DNS (private zones for internal resolution)
```

**GCP networking component summary:**

|Component                  |Abbreviation|Description                                                    |
|---------------------------|------------|---------------------------------------------------------------|
|VPC                        |VPC         |Global virtual network (spans regions)                         |
|GFE                        |GFE         |Google Frontend — global L7 load balancer / anycast entry point|
|Cloud Armor                |—           |WAF + DDoS protection                                          |
|Cloud Router               |—           |BGP routing for VPN/Interconnect                               |
|HA VPN                     |—           |High Availability VPN (99.99% SLA)                             |
|Cloud Interconnect         |—           |Dedicated/Partner private connectivity (like ExpressRoute)     |
|PSC                        |PSC         |Private Service Connect — private access to GCP services       |
|VPC Service Controls       |VPC-SC      |Perimeter around GCP APIs to prevent data exfiltration         |
|Cloud NAT                  |—           |Managed SNAT for private instances                             |
|Cloud DNS                  |—           |Managed DNS service                                            |
|Shared VPC / XPN           |XPN         |Share a VPC across multiple GCP projects                       |
|Cloud CDN                  |—           |CDN integrated with GFE                                        |
|Network Intelligence Center|NIC         |Topology, connectivity tests, performance dashboard            |

-----

## 19. Quick Reference — Port Numbers

|Port   |Protocol|Service                     |Notes                                     |
|-------|--------|----------------------------|------------------------------------------|
|20/21  |TCP     |FTP (data / control)        |Insecure — use SFTP (port 22) instead     |
|22     |TCP     |SSH / SFTP / SCP            |Secure remote access — restrict source IP |
|23     |TCP     |Telnet                      |Insecure (plaintext) — never use          |
|25     |TCP     |SMTP (server-to-server)     |Block inbound on VMs (spam prevention)    |
|53     |TCP/UDP |DNS                         |UDP for queries, TCP for large responses  |
|67/68  |UDP     |DHCP (server/client)        |                                          |
|80     |TCP     |HTTP                        |Redirect to 443 in production             |
|110    |TCP     |POP3                        |Email retrieval (legacy)                  |
|123    |UDP     |NTP                         |Time sync — needed for TLS, Kerberos      |
|143    |TCP     |IMAP                        |Email retrieval                           |
|161/162|UDP     |SNMP (get/trap)             |Network device monitoring                 |
|389    |TCP/UDP |LDAP                        |Active Directory queries                  |
|443    |TCP     |HTTPS                       |TLS-encrypted HTTP                        |
|445    |TCP     |SMB                         |Windows file shares — block externally!   |
|587    |TCP     |SMTP (submission)           |Authenticated email sending               |
|636    |TCP     |LDAPS                       |LDAP over TLS                             |
|993    |TCP     |IMAPS                       |IMAP over TLS                             |
|995    |TCP     |POP3S                       |POP3 over TLS                             |
|1433   |TCP     |Microsoft SQL Server        |Never expose publicly                     |
|1521   |TCP     |Oracle Database             |Never expose publicly                     |
|2375   |TCP     |Docker daemon (unencrypted) |Very dangerous if exposed — never use     |
|2376   |TCP     |Docker daemon (TLS)         |                                          |
|3306   |TCP     |MySQL / MariaDB             |Never expose publicly                     |
|3389   |TCP     |RDP                         |High-risk — use Bastion instead           |
|4443   |TCP     |HTTPS (alt)                 |Alternate HTTPS port                      |
|5432   |TCP     |PostgreSQL                  |Never expose publicly                     |
|5672   |TCP     |RabbitMQ AMQP               |                                          |
|5671   |TCP     |AMQP over TLS               |                                          |
|6379   |TCP     |Redis                       |Never expose publicly (no auth by default)|
|6443   |TCP     |Kubernetes API Server       |                                          |
|8080   |TCP     |HTTP (alternate / dev proxy)|                                          |
|8443   |TCP     |HTTPS (alternate)           |                                          |
|9092   |TCP     |Apache Kafka                |                                          |
|9200   |TCP     |Elasticsearch HTTP          |                                          |
|9300   |TCP     |Elasticsearch cluster comm  |                                          |
|15432  |TCP     |PostgreSQL (alt)            |                                          |
|27017  |TCP     |MongoDB                     |Never expose publicly                     |
|50000  |TCP     |Jenkins                     |                                          |

-----

## 20. Abbreviations Glossary

|Abbreviation|Full Name                                |Context                                         |
|------------|-----------------------------------------|------------------------------------------------|
|ACL         |Access Control List                      |Network (NACLs in AWS) or file system           |
|ALB         |Application Load Balancer                |AWS L7 load balancer                            |
|APIPA       |Automatic Private IP Addressing          |IP assigned when DHCP fails (169.254.x.x)       |
|ARP         |Address Resolution Protocol              |Maps IP → MAC on local network                  |
|AS          |Autonomous System                        |Network under single routing policy             |
|ASG         |Application Security Group               |Azure: logical grouping for NSG rules           |
|ASN         |Autonomous System Number                 |Unique ID for BGP routing entities              |
|AZ          |Availability Zone                        |Isolated data center within a region            |
|BGP         |Border Gateway Protocol                  |Internet’s dynamic routing protocol             |
|CDN         |Content Delivery Network                 |Edge caching / acceleration network             |
|CIDR        |Classless Inter-Domain Routing           |IP addressing with variable-length prefixes     |
|CMK         |Customer-Managed Key                     |Encryption key managed by customer              |
|DNAT        |Destination NAT                          |Rewrite destination IP (port forwarding)        |
|DNS         |Domain Name System                       |Name → IP resolution                            |
|DNSSEC      |DNS Security Extensions                  |Cryptographic signing of DNS records            |
|DDNS        |Dynamic DNS                              |Auto-update DNS when IP changes                 |
|DDoS        |Distributed Denial of Service            |Attack flooding a service with traffic          |
|DHCP        |Dynamic Host Configuration Protocol      |Automatic IP assignment                         |
|DRT         |DDoS Response Team                       |AWS/Azure expert team for attack mitigation     |
|EIP         |Elastic IP                               |AWS static public IPv4 address                  |
|ENI         |Elastic Network Interface                |AWS virtual NIC                                 |
|eBGP        |External BGP                             |BGP between different ASNs                      |
|FQDN        |Fully Qualified Domain Name              |Complete domain name (e.g., api.example.com)    |
|GFE         |Google Frontend                          |GCP’s global L7 load balancer infrastructure    |
|HA          |High Availability                        |System designed to minimize downtime            |
|HTTP        |HyperText Transfer Protocol              |Web communication (unencrypted)                 |
|HTTPS       |HTTP Secure                              |HTTP over TLS                                   |
|iBGP        |Internal BGP                             |BGP within the same AS                          |
|ICMP        |Internet Control Message Protocol        |Ping, traceroute, error messages                |
|IDPS        |Intrusion Detection and Prevention System|Network threat detection/blocking               |
|IGW         |Internet Gateway                         |AWS: gateway enabling internet access           |
|IKE         |Internet Key Exchange                    |VPN key negotiation protocol                    |
|IP          |Internet Protocol                        |L3 addressing and routing                       |
|IPsec       |IP Security                              |VPN encryption suite                            |
|JIT         |Just-In-Time                             |Temporary access provisioning                   |
|L2/L3/L4/L7 |OSI Layer 2/3/4/7                        |Data Link / Network / Transport / Application   |
|LDAP        |Lightweight Directory Access Protocol    |Directory (AD) query protocol                   |
|LB          |Load Balancer                            |Traffic distribution across backends            |
|MAC         |Media Access Control                     |L2 hardware address of a NIC                    |
|MITM        |Man-in-the-Middle                        |Network interception attack                     |
|MTU         |Maximum Transmission Unit                |Largest packet size in bytes (usually 1500)     |
|mTLS        |Mutual TLS                               |Both sides authenticate via certificates        |
|NACL        |Network Access Control List              |AWS: stateless subnet-level firewall            |
|NAT         |Network Address Translation              |Rewrites source/dest IP/port                    |
|NIC         |Network Interface Card                   |Physical or virtual network adapter             |
|NLB         |Network Load Balancer                    |AWS L4 load balancer                            |
|NSG         |Network Security Group                   |Azure: stateful L3/L4 packet filter             |
|NTP         |Network Time Protocol                    |Time synchronization                            |
|NVA         |Network Virtual Appliance                |Third-party firewall/router in the cloud        |
|OWASP       |Open Web Application Security Project    |Security standards and Top 10 list              |
|PAT         |Port Address Translation                 |Multiple private IPs share one public IP (ports)|
|PE          |Private Endpoint                         |Azure private IP for PaaS services              |
|PIM         |Privileged Identity Management           |Azure: JIT privileged access management         |
|PoP         |Point of Presence                        |CDN edge location                               |
|PSC         |Private Service Connect                  |GCP equivalent of Private Endpoints             |
|QUIC        |Quick UDP Internet Connections           |Google’s transport protocol (HTTP/3 base)       |
|RBAC        |Role-Based Access Control                |Permissions assigned to roles, not users        |
|RFC         |Request for Comments                     |Internet standards documents (e.g., RFC 1918)   |
|RDP         |Remote Desktop Protocol                  |Windows GUI remote access                       |
|S2S         |Site-to-Site                             |VPN type connecting two networks                |
|P2S         |Point-to-Site                            |VPN type for individual client device           |
|SAN         |Subject Alternative Name                 |TLS cert covering multiple domains              |
|SG          |Security Group                           |AWS: stateful firewall (instance-level)         |
|SLAAC       |Stateless Address Autoconfiguration      |IPv6 auto-configuration mechanism               |
|SNAT        |Source NAT                               |Rewrite source IP (outbound masquerade)         |
|SSH         |Secure Shell                             |Encrypted remote terminal                       |
|SSL         |Secure Sockets Layer                     |Deprecated — replaced by TLS                    |
|TCP         |Transmission Control Protocol            |Reliable, ordered transport protocol            |
|TGW         |Transit Gateway                          |AWS: hub for transitive VPC routing             |
|TLS         |Transport Layer Security                 |Encryption for in-transit data                  |
|TTL         |Time To Live                             |DNS: cache duration; IP: hop limit              |
|UDP         |User Datagram Protocol                   |Fast, connectionless transport protocol         |
|UDR         |User Defined Route                       |Azure custom routing rule                       |
|VGW         |Virtual Private Gateway                  |AWS: VPN/Direct Connect endpoint                |
|VLAN        |Virtual Local Area Network               |L2 network segmentation via tagging             |
|VNet        |Virtual Network                          |Azure: isolated private cloud network           |
|VPC         |Virtual Private Cloud                    |AWS/GCP: isolated private cloud network         |
|VPN         |Virtual Private Network                  |Encrypted tunnel over public network            |
|VPCE        |VPC Endpoint                             |AWS: PrivateLink interface or gateway endpoint  |
|WAF         |Web Application Firewall                 |L7 firewall protecting web apps (OWASP rules)   |
|XPN         |Cross-Project Network                    |GCP: Shared VPC across projects                 |

-----

*v2 — Expanded Edition | April 2026 | Applicable to Azure, AWS, GCP, and general networking concepts*
