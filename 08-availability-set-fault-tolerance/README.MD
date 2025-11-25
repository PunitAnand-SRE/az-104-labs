# Lab 08: Implement High Availability using Azure Availability Sets

## Overview and Purpose

This lab demonstrates **Azure Availability Sets**, a fundamental feature for ensuring high availability (HA) of virtual machines (VMs) within a single Azure region. An Availability Set is a logical grouping of VMs that distributes them across **Fault Domains (FDs)** and **Update Domains (UDs)** to protect against unplanned hardware failures and planned maintenance events. In this setup, we deployed two VMs (`WebVM01` and `WebVM02`) into `AvailabilitySet01`, ensuring they are placed in different fault domains (FD 0 and FD 1) for fault tolerance.

### Why This Lab?
- **AZ-104 Exam Relevance**: Directly aligns with the "Deploy and manage Azure compute resources" objective (15-20% of exam), specifically configuring VM availability options like Availability Sets, Zones, and Scale Sets. Expect questions on HA concepts, SLAs (99.95% for 2+ VMs in an Availability Set with managed disks), and troubleshooting (e.g., FD/UD distribution). It's a core topic for demonstrating fault tolerance without advanced features like Zones.
- **Real-World Value**: In production, single VMs risk downtime from rack failures (FD) or Azure updates (UD). Availability Sets provide cost-effective HA for multi-tier apps (e.g., web/app servers) without the overhead of Availability Zones (cross-datacenter) or Scale Sets (auto-scaling). This lab builds on Lab 07's VMs, showing how to group them for resiliency – essential for SRE/DevOps roles managing mission-critical workloads.
- **Learning Outcomes**:
  - Understand FD (hardware isolation: power/network) vs. UD (maintenance sequencing).
  - Achieve 99.95% SLA by distributing VMs across at least 2 FDs.
  - Best Practices: Create Sets during VM deployment; use with Load Balancer for traffic distribution.
  - Cost Awareness: No extra charge for Availability Sets; pay only for VMs/disks.
- **Lab Duration**: 30-45 minutes (VM creation + verification).
- **Difficulty**: Beginner-Intermediate (builds on VM basics from Labs 01-02).

**Key Concepts**:
- **Fault Domains (FDs)**: Up to 3 isolated hardware groups (e.g., racks); VMs spread to avoid single-point failures.
- **Update Domains (UDs)**: Up to 20 logical groups; Azure staggers reboots during maintenance (e.g., one VM per UD).
- **SLA**: 99.95% uptime for 2+ VMs in a Set with Premium SSD managed disks.
- **Limitations**: Single-region only (use Zones for datacenter failover); max 200 VMs per Set; can't add existing VMs directly (recreate via PowerShell).

## Prerequisites
- Azure Subscription with Contributor role.
- Resource Group: `App-Group` (Central India; reuse from prior labs).
- Existing VMs: `WebVM01` (Windows) and `WebVM02` (Linux) from Lab 07 (or create new B1s instances).
- Azure Portal access.
- NSG: Inbound TCP/80 allowed (for web verification if reusing Lab 07).
- Costs: ~$0.05-0.10/hour for B1s VMs; monitor via Cost Management.

## High-Level Architecture

```
[Azure Subscription]
    |
[Resource Group: App-Group (Central India)]
    |
    ├── [AvailabilitySet01]
    │   ├── Fault Domain 0: WebVM01 (Windows Server 2025, Standard B1s)
    │   └── Fault Domain 1: WebVM02 (Ubuntu 24.04, Standard B1s)
    │
    ├── [Networking: Default VNet/NSG (Allow TCP/80)]
    └── [Storage: Managed Disks (Premium SSD for SLA)]

[Load Balancer (Optional)] --> Traffic Distribution --> VMs in Different FDs/UDs
```

- **Data Flow**: VMs created/deployed into Set → Azure auto-distributes across FDs/UDs → HA during failures/maintenance.
- **Tools Used**: Azure Portal (creation), PowerShell (if adding existing VMs).

## Step-by-Step Instructions

### Step 1: Create the Availability Set (Optional Standalone Step)
While typically created during VM deployment, you can pre-provision:
1. In Azure Portal, search **Availability sets** > Create.
2. **Basics**:
   - Resource Group: `App-Group`.
   - Name: `AvailabilitySet01`.
   - Location: Central India.
   - Fault domains: `2` (minimum for HA).
   - Update domains: `5` (default; balances maintenance).
3. Review + Create > Deploy.
4. Note: This Set will be selected during VM creation.

### Step 2: Create or Redeploy First VM into Availability Set
Use existing VMs from Lab 07 or create new ones. (Note: Direct addition to existing VMs requires PowerShell recreation – see Troubleshooting.)

1. Search **Virtual machines** > Create > Azure virtual machine (or edit existing via PowerShell).
2. **Basics**:
   - Resource Group: `App-Group`.
   - VM Name: `WebVM01` (if new).
   - Region: Central India.
   - Image: Windows Server 2025 Datacenter (or reuse existing).
   - Size: Standard_B1s (1 vCPU, 1 GiB).
   - Authentication: Password (admin user).
3. **Availability options** (key step):
   - Select **Availability set** from dropdown.
   - Choose `AvailabilitySet01` (or create new with 2 FDs, 5 UDs).
4. **Networking**: Default VNet; Public IP: Enabled; NSG: Basic (allow HTTP).
5. **Disks**: Use managed disks (Premium SSD for SLA).
6. Review + Create > Deploy (~5 min).
7. Post-Deploy: VM > Overview > Confirm "Availability set: AvailabilitySet01".

### Step 3: Create or Redeploy Second VM into the Same Availability Set
Repeat for the second VM, ensuring different FD assignment.

1. Virtual machines > Create > Azure virtual machine.
2. **Basics**:
   - VM Name: `WebVM02`.
   - Image: Ubuntu Server 24.04 LTS.
   - Size: Standard_B1s.
   - Authentication: SSH public key.
3. **Availability options**:
   - Select **Availability set** > `AvailabilitySet01`.
   - Azure auto-assigns to FD 1 (different from WebVM01).
4. **Networking/Disks**: Same as Step 2.
5. Review + Create > Deploy (~5 min).
6. VM > Overview > Confirm "Availability set: AvailabilitySet01" + FD: 1.

**Screenshot**: Availability Set overview showing both VMs in different fault domains.

![Availability Set with Two VMs in Different Fault Domains](https://raw.githubusercontent.com/PunitAnand-SRE/az-104-labs/main/08-availability-set-fault-tolerance/screenshots/availability-set-with-two-vms-in-different-fault-domains.png)

### Step 4: Verify Distribution and High Availability
1. Navigate to **AvailabilitySet01** > Overview:
   - Confirm: 2 Fault domains, 5 Update domains, 2 VMs (WebVM01 in FD 0, WebVM02 in FD 1).
   - Colocation status: N/A (no issues).
2. VMs > Virtual machines tab: Both running, different FDs.
3. (Optional) Test HA: Stop one VM via Portal > Confirm other remains available (e.g., via public IP browser check from Lab 07).
4. Resource Group view: All resources (VMs, NSGs, Disks) grouped under App-Group.

## Troubleshooting
- **Can't Add Existing VM**: Portal doesn't support direct addition; use PowerShell to recreate (deallocate VM, detach disks, recreate with Set ID). Script example:
  ```powershell
  # Get existing VM and new Set
  $originalVM = Get-AzVM -ResourceGroupName "App-Group" -Name "WebVM01"
  $availSet = Get-AzAvailabilitySet -ResourceGroupName "App-Group" -Name "AvailabilitySet01"
  
  # Recreate VM config with Set
  $newVM = New-AzVMConfig -VMName $originalVM.Name -VMSize $originalVM.HardwareProfile.VmSize -AvailabilitySetId $availSet.Id
  # Attach OS/Data disks, then Update-AzVM
  ```
- **Same FD Assignment**: Redeploy VM; Azure guarantees distribution for 2+ FDs.
- **SLA Not Met**: Ensure 2+ VMs, managed disks; check via VM > Overview.
- **Maintenance Impact**: Monitor Azure Status for UD reboots; use Scale Sets for auto-healing.
- **Costs**: Stop VMs when idle; delete post-lab.

## Cleanup
1. Deallocate/Stop VMs: Virtual machines > Select both > Stop (saves compute costs).
2. Delete VMs: Select > Delete (includes disks/IPs; confirm).
3. Delete Availability Set: Availability sets > `AvailabilitySet01` > Delete (if empty).
4. Resource Group: App-Group > Delete unused (e.g., NSGs if not shared).
5. Verify: Subscription > Cost Management > No charges.

**Total Lab Cost**: <$0.20 (if cleaned promptly).

## Exam Tips (AZ-104)
- **Key Topics**: Differentiate Availability Sets (intra-datacenter HA) vs. Zones (cross-datacenter) vs. Scale Sets (HA + scaling). Know FD/UD limits, SLA requirements.
- **Scenarios**: Questions on selecting options during VM creation; PowerShell for migration; combining with Load Balancer.
- **Practice**: Deploy with ARM templates; simulate FD failure (stop VM). Review MS Learn: "Availability options for Azure VMs".
- **Common Pitfalls**: Forgetting managed disks for SLA; creating Sets post-VM (requires recreation).

## References
- [Availability options for Azure Virtual Machines](https://learn.microsoft.com/en-us/azure/virtual-machines/availability)
- [Change a VM's availability set using PowerShell](https://learn.microsoft.com/en-us/azure/virtual-machines/windows/change-availability-set)
- [AZ-104: Deploy and manage compute resources](https://learn.microsoft.com/en-us/training/paths/az-104-manage-compute-resources/)
- [Study guide for AZ-104](https://learn.microsoft.com/en-us/credentials/certifications/resources/study-guides/az-104)
- [Step-by-Step: Deploy VMs in Availability Set](https://k21academy.com/microsoft-azure/architect/azure-add-vm-to-availability-set/)

---

*Lab Completed: November 25, 2025. Author: Punit Anand-SRE. For questions, open GitHub issue.*
