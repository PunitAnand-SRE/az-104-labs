# Azure NSG Stateful Behavior and Intra-VNet Communication

**Lab 14 – AZ-104**

---

## 📌 Lab Objective

The goal of this lab is to **deeply understand how Azure Network Security Groups (NSGs) work**, especially:

* Default NSG rules
* Intra-VNet (east-west) communication
* Inbound vs outbound traffic control
* **Stateful nature of NSGs**
* Why internal traffic works even when outbound traffic is denied

This lab demonstrates **real-world Azure security behavior**, not just theory.

---

## 🧱 Lab Architecture Overview

* **VNet**: Single Azure Virtual Network
* **VMs**:

  * `vm-dev-cind-web-01` → Web VM (NGINX installed)
  * `vm-dev-cind-db-01` → DB VM (used as client)
* **NSG**:

  * Attached at NIC level
  * Initially using **only default rules**
* **Connectivity**:

  * No inbound rules from Internet
  * Communication tested using **private IPs**

---

## 🔑 Key Azure Concepts Used in This Lab

### 1. Network Security Group (NSG)

An NSG is a **stateful firewall** that controls:

* **Inbound traffic**
* **Outbound traffic**

It works using:

* Rules
* Priorities (lower number = higher priority)
* Allow / Deny actions

---

### 2. Default NSG Rules (Very Important)

Every NSG automatically includes these rules:

| Rule Name             | Direction | Purpose                             |
| --------------------- | --------- | ----------------------------------- |
| AllowVnetInbound      | Inbound   | Allows traffic within the same VNet |
| AllowVnetOutbound     | Outbound  | Allows traffic within the same VNet |
| AllowInternetOutbound | Outbound  | Allows outbound Internet traffic    |
| DenyAllInbound        | Inbound   | Blocks all inbound traffic          |
| DenyAllOutbound       | Outbound  | Blocks all outbound traffic         |

These rules are the **foundation** of this lab.

---

### 3. Stateful Firewall Behavior

Azure NSGs are **stateful**, which means:

* If inbound traffic is allowed,
* The **return traffic is automatically allowed**,
* Even if outbound rules would normally deny it.

This is the **most critical learning** from this lab.

---

## 🧪 Step-by-Step Lab Walkthrough

---

## 1️⃣ Default NSG blocks inbound traffic from Internet

**Screenshot:**
`01-default-nsg-blocks-inbound-traffic-from-internet.png`

![Default NSG blocks inbound traffic](https://raw.githubusercontent.com/PunitAnand-SRE/az-104-labs/main/14-azure-nsg-stateful-behavior-and-intra-vnet-communication/screenshots/01-default-nsg-blocks-inbound-traffic-from-internet.png)

### What was done

* All **custom NSG rules were deleted**
* Only **default NSG rules** remained
* Tried accessing:

  * SSH from local machine
  * NGINX page from browser

### Result

* ❌ SSH failed
* ❌ Browser access failed

### Why this happens

* `DenyAllInbound` rule blocks all Internet traffic
* Azure is **secure by default**
* No public access unless explicitly allowed

✅ **This confirms baseline security posture**

---

## 2️⃣ HTTP access from DB VM to Web VM using private IP

**Screenshot:**
`02-http-access-from-db-vm-to-web-vm-using-private-ip.png`

![HTTP access DB to Web](https://raw.githubusercontent.com/PunitAnand-SRE/az-104-labs/main/14-azure-nsg-stateful-behavior-and-intra-vnet-communication/screenshots/02-http-access-from-db-vm-to-web-vm-using-private-ip.png)

### What was done

* Logged into **DB VM**
* Sent HTTP request to Web VM using:

  ```
  curl http://<web-vm-private-ip>
  ```

### Result

* ✅ NGINX page returned successfully

### Why this works

* Both VMs are in the **same VNet**
* `AllowVnetOutbound` (DB VM)
* `AllowVnetInbound` (Web VM)

✅ Internal (east-west) traffic is **allowed by default**

---

## 3️⃣ SSH from DB VM to Web VM using private IP

**Screenshot:**
`03-ssh-from-db-vm-to-web-vm-using-private-ip.png`

![SSH DB to Web](https://raw.githubusercontent.com/PunitAnand-SRE/az-104-labs/main/14-azure-nsg-stateful-behavior-and-intra-vnet-communication/screenshots/03-ssh-from-db-vm-to-web-vm-using-private-ip.png)

### What was done

* SSH attempted from DB VM to Web VM using private IP

### Result

* ✅ SSH connection successful

### Why this works

* No SSH rule was created
* Still allowed because:

  * Same VNet
  * `AllowVnetInbound` rule

📌 **Important takeaway**
Internal admin access works even without explicit rules.

---

## 4️⃣ Outbound NSG rule denying all outbound traffic on Web VM

**Screenshot:**
`04-outbound-nsg-rule-deny-all-outbound-traffic-on-web-vm.png`

![Outbound deny rule](https://raw.githubusercontent.com/PunitAnand-SRE/az-104-labs/main/14-azure-nsg-stateful-behavior-and-intra-vnet-communication/screenshots/04-outbound-nsg-rule-deny-all-outbound-traffic-on-web-vm.png)

### What was done

* Created a **custom outbound NSG rule** on Web VM
* Rule:

  * Direction: Outbound
  * Action: Deny
  * Destination: Any
  * Priority: Higher than default allow

### Purpose

* Restrict Web VM from accessing Internet
* Simulate locked-down production environment

---

## 5️⃣ Stateful NSG behavior: HTTP works but `apt update` fails

**Screenshot:**
`05-stateful-nsg-behavior-http-works-but-apt-update-fails.png`

![Stateful NSG behavior](https://raw.githubusercontent.com/PunitAnand-SRE/az-104-labs/main/14-azure-nsg-stateful-behavior-and-intra-vnet-communication/screenshots/05-stateful-nsg-behavior-http-works-but-apt-update-fails.png)

### Observations

#### ✅ HTTP from DB VM to Web VM still works

* Even though outbound traffic is denied
* Because:

  * Inbound request was allowed
  * Return traffic is **automatically allowed**

➡️ **This proves NSGs are stateful**

---

#### ❌ `sudo apt update` fails on Web VM

* Requires new outbound Internet connection
* Outbound deny rule blocks it

➡️ **Outbound deny rule works as expected**

---

## 🧠 Key Learnings & Why This Matters

### 🔐 Security Design

* Azure follows **default deny** for inbound traffic
* Encourages least-privilege networking

### 🔄 Stateful Firewalls

* Prevents accidental breakage of applications
* Return traffic is automatically allowed

### 🌐 Internal vs External Traffic

* Intra-VNet communication is trusted
* Internet access must be explicitly designed

### 🏢 Real-World Usage

* Common in:

  * Production environments
  * Private backends
  * Zero-trust networks
  * Hub-and-spoke architectures

---

## 🎯 When to Use This Knowledge

* Debugging connectivity issues
* Designing secure VNets
* Explaining “why traffic still works” to managers
* AZ-104 and real interview scenarios
* Production incident analysis

---

## ✅ Final Summary

This lab demonstrates that:

* Azure NSGs are **stateful**
* Internal VNet traffic works by default
* Outbound deny rules block **new** outbound connections
* Return traffic is always allowed
* Secure design does not break applications

📌 **This is foundational Azure networking knowledge**
