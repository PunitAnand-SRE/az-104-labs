# **Lab 17: Azure NSG at Subnet Level – Shared NSG Design for Centralized Security Management**

---

## 📘 **Lab Objective**

The goal of this lab is to **design and implement a centralized Network Security Group (NSG) at the subnet level** instead of using individual NSGs on each VM Network Interface (NIC).

This lab demonstrates:

* Why **subnet-level NSGs are preferred in enterprise environments**
* How **one shared NSG can secure multiple workloads**
* How Azure evaluates traffic when **NSGs are applied at subnet vs NIC**
* How to **simplify security management** without reducing control

---

## 🏗️ **Architecture Overview**

**Environment used in this lab:**

* **Virtual Network (VNet):** Single VNet
* **Subnets:**

  * Web Subnet
  * DB Subnet
* **Virtual Machines:**

  * Web VM (running NGINX)
  * DB VM (running MySQL)
* **Security Model:**

  * One **shared NSG**
  * Associated at **subnet level**
  * No NSG attached at VM NIC level

---

## 🔐 **Why Subnet-Level NSGs Matter (Real-World Context)**

In enterprise Azure environments:

* Hundreds of VMs may exist in the same tier (Web / App / DB)
* Managing **individual NSGs per VM** becomes:

  * Error-prone
  * Difficult to audit
  * Hard to scale

**Subnet-level NSGs provide:**

* Centralized rule management
* Consistent enforcement
* Easier auditing & troubleshooting
* Cleaner architecture

> **Best Practice:**
> Use **subnet-level NSGs for baseline security**
> Use **NIC-level NSGs only for exceptional cases**

---

## 🧠 **Key Azure NSG Concepts (Quick Refresher)**

### Network Security Group (NSG)

A set of **stateful** security rules that control:

* Inbound traffic
* Outbound traffic

### Stateful Behavior

If traffic is **allowed inbound**, the **response is automatically allowed outbound**, even if outbound rules are restrictive.

### Rule Priority

* Lower number = higher priority
* Custom rules override default rules

### Default NSG Rules (Important)

| Rule                  | Description                |
| --------------------- | -------------------------- |
| AllowVNetInBound      | Allows traffic within VNet |
| AllowVNetOutBound     | Allows traffic within VNet |
| AllowInternetOutBound | Allows outbound internet   |
| DenyAllInBound        | Denies all other inbound   |
| DenyAllOutBound       | Denies all other outbound  |

---

## 🛠️ **Lab Implementation – Step by Step**

---

### **Step 1: Create a Shared NSG with Default Rules**

A new NSG was created without any custom rules.
At this stage, only **default Azure rules** are present.

📸 **Screenshot:** Shared NSG created with default rules
![Shared NSG default rules](https://raw.githubusercontent.com/PunitAnand-SRE/az-104-labs/main/17-azure-nsg-subnet-level-shared-nsg-centralized-security/screenshots/01-shared-nsg-created-with-default-rules.png)

**Why this matters:**

* Establishes a clean baseline
* Helps understand how default rules behave before customization

---

### **Step 2: Configure Centralized Inbound & Outbound Rules**

Custom rules were added to the shared NSG to cover **both Web and DB security needs**, such as:

**Inbound rules (examples):**

* Allow HTTP (80) from Internet → Web Subnet
* Allow MySQL (3306) from Web Subnet → DB Subnet
* Deny unwanted ports

**Outbound rules (examples):**

* Restrict unnecessary internet access
* Allow only required traffic paths

📸 **Screenshot:** Shared NSG with custom inbound and outbound rules
![Custom NSG rules](https://raw.githubusercontent.com/PunitAnand-SRE/az-104-labs/main/17-azure-nsg-subnet-level-shared-nsg-centralized-security/screenshots/02-shared-nsg-custom-inbound-outbound-rules-configured.png)

**Why this matters:**

* One NSG now enforces **tier-based security**
* Rules are reusable and scalable

---

### **Step 3: Remove NSGs from VM Network Interfaces**

Previously, NSGs were attached directly to VM NICs.
These were **removed** to avoid:

* Rule duplication
* Conflicting security logic
* Troubleshooting complexity

📸 **Screenshot:** NSGs disassociated from VM NICs
![NSG removed from NIC](https://raw.githubusercontent.com/PunitAnand-SRE/az-104-labs/main/17-azure-nsg-subnet-level-shared-nsg-centralized-security/screenshots/03-nsg-disassociated-from-vm-network-interfaces.png)

**Important Azure Behavior:**

> If NSGs exist on **both subnet and NIC**, traffic is evaluated:
>
> 1. Subnet NSG
> 2. NIC NSG

Removing NIC-level NSGs ensures **single-point control**.

---

### **Step 4: Associate Shared NSG with Web & DB Subnets**

The shared NSG was attached to:

* Web Subnet
* DB Subnet

This ensures:

* All VMs in these subnets automatically inherit security rules
* Any future VM added is secured **by default**

📸 **Screenshot:** Shared NSG associated with both subnets
![NSG at subnet level](https://raw.githubusercontent.com/PunitAnand-SRE/az-104-labs/main/17-azure-nsg-subnet-level-shared-nsg-centralized-security/screenshots/04-shared-nsg-associated-with-web-and-db-subnets.png)

---

## 🔍 **Traffic Flow After Implementation**

### Internet → Web VM

✔ Allowed (HTTP rule)

### Web VM → DB VM (MySQL)

✔ Allowed (VNet traffic + custom rule)

### DB VM → Internet

❌ Restricted (based on outbound policy)

### East-West Traffic (within VNet)

✔ Allowed by default VNet rules

---

## 🧪 **Validation & Results**

* Web application accessible from Internet
* DB accessible **only** from Web subnet
* No direct Internet access to DB VM
* Security rules enforced consistently
* No VM-specific configuration required

---

## 🧠 **What This Lab Proves (Key Takeaways)**

* Subnet-level NSGs provide **cleaner architecture**
* Shared NSGs reduce operational overhead
* Azure NSGs are **stateful**
* Default rules are powerful and should be understood
* This design mirrors **real enterprise Azure deployments**

---

## 🎯 **AZ-104 Exam Relevance**

This lab directly maps to:

* Implement network security groups
* Control inbound & outbound traffic
* Secure multi-tier applications
* Understand NSG evaluation order
* Apply least-privilege networking

---

## 🏁 **Conclusion**

This lab demonstrates how **centralized security using subnet-level NSGs** is:

* Easier to manage
* More secure
* More scalable
* Enterprise-ready

A strong foundation for:

* Hub-and-spoke networks
* Production workloads
* Zero-trust designs
