# Lab 15 – Azure NSG ICMP Traffic Control and Protocol-Specific Filtering

## 📌 Lab Objective

This lab demonstrates **how Azure Network Security Groups (NSGs handle ICMP traffic**, how **outbound rules impact connectivity**, and why **ICMP behaves differently from application protocols like HTTP and SSH**.

By the end of this lab, you will clearly understand:

* Why **ping may fail even when VMs are in the same VNet**
* How **outbound NSG rules affect ICMP**
* Why ICMP **requires explicit inbound allow rules**
* How to troubleshoot **real-world connectivity issues** using NSGs

This lab builds directly on **Lab 14 (NSG Stateful Behavior and Intra-VNet Communication)** and uses the **same infrastructure**.

---

## 🏗️ Lab Architecture Overview

**Virtual Network**

* Single VNet (same as previous lab)
* Both VMs are deployed in the **same VNet**

**Virtual Machines**

* **Web VM**: `vm-dev-cind-web-01`

  * Private IP: `10.0.0.4`
  * NSG attached to NIC: `vm-dev-cind-web-01-nsg`
* **DB VM**: `vm-dev-cind-db-01`

  * No public IP

**Security**

* NSGs attached at **NIC level**
* Focus on **Inbound vs Outbound rules**
* Protocol under test: **ICMP (Ping)**

---

## 🧠 Why This Lab Is Important

Many engineers assume:

> “If two VMs are in the same VNet, ping will always work.”

This is **false**.

In real production environments:

* ICMP is often **blocked intentionally**
* Outbound rules are **tightly restricted**
* Ping failures are **misdiagnosed as network outages**

This lab teaches:

* **How NSGs actually process ICMP**
* **Why outbound rules matter**
* **How to explicitly allow ICMP safely**

---

## 🔑 Core Concepts Explained

### What is ICMP?

* ICMP (Internet Control Message Protocol) is used for:

  * `ping`
  * Network diagnostics
* ICMP is **not TCP or UDP**
* It does **not use ports**

---

### How NSGs Handle ICMP

* NSGs evaluate:

  * **Direction** (Inbound / Outbound)
  * **Protocol** (ICMP / TCP / UDP)
  * **Priority**
* ICMP **must be explicitly allowed**
* Outbound deny rules **block ping responses**

---

### Default NSG Rules (Key Ones Used Here)

| Rule              | Direction | Purpose                         |
| ----------------- | --------- | ------------------------------- |
| AllowVnetInbound  | Inbound   | Allows traffic within same VNet |
| AllowVnetOutbound | Outbound  | Allows traffic within same VNet |
| DenyAllOutbound   | Outbound  | Blocks all outbound traffic     |

⚠️ **Custom rules override default rules**

---

## 🧪 Lab Walkthrough & Observations

---

### 🔹 Step 1 – ICMP Blocked Due to Outbound NSG Rule

In the previous lab, an **outbound deny-all rule** was created on the **Web VM NSG**.

As a result:

* Ping **from Web VM → DB VM fails**
* Even though both VMs are in the same VNet
* Even though default `AllowVnetOutbound` exists

📸 **Evidence**

![ICMP blocked due to outbound NSG rule](https://raw.githubusercontent.com/PunitAnand-SRE/az-104-labs/main/15-azure-nsg-icmp-traffic-control-and-protocol-filtering/screenshots/01-icmp-blocked-due-to-outbound-nsg-rule-on-web-vm.png)

**Key Insight**

> Outbound NSG rules apply to ICMP exactly like any other protocol.

---

### 🔹 Step 2 – Fixing ICMP by Adjusting NSG Rules

To restore ICMP functionality:

#### Actions Taken

1. **Removed the outbound deny-all rule** from:

   * `vm-dev-cind-web-01-nsg`
2. Created a **custom inbound ICMP allow rule**

   * Protocol: ICMP
   * Source: Any
   * Destination: `10.0.0.4` (Web VM private IP)
   * Action: Allow
   * Priority: Higher than deny rules

---

### 🔹 Step 3 – ICMP Allowed Successfully

After applying the fixes:

* Ping from **local machine → Web VM** works
* ICMP traffic is explicitly allowed
* NSG behavior is predictable and controlled

📸 **Evidence**

![ICMP allowed after inbound rule and outbound fix](https://raw.githubusercontent.com/PunitAnand-SRE/az-104-labs/main/15-azure-nsg-icmp-traffic-control-and-protocol-filtering/screenshots/02-icmp-allowed-after-inbound-icmp-rule-and-outbound-fix.png)

**Key Insight**

> ICMP must be explicitly allowed, and outbound rules must permit return traffic.

---

## 🔍 Important Technical Learnings

### ICMP vs HTTP (Critical Difference)

| Behavior             | HTTP     | ICMP        |
| -------------------- | -------- | ----------- |
| Uses ports           | Yes      | No          |
| Stateful behavior    | Yes      | Limited     |
| Needs outbound allow | Implicit | Explicit    |
| Commonly blocked     | Rare     | Very common |

---

### Why Ping Failed Earlier but HTTP Worked (Lab 14)

* HTTP worked because:

  * NSGs are **stateful**
  * Response traffic was allowed automatically
* ICMP failed because:

  * Outbound deny rule blocked ICMP replies
  * ICMP doesn’t behave like application sessions

---

## 🏢 Real-World Use Cases

This lab mirrors real enterprise scenarios:

* Production VMs **block ICMP intentionally**
* Security teams **restrict outbound traffic**
* Engineers must:

  * Diagnose connectivity without assuming ping works
  * Know when to allow ICMP safely

Used in:

* Network troubleshooting
* Security audits
* Zero Trust architectures
* Azure certification (AZ-104 / AZ-700)

---

## ✅ Final Summary

* ICMP is **not automatically allowed**
* Outbound NSG rules **can break ping**
* Explicit inbound ICMP rules are required
* NSGs treat ICMP differently from HTTP/SSH
* Understanding this avoids **false outage alerts**

---

## 📌 Key Takeaway

> **Never assume ping failure means network failure.
> Always evaluate NSG direction, protocol, and priority.**

