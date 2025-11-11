# Lab 1: Create a Windows Virtual Machine in Azure

## Objective
Deploy a Windows Server VM using the Azure Portal and connect via RDP. (Udemy Section 1, Alan Rodrigues.)

## Prerequisites
- Azure free subscription.
- Access to Azure Portal (portal.azure.com).

## Step-by-Step Execution
1. **Resource Group**: Created `FirstVM` in Central India region (low latency for India-based ops).
2. **VM Setup**:
   - Name: `AppVM`.
   - Image: Windows Server 2025 Datacenter (Gen2, x64 architecture).
   - Size: Standard B1s (1 vCPU, 1 GiB RAM—cost-effective for learning).
   - Authentication: Admin username (redacted for security), password (generated/secure).
3. **Networking**:
   - Public IP: New dynamic (41.87.210.208 via NIC appvmnic2).
   - Network Security Group (NSG): Default inbound rules; RDP (TCP port 3389) allowed (source: Any—tighten for prod).
   - VNet/Subnet: vnet-centralindia/snet-centralindia-1 | Private IP: 172.16.0.4.
4. **Deploy & Connect**:
   - VM provisioned in ~5 minutes (Created: Nov 11, 2025, 12:57 PM UTC).
   - Downloaded RDP file from Portal, connected from my Windows machine—successful login (Agent status: Ready, Version 2.7.4149.11172).
   - Inside VM: Verified setup (e.g., ran `systeminfo` in Command Prompt to confirm OS and uptime).

**Commands Used** (Portal-based for this intro lab; Azure CLI next):
- None yet—wizard clicks in Portal. Future: `az vm create --resource-group FirstVM --name AppVM --image Win2025Datacenter --size Standard_B1s --admin-username [user]`.

## Challenges & Fixes
- No major issues—smooth deploy. Note: Default NSG allowed RDP out-of-box; in real SRE scenarios, I'd lock to specific IPs.
- Cost Note: B1s ~$0.008/hour—stopped VM post-lab to pause billing (no auto-shutdown enabled).

## SRE Tie-In
Hands-on with secure boot (vTPM enabled) and NSG basics—critical for reliable, hardened infra. Ties to observability: Health monitoring disabled here, but I'd enable Azure Monitor for alerts in prod.

## Screenshots
*(Upload your PNGs to /screenshots/ folder, then add links like below—see Step 3.)*
![VM Overview - Running Status](./screenshots/vm-overview.png)
![Basics Tab - Config Details](./screenshots/vm-basics.png)
![Networking Tab - IP & NSG Rules](./screenshots/vm-networking.png)
![RDP Connection Inside VM](./screenshots/rdp-success.png) *(If you have this; otherwise, skip.)*

## Cleanup
- Stopped VM via Portal (Overview > Stop) to halt costs.
- Optional full delete: Resource Groups > FirstVM > Delete (nukes VM + all).
- Total lab time: ~45 mins. Zero lingering charges.

---
*Completed: Nov 11, 2025 | Reflections: Quick win on provisioning—excited for networking labs. Next: Azure Storage.*
