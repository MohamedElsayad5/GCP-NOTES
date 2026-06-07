<div align="center">

# ☁️ Google Cloud IAM & Service Accounts Cheat Sheet

**A practical, production-ready guide for GCP Identity & Access Management**

[[IAM](https://img.shields.io/badge/Google%20Cloud-IAM-4285F4?style=for-the-badge&logo=googlecloud&logoColor=white)](https://cloud.google.com/iam)
[[Security](https://img.shields.io/badge/Security-Zero%20Trust-34A853?style=for-the-badge)](#)
[[Interview Ready](https://img.shields.io/badge/Status-Interview%20Ready-00C853?style=for-the-badge)](#)

[📖 Core Concepts](#1-what-is-iam-the-core-paradigm) • [👥 Principals](#2-principals--identities) • [🔐 Service Accounts](#5-service-accounts-deep-dive) • [🚫 Keyless](#7-keyless-modern-solutions-adc--impersonation)

</div>

---

## 📖 Table of Contents

1. [What is IAM? (The Core Paradigm)](#1-what-is-iam-the-core-paradigm)
2. [Principals & Identities](#2-principals--identities)
3. [Permissions vs. Roles](#3-permissions-vs-roles)
4. [Resource Hierarchy & Policy Inheritance](#4-resource-hierarchy--policy-inheritance)
5. [Service Accounts Deep Dive](#5-service-accounts-deep-dive)
6. [The Danger of Service Account Keys](#6-the-danger-of-service-account-keys)
7. [Keyless Modern Solutions (ADC & Impersonation)](#7-keyless-modern-solutions-adc--impersonation)
8. [The Ultimate gcloud IAM Cheat Sheet](#8-the-ultimate-gcloud-iam-cheat-sheet)
9. [Interview Q&A Flashcards](#9-interview-qa-flashcards)
10. [IAM Best Practices & Golden Rule](#10-iam-best-practices--golden-rule)

---

## 1. What is IAM? (The Core Paradigm)

**Identity and Access Management (IAM)** defines security by regulating **WHO** can do **WHAT** on **WHICH** specific resource.

```text
┌───────────┐ ┌───────────────┐ ┌──────────────┐
│ WHO? │ ───► │ CAN DO WHAT? │ ───► │ WHICH RES.? │
│(Principal)│ │ (Role) │ │ (Resource) │
└───────────┘ └───────────────┘ └──────────────┘
```

**Example:** `john@company.com` → `Storage Object Viewer` → `gs://my-bucket`

---

## 2. Principals & Identities

A **Principal** (formerly "member") is an identity that can be granted access to cloud assets.

| Principal Type | Syntax / Identity Example | Primary Use Case |
| :--- | :--- | :--- |
| **Google Account** | `john@company.com` | Human developers, admins, users |
| **Google Group** | `devops-team@company.com` | Grouping users to apply permissions at scale |
| **Workspace Domain** | `@company.com` | All accounts inside enterprise domain |
| **Service Account** | `app-runner@proj.iam.gserviceaccount.com` | Apps, scripts, compute workloads, CI/CD |
| **Special Identities** | `allAuthenticatedUsers` / `allUsers` | Anyone with Google ID / Total public access |

> **💡 Best Practice:** Always prefer Groups over individual users for easier management.

---

## 3. Permissions vs. Roles

* **Permission:** A granular authority matching a precise REST API action.
  Format: `<service>.<resource>.<action>` - e.g., `compute.instances.start`
* **Role:** A named bundle containing hundreds/thousands of permissions. You bind roles, not permissions.

### Role Classifications

| Type | Description | Production Use |
| :--- | :--- | :--- |
| **1. Basic Roles (Legacy)** | `Owner`, `Editor`, `Viewer` | ❌ **Avoid** - Highly permissive, massive blast radius |
| **2. Predefined Roles** | Google-maintained - `roles/storage.objectViewer` | ✅ **Recommended** for standard architectures |
| **3. Custom Roles** | User-defined permission sets | ✅ For specific org workflows when predefined isn't enough |

**Rule:** Start with predefined. Only create custom if you need specific combo.

---

## 4. Resource Hierarchy & Policy Inheritance

Google Cloud resources are governed by a strict logical hierarchy. **Policies applied at higher node inherit downwards** to all children.

```text
[Organization] ──► [Folder] ──► [Project] ──► [Resource: VM/Bucket/DB]
     Policy Policy Policy Policy
```

### Effective Policy Rule
An identity's **Effective Policy** on any resource = **Union** of all permissions inherited from parents + policies applied directly to that resource.

**Example:**
```
Org: Viewer
  └── Project: Editor on storage.googleapis.com
        └── Bucket-A: Storage Admin
Result on Bucket-A: Viewer + Editor + Storage Admin = Storage Admin wins
```

> ⚠️ **Important:** Permissions are **additive only**. Higher-level permissions **cannot be restricted** lower down. If Org gives Editor, you can't remove it at Project level.

---

## 5. Service Accounts Deep Dive

A **Service Account (SA)** represents a **non-human machine identity**. Uses cryptographic credentials, not passwords.

### Type Breakdown

| Type | Description | Use Case |
| :--- | :--- | :--- |
| **User-Managed SAs** | Explicitly created by engineers | ✅ **Recommended** - Compartmentalize apps |
| **Default SAs** | Auto-created when API enabled | ❌ **Not recommended** - Broad legacy Editor access |
| **Service Agents** | Google-owned system accounts | Background infra - e.g., GKE Service Agent |

**Default SAs Examples:**
- Compute Engine default SA: `PROJECT_NUMBER-compute@developer.gserviceaccount.com`
- App Engine default SA: `PROJECT_ID@appspot.gserviceaccount.com`

**Problem:** Default SAs often have `Editor` role. If VM compromised, attacker gets Editor on entire project.

---

## 6. The Danger of Service Account Keys

A **Service Account Key** is a long-lived, portable JSON file with private key.

### 🚫 Why Keys Are a Security Risk

| Risk | Explanation |
| :--- | :--- |
| **No Authentication Boundary** | Anyone with key file has instant access. No SSO, no MFA challenge |
| **Leak Vectors** | Pushed to GitHub, exposed in logs, shared via Slack/email |
| **Tracking Overhead** | Hard to audit who has key, hard to revoke without breaking apps |
| **No Auto-Rotation** | Keys don't expire. Must manually rotate |
| **Portable** | Can be copied anywhere, used from any machine |

**Real Incident:** Thousands of keys leaked on GitHub annually. Attackers scan repos automatically.

---

## 7. Keyless Modern Solutions (ADC & Impersonation)

### A. Application Default Credentials (ADC)
**Don't hardcode key paths.** Let client libraries auto-discover credentials.

```python
# ❌ BAD - Hardcoded key
from google.cloud import storage
client = storage.Client.from_service_account_json('key.json')

# ✅ GOOD - ADC auto-discovers
from google.cloud import storage
client = storage.Client() # Finds creds automatically
```

**ADC Search Order:**
1. `GOOGLE_APPLICATION_CREDENTIALS` env var
2. `gcloud auth application-default login` creds
3. Attached SA on GCE/Cloud Run/GKE - Metadata Server
4. Fallback to anonymous

### B. Workload/Service Account Attachment
**Best Practice:** Directly bind SA to runtime - GCE VM, Cloud Run, GKE Pod.

Platform injects temporary creds into **Metadata Server**: `http://169.254.169.254`

```bash
# Create VM with SA
gcloud compute instances create my-vm \
  --service-account=app-sa@PROJECT.iam.gserviceaccount.com \
  --scopes=cloud-platform
```

**Result:** No keys on disk. Creds auto-rotated by Google every hour.

### C. Service Account Impersonation
Assume identity of privileged SA for short periods **without downloading keys**.

```text
┌───────────┐ gcloud request ┌────────────┐ Short-lived Token ~1hr ┌─────────────────┐
│ Developer │ ──────────────► │ Google IAM │ ──────────────────────► │ Service Account │
└───────────┘ └────────────┘ └─────────────────┘
```

**Required Role:** `roles/iam.serviceAccountTokenCreator` on target SA

```bash
# Impersonate SA for commands
gcloud auth application-default login \
  --impersonate-service-account=prod-sa@PROJECT.iam.gserviceaccount.com

# Now all gcloud/storage commands run as prod-sa
gcloud storage ls gs://prod-bucket
```

---

## 8. The Ultimate gcloud IAM Cheat Sheet

### Context & Configuration
```bash
gcloud auth login # Authenticate human user
gcloud config set project PROJECT_ID # Set active project
gcloud config list # Show current config
```

### Keyless Operations & Testing
```bash
gcloud auth application-default login # Generate local ADC creds
gcloud auth application-default print-access-token # Get token for testing
```

### Service Account Impersonation
```bash
# Run single command as SA
gcloud storage ls gs://bucket \
  --impersonate-service-account=sa@PROJECT.iam.gserviceaccount.com

# Login as SA for session
gcloud auth application-default login \
  --impersonate-service-account=sa@PROJECT.iam.gserviceaccount.com
```

### IAM Policy Management
```bash
# View IAM policy on project
gcloud projects get-iam-policy PROJECT_ID

# View IAM policy on bucket
gcloud storage buckets get-iam-policy gs://BUCKET_NAME

# Add IAM binding
gcloud projects add-iam-policy-binding PROJECT_ID \
  --member=user:john@company.com \
  --role=roles/storage.objectViewer

# Remove IAM binding
gcloud projects remove-iam-policy-binding PROJECT_ID \
  --member=user:john@company.com \
  --role=roles/storage.objectViewer
```

### Service Account Management
```bash
# Create SA
gcloud iam service-accounts create SA_NAME \
  --display-name="My Service Account"

# List SAs
gcloud iam service-accounts list

# Grant SA access to resource
gcloud projects add-iam-policy-binding PROJECT_ID \
  --member=serviceAccount:SA_NAME@PROJECT.iam.gserviceaccount.com \
  --role=roles/storage.admin
```

---

## 9. Interview Q&A Flashcards

#### Q: What is the Principle of Least Privilege?
**A:** Security design concept: a principal should only be granted the **minimum exact permissions** required to perform their job function, and nothing more. Reduces blast radius if compromised.

#### Q: User has Viewer at Project level, but Editor at Bucket level. What can they do?
**A:** They can **edit that specific bucket**. Permissions are additive. Direct bucket policy expands access beyond inherited Viewer. They still have Viewer on other resources.

#### Q: Why should you avoid Default Service Accounts?
**A:** Default SAs historically have **Editor role** automatically. If a VM is compromised, attacker gets Editor on entire project. Violates least privilege.

#### Q: What is the mechanism behind Service Account Impersonation?
**A:** IAM validates calling principal has `ServiceAccountTokenCreator` role on target SA, then **dynamically mints short-lived access token** - typically 1 hour - instead of exposing static long-lived keys.

#### Q: Difference between Basic, Predefined, and Custom roles?
**A:** **Basic** - Legacy Owner/Editor/Viewer, too broad. **Predefined** - Google-managed, granular, recommended. **Custom** - User-defined, for specific needs when predefined doesn't fit.

#### Q: Can you deny permissions in GCP IAM?
**A:** **No explicit deny.** IAM is allow-only and additive. To restrict, you must not grant permission at higher levels. Use VPC Service Controls or Org Policies for restrictions.

---

## 10. IAM Best Practices & Golden Rule

### ✅ Checklist
- **Enforce Least Privilege:** Replace Basic roles with narrow predefined/custom roles
- **Prefer Groups over Users:** Map permissions to Google Groups. Add/remove users from groups
- **Audit Access Regularly:** Use Policy Analyzer, remove unused permissions
- **Use Service Account per App:** Don't share SAs across applications
- **Disable Default SAs:** Create user-managed SAs with minimal roles
- **Enable Audit Logs:** Track who accessed what
- **Rotate Keys if Used:** If you must use keys, rotate every 90 days max[x]

### 🏆 The Golden Rule

> **If you can avoid creating or downloading Service Account Keys, avoid them completely.**
>
> **Use:** Application Default Credentials (ADC) + Attach SAs to compute + Service Account Impersonation

**Keyless = Secure.** Keys = Risk.

---

## 🔐 Quick Decision Tree

```text
Need app to access GCP?
    │
    ├─ Running on GCP? ──► Attach SA to VM/Cloud Run/GKE
    │ Use ADC in code
    │
    └─ Running outside GCP?
        │
        ├─ Can use Workload Identity Federation? ──► Use WIF - Keyless
        │
        └─ Must use keys? ──► Create SA Key
                              Store in Secret Manager
                              Rotate every 90 days
```

---

<div align="center">

**⭐ Star this repo if it helped you ace your GCP interview!**

Made with ❤️ for Cloud Security

[⬆️ Back to Top](#️-google-cloud-iam--service-accounts-cheat-sheet)

</div>