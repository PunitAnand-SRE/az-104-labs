# **Lab 18: Azure Application Security Groups (ASG) with Network Security Groups (NSG)**

---

## 📘 Lab Overview

In this lab, we explore **Application Security Groups (ASG)** and understand **why they are critical in modern, scalable Azure architectures**.

We start by identifying the **limitations of IP-based NSG rules**, then introduce ASGs as an abstraction layer that enables **role-based, dynamic, and maintainable security rules** — without relying on static IP addresses.

This lab demonstrates how ASGs work together with NSGs to secure **Web-to-Database traffic** in a **two-tier architecture**.

---

## 🎯 Objectives

By the end of this lab, you will understand:

* Why **IP-based NSG rules do not scale**
* What **Application Security Groups (ASG)** are
* How ASGs simplify NSG rule management
* How to associate ASGs with virtual machines
* How NSG rules work using ASGs instead of IP addresses
* Why ASGs are **preferred in enterprise and production environments**

---

## 🧱 Architecture Used in This Lab

* **Web VM** (Application Tier)
* **DB VM** (Database Tier)
* **Shared NSG** applied at subnet level
* **Application Security Groups**

  * ASG for Web tier
  * ASG for DB tier

Traffic flow:

```
Internet → Web VM → DB VM (MySQL)
```

---

## 🔹 Part 1: Problem with Traditional IP-Based NSG Rules

### ❌ Static IP-Based Security Rules

Initially, the NSG rules were configured using **private IP addresses** of the Web VM and DB VM.

📸 **Screenshot: NSG using static IP addresses**
![Static IP based NSG rules](https://raw.githubusercontent.com/PunitAnand-SRE/az-104-labs/main/18-azure-application-security-groups-asg-with-nsg/screenshots/01-nsg-using-static-ip-addresses-for-web-and-db-traffic.png)

### ⚠️ Why This Is a Problem

Using IP addresses in NSG rules causes several issues:

* VM IPs can change (redeployments, scaling, re-creation)
* Adding a new Web VM requires **manual NSG updates**
* High operational overhead
* Error-prone and hard to audit
* Poor scalability in auto-scaling scenarios

👉 **This approach does not work well in real-world enterprise environments.**

---

## 🔹 Part 2: What Is an Application Security Group (ASG)?

### ✅ Definition

**Application Security Groups (ASG)** allow you to group Azure VMs based on their **application role** (Web, App, DB), rather than their IP address.

ASGs act as **logical identity labels** that can be referenced inside NSG rules.

### 🔑 Key Concept

> **NSGs control traffic, ASGs define who the traffic applies to**

---

## 🔹 Part 3: Creating ASGs for Application Tiers

We created two ASGs:

* **asg-az104-cind-web** → for Web tier VMs
* **asg-az104-cind-db** → for Database tier VMs

📸 **Screenshot: ASGs created for Web and DB tiers**
![ASG creation](https://raw.githubusercontent.com/PunitAnand-SRE/az-104-labs/main/18-azure-application-security-groups-asg-with-nsg/screenshots/02-application-security-groups-created-for-web-and-db-tiers.png)

### 🧠 Why This Matters

Instead of saying:

> “Allow traffic from 10.0.1.4”

We now say:

> “Allow traffic from **Web tier**”

This is **role-based security**, not IP-based security.

---

## 🔹 Part 4: Associating ASGs with Virtual Machines

Each VM network interface was associated with the appropriate ASG:

* Web VM → **asg-az104-cind-web**
* DB VM → **asg-az104-cind-db**

📸 **Screenshot: ASGs associated with VMs**
![ASG association](https://raw.githubusercontent.com/PunitAnand-SRE/az-104-labs/main/18-azure-application-security-groups-asg-with-nsg/screenshots/03-asg-associated-with-web-vm-and-db-vm.png)

### 🔎 Important Notes

* ASGs are associated at the **NIC level**
* A VM can belong to **multiple ASGs**
* ASG association does **not interrupt traffic**

---

## 🔹 Part 5: Updating NSG Rules to Use ASGs

The NSG rule was updated to:

* **Source** → ASG (Web tier)
* **Destination** → ASG (DB tier)
* **Port** → 3306 (MySQL)

📸 **Screenshot: NSG rules using ASGs**
![NSG with ASG](https://raw.githubusercontent.com/PunitAnand-SRE/az-104-labs/main/18-azure-application-security-groups-asg-with-nsg/screenshots/04-nsg-rules-updated-to-use-asg-instead-of-ip-addresses.png)

### ✅ Result

* Web VMs can connect to DB VMs
* No IP addresses used
* Security is **clean, readable, and scalable**

---

## 🔐 Security Flow After ASG Implementation

| Source Tier  | Destination Tier | Allowed | Reason              |
| ------------ | ---------------- | ------- | ------------------- |
| Internet     | Web              | ✅       | HTTP allowed        |
| Internet     | DB               | ❌       | No ASG match        |
| Web ASG      | DB ASG           | ✅       | Explicit NSG rule   |
| Any Other VM | DB               | ❌       | Not part of Web ASG |

---

## 🚀 Why ASGs Are Critical in Production

### ✔ Scalability

* Add or remove VMs without touching NSG rules

### ✔ Reduced Operational Risk

* No IP management
* No broken rules due to IP changes

### ✔ Cleaner Security Design

* Rules are **intent-based**, not network-based

### ✔ DevOps & Auto-Scaling Friendly

* Works seamlessly with VMSS and automation

### ✔ Interview & AZ-104 Favorite

Common question:

> *“How do you manage NSG rules in dynamic environments?”*

Correct answer:

> **Using Application Security Groups**

---

## 🧠 Key Takeaways

* NSGs control **traffic**
* ASGs define **who**
* ASGs eliminate IP dependency
* ASGs are essential for enterprise-grade Azure networking
* This pattern is used in **real production systems**

---

## 🏁 Final Thoughts

This lab demonstrates **how Azure networking evolves**:

1. IP-based rules (basic)
2. Subnet-level NSGs (centralized)
3. ASG-based rules (**enterprise-ready**)

