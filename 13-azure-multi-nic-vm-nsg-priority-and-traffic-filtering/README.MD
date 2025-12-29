# Lab 13: Multi-NIC Virtual Machine with NSG Priority-Based Traffic Filtering in Azure

## 📌 Overview

In this lab, I implemented and validated a **multi-network-interface (multi-NIC) Virtual Machine** architecture in Azure and demonstrated how **Network Security Group (NSG) rule priority** directly affects application accessibility — even when the application (NGINX) is correctly installed and running.

This lab focuses on **real-world Azure networking behavior**, including:

* Azure NAT (Public IP → Private IP translation)
* NIC-level vs Subnet-level NSG association
* NSG rule priority evaluation
* Traffic filtering using deny rules
* Troubleshooting application connectivity at the network layer

This design is commonly used when a VM acts as a **network appliance** (firewall, proxy, traffic inspection VM, or routing VM).

---

## 🎯 Objectives

* Attach a **secondary network interface** to an existing VM
* Understand **why multi-NIC VMs are used** in enterprise environments
* Apply **NSG rules at NIC level**
* Demonstrate **Azure NAT behavior**
* Prove **NSG priority order** using allow and deny rules
* Validate how network rules override application availability

---

## 🧱 Architecture Summary

| Component          | Details                                 |
| ------------------ | --------------------------------------- |
| Virtual Network    | Same VNet and subnet used for both NICs |
| Virtual Machine    | Web VM with NGINX installed             |
| Network Interfaces | Primary NIC + Secondary NIC             |
| NSG Scope          | Applied to **Network Interface**        |
| Public Access      | Via Public IP (NATed to Private IP)     |
| Application        | NGINX (HTTP – port 80)                  |

---

## 🔹 Why Use a Multi-NIC Virtual Machine?

Multi-NIC VMs are commonly used when a VM must:

* Act as a **traffic inspection appliance**
* Receive traffic from the internet
* Filter or analyze traffic
* Forward allowed traffic to internal workloads

Examples:

* Firewall VM
* Reverse proxy
* Load inspection VM
* Bastion-style routing VM

Azure allows multiple NICs **only when all NICs are in the same VNet**.

---

## 🖧 Step 1: Secondary Network Interface Attached to Web VM

A secondary network interface was created and attached to the existing web VM.
Both NICs are connected to the **same VNet and subnet**, enabling advanced traffic handling scenarios.

### 🔍 Key Points

* One VM can have multiple NICs
* Each NIC can have its **own NSG**
* Traffic can be controlled per interface
* Ideal for appliance-style VMs

### 📸 Screenshot

![Multi NIC VM](https://raw.githubusercontent.com/PunitAnand-SRE/az-104-labs/main/13-azure-multi-nic-vm-nsg-priority-and-traffic-filtering/screenshots/01-web-vm-with-secondary-network-interface-attached.png)

---

## 🔐 Step 2: NSG Allow Rule Using Private IP (Understanding Azure NAT)

An inbound NSG rule was created to **allow HTTP (port 80)** traffic.

### ❗ Important Concept: Azure NAT

Even though users access the VM using a **Public IP**, Azure internally:

* Performs **Destination NAT**
* Converts public IP traffic to the VM’s **private IP**
* NSG rules must therefore reference the **private IP**

### 🔧 Rule Configuration

* **Source**: Any
* **Destination**: VM Private IP (example: `10.0.0.4`)
* **Protocol**: TCP
* **Port**: 80
* **Action**: Allow
* **NSG Scope**: Network Interface

### 📸 Screenshot

![Allow HTTP Rule](https://raw.githubusercontent.com/PunitAnand-SRE/az-104-labs/main/13-azure-multi-nic-vm-nsg-priority-and-traffic-filtering/screenshots/02-nsg-allow-http-to-private-ip-address.png)

---

## 🚦 Step 3: Understanding NSG Rule Priority (Critical Concept)

Azure NSG rules are evaluated based on **priority number**:

* **Lower number = higher priority**
* Once a rule matches, **evaluation stops**
* Later rules are **not checked**

### 🚫 Deny Rule Created

A **deny rule** was intentionally added to block traffic on port range **70–100**.

### 🔧 Rule Details

* **Priority**: 290 (higher priority than allow rule)
* **Port Range**: 70–100
* **Protocol**: TCP
* **Destination**: VM Private IP
* **Action**: Deny

Because HTTP (80) falls within this range:

* Traffic is denied **before** the allow rule is evaluated
* NGINX becomes inaccessible
* Browser shows **connection timeout**

### 📸 Screenshot

![NSG Priority Blocking Traffic](https://raw.githubusercontent.com/PunitAnand-SRE/az-104-labs/main/13-azure-multi-nic-vm-nsg-priority-and-traffic-filtering/screenshots/03-nsg-deny-rule-priority-blocking-nginx-access.png)

---

## 🧠 Key Learning: Why NGINX Is Not Loading

| Check                     | Status             |
| ------------------------- | ------------------ |
| VM Running                | ✅                  |
| NGINX Installed           | ✅                  |
| Public IP Reachable       | ✅                  |
| NSG Allow Rule Exists     | ✅                  |
| Higher Priority Deny Rule | ❌ (Blocks traffic) |

### 🔑 Conclusion

**Networking rules override application availability.**
Even if an application is healthy, **NSG priority determines traffic flow**.

---

## 🧩 NSG Association: NIC vs Subnet

| Scope            | Use Case                               |
| ---------------- | -------------------------------------- |
| Subnet-level NSG | Broad control for multiple VMs         |
| NIC-level NSG    | Fine-grained control per VM or per NIC |

In this lab:

* NSG was applied at **NIC level**
* Enabled precise traffic filtering
* Demonstrated appliance-style control

---

## 💼 Real-World Use Cases

This lab design mirrors:

* Firewall VM setups
* Zero Trust architectures
* DMZ-based designs
* Network troubleshooting scenarios
* Azure security interviews (very common topic)

---

## ✅ Final Takeaways

* Azure uses **NAT**, not direct public IP routing
* NSG rules are evaluated strictly by **priority**
* A single deny rule can override multiple allow rules
* Multi-NIC VMs are powerful but must be designed carefully
* Network troubleshooting must always start at **NSG level**

---

## 📘 AZ-104 Exam Relevance

This lab directly covers:

* Azure VNets
* NSGs
* NIC-level security
* Traffic flow analysis
* Troubleshooting connectivity
