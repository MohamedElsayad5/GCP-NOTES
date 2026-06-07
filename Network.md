<div align="center">

# 🌐 Google Cloud Interconnect - Complete Study Notes

**The ultimate visual guide for GCP private connectivity**

[[Cloud Interconnect](https://img.shields.io/badge/Google%20Cloud-Interconnect-4285F4?style=for-the-badge&logo=googlecloud&logoColor=white)](https://cloud.google.com/network-connectivity/docs/interconnect)
[[Networking](https://img.shields.io/badge/Networking-Hybrid%20Cloud-34A853?style=for-the-badge)](#)
[[Interview Ready](https://img.shields.io/badge/Status-Interview%20Ready-00C853?style=for-the-badge)](#)

[🔐 VPN vs Interconnect](#cloud-vpn-vs-cloud-interconnect) • [🏗️ Types](#types-of-interconnect) • [⚙️ Configuration](#complete-interconnect-workflow) • [🛡️ Security](#encryption)

</div>

---

## 📌 Overview

Google Cloud provides multiple connectivity options between:

- **On-Premises Networks**
- **Other Cloud Providers** - AWS, Azure, etc.
- **Google Cloud VPC Networks**

The two major solutions are:

1. **Cloud VPN** - Encrypted tunnels over public internet
2. **Cloud Interconnect** - Private physical connectivity

---

# 🔐 Cloud VPN vs Cloud Interconnect

## Cloud VPN

### What is Cloud VPN?
Cloud VPN creates **encrypted IPsec tunnels over the public Internet** between your network and Google Cloud.

### Characteristics
- ✅ **Quick deployment** - Hours to days
- ✅ **Low operational effort**
- ✅ **Cost-effective** - No physical circuits
- ✅ **Encrypted by default** - IPsec

### Bandwidth
- **1 Gbps to 3 Gbps per tunnel**
- Actual throughput depends on internet quality, distance, congestion

### Limitations
Because traffic traverses the Internet:
- ⚠️ **Higher latency**
- ⚠️ **Jitter**
- ⚠️ **Variable performance**
- ⚠️ **Less predictable throughput**

### Best Use Cases
- Small and medium workloads
- Fast connectivity setup
- Backup connectivity
- Temporary environments / POC

---

# 🌐 Cloud Interconnect

## What is Cloud Interconnect?
Cloud Interconnect provides a **private physical connection** between your network and Google Cloud. **Traffic does NOT traverse the public Internet.**

## Advantages

### ⚡ High Bandwidth
Supported port speeds:
- **10 Gbps**
- **100 Gbps**

Multiple circuits can be bonded for higher capacity.

### 🚀 Lower Latency
Traffic travels directly through **Google's global private backbone**.

Benefits:
- Lower latency
- Lower jitter  
- More stable, predictable performance

### 🏢 Enterprise Ready
Ideal for:
- Large enterprises
- Mission-critical applications
- Large data transfers - Petabytes
- Hybrid cloud architectures

---

# 🗺️ Google Global Backbone

Google operates a global private backbone network connecting all regions.

```text
Europe-West1 (Belgium)
      |
      | Google Private Backbone
      |
US-Central1 (Iowa)
```

Resources inside a VPC communicate through Google's backbone automatically. No public internet.

---

# 🏢 Physical Infrastructure Terms

## Colocation Facility
A neutral data center where:
- **Google equipment** exists
- **Cloud providers** have presence
- **ISPs** and **carriers** operate
- **Enterprises** may place equipment

Interconnect connections are established inside these facilities.

## Meet-Me Room (MMR)
Inside a colocation facility there is a room called **Meet-Me Room**.

**Purpose:** Physical cross-connections occur here

**Contains:**
- Patch panels
- Fiber termination points
- Cross-connect infrastructure

## Cross Connect
A **Cross Connect** is the physical fiber cable connecting:

```text
Customer Equipment
         |
         | Fiber Cross Connect
         |
Google Equipment
```

This creates private layer-1 connectivity.

## LOA-CFA
**Letter of Authorization and Connecting Facility Assignment**

When ordering Dedicated Interconnect, Google generates an LOA-CFA containing:
- Port information
- Facility information
- Patch panel information
- Technical connection details

Required by the colocation provider to complete the cross-connect.

---

# 🏗️ Types of Interconnect

There are three main types.

---

## 1. Dedicated Interconnect

### Definition
A **direct physical connection** between Customer Equipment and Google Equipment inside the **same colocation facility**.

### Characteristics
You receive:
- **Dedicated physical port** on Google's router
- **Dedicated physical circuit**

Supported port sizes:
- **10 Gbps**
- **100 Gbps**

### Advantages
- ✅ **Maximum performance**
- ✅ **Lowest latency**
- ✅ **Highest bandwidth**
- ✅ **Direct connectivity** - No intermediary

### Requirements
Must have:
- Equipment inside Google's colocation facility
- OR Partner hosting your equipment

---

## 2. Partner Interconnect

### Definition
Used when **customer equipment is NOT present** in a Google colocation facility.

A **service provider** already connected to Google acts as intermediary.

### Architecture
```text
Customer On-Prem
    |
Partner Network (Equinix, Megaport, etc.)
    |
Google Network
```

### Characteristics
- **No dedicated Google port** for customer
- **Flexible bandwidth options**

Typical bandwidth:
- 50 Mbps
- 100 Mbps, 200 Mbps, 300 Mbps, 400 Mbps, 500 Mbps
- 1 Gbps to 10 Gbps
- 50 Gbps

### Benefits
- ✅ **Easier deployment** - No colocation presence required
- ✅ **Lower complexity**
- ✅ **Pay for what you need**

---

## 3. Cross-Cloud Interconnect

### Definition
**Private connectivity between Google Cloud and other cloud providers** like AWS and Azure.

### Example
```text
Google Cloud VPC
      |
Cross Cloud Interconnect (Google Managed)
      |
AWS Direct Connect Location
      |
AWS VPC
```

**Key Point:** Google manages the connectivity to the other cloud. No internet traversal.

---

# 📍 Metro & Redundancy Concepts

## Metro Concept
Google groups nearby colocation facilities into **Metro Areas**.

**Examples:**
- **Amsterdam Metro** - ams-zone1, ams-zone2
- **Brussels Metro**
- **Ashburn Metro** - ash-zone1, ash-zone2, ash-zone3

A Metro may contain one or multiple facilities.

## Edge Availability Domain (EAD)
Within a Metro, Google divides infrastructure into **Edge Availability Domains**.

**Purpose:**
- **Fault isolation** - Maintenance in EAD1 doesn't affect EAD2
- **Redundancy** - Independent power, cooling, network

Think of them as independent groups of edge devices.

---

# 🎯 High Availability Design

## 99.9% SLA
Requires:
- **Two Interconnect connections**
- **Same Metro**
- **Different Edge Availability Domains**

```text
Metro: Ashburn
  ├── Connection 1 → EAD1
  └── Connection 2 → EAD2
```

## 99.99% SLA
Requires:
- **Four Interconnect connections**
- **Two Metro Areas**
- **Different EADs in each Metro**

```text
Metro A: Ashburn
  ├── EAD1 - Connection 1
  └── EAD2 - Connection 2

Metro B: Chicago  
  ├── EAD1 - Connection 3
  └── EAD2 - Connection 4
```

---

# ⚙️ Logical Configuration Layer

## Physical Connection is NOT Enough
After installing the fiber connection, **traffic still cannot flow**. Logical configuration is required.

---

## VLAN Attachment

### What is VLAN Attachment?
A **logical connection created over the physical Interconnect**. Equivalent to 802.1Q VLAN.

### Functions
Provides:
- **VLAN ID** - For 802.1Q tagging
- **BGP IP addresses** - /30 or /31 for peering

### Process
**Google Side:**
```text
1. Assigns VLAN ID - e.g. 1010
2. Assigns BGP IPs - 169.254.10.1/30 (Google), 169.254.10.2/30 (Customer)
```

**Customer Side:**
```text
1. Creates matching VLAN interface on router
2. Configures BGP with provided IPs
```

---

## Cloud Router

### Purpose
**Cloud Router is required for:**
- **Dynamic routing** via BGP
- **Route advertisement** between VPC and on-prem

### Functions
- Establishes BGP sessions with on-prem router
- **Learns on-prem routes** and installs in VPC
- **Advertises VPC subnet routes** to on-prem

**Note:** Cloud Router is a Google-managed service. No VM to manage.

---

## BGP Peering

After VLAN Attachment creation, Google provides:

```text
Google BGP IP: 169.254.10.1
Customer BGP IP: 169.254.10.2
ASN: Google - 16550, Customer - Your ASN
```

BGP session is established over the VLAN.

### Result
**Automatic route exchange:**

Google learns:
```text
On-Prem Networks - 10.1.0.0/16, 192.168.1.0/24
```

On-Prem learns:
```text
VPC Networks - 10.128.0.0/20, 10.132.0.0/20
```

---

# 🗺️ Dynamic Routes & Routing Modes

## Dynamic Routes
Routes learned through BGP become **Dynamic Routes** in your VPC routing table.

## Regional Dynamic Routing - Default
Routes are visible **only to workloads within the same region** as the Cloud Router.

```text
Cloud Router in europe-west1
    |
    ↓ Advertises routes to
VMs in europe-west1 ✅
VMs in us-central1 ❌ Cannot use routes
```

## Global Dynamic Routing - Optional
When enabled on VPC: **All regions can use learned routes**.

```text
Cloud Router in europe-west1
    |
    ↓ Advertises routes to
VMs in europe-west1 ✅
VMs in us-central1 ✅
VMs in asia-east1 ✅
```

## Subnet Routes vs Dynamic Routes

| Type | Scope | Config Required |
| :--- | :--- | :--- |
| **Subnet Routes** | **Automatically global** | No config needed |
| **Dynamic Routes** | **Regional by default** | Enable Global Dynamic Routing for cross-region |

---

# 🔀 Multiple VLAN Attachments

One physical Interconnect can contain **multiple VLAN Attachments**.

**Benefits:**
- **Traffic separation** - Prod vs Dev
- **Multi-tenancy** - Different BUs
- **Capacity distribution**

Each VLAN has:
- Independent VLAN ID
- Independent BGP session
- Independent routing

---

# 🔧 Partner Interconnect Models

## Partner Interconnect Layer 2
**Partner only transports VLAN**. Customer performs BGP directly with Google.

```text
Customer Router ←--BGP--→ Google Cloud Router
      |                         |
   Partner L2 Transport         |
      |-------------------------|
```

## Partner Interconnect Layer 3
**Partner handles BGP with Google**. Customer peers with Partner.

```text
Customer Router ←--BGP--→ Partner Router ←--BGP--→ Google Router
```

Partner redistributes routes between customer and Google.

---

# 🛡️ Encryption

## Default Behavior
**Interconnect traffic is NOT encrypted by default.**

- **Layer 2:** Not encrypted
- **Layer 3:** Not encrypted

Traffic is private but not encrypted on the wire.

---

## MACsec Encryption

### What is MACsec?
**Media Access Control Security** - IEEE 802.1AE

Layer 2 encryption protocol.

### Purpose
Encrypts traffic between:
```text
Customer Device ←--MACsec--→ Google Edge Device
```

### Important Limitation
**Not End-to-End Encryption.** Only protects the **local Interconnect segment** from customer to Google edge. Traffic inside Google backbone is unencrypted.

**Availability:** Only on **10G and 100G Dedicated Interconnect**.

---

## VPN over Interconnect

### Why?
To achieve **End-to-End Encryption** while keeping:
- ✅ Private connectivity
- ✅ High bandwidth
- ✅ Low latency

### Architecture
```text
On-Prem
    |
HA VPN Tunnel - IPsec
    |
Interconnect VLAN Attachment
    |
Google Cloud VPC
```

### How it Works
1. Deploy **HA VPN gateways** on both sides
2. VPN tunnels run **over VLAN Attachments** instead of public Internet
3. Traffic is encrypted before entering Interconnect

### Benefits
- ✅ **End-to-End Encryption** - IPsec
- ✅ **Private transport** - No internet
- ✅ **Enterprise-grade security**
- ✅ **High performance** - Uses Interconnect bandwidth

**This is the most secure Interconnect architecture.**

---

# 📋 Complete Interconnect Workflow

| Step | Action | Details |
| :--- | :--- | :--- |
| **1** | **Order Interconnect** | Choose Dedicated or Partner Interconnect |
| **2** | **Physical Setup** | Cross-connect installed in colocation facility |
| **3** | **Create VLAN Attachment** | Specify Interconnect, VLAN ID, Cloud Router |
| **4** | **Receive Config** | Google provides VLAN ID + BGP IPs |
| **5** | **Configure On-Prem** | Create VLAN interface on customer router |
| **6** | **Deploy Cloud Router** | In same region as VLAN Attachment |
| **7** | **Establish BGP** | BGP session comes up, routes exchanged |
| **8** | **Verify Routes** | Check VPC routes + on-prem routing table |
| **9** | **Optional: Global Routing** | Enable for cross-region access |
| **10** | **Optional: Security** | Enable MACsec or VPN over Interconnect |

---

# 🎯 Interview Summary

## Cloud VPN
- **Internet based** - IPsec encrypted
- **Fast deployment** - Hours
- **Lower bandwidth** - 1-3 Gbps per tunnel
- **Higher latency** - Variable performance
- **Use case:** Backup, small workloads, POC

---

## Cloud Interconnect
- **Private connectivity** - No public internet
- **High bandwidth** - 10G/100G
- **Enterprise solution** - Mission critical
- **Lower latency** - Predictable performance

---

## Dedicated Interconnect
- **Direct physical Google port**
- **10G / 100G** dedicated
- **Highest performance**
- **Requires colocation presence**

---

## Partner Interconnect
- **Through service provider** - Equinix, Megaport
- **Flexible bandwidth** - 50Mbps to 50Gbps
- **Easier deployment** - No colocation needed

---

## Cross Cloud Interconnect
- **Connects GCP with AWS/Azure** privately
- **Google managed** connectivity

---

## VLAN Attachment
**Logical connection over physical circuit.** Provides VLAN ID + BGP IPs. Required for traffic flow.

---

## Cloud Router
**Managed BGP router.** Provides dynamic routing, learns on-prem routes, advertises VPC routes.

---

## BGP
**Border Gateway Protocol.** Used for dynamic route exchange between on-prem and GCP.

---

## Global Dynamic Routing
Makes **learned routes available across all VPC regions**. Disabled by default.

---

## MACsec
**Layer 2 encryption** between customer and Google edge. **Not end-to-end.** Only on 10G/100G Dedicated.

---

## VPN over Interconnect
**IPsec VPN tunnels over Interconnect VLANs.** Provides **End-to-End Encryption + Private Connectivity + High Performance**. Most secure architecture.

---

<div align="center">

**⭐ Save this for your GCP Network Engineer interview!**

Made with ❤️ for Cloud Networking

[⬆️ Back to Top](#-google-cloud-interconnect---complete-study-notes)

</div>