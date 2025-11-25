# Lab 09: Implement High Availability using Azure Availability Zones

## Overview and Purpose

This lab demonstrates **Azure Availability Zones (AZs)** – the highest level of built-in high availability for Azure Virtual Machines within a single region. Unlike Availability Sets (rack-level protection), Availability Zones protect against **entire datacenter failures** by placing VMs in physically separate datacenters (each with independent power, cooling, and networking) while maintaining low latency.

In this lab, we deployed a Windows VM named **AppVM** explicitly into **Availability Zone 1** in the **Central India** region. While a single-zone VM provides 99.9% SLA, deploying identical VMs across **2 or 3 zones** (e.g., Zone 1 + Zone 2 + Zone 3) unlocks the **99.99% uptime SLA** – critical for production workloads.

### Why This Lab Matters
- **AZ-104 Exam Weight**: 15–20% of the exam focuses on compute availability. Questions frequently compare **Availability Sets (99.95%)** vs **Availability Zones (99.99%)** vs **Zone-redundant services**.
- **Real-World Impact**: A single datacenter outage (fire, flood, cooling failure) can take down an entire Availability Set. Zones survive such events. Used by Netflix, banks, e-commerce, and any zero-downtime application.
- **SLA Jump**:  
  → 1 VM (no zone/set) → 99.9% (~8.7 hrs downtime/year)  
  → Availability Set (2+ VMs) → 99.95% (~4.3 hrs/year)  
  → **Availability Zones (2+ VMs)** → **99.99% (~52 min/year)**

### Learning Outcomes
- Select Availability Zone during VM creation
- Understand zone-aware vs zone-redundant resources
- Recognize which Azure regions support zones (Central India has 3)
- Prepare for exam scenarios requiring maximum resilience

## Lab Details
- **Resource Group**: `Resource-by-zone`
- **Region**: Central India (supports 3 zones)
- **VM Name**: `AppVM`
- **Image**: Windows Server 2022 Datacenter (or 2025)
- **Size**: Standard_B1s (burstable, cost-effective)
- **Zone Selected**: **Zone 1**
- **Date Completed**: November 21–25, 2025

## Step-by-Step Recap (What You Did)

1. Created a new resource group `Resource-by-zone`
2. Navigated to **Virtual machines → Create → Azure virtual machine**
3. In **Basics** tab → **Availability zone** dropdown → explicitly selected **1**
4. Chose Windows Server 2022/2025, Standard_B1s size
5. Configured networking (default VNet/subnet in Central India are zone-redundant)
6. Deployed the VM (~5–7 minutes)
7. Verified in VM Overview: **Availability zone: 1** clearly displayed

**Proof Screenshot**  
(AppVM successfully running in Availability Zone 1 – Central India)

![AppVM deployed in Availability Zone 1](https://raw.githubusercontent.com/PunitAnand-SRE/az-104-labs/main/09-implement-high-availability-availability-zones/screenshots/appvm-deployed-in-availability-zone-1.png)

## How to Achieve Full 99.99% HA (Next-Level Production Setup)
To get the real benefit, repeat the same steps and deploy:
- `AppVM-2` → Availability zone **2**
- `AppVM-3` → Availability zone **3** (optional

Then front them with:
- **Standard Load Balancer** (SKU: Standard, Zone-redundant)
- Backend pool containing all three VMs
- Health probe + load-balancing rule (e.g., port 80/443)

Result: Even if an entire datacenter goes down, traffic fails over instantly.

## Key Exam & Interview Points
| Feature                    | Availability Set       | Availability Zones         |
|----------------------------|------------------------|----------------------------|
| Protects against           | Rack/power failures     | Full datacenter failures   |
| Max SLA (2+ VMs)           | 99.95%                | **99.99%                   |
| Latency between instances   | Very low               | Low (same region)          |
| Cross-zone data transfer    | Free                   | ~$0.01/GB (egress)        |
| Supported regions           | All regions            | 60+ regions (including Central India) |

## Cleanup (Always Do This!)
1. Go to resource group → **Resource-by-zone**
2. Click **Delete resource group**
3. Type the name to confirm → Delete

Cost for this lab: <$0.15 if cleaned promptly.

## References
- Microsoft Learn: Availability Zones Overview → https://learn.microsoft.com/en-us/azure/reliability/availability-zones-overview
- Azure VM Availability Options → https://learn.microsoft.com/en-us/azure/virtual-machines/availability
- AZ-104 Study Guide – Compute Section → https://learn.microsoft.com/en-us/certifications/resources/study-guide-az-104

---

**Lab Completed Successfully – November 2025**  
Author: Punit Anand | SRE & Azure Enthusiast
