
# **Lab 19: Azure Virtual Network (VNet) Peering – Private Connectivity Across VNets**

---

## 🧠 Purpose of This Lab

This lab demonstrates **Azure Virtual Network (VNet) Peering**, a core networking concept used to enable **private, secure communication between isolated VNets**.

Although this lab could not be fully executed due to **Azure free subscription vCPU limits**, the **architecture, design principles, and configuration steps are fully documented**, exactly as used in real-world Azure environments.

> ⚠️ **Important Note**
> Azure Free Trial allows limited vCPUs. This lab conceptually requires **3 VMs**, but only **2 VMs were deployed**.
> The design, behavior, and learning outcomes remain **100% valid and production-aligned**.

---

## 1️⃣ Why Do We Need VNet Peering?

### 🔹 Default Azure Behavior (The Problem)

In Azure:

* Every **Virtual Network (VNet) is isolated by default**
* Even if VNets are:

  * In the same subscription
  * Same resource group
  * Same region

➡️ **VMs in different VNets cannot communicate using private IP addresses**

This isolation is intentional for **security and tenant separation**.

---

### 🔹 Real-World Scenarios Requiring VNet Peering

VNet Peering is commonly used when:

* Dev, Test, and Prod environments are separated into VNets
* Applications are split across multiple VNets
* Multiple teams manage different VNets but require connectivity
* Hub-and-Spoke network architecture
* Shared services (DNS, monitoring, AD) in a central VNet

---

## 2️⃣ What Is Azure VNet Peering?

### ✅ Definition

**VNet Peering** connects two Azure VNets so that resources can communicate **privately** using private IP addresses.

### 🔐 Key Characteristics

* Traffic flows over **Microsoft’s private backbone**
* No need for:

  * VPN Gateway
  * Azure Firewall
  * Public IPs
  * NAT
* Low latency and high bandwidth
* Secure and private

📌 **Simple analogy**:

> VNet Peering is like **stitching two isolated networks together securely**.

---

## 3️⃣ High-Level Architecture (Lab Design)

### 🔹 Logical Design

```
Azure Subscription
└── Resource Group: rg-vnet-peering-lab
    ├── Virtual Network: vnet-dev (10.0.0.0/16)
    │   ├── subnet-web (10.0.1.0/24)
    │   │   └── vm-dev-web-01 (10.0.1.4)
    │   └── subnet-db (10.0.2.0/24)
    │       └── vm-dev-db-01 (planned)
    │
    ├── Virtual Network: vnet-test (10.1.0.0/16)
    │   └── subnet-web (10.1.1.0/24)
    │       └── vm-test-web-01 (10.1.1.4)
    │
    └── VNet Peering
        ├── vnet-dev → vnet-test
        └── vnet-test → vnet-dev
```

---

## 4️⃣ Critical Design Rules (Very Important)

### ✅ Address Space Must NOT Overlap

```
vnet-dev  → 10.0.0.0/16
vnet-test → 10.1.0.0/16
```

Overlapping address spaces **will break peering**.

---

### ✅ Peering Is Bi-Directional

You must configure **two peerings**:

* VNet-A → VNet-B
* VNet-B → VNet-A

One-way peering is **not sufficient**.

---

### ❌ Peering Is NOT Transitive

If:

* VNet-A peers with VNet-B
* VNet-B peers with VNet-C

➡️ **VNet-A cannot talk to VNet-C automatically**

This is a **very common interview trap question**.

---

## 5️⃣ Step-by-Step Lab Configuration

### 🔹 STEP 1: Create Resource Group

* Name: `rg-vnet-peering-lab`
* Region: Central India

---

### 🔹 STEP 2: Create First VNet (Dev)

* Name: `vnet-dev`
* Address space: `10.0.0.0/16`
* Subnets:

  * `subnet-web` → `10.0.1.0/24`
  * `subnet-db` → `10.0.2.0/24`

---

### 🔹 STEP 3: Deploy VM in VNet-Dev

* VM Name: `vm-dev-web-01`
* Size: B1s (1 vCPU)
* VNet: `vnet-dev`
* Subnet: `subnet-web`
* Public IP: Optional (for admin access)

Optional web server:

```bash
sudo apt update
sudo apt install apache2 -y
```

---

### 🔹 STEP 4: Create Second VNet (Test)

* Name: `vnet-test`
* Address space: `10.1.0.0/16`
* Subnet:

  * `subnet-web` → `10.1.1.0/24`

---

### 🔹 STEP 5: Deploy VM in VNet-Test

* VM Name: `vm-test-web-01`
* Size: 1 vCPU
* VNet: `vnet-test`
* Subnet: `subnet-web`

---

## 6️⃣ Connectivity Test (Before Peering)

From `vm-test-web-01`:

```bash
ping 10.0.1.4
```

❌ **Fails**

### ❓ Why?

Because VNets are **isolated by default**.

---

## 7️⃣ Configure VNet Peering (Most Important Step)

### 🔹 STEP 6A: Dev → Test

* Go to `vnet-dev`
* Peerings → Add
* Name: `dev-to-test`
* Remote VNet: `vnet-test`
* Allow virtual network access ✅

---

### 🔹 STEP 6B: Test → Dev

* Go to `vnet-test`
* Peerings → Add
* Name: `test-to-dev`
* Remote VNet: `vnet-dev`
* Allow virtual network access ✅

📌 Both peerings should show **Connected**

---

## 8️⃣ NSG Considerations

Even with peering:

* **NSGs still apply**
* Traffic can be blocked by NSG rules

Ensure NSG allows:

* ICMP (for ping), or
* TCP 80 (for HTTP testing)

Example:

```
Source: 10.0.0.0/8
Destination: Any
Protocol: ICMP / TCP
Action: Allow
```

---

## 9️⃣ Connectivity Test (After Peering)

From `vm-test-web-01`:

```bash
ping 10.0.1.4
```

✅ **Success**

If Apache installed:

```bash
curl http://10.0.1.4
```

---

## 🔟 Key Takeaways (AZ-104 + Interview Gold)

* VNets are isolated by default
* VNet peering enables private IP communication
* Peering:

  * Is bi-directional
  * Is non-transitive
* Address spaces must not overlap
* No gateways or public IPs required
* Traffic stays on Azure backbone

---

## 🧠 Why This Matters in Real Projects

* Enables multi-environment separation
* Reduces cost (no VPN gateway)
* Improves security posture
* Forms foundation for hub-and-spoke design
* Frequently used in enterprise Azure architectures
