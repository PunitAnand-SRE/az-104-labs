# Lab 10: Azure Virtual Machine Scale Sets (VMSS) with CPU-Based Autoscale – Scale Out and Scale In

![Azure VMSS Autoscale Banner](https://img.shields.io/badge/Azure-VMSS%20Autoscale-blue?logo=microsoftazure&logoColor=white&style=flat-square)

## Overview

Welcome to **Lab 10** in the AZ-104 Azure Administrator certification preparation series! This hands-on lab focuses on Azure Virtual Machine Scale Sets (VMSS), a powerful Azure Compute service for deploying and managing groups of identical, load-balanced virtual machines. VMSS enables horizontal scaling to handle varying workloads efficiently, ensuring high availability and cost optimization.

In this lab, you'll:
- Create a VMSS with basic networking configuration.
- Configure CPU-based autoscale rules to automatically **scale out** (add instances) when CPU demand exceeds 70% and **scale in** (remove instances) when it drops below 50%.
- Simulate load using the `stress` tool on a VM instance to trigger scaling.
- Troubleshoot and resolve a common error related to Azure Monitor (Microsoft.Insights) registration.
- Verify scaling events through Azure Monitor and instance views.

This lab aligns with AZ-104 exam objectives on implementing and managing Azure Compute resources, including autoscaling configurations. By the end, you'll have a scalable infrastructure setup that's resilient to demand fluctuations.

**Lab Duration:** 45-60 minutes  
**Difficulty Level:** Intermediate  
**Cost Estimate:** ~$0.10-0.50 (depending on instance runtime; use free tier where possible and clean up resources post-lab).

## Learning Objectives

- Understand VMSS architecture and its role in scalable cloud applications.
- Configure inbound ports, public IP, and Network Security Groups (NSGs) for VMSS instances.
- Implement metric-based autoscale rules using Azure Monitor.
- Simulate and validate scale-out/scale-in behaviors.
- Troubleshoot subscription-level prerequisites for monitoring services.
- Recognize how autoscaling integrates with load balancers for traffic distribution.

## Prerequisites

- An active Azure subscription (free tier eligible).
- Access to the Azure Portal with Contributor or Owner role on a resource group.
- Basic familiarity with Azure VMs (from previous labs like Lab 08: Azure Virtual Machines).
- SSH client (e.g., Azure Cloud Shell, PuTTY, or OpenSSH) for connecting to Linux instances.
- A resource group (e.g., "DefaultDirectory" from your screenshots) in Central India region.

**Tools Needed:**
- Azure Portal (browser-based).
- Azure Cloud Shell (for SSH and stress testing).
- GitHub repo for documentation (this lab's folder: `10-Azure Virtual Machine Scale Sets (VMSS) with CPU-Based Autoscale – Scale Out and Scale In`).

**Cleanup Note:** After the lab, delete the resource group or VMSS to avoid charges. Use the "Delete resource group" option in the Azure Portal.

## Key Concepts

Before diving in, let's clarify core terms and Azure best practices (sourced from Microsoft documentation and SRE guidelines):

### What is a Virtual Machine Scale Set (VMSS)?
VMSS is an Azure service that lets you create and manage a collection of identical VMs (instances) that are load-balanced and autoscalable. It's ideal for applications needing high availability across fault/update domains or availability zones. Unlike single VMs, VMSS handles orchestration modes like **Uniform** (identical instances) or **Flexible** (diverse instances for advanced scenarios). In this lab, we use Uniform mode.

- **Instances:** Individual VMs in the scale set. They are auto-created/deleted during scaling and share configurations (e.g., OS image, size like Standard_B1s).
- **Load Balancer Integration:** Traffic is distributed via Azure Load Balancer, ensuring even workload across instances.

### Scale Out vs. Scale In
- **Scale Out (Horizontal Scaling):** Automatically adds VM instances when demand rises (e.g., high CPU). This increases capacity without downtime, as new instances are provisioned and load-balanced in. Best practice: Trigger on sustained thresholds (e.g., average CPU > 70% for 5 minutes) to avoid "flapping" (rapid up/down cycles).
- **Scale In (Horizontal Scaling):** Removes excess instances when demand drops (e.g., low CPU), optimizing costs. Use a hysteresis margin (e.g., scale in at <50% if out at >70%) to prevent flapping. Azure prioritizes terminating "OldestVM" or "NewestVM" based on scale-in policy.

Vertical scaling (up/down VM size) is manual and not covered here—VMSS excels at horizontal scaling.

### Scale Set Rules (Autoscale Rules)
Autoscale rules are defined in Azure Monitor and evaluate metrics every 1 minute. They trigger actions like "Increase count by 1" within cooldown periods (e.g., 5 minutes) to stabilize the system.

- **Types of Rules:**
  - **Metric-Based (Reactive):** Trigger on host metrics (e.g., CPU, Memory, Disk I/O) or custom app metrics. Example: Scale out if queue length > 50 messages per instance.
  - **Schedule-Based (Proactive):** Scale for predictable patterns, e.g., add instances weekdays 9 AM-5 PM or during events like Black Friday.
  - **Profile-Based:** Combine rules into profiles (default, fixed-date, recurring). Evaluation: Scale out if **any** rule met; scale in if **all** rules met.

- **Other Rule Examples (Beyond CPU):**
  - **Memory Usage:** Scale out if average memory > 80%.
  - **Network Traffic:** Scale in if inbound bytes < 10 MB/min.
  - **Custom Metrics:** Use Application Insights for app-specific rules, e.g., HTTP queue length > 100.
  - **Queue Length:** For Azure Storage/Service Bus: Scale out if messages >= 50 (per instance).
  - **Composite Rules:** Multiple conditions, e.g., (CPU > 70% AND Memory > 60%).

Best practices: Set min=1, max=3-10 instances; use average aggregation; enable predictive autoscale for ML-based forecasting.

### Alignment with SRE Goals
As a Site Reliability Engineer (SRE), your focus is on reliability (SLOs like 99.9% availability), scalability, and efficiency (error budgets for innovation). This lab builds SRE muscle by:
- **Reliability:** Autoscaling ensures SLOs during traffic spikes, reducing MTTR (Mean Time to Recovery) via proactive load handling.
- **Scalability:** Horizontal scaling supports elastic workloads, aligning with Azure Well-Architected Framework's Performance Efficiency pillar.
- **Efficiency/Toil Reduction:** Automates manual scaling, freeing time for higher-value tasks like chaos engineering or observability.
- **Cost Optimization:** Scale-in prevents overprovisioning, tying to SRE's error budget—balance reliability with sustainable ops.
- **Observability:** Integrates Azure Monitor for SLIs (e.g., CPU as proxy for latency), enabling data-driven decisions.

In production, extend with Azure SRE tools like SRE Agent for AI-driven incident response or Chaos Studio for resilience testing.

## Lab Instructions

Follow these steps in the Azure Portal. All screenshots reference your uploaded images for visual guidance.

### Step 1: Create VMSS with Networking Configuration
1. In the Azure Portal, search for "Virtual machine scale sets" and select **Create**.
2. Under **Basics**:
   - Resource group: Select your existing group (e.g., "DefaultDirectory").
   - VMSS name: `scaleSet01`.
   - Region: Central India.
   - Availability zone: No (uses fault domains).
   - Orchestration mode: **Uniform**.
   - Image: Ubuntu Server 22.04 LTS - x64 Gen 2.
   - Size: Standard_B1s (1 vCPU, 1 GiB memory) – cost-effective for testing.
   - Instance count: 1.
   - Admin username: `azureuser`.
   - Authentication: SSH public key (generate new or use existing).
3. Under **Networking** (key step – see screenshot):
   - Virtual network: Create new (`vnet-centralindia`).
   - Subnet: Create new (`snet-centralindia-1`, 172.16.0.0/24).
   - Load balancing services: **Azure Load Balancer**.
   - Edit public IP: **Enabled** (for SSH access; nominal charge applies).
   - NIC network security group: **Basic** (allows HTTP/HTTPS).
   - Inbound ports: **Allow selected ports** → Select **HTTP (80)** and **SSH (22)**.
   
   ![VMSS NIC Configuration](https://github.com/PunitAnand-SRE/az-104-labs/blob/main/10-Azure%20Virtual%20Machine%20Scale%20Sets%20(VMSS)%20with%20CPU-Based%20Autoscale%20%E2%80%93%20Scale%20Out%20and%20Scale%20In/screenshots/01-VMSS-NIC-Configuration-PublicIP-Enabled-and-Inbound-Ports-HTTP-SSH.png)
   
   > **Tip:** The warning about NSG rules is standard—use advanced networking tab for IP restrictions in production.

4. Review + Create → **Create**. Wait 5-10 minutes.

### Step 2: Verify VMSS Creation
1. Navigate to **Resource groups** > Your group > `scaleSet01`.
2. In the **Overview** blade, confirm:
   - Status: Succeeded.
   - Instance count: 1.
   - Orchestration mode: Uniform.
   - OS: Linux.

   ![VMSS scaleSet01 Overview](https://github.com/PunitAnand-SRE/az-104-labs/blob/main/10-Azure%20Virtual%20Machine%20Scale%20Sets%20(VMSS)%20with%20CPU-Based%20Autoscale%20%E2%80%93%20Scale%20Out%20and%20Scale%20In/screenshots/02-VMSS-scaleSet01-Overview-After-Creation.png)

### Step 3: Check Initial Instances
1. Select **Instances** under Settings.
2. Verify one running instance (`scaleSet01_0`):
   - Status: Running.
   - Public IP: e.g., 13.71.23.52 (note for SSH).
   - Private IP: e.g., 172.16.0.4.
   - Provisioning state: Succeeded.

   ![Initial Instance View](https://github.com/PunitAnand-SRE/az-104-labs/blob/main/10-Azure%20Virtual%20Machine%20Scale%20Sets%20(VMSS)%20with%20CPU-Based%20Autoscale%20%E2%80%93%20Scale%20Out%20and%20Scale%20In/screenshots/03-VMSS-scaleSet01-Instances-Initial-State-with-PublicIP.png)

### Step 4: Configure Scale-Out Rule
1. In `scaleSet01` > **Scaling** (under Settings) > **Autoscale** > **Custom autoscale**.
2. Enable autoscale.
3. Add rule:
   - Metric: **Percentage CPU** (Average).
   - Operator: **Greater than**.
   - Threshold: **70**.
   - Duration: **5 minutes**.
   - Operation: **Increase count by** → **1**.
   - Cool down: **5 minutes**.
   - Instance limits: Min **1**, Max **3**.

   ![Scale-Out Rule Creation](https://github.com/PunitAnand-SRE/az-104-labs/blob/main/10-Azure%20Virtual%20Machine%20Scale%20Sets%20(VMSS)%20with%20CPU-Based%20Autoscale%20%E2%80%93%20Scale%20Out%20and%20Scale%20In/screenshots/04-VMSS-Autoscale-Rule-Creation-CPU-Greater-Than-70-Percent.png)

4. **Save**. If error occurs (e.g., "MissingSubscriptionRegistration" for 'microsoft.insights'), proceed to Step 5.

### Step 5: Troubleshoot and Register Microsoft.Insights Provider
1. Go to **Subscriptions** > Your subscription > **Resource providers**.
2. Search for `Microsoft.Insights` → Status: **Registering** or **NotRegistered**.
3. Select **Register**.

   ![Registering Microsoft Insights](https://github.com/PunitAnand-SRE/az-104-labs/blob/main/10-Azure%20Virtual%20Machine%20Scale%20Sets%20(VMSS)%20with%20CPU-Based%20Autoscale%20%E2%80%93%20Scale%20Out%20and%20Scale%20In/screenshots/05-Subscription-Registering-Microsoft-Insights-Provider.png)

4. Wait 2-3 minutes, refresh, and retry saving the rule. This enables Azure Monitor metrics for autoscale.

### Step 6: Simulate CPU Load and Trigger Scale-Out
1. SSH to the instance: `ssh azureuser@<public-ip>` (e.g., 13.71.23.52).
2. Install `stress`: `sudo apt update && sudo apt install -y stress`.
3. Run stress: `stress --cpu 1 --timeout 600s` (stresses 1 CPU core for 10 minutes).
4. In Portal: Monitor **Scaling** > **Run history** for scale-up events.

   ![Scale-Up Triggered](https://github.com/PunitAnand-SRE/az-104-labs/blob/main/10-Azure%20Virtual%20Machine%20Scale%20Sets%20(VMSS)%20with%20CPU-Based%20Autoscale%20%E2%80%93%20Scale%20Out%20and%20Scale%20In/screenshots/06-VMSS-Autoscale-Run-History-Scale-Up-Triggered.png)

   > **Observed:** Capacity increased from 1 to 3 instances after ~5-10 minutes.

### Step 7: Verify Scaled-Out Instances
1. Go to **Instances**.
2. Confirm 3 running instances: `scaleSet01_0`, `scaleSet01_1`, `scaleSet01_2`.

   ![Instances After Scale-Out](https://github.com/PunitAnand-SRE/az-104-labs/blob/main/10-Azure%20Virtual%20Machine%20Scale%20Sets%20(VMSS)%20with%20CPU-Based%20Autoscale%20%E2%80%93%20Scale%20Out%20and%20Scale%20In/screenshots/07-VMSS-Instances-After-Scale-Up-Three-Instances-Running.png)

### Step 8: Configure Scale-In Rule
1. Back to **Scaling** > Edit autoscale.
2. Add rule:
   - Metric: **Percentage CPU** (Average).
   - Operator: **Less than**.
   - Threshold: **50**.
   - Duration: **5 minutes**.
   - Operation: **Decrease count by** → **1**.
   - Cool down: **5 minutes**.

   ![Scale-In Rule](https://github.com/PunitAnand-SRE/az-104-labs/blob/main/10-Azure%20Virtual%20Machine%20Scale%20Sets%20(VMSS)%20with%20CPU-Based%20Autoscale%20%E2%80%93%20Scale%20Out%20and%20Scale%20In/screenshots/08-VMSS-Autoscale-Scale-In-Rule-CPU-Less-Than-50-Percent.png)

3. **Save**.

### Step 9: Trigger and Verify Scale-In
1. Stop stress on the original instance: `Ctrl+C`.
2. Wait 5-10 minutes; monitor **Run history** for scale-down.

   ![Scale-Down Triggered](https://github.com/PunitAnand-SRE/az-104-labs/blob/main/10-Azure%20Virtual%20Machine%20Scale%20Sets%20(VMSS)%20with%20CPU-Based%20Autoscale%20%E2%80%93%20Scale%20Out%20and%20Scale%20In/screenshots/09-VMSS-Autoscale-Run-History-Scale-Down-Triggered.png)

3. Check **Instances**: Should reduce to 1-2.

## Verification Steps

- **Scale-Out:** CPU >70% → Instances increase to max (3). Check Metrics blade for CPU graph.
- **Scale-In:** CPU <50% → Instances decrease to min (1). Review Activity Log for events.
- **Health:** All instances "Running"; load balancer health probes green.
- **Costs:** Use Azure Cost Management to confirm no unexpected charges.
- **Edge Case:** If flapping occurs, adjust thresholds (e.g., out at 70%, in at 40%).

## Troubleshooting

- **Insights Registration Error:** As in Step 5; registration takes ~5 minutes.
- **No Scaling:** Ensure duration=5min, cooldown=5min; verify metrics in Azure Monitor. Check if predictive autoscale is enabled (optional for forecasting).
- **SSH Issues:** Confirm inbound SSH(22) allowed; use public IP.
- **Flapping:** Increase hysteresis (difference between out/in thresholds) per best practices.
- **Quota Limits:** If max instances hit, request quota increase via Portal > Subscriptions > Usage + quotas.

## Cleanup

1. Portal > Resource groups > Your group > **Delete resource group**.
2. Confirm deletion (includes VMSS, VNet, LB).
3. Verify in **Activity log** no lingering resources.

## Next Steps & Further Reading

- Extend with Application Gateway for HTTPS/WAF.
- Explore predictive autoscale or custom metrics via Application Insights.
- AZ-104 Prep: Practice exam questions on Compute scaling.
- Resources:
  - [Azure VMSS Autoscale Overview](https://learn.microsoft.com/en-us/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-autoscale-overview)
  - [Best Practices for Autoscale](https://learn.microsoft.com/en-us/azure/azure-monitor/autoscale/autoscale-best-practices)
  - [SRE in Azure](https://learn.microsoft.com/en-us/azure/site-reliability-engineering/)

## Personal Reflection

This lab was a deep dive into elastic infrastructure—seeing instances auto-provision under load was eye-opening! It directly supports my SRE journey by automating reliability, reducing toil, and aligning with SLO-driven ops. Future labs: Integrate with AKS for container scaling.

**Lab Author:** Punit Anand (SRE Aspirant)  
**Date Completed:** November 21, 2025  
**GitHub Repo:** [PunitAnand-SRE/az-104-labs](https://github.com/PunitAnand-SRE/az-104-labs)
