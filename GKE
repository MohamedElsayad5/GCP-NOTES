<div align="center">

# 🚀 Google Kubernetes Engine (GKE) - Ultimate Revision Guide

**A concise, visual, interview-focused cheat sheet for GKE**

[[GKE](https://img.shields.io/badge/Google%20Kubernetes%20Engine-GKE-4285F4?style=for-the-badge&logo=googlecloud&logoColor=white)](https://cloud.google.com/kubernetes-engine)
[[Kubernetes](https://img.shields.io/badge/Kubernetes-v1.28+-326CE5?style=for-the-badge&logo=kubernetes&logoColor=white)](https://kubernetes.io)
[[Interview Ready](https://img.shields.io/badge/Status-Interview%20Ready-00C853?style=for-the-badge)](#)

[🎯 Quick Start](#-1-what-is-gke) • [🏗️ Architecture](#️-2-gke-architecture) • [⚡ Cheat Sheet](#-4-workloads-cheat-sheet) • [🛡️ Security](#️-12-security-deep-dive)

</div>

---

## 📌 1. What is GKE?

> **Google Kubernetes Engine = Managed Kubernetes + Google Cloud Superpowers**

| 🛠️ **Google Manages** | 💻 **You Focus On** |
| :--- | :--- |
| Control Plane, Upgrades, Patching | Deploying Apps, Writing Code |
| Infrastructure Provisioning, Auto-healing | Workload Configs, Services |
| Monitoring & Logging Integration | Business Logic & Features |

**Bottom Line:** You ship code. Google handles the operational pain.

---

## 🏗️ 2. GKE Architecture

```mermaid
graph TB
    subgraph "🧠 Control Plane - Google Managed"
        API[API Server]
        ETCD
        SCHED[Scheduler]
        CM[Controller Manager]
    end

    subgraph "💻 Node Pool 1"
        N1[Node 1]
        P1[📦 Pod A]
        P2[📦 Pod B]
        N1 --> P1
        N1 --> P2
    end

    subgraph "💻 Node Pool 2"
        N2[Node 2]
        P3[📦 Pod C]
        P4[📦 Pod D]
        N2 --> P3
        N2 --> P4
    end

    API --> N1
    API --> N2

    style API fill:#4285F4,stroke:#333,stroke-width:2px,color:#fff
    style N1 fill:#34A853,stroke:#333,stroke-width:2px,color:#fff
    style N2 fill:#34A853,stroke:#333,stroke-width:2px,color:#fff
```

| Component | Responsibility |
| :--- | :--- |
| **🧠 Control Plane** | Brain of the cluster. API entry point, scheduling, state management |
| **💻 Worker Nodes** | GCE VMs running kubelet, kube-proxy, and your containerized **Pods** |

---

## 🏎️ 3. GKE Operational Modes

| Feature | ✈️ **Autopilot Mode** | ⚙️ **Standard Mode** |
| :--- | :--- | :--- |
| **Management** | **Fully managed by Google** | **Shared** - You manage nodes |
| **Node Provisioning** | Google provisions & scales automatically | You manually configure Node Pools |
| **Pricing Model** | 💰 **Pay per Pod** - vCPU, Memory, Storage | 💰 **Pay per Node** - Compute Engine VMs |
| **SLA** | 99.9% for Pods | 99.95% for Control Plane only |
| **Best For** | 🔥 **Recommended** - Hands-off, fast deployments | Full customization, custom kernels, GPUs |

> **💡 Pro Tip:** Choose Autopilot unless you need custom OS, kernel modules, or specific GPU drivers.

---

## 📦 4. Workloads Cheat Sheet

| Workload | Use Case | Example | Key Feature |
| :--- | :--- | :--- | :--- |
| **Pod** | Smallest deployable unit | 1+ containers | Ephemeral by design |
| **Deployment** | 🌐 **Stateless** apps | Web API, Frontend | Rolling updates, replicas |
| **StatefulSet** | 💾 **Stateful** apps | Databases, Kafka | Stable identity, ordered deploy |
| **DaemonSet** | Run on every node | Log collectors, Monitoring agents | One Pod per Node |
| **Job** | Run to completion | Batch processing, DB migration | Runs once |
| **CronJob** | Scheduled tasks | Backups at 2 AM daily | Cron syntax scheduling |

---

## 🌍 5. Cluster Topologies

### 📍 Zonal Clusters
* **Single-Zone:** Control plane + nodes in **one zone**. Cheapest, but ❌ **Single Point of Failure**
* **Multi-Zonal:** Control plane in one zone, nodes spread across **multiple zones** in a region

### 🌍 Regional Clusters - 🔥 Production Best Practice
* Replicates **Control Plane AND Nodes across 3 zones** within a region
* ✅ Zero-downtime control plane upgrades
* ✅ Survives full zone outage
* **Always use Regional for production**

---

## 🛣️ 6. Release Channels

| Channel | Speed | Stability | Best For |
| :--- | :--- | :--- | :--- |
| **🚀 Rapid** | Fastest | 🟡 Lower | Dev/Staging - Latest features |
| **⚖️ Regular** | Balanced | 🟢 **Default** | Most production workloads |
| **🔒 Stable** | Slowest | 🟢🟢 Highest | Critical, compliance-heavy systems |

---

## 👥 7. Node Pools

A **Node Pool** is a group of nodes in a cluster that share the same configuration.

```yaml
Examples:
Pool-A: n2-standard-4 → General purpose workloads
Pool-B: n2-highmem-16 → Memory-intensive - Databases
Pool-C: g2-standard-8 → GPU-enabled - AI/ML training
Pool-D: e2-micro → Spot/Preemptible - Batch jobs 💸
```

**Why use them:** Run different workloads on optimized hardware + cost control + taints/tolerations.

---

## 🔌 8. Networking & CNI

### 🕸️ VPC-Native Clusters - Best Practice
Pods get IPs **directly from your Google Cloud VPC** secondary ranges.
* ✅ Better performance - No NAT overhead
* ✅ Native GCP firewall rules work on Pods
* ✅ Direct access to Cloud SQL, Memorystore, etc.

### 🔌 CNI vs kube-proxy

| | **CNI - Container Network Interface** | **kube-proxy** |
| :--- | :--- | :--- |
| **Role** | Pod IP assignment, Pod-to-Pod networking | Service routing, load-balancing ClusterIP |
| **Examples** | **Calico** - Rich Network Policies 🔥<br>**Cilium** - eBPF performance 🚀<br>**Flannel** - Simple, lightweight | iptables or IPVS mode |

---

## 🚦 9. Exposing Workloads

```text
🌐 Internet
    ↓
🌐 Ingress - HTTP/S LB - Layer 7 Routing - Path/Domain based
    ↓
⚙️ Service - ClusterIP / NodePort / LoadBalancer
    ↓
📦 Pods - Your application containers
```

| Service Type | Scope | Use Case |
| :--- | :--- | :--- |
| **ClusterIP** | Internal only 🔒 | Default. Backend services |
| **NodePort** | `<NodeIP>:30000-32767` | Dev/Test only |
| **LoadBalancer** | External IP via GCP NLB 🌐 | Single service - Expensive |
| **Ingress** | Single HTTP/S LB + routing | 🔥 **Best** - Multiple services via `/api`, `/app` |

---

## 📈 10. The Autoscaling Trio

| Autoscaler | Scales What | Trigger |
| :--- | :--- | :--- |
| **📈 HPA** | **Pods** | CPU > 70%, Memory, Custom metrics |
| **📉 VPA** | **Pod Resources** | Adjusts CPU/Memory requests/limits |
| **🖥️ Cluster Autoscaler** | **Nodes** | Adds GCE VMs when Pods are pending |

> **⚠️ Important:** Do not run HPA + VPA on the same metric. They will conflict.

---

## 💾 11. Storage Architecture

```text
1. StorageClass → Blueprint - SSD vs HDD, reclaim policy
2. PVC → Your request - "Give me 100Gi"
3. PV → Actual disk provisioned by GKE
```

**Common GKE StorageClasses:**
- `premium-rwo` - SSD Persistent Disk - Databases 🔥
- `standard-rwo` - Standard HDD - Logs, backups
- `standard-rwx` - Filestore - ReadWriteMany shared storage

---

## 🛡️ 12. Security Deep Dive

### 🔑 Workload Identity - 🔥 Must-Know for Interviews
Safest way for Pods to access GCP services like Cloud Storage, BigQuery.

* ❌ **Old/Bad Way:** Download JSON service account key → Mount as Secret → Risk of leak
* ✅ **GKE Way:** Bind Kubernetes Service Account to IAM Google Service Account

```bash
# Enable Workload Identity
kubectl annotate serviceaccount KSA_NAME \
  --namespace NAMESPACE \
  iam.gke.io/gcp-service-account=GSA_NAME@PROJECT_ID.iam.gserviceaccount.com
```
**Result:** Pod authenticates to GCP APIs with no keys in your cluster.

### 🧱 Network Policies
Built-in Pod-to-Pod firewall.

```yaml
Example: Allow frontend pods to talk to backend, but block frontend → database
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
```

### 🚪 Private Clusters & Authorized Networks

| Feature | Benefit |
| :--- | :--- |
| **Private Nodes** | Nodes have no public IPs - Not exposed to internet |
| **Authorized Networks** | Restrict `kubectl` access to specific CIDRs - VPN/Bastion only |
| **Master Authorized** | Control Plane endpoint not public |

---

## 🔄 13. Surge Upgrades

How GKE upgrades Node Pools with zero downtime:
1. 🟢 **maxSurge:** Creates new upgraded nodes first
2. 🔀 **Drain:** Safely evicts Pods from old nodes to new ones
3. 🔴 **maxUnavailable:** Terminates old nodes

**Result:** Rolling upgrade with no service interruption 💪

---

## 🎯 14. Ultimate Production Checklist

```markdown
[ ] ✅ Regional Cluster - HA across 3 zones
[ ] ✅ Private Cluster + Authorized Networks - Network security
[ ] ✅ Workload Identity - No JSON key secrets
[ ] ✅ VPC-Native + Network Policies - Pod-level firewall
[ ] ✅ HPA + Cluster Autoscaler - Auto scale workloads & nodes
[ ] ✅ Regular Release Channel - Stable feature velocity
[ ] ✅ Cloud Logging/Monitoring - Full observability
[ ] ✅ Autopilot Mode - Unless you need Standard for custom needs
```

---

### 💡 Interview Tip Summary

> **When asked "Why GKE?" say:**
>
> "GKE eliminates the pain of managing the control plane, HA, and upgrades. I get 100% upstream Kubernetes, but integrated with GCP. That means IAM instead of secrets via Workload Identity, Global HTTP/S Load Balancer instead of nginx-ingress, VPC-native networking instead of overlay networks, and integrated Cloud Logging. With Autopilot, I pay per Pod and Google handles all node management, so I focus on code not infra."

---

<div align="center">

**⭐ If this repo helped you, give it a Star!**

Made with ❤️ for GKE interviews

[⬆️ Back to Top](#-google-kubernetes-engine-gke---ultimate-revision-guide)

</div>