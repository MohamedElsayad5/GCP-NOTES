<div align="center">

# 🖥️ Google Cloud Compute Engine – Ultimate Revision Notes

**A concise, visual, interview-focused cheat sheet for GCE (IaaS)**

[[Compute Engine](https://img.shields.io/badge/Google%20Cloud-Compute%20Engine-4285F4?style=for-the-badge&logo=googlecloud&logoColor=white)](https://cloud.google.com/compute)
[[IaaS](https://img.shields.io/badge/Service-IaaS-34A853?style=for-the-badge)](#)
[[Interview Ready](https://img.shields.io/badge/Status-Interview%20Ready-00C853?style=for-the-badge)](#)

[📌 Shared Responsibility](#-1-shared-responsibility-model) • [⚙️ Machine Types](#️-2-machine-families-cheat-sheet) • [💾 Storage](#-3-storage-options-comparison) • [📈 Scaling](#-5-scaling--high-availability-migs)

</div>

---

## 📌 1. Shared Responsibility Model

Compute Engine is **Infrastructure as a Service (IaaS)**. Here is who owns what:

| 💻 **You Are Responsible For (Software)** | 🧱 **Google Is Responsible For (Hardware)** |
| :--- | :--- |
| 💿 Operating System & Patches | 🏢 Physical Data Centers & Security |
| 🛠️ Software Installation & Runtimes | 🔌 Power, Cooling, & Networking |
| 🔒 Application Security & IAM | 🧰 Physical Server Hardware & Disks |
| ⚙️ Network Firewall Configurations | 🔄 Hardware Lifecycle & Maintenance |

**Bottom Line:** Google owns the metal. You own everything above the hypervisor.

---

## ⚙️ 2. Machine Families Cheat Sheet

### 🔹 General Purpose - Balanced CPU/RAM
* **Types:** **E2** - Cost-optimized, shared-core | **N2 / N2D** - Balanced performance | **N1** - Legacy
* 🎯 **Use Cases:** Web applications, microservices, medium databases, dev/test

### 🔹 Compute Optimized - High CPU
* **Types:** **C2 / C3** - Highest performance per core, high clock speed
* 🎯 **Use Cases:** High-Performance Computing (HPC), gaming servers, media encoding, scientific modeling

### 🔹 Memory Optimized - High RAM
* **Types:** **M1 / M2 / M3** - Up to 12TB RAM
* 🎯 **Use Cases:** Large In-memory databases like SAP HANA, real-time analytics, large caches

### 🔹 Accelerator Optimized - GPU Powered
* **Types:** **A2 / A3 / G2** - NVIDIA GPUs attached
* 🎯 **Use Cases:** AI/ML training, Deep Learning, heavy video rendering, 3D visualization

> **💡 Interview Tip:** Know E2 vs N2. E2 is cheaper but burstable. N2 is consistent performance.

---

## 💾 3. Storage Options Comparison

| Feature | 💿 **Persistent Disk (PD)** | ⚡ **Local SSD** | 🚀 **Hyperdisk** |
| :--- | :--- | :--- | :--- |
| **Storage Type** | Network-attached Block Storage | Physically attached to host | Next-gen scalable block storage |
| **Data Persistence** | ✅ Survives VM stop/start & deletion | ❌ **Lost if VM stops/preempted** | ✅ Survives VM life cycle |
| **Performance** | Reliable, scales with disk size | 🔥 **Ultra-low latency**, insane IOPS | 🚀 **Dynamically scalable** IOPS/Throughput |
| **Max Size** | 64 TB per disk | 9 TB per VM | 64 TB per disk |
| **Best For** | Boot disks, standard databases | Caching, scratch space, swap | Enterprise apps with extreme performance needs |

**Key Rules:**
1. **PD-Standard:** HDD - Cheap, backups
2. **PD-Balanced:** SSD - Default for most workloads
3. **PD-SSD:** SSD - High performance DBs
4. **Local SSD:** Ephemeral, fastest, use for temp data only
5. **Hyperdisk Extreme/Balanced:** Decouple size from performance

---

## 🌐 4. Networking & Access Control

### 🌐 IP Addresses & Firewalls
* **Internal IP:** Assigned to every VM automatically. Used for free private communication within same VPC.
* **External IP:** Optional. Used to face public internet. **Costs money if VM is stopped but IP reserved**.
* **Firewall Rules:** VPC-level rules defined by **Network Tags** or **Service Accounts** to allow/deny traffic. Default deny ingress.

### 🔐 Safe Access Methods
1. **SSH / RDP:** Standard access via external IP or IAP tunnel.
2. **OS Login:** 🔥 Links your IAM user account directly to Linux SSH profiles. **No more managing SSH keys manually!** Centralized access control.
3. **IAP (Identity-Aware Proxy):** 🔥 **Production Best Practice.** Allows you to SSH into a VM **without a public IP address** via Cloud Console. Traffic tunneled through Google.

```bash
# SSH via IAP - No public IP needed
gcloud compute ssh VM_NAME --tunnel-through-iap --zone=ZONE
```

---

## 📈 5. Scaling & High Availability (MIGs)

Instead of managing single VMs, production uses **Managed Instance Groups (MIGs)**.

```text
[ Cloud Load Balancer - Global HTTP/S LB ]
          │
          ├──▶ [ Regional Managed Instance Group (MIG) ]
          │          ├── VM-1 (Zone A) ── 🟢 Healthy
          │          ├── VM-2 (Zone B) ── 🟢 Healthy
          │          └── VM-3 (Zone C) ── 🔄 Auto-healing (Replacing broken VM)
```

### 🧠 Instance Templates
A **reusable blueprint** specifying:
- Machine Type
- Boot Image or Custom Image
- Network, Subnet, Firewall tags
- **Startup Scripts** - Automation commands executed on boot
- Service Account

**Important:** Instance Templates are **immutable**. To change, create new version and do rolling update.

### 📈 Managed Instance Groups (MIGs) Features
* 🔄 **Auto-healing:** Uses health checks to recreate failed/unresponsive VMs automatically.
* 📈 **Autoscaling:** Dynamically grows/shrinks VM count based on:
    - CPU utilization
    - Load Balancing serving capacity
    - Pub/Sub queue size
    - Custom Cloud Monitoring metrics
* 🌍 **Regional Distribution:** Automatically spreads VMs across multiple zones to survive single-zone outages.
* 🔄 **Rolling Updates:** Update VMs gradually without downtime.

---

## ⚡ 6. Spot (Preemptible) VMs vs Standard VMs

* **What are they?** Excess GCP compute capacity offered at **huge discount - 60-91% off**.
* **The Catch:** Google can terminate (preempt) them **at any time with 30-second warning notice**.
* **Max Runtime:** 24 hours for preemptible. Spot VMs have no max runtime but same preemption risk.

| Feature | **Standard VM** | **Spot/Preemptible VM** |
| :--- | :--- | :--- |
| **Pricing** | Full price | **60-91% discount** |
| **Availability** | Guaranteed | Can be preempted anytime |
| **Use Case** | Production workloads | Fault-tolerant workloads |

❌ **DO NOT USE FOR:** Stateful databases, core web APIs, user-facing applications, anything that can't handle interruption.

✅ **PERFECT FOR:** Stateless batch processing, CI/CD runners, distributed big-data jobs like Hadoop/Spark, rendering farms, fault-tolerant workloads.

---

## 📸 7. Snapshots vs Custom Images

| Feature | **📸 Snapshot** | **🧩 Custom Image** |
| :--- | :--- | :--- |
| **What is it** | Point-in-time **incremental backup** of a Persistent Disk | Curated **golden boot disk** with OS + software |
| **Source** | From Persistent Disk | From Disk, Snapshot, or another Image |
| **Use Case** | Backup, disaster recovery, quick rollbacks | Launching new VMs/MIGs with pre-installed software |
| **Storage** | Incremental - Only changed blocks | Full image |
| **Best Practice** | Daily backups | Create hardened base image for templates |

**Workflow:** Create Custom Image → Use in Instance Template → Deploy MIG

---

## 🔄 8. VM Lifecycle States

```text
[ PROVISIONING ] ──▶ [ STAGING ] ──▶ [ RUNNING ] ──▶ [ STOPPING ] ──▶ [ TERMINATED ]
                                                           │                    │
                                                           └─[ SUSPENDING ]─────┘
```

> ⚠️ **Cost Note:** You do **NOT** pay for vCPU/Memory when VM is `TERMINATED` or `STOPPED`. 
> 
> But you **CONTINUE to pay for:**
> 1. Attached Persistent Disks
> 2. Reserved External IPs
> 3. Custom Images & Snapshots

---

## 🔐 9. IAM & Service Accounts

### How do you give a VM permissions to call Cloud Storage?
**✅ Correct:** Attach a **Service Account** to the VM with required IAM roles like `Storage Object Viewer`.

**❌ Wrong:** Embed service account JSON keys in VM. This is a security risk.

```bash
# Create VM with service account
gcloud compute instances create my-vm \
  --service-account=SA_NAME@PROJECT.iam.gserviceaccount.com \
  --scopes=https://www.googleapis.com/auth/cloud-platform
```

**Best Practice:** Use **least privilege**. Don't give `Editor` to every VM.

---

## 🛠️ 10. Metadata & Startup Scripts

### Metadata
Key-value pairs attached to VM or project. Used for configuration.

### Startup Scripts
**How to pass metadata or execute boot tasks?** Use **Startup Scripts**.

```bash
# In Instance Template
metadata:
  startup-script: |
    #!/bin/bash
    apt-get update
    apt-get install -y nginx
    systemctl start nginx
```

Runs on first boot. Use for bootstrapping, installing software, joining clusters.

---

## 🎯 11. Top Interview Keywords & Quick Answers

| **Question** | **Answer** |
| :--- | :--- |
| **Q: How do you pass metadata or execute boot tasks?** | **A:** Using **Startup Scripts** in metadata |
| **Q: How do you give a VM permissions to call GCP APIs?** | **A:** Attach a **Service Account** to VM with required IAM roles. Never use access keys |
| **Q: What is the fastest disk option available?** | **A:** **Local SSD** - But remember, it is ephemeral/temporary storage |
| **Q: How do you design a highly available app on GCE?** | **A:** **Regional MIG** behind **Global HTTP/S Load Balancer** with active health checks |
| **Q: Difference between Preemptible and Spot VMs?** | **A:** Preemptible = max 24h life. Spot = no max life but same preemption risk. Spot is newer |
| **Q: How to SSH without public IP?** | **A:** **IAP TCP Tunneling** - `gcloud compute ssh --tunnel-through-iap` |
| **Q: E2 vs N2?** | **A:** E2 = cost-optimized, burstable. N2 = consistent performance, no burst |
| **Q: How to reduce VM costs?** | **A:** Committed Use Discounts, Sustained Use Discounts, Spot VMs, Rightsizing, Stop idle VMs |

---

## 💰 12. Cost Optimization

1. **Committed Use Discounts (CUDs):** 1-year or 3-year commitment for 57% discount
2. **Sustained Use Discounts (SUDs):** Automatic discount for running >25% of month
3. **Spot VMs:** 60-91% off for fault-tolerant workloads
4. **Rightsizing:** Use Recommender to pick correct machine type
5. **Stop Idle VMs:** Automate shutdown of dev/test VMs at night
6. **Delete Unattached Disks:** Disks cost money even when VM deleted

---

<div align="center">

**⭐ Save this for your GCP Associate Cloud Engineer interview!**

Made with ❤️ for GCE interviews

[⬆️ Back to Top](#️-google-cloud-compute-engine--ultimate-revision-notes)

</div>