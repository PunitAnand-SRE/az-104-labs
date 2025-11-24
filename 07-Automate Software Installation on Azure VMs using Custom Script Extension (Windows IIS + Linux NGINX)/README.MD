# Lab 07: Automate Software Installation on Azure VMs using Custom Script Extension (Windows IIS + Linux NGINX)

## Overview and Purpose

This lab demonstrates the use of **Azure Custom Script Extension (CSE)** to automate post-deployment configuration and software installation on Azure Virtual Machines (VMs). CSE is a powerful VM extension that downloads and executes scripts (PowerShell for Windows, Bash for Linux) on VMs without manual intervention. It's ideal for bootstrapping environments, installing applications like web servers, and ensuring consistent configurations across deployments.

### Why This Lab?
- **AZ-104 Exam Relevance**: Aligns with exam objectives under "Manage Azure identities and governance" and "Implement and manage storage" (via Azure Storage for script hosting), but primarily "Deploy and manage Azure compute resources" – specifically, configuring VMs with extensions for automation. CSE is a key topic for automating VM provisioning, which appears in 10-15% of questions (e.g., post-deployment tasks, idempotent scripting, and troubleshooting extensions).
- **Real-World Value**: In production, manual logins to VMs for software installs (e.g., IIS for .NET apps or NGINX for Linux web apps) is error-prone and non-scalable. CSE enables Infrastructure-as-Code (IaC) principles, integrates with ARM templates/CI/CD pipelines (e.g., Azure DevOps), and supports hybrid/multi-cloud setups. This lab shows cross-platform automation (Windows/Linux), secure script storage in Azure Blob, and verification via public endpoints – skills for SRE/DevOps roles.
- **Learning Outcomes**:
  - Understand CSE architecture: Scripts stored in Azure Storage (public/protected settings), executed under LocalSystem (Windows) or root (Linux).
  - Best Practices: Idempotent scripts, managed identities for security, logging for troubleshooting.
  - Cost Awareness: ~$0.01-0.05/hour for small VMs + storage; cleanup prevents charges.
- **Lab Duration**: 45-60 minutes (including verification).
- **Difficulty**: Intermediate (assumes basic Azure Portal/CLI familiarity from prior labs).

**Key Concepts**:
- **CSE Version**: Used v1.10 (Windows) and v2.1 (Linux) – supports managed identities, timestamp for re-runs.
- **Idempotency**: Scripts check if software is installed before running (prevents errors on re-execution).
- **Security**: Public settings (fileUris) are unencrypted; protected settings (commandToExecute, keys) are encrypted. Used storage access keys here; prefer managed identities in prod.
- **Limitations**: 90-min runtime; no reboots in scripts (use scheduled tasks); one extension instance per VM.

## Prerequisites
- Azure Subscription with Contributor role (or Owner).
- Resource Group: `App-Group` (Central India; reuse from prior labs).
- Azure Portal access.
- Basic PowerShell/Bash knowledge.
- NSG Rules: Allow inbound TCP/80 (HTTP) for web verification.
- Costs: ~$0.10-0.20 total (VMs + storage); monitor via Cost Management.

## High-Level Architecture

```
[Azure Subscription]
    |
[Resource Group: App-Group (Central India)]
    ├── [Storage Account: mydatastore69 (LRS, Hot Tier, Public Access Enabled)]
    │   └── [Container: scripts (Public Read)]
    │       ├── setup-iis.ps1 (PowerShell: Install IIS + Custom HTML)
    │       └── setup-nginx.sh (Bash: Install NGINX + Custom HTML)
    |
    ├── [Windows VM: WebVM01 (Win Server 2025, Standard B1s, Public IP)]
    │   └── [Extension: CustomScriptExtension (Runs setup-iis.ps1)]
    │       └── [NSG: Allow TCP/80]
    |
    └── [Linux VM: WebVM02 (Ubuntu 24.04, Standard B1s, Public IP)]
        └── [Extension: CustomScript (Runs setup-nginx.sh)]
            └── [NSG: Allow TCP/80]

[Browser] --> HTTP (Port 80) --> Public IPs --> VMs --> IIS/NGINX Default Pages (Customized)
```

- **Data Flow**: Scripts uploaded to Blob Storage → CSE downloads/executes on VM boot → Web servers start → Verify via browser.
- **Tools Used**: Azure Portal (for creation/upload), PowerShell/Bash (scripting), Browser (verification).

## Step-by-Step Instructions

### Step 1: Create Azure Storage Account for Script Hosting
CSE requires scripts to be accessible (e.g., via Blob Storage). We use a General Purpose v2 account with public access for simplicity (use private + SAS/managed identity in prod).

1. In Azure Portal, search for **Storage accounts** > Create.
2. **Basics**:
   - Resource Group: `App-Group`.
   - Storage account name: `mydatastore69` (unique; auto-generates if needed).
   - Location: Central India.
   - Performance: Standard.
   - Redundancy: Locally-redundant storage (LRS) – cost-effective for non-critical scripts.
3. **Security**:
   - Enable "Allow storage account key access" (for this lab).
   - Blob public access: Enabled from all networks (for easy download; restrict in prod).
4. **Advanced** > Data protection: Enable soft delete (7 days) for blobs.
5. Review + Create > Deploy.
6. Navigate to the account > **Overview** to confirm (LRS, Hot tier, Public access enabled).

**Screenshot**: Storage account overview showing LRS, Hot tier, and public access.

![Storage Account Overview](https://raw.githubusercontent.com/PunitAnand-SRE/az-104-labs/main/07-Automate%20Software%20Installation%20on%20Azure%20VMs%20using%20Custom%20Script%20Extension%20(Windows%20IIS%20%2B%20Linux%20NGINX)/screenshots/storage-account-overview.png)

### Step 2: Create Container and Upload IIS PowerShell Script
1. In the storage account > **Containers** > + Container > Name: `scripts` > Public access level: Blob (allows anonymous read).
2. Create > Navigate to container.
3. **Upload** > Select local file `setup-iis.ps1` (create if needed; see script below).
4. Verify upload: List shows `setup-iis.ps1` (Hot tier, Block blob).

**Script: setup-iis.ps1** (Idempotent; installs IIS if not present, creates custom HTML):
```powershell
# Install IIS Feature (idempotent: skips if already installed)
Install-WindowsFeature -Name Web-Server -IncludeManagementTools

# Define IIS root path
$path = "C:\inetpub\wwwroot"

# Create simple HTML page
$html = @"
<!DOCTYPE html>
<html>
<head><title>Custom Script Extensions Demo</title></head>
<body><h1>We now have Internet Information Services running on the machine</h1></body>
</html>
"@

# Save as default.html
Set-Content -Path "$path\default.html" -Value $html -Force
```

**Screenshot**: Container showing uploaded IIS script with content preview.

![Container with IIS Script and Content](https://raw.githubusercontent.com/PunitAnand-SRE/az-104-labs/main/07-Automate%20Software%20Installation%20on%20Azure%20VMs%20using%20Custom%20Script%20Extension%20(Windows%20IIS%20%2B%20Linux%20NGINX)/screenshots/container-with-iis-script-and-content.png)

### Step 3: Create Windows VM and Configure Custom Script Extension
1. Search **Virtual machines** > Create > Azure virtual machine.
2. **Basics**:
   - Resource Group: `App-Group`.
   - VM Name: `WebVM01`.
   - Region: Central India.
   - Image: Windows Server 2025 Datacenter: Azure Edition – Gen2.
   - Size: Standard_B1s (1 vCPU, 1 GiB RAM – cost-optimized).
   - Admin: Your username/password.
3. **Networking**: Default VNet/Subnet; Public IP: Enabled; NIC: Default; Delete public IP on VM delete: No.
4. **Extensions + apps** > **Add extensions** > Search "Custom Script" > Select **CustomScriptExtension** > Create (uses latest version).
5. In Extension settings:
   - **Storage Account**: Select `mydatastore69`.
   - **Container**: `scripts`.
   - **File**: `setup-iis.ps1`.
6. Review + Create > Deploy (~5-10 min).
7. Post-Deploy: VM > Extensions > Confirm "CustomScriptExtension" Provisioning state: Succeeded.

**Screenshots**:
- Extension configuration wizard selecting script.

![Windows VM Selecting IIS Script](https://raw.githubusercontent.com/PunitAnand-SRE/az-104-labs/main/07-Automate%20Software%20Installation%20on%20Azure%20VMs%20using%20Custom%20Script%20Extension%20(Windows%20IIS%20%2B%20Linux%20NGINX)/screenshots/windows-vm-selecting-iis-script.png)

- Extension status post-install.

![Windows VM Extension Installed](https://raw.githubusercontent.com/PunitAnand-SRE/az-104-labs/main/07-Automate%20Software%20Installation%20on%20Azure%20VMs%20using%20Custom%20Script%20Extension%20(Windows%20IIS%20%2B%20Linux%20NGINX)/screenshots/windows-vm-extension-installed.png)

### Step 4: Verify IIS Installation on Windows VM
1. VM > Overview > Copy Public IP (e.g., 4.224.155.86).
2. Browser: `http://<Public-IP>` > See IIS welcome with custom message: "We now have Internet Information Services running on the machine".
3. (Optional) RDP to VM > Check `C:\inetpub\wwwroot\default.html` and IIS Manager.

**Screenshot**: Browser proof of IIS running.

![Windows IIS Running Proof](https://raw.githubusercontent.com/PunitAnand-SRE/az-104-labs/main/07-Automate%20Software%20Installation%20on%20Azure%20VMs%20using%20Custom%20Script%20Extension%20(Windows%20IIS%20%2B%20Linux%20NGINX)/screenshots/windows-iis-running-proof.png)

### Step 5: Upload NGINX Bash Script to Container
1. Back to Storage > Container `scripts` > Upload `setup-nginx.sh` (see script below).
2. Verify: Container now shows both files.

**Script: setup-nginx.sh** (Idempotent; updates packages, installs NGINX, custom HTML, enables on boot):
```bash
#!/bin/bash

# Update packages and install NGINX (idempotent via apt)
apt-get update
apt-get install -y nginx

# Create simple index.html
cat > /var/www/html/index.html << EOF
<!DOCTYPE html>
<html>
<head><title>Custom Script Extensions Demo</title></head>
<body><h1>We now have NGINX running as a web server</h1></body>
</html>
EOF

# Ensure NGINX starts on boot
systemctl enable nginx
systemctl start nginx
```

**Screenshot**: Container with both scripts uploaded.

![Container Both Scripts Uploaded](https://raw.githubusercontent.com/PunitAnand-SRE/az-104-labs/main/07-Automate%20Software%20Installation%20on%20Azure%20VMs%20using%20Custom%20Script%20Extension%20(Windows%20IIS%20%2B%20Linux%20NGINX)/screenshots/container-both-scripts-uploaded.png)

**Screenshot**: NGINX script content.

![Setup NGINX SH Script Content](https://raw.githubusercontent.com/PunitAnand-SRE/az-104-labs/main/07-Automate%20Software%20Installation%20on%20Azure%20VMs%20using%20Custom%20Script%20Extension%20(Windows%20IIS%20%2B%20Linux%20NGINX)/screenshots/setup-nginx-sh-script-content.png)

### Step 6: Create Linux VM and Configure Custom Script Extension
1. Virtual machines > Create > Azure virtual machine.
2. **Basics**:
   - Resource Group: `App-Group`.
   - VM Name: `WebVM02`.
   - Region: Central India.
   - Image: Ubuntu Server 24.04 LTS – Gen2.
   - Size: Standard_B1s.
   - Authentication: SSH public key (generate if needed).
3. **Networking**: Default; Public IP: Enabled.
4. **Extensions + apps** > Add "Custom Script for Linux" > Create.
5. Settings:
   - Storage Account: `mydatastore69`.
   - Container: `scripts`.
   - File: `setup-nginx.sh`.
6. Review + Create > Deploy (~5-10 min).
7. VM > Extensions > Confirm "CustomScript" Succeeded.

**Screenshot**: Extension wizard for Linux.

![Linux VM Selecting NGINX Script](https://raw.githubusercontent.com/PunitAnand-SRE/az-104-labs/main/07-Automate%20Software%20Installation%20on%20Azure%20VMs%20using%20Custom%20Script%20Extension%20(Windows%20IIS%20%2B%20Linux%20NGINX)/screenshots/linux-vm-selecting-nginx-script.png)

### Step 7: Verify NGINX Installation on Linux VM
1. VM > Overview > Copy Public IP (e.g., 74.225.202.25).
2. Browser: `http://<Public-IP>` > See custom message: "We now have NGINX running as a web server".
3. (Optional) SSH to VM > `systemctl status nginx` and check `/var/www/html/index.html`.

**Screenshot**: Browser proof of NGINX running.

![Linux NGINX Running Proof](https://raw.githubusercontent.com/PunitAnand-SRE/az-104-labs/main/07-Automate%20Software%20Installation%20on%20Azure%20VMs%20using%20Custom%20Script%20Extension%20(Windows%20IIS%20%2B%20Linux%20NGINX)/screenshots/linux-nginx-running-proof.png)

## Troubleshooting
- **Extension Failed**: Check VM > Extensions > Status > Logs (Windows: `C:\WindowsAzure\Logs\Plugins`; Linux: `/var/log/azure/custom-script`). Common: Network blocks to storage, script syntax errors.
- **Script Not Running**: Ensure idempotency; add `timestamp: 1` in settings to force re-run.
- **Public Access Denied**: Verify Blob container public level; test download URL.
- **VM Not Reachable**: NSG inbound rule for TCP/80; public IP associated.
- **Costs**: Use Azure Cost Management; stop/delete VMs post-lab.

## Cleanup
1. Delete VMs: Search Virtual machines > Select both > Delete (includes disks, IPs).
2. Delete Storage Account: Storage accounts > `mydatastore69` > Delete.
3. Empty Resource Group: App-Group > (Optional) Delete unused resources.
4. Verify: No running resources in Subscription > Cost Management.

**Total Lab Cost**: <$0.50 (if cleaned promptly).

## Exam Tips (AZ-104)
- **Key Topics**: Know CSE vs. DSC (CSE for simple scripts; DSC for complex configs). Use PowerShell/CLI examples (e.g., `Set-AzVMExtension`).
- **Scenarios**: Expect questions on deploying extensions via ARM/CLI, secure settings (protected vs. public), and troubleshooting (logs, status).
- **Practice**: Replicate with ARM templates; add managed identity. Review MS Learn modules on VM extensions.
- **Common Pitfalls**: Forgetting NSG rules; non-idempotent scripts causing failures on scale sets.

## References
- [Azure Custom Script Extension for Windows](https://learn.microsoft.com/en-us/azure/virtual-machines/windows/extensions-customscript)
- [Azure Custom Script Extension for Linux](https://learn.microsoft.com/en-us/azure/virtual-machines/linux/extensions-customscript)
- [Tutorial: Automate IIS on Windows VM](https://learn.microsoft.com/en-us/azure/virtual-machines/windows/tutorial-automate-vm-deployment)
- [Tutorial: Install NGINX on Linux VM](https://learn.microsoft.com/en-us/azure/virtual-machines/linux/quick-create-portal#add-the-custom-script-extension-to-your-vm)
- [AZ-104 Study Guide](https://learn.microsoft.com/en-us/credentials/certifications/resources/study-guides/az-104)

---

*Lab Completed: November 20, 2025. Author: Punit Anand-SRE. For questions, open GitHub issue.*
