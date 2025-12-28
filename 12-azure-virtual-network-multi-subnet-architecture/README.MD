# Lab 12: Azure Virtual Network Design with Multi-Subnet Architecture (Web & DB Tier Isolation)

---

## 📌 Overview

In this lab, I designed and implemented a **custom Azure Virtual Network (VNet)** with **multiple subnets** to represent a **real-world, production-grade application architecture**.

Instead of allowing Azure to auto-create networking during VM deployment, the **VNet and subnets were designed first**, and then virtual machines were deployed into the appropriate subnets based on their role (Web tier vs Database tier).

This approach reflects how **enterprise cloud environments** and **SRE teams** design networks for **security, scalability, and operational clarity**.

---

## 🎯 Objective

* Understand **why VNets are required in enterprise environments**
* Design a **multi-subnet Azure network**
* Deploy:

  * A **Web VM** exposed to the internet
  * A **Database VM** isolated from the internet
* Understand and apply:

  * IP addressing
  * Subnetting
  * CIDR notation
  * NIC behavior
* Align networking decisions with **SRE principles**

---

## 🏢 Why Companies Need a Virtual Network (VNet)

In real organizations:

* Applications **must not run directly on the public internet**
* Traffic must be **segmented, secured, and predictable**
* Failures must be **isolated** to reduce blast radius
* Operations teams need **clear visibility and control**

An **Azure Virtual Network (VNet)** provides:

* A **private network boundary** in Azure
* Control over:

  * IP address ranges
  * Subnets
  * Routing
  * Security rules (NSGs, firewalls)

> 🔎 **SRE perspective:**
> A well-designed VNet simplifies incident response, limits security exposure, and supports scalable growth.

---

## 🌐 VNet Design Used in This Lab

### Virtual Network Details

* **VNet Name:** `vnet-dev-cind-01`
* **Address Space:** `10.0.0.0/16`
* **Region:** Central India
* **Resource Group:** `rg-az104-dev-cind`

This `/16` address space allows **65,536 private IP addresses**, enabling future expansion without re-architecting the network.

### VNet Overview

![VNet Overview](https://raw.githubusercontent.com/PunitAnand-SRE/az-104-labs/main/12-azure-virtual-network-multi-subnet-architecture/screenshots/01-vnet-overview-and-address-space.png)

---

## 🧠 Core Networking Concepts (Explained with Examples)

### 1️⃣ IP Address

An **IP address** uniquely identifies a resource in a network.

**Examples from this lab:**

* Web VM private IP: `10.0.0.4`
* DB VM private IP: `10.0.1.4`

> 💡 **SRE relevance:**
> Predictable IP addressing is critical for monitoring, firewall rules, and troubleshooting production incidents.

---

### 2️⃣ CIDR (Classless Inter-Domain Routing)

CIDR defines how many IP addresses a network contains.

**CIDRs used in this lab:**

| CIDR  | Usage       | Total IPs |
| ----- | ----------- | --------- |
| `/16` | VNet        | 65,536    |
| `/24` | Each subnet | 256       |

Examples:

* `10.0.0.0/16` → Entire VNet
* `10.0.0.0/24` → Web subnet
* `10.0.1.0/24` → DB subnet

---

### 3️⃣ Subnet

A **subnet** is a logical subdivision of a VNet.

**Why subnets matter:**

* Security isolation
* Role-based segmentation
* Controlled traffic flow

---

## 🧩 Subnet Architecture (Web & DB Tiers)

| Subnet Name            | Address Range | Purpose       |
| ---------------------- | ------------- | ------------- |
| `snet-dev-cind-web-01` | `10.0.0.0/24` | Web tier      |
| `snet-dev-cind-db-01`  | `10.0.1.0/24` | Database tier |

### Subnet Design View

![Subnets](https://raw.githubusercontent.com/PunitAnand-SRE/az-104-labs/main/12-azure-virtual-network-multi-subnet-architecture/screenshots/04-db-subnet-and-private-db-vm-without-public-ip.png)

---

## 🖥️ Web Tier VM Deployment

### Web VM Details

* **VM Name:** `vm-dev-cind-web-01`
* **Subnet:** `snet-dev-cind-web-01`
* **Private IP:** `10.0.0.4`
* **Public IP:** Enabled
* **Purpose:** Internet-facing application server

### Web VM Overview

![Web VM](https://raw.githubusercontent.com/PunitAnand-SRE/az-104-labs/main/12-azure-virtual-network-multi-subnet-architecture/screenshots/02-web-vm-deployed-in-web-subnet-with-public-ip.png)

---

## 🔌 Network Interface (NIC) Behavior

When a VM is deployed, Azure automatically creates:

* A **Network Interface (NIC)**
* Assigns:

  * A private IP from the subnet
  * A public IP (if enabled)

### NIC IP Assignment

![NIC IP Assignment](https://raw.githubusercontent.com/PunitAnand-SRE/az-104-labs/main/12-azure-virtual-network-multi-subnet-architecture/screenshots/03-network-interface-ip-assignment-from-subnet.png)

**Key observation:**

* IP `10.0.0.4` was dynamically allocated from `snet-dev-cind-web-01`

> 📘 **AZ-104 key point:**
> IPs and NSGs are associated with **NICs**, not directly with VMs.

---

## 🛢️ Database Tier VM Deployment (Secure by Design)

### DB VM Details

* **VM Name:** `vm-dev-cind-db-01`
* **Subnet:** `snet-dev-cind-db-01`
* **Private IP:** `10.0.1.4`
* **Public IP:** ❌ Not assigned
* **Purpose:** Internal database server

### Why No Public IP?

* Prevents direct internet access
* Reduces attack surface
* Enforces internal-only communication

> 🔐 **SRE principle:**
> “If a resource doesn’t need internet access, it should not have a public IP.”

---

## 🔁 End-to-End Traffic Flow

```
Internet
   |
   | (Public IP)
   v
Web VM (10.0.0.4)
   |
   | (Private VNet traffic)
   v
DB VM (10.0.1.4)
```

---

## 🧪 Verification

* Web VM accessible from the internet
* DB VM not reachable from the internet
* Both VMs can communicate internally
* IPs correctly assigned from respective subnets

---

## 🔐 Security & SRE Design Principles Applied

✔ Network segmentation
✔ Least exposure principle
✔ Tier-based isolation
✔ Scalable IP planning
✔ Production-aligned architecture

---

## 📚 AZ-104 Skills Covered

* Azure Virtual Networks
* Subnet design & CIDR
* NIC and IP behavior
* Public vs private IPs
* Secure VM deployment

---

## 🧹 Cleanup

To avoid unnecessary Azure charges:

* Delete resource group: `rg-az104-dev-cind`

---

## 🚀 Why This Lab Matters for an SRE Role

This lab demonstrates:

* Intentional network design
* Security-first thinking
* Clear operational boundaries
* Troubleshooting-friendly architecture

This is **how real production systems are designed and reviewed** by SRE and Cloud Engineering teams.
