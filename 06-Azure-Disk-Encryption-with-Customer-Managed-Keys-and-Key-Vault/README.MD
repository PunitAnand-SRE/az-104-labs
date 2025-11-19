# Azure Disk Encryption with Customer-Managed Keys and Key Vault

This lab demonstrates the implementation of **customer-managed disk encryption** for Azure Virtual Machines (VMs) using **Azure Key Vault** and **Disk Encryption Sets**. It covers securing data at rest on both Linux and Windows VMs, transitioning from platform-managed to customer-managed keys for enhanced control, compliance, and security.

## Overview

### Why This Lab?
- **Security Enhancement**: Azure managed disks are encrypted by default with platform-managed keys (PMKs), but customer-managed keys (CMKs) provide granular control over key lifecycle (e.g., rotation, auditing) and meet regulatory requirements (e.g., GDPR, HIPAA).
- **Key Benefits**:
  - Isolate keys in a dedicated vault to prevent unauthorized access.
  - Use Disk Encryption Sets (DES) to apply encryption across multiple resources efficiently.
  - Support for both OS and data disks on Linux (dm-crypt/LUKS) and Windows (BitLocker) VMs.
- **Real-World Application**: Ideal for workloads handling sensitive data, such as financial apps or healthcare systems, where you need to revoke keys instantly or integrate with on-premises HSMs.
- **Learning Outcomes**:
  - Configure Azure Key Vault with RBAC permissions.
  - Create and manage encryption keys.
  - Apply DES to existing VMs without downtime.
  - Verify encryption transparency.

### High-Level Architecture
1. **Azure Key Vault**: Stores CMKs securely.
2. **Disk Encryption Set (DES)**: Links CMK to disks/VMs.
3. **VM Disks**: Encrypted using the DES-referenced key.
4. **Access Control**: RBAC roles ensure least-privilege access.

## Prerequisites
- Azure subscription with Contributor/Owner permissions.
- Existing Linux and Windows VMs (e.g., "LinuxAppVM" and "AppVM") with unencrypted managed disks.
- Azure Portal access.
- Basic familiarity with Azure RBAC and Key Vault.

**Estimated Time**: 45-60 minutes.

## Step-by-Step Implementation

### Step 1: Create and Configure Azure Key Vault
Create a Key Vault to store encryption keys. Enable soft-delete and purge protection for recovery.

- Navigate to **Azure Portal > Key Vaults > Create**.
- Resource Group: `FirstVM` (or your existing group).
- Name: `keyVault69`.
- Region: `Central India`.
- Pricing Tier: Standard.
- Enable: Soft-delete and Purge protection.

![Key Vault Overview Dashboard](https://raw.githubusercontent.com/PunitAnand-SRE/az-104-labs/main/06-Azure-Disk-Encryption-with-Customer-Managed-Keys-and-Key-Vault/screenshots/key-vault69-overview-dashboard..png
)

**Key Concept**: Key Vault isolates secrets/keys from compute resources, logging all access for auditing.

### Step 2: Resolve Permissions for Key Management
By default, even owners lack permissions to create keys. Assign RBAC roles.

- In Key Vault > **Access control (IAM) > Add role assignment**.
- Role: `Key Vault Administrator`.
- Assign to: Your user account (e.g., `Punit Anand`).
- Scope: This resource.
- Save and wait 5-10 minutes for propagation.

![RBAC Role Assignment for Key Vault Admin](https://raw.githubusercontent.com/PunitAnand-SRE/az-104-labs/main/06-Azure-Disk-Encryption-with-Customer-Managed-Keys-and-Key-Vault/screenshots/rbac-role-assignment-for-key-vault-admin.png
)

**Troubleshooting Note**: If you encounter authorization errors during key creation, verify role propagation via Activity Log.

### Step 3: Create Encryption Key for Linux VM (dataKey)
With permissions in place, generate a CMK.

- In Key Vault > **Keys > Generate/Import**.
- Name: `dataKey`.
- Key Type: RSA.
- Key Size: 2048 bits.
- Create.

![Successful Creation of dataKey in Vault](https://raw.githubusercontent.com/PunitAnand-SRE/az-104-labs/main/06-Azure-Disk-Encryption-with-Customer-Managed-Keys-and-Key-Vault/screenshots/successful-creation-of-datakey-in-vault.png
)

**Key Concept**: Keys are versioned (e.g., `dataKey-abc123`). Rotate by creating new versions without disrupting encryption.

### Step 4: Create Disk Encryption Set (DES) for Linux Disk
DES bundles the CMK for scalable application.

- Navigate to **Azure Portal > Disk Encryption Sets > Create**.
- Resource Group: `FirstVM`.
- Name: `DiskEncryptionSet-2025118675` (auto-generated or custom).
- Key Vault: Select `keyVault69`.
- Key: Select `dataKey`.
- Grant permissions: Add `Disk Encryption Key Encipherment` role to the vault's managed identity.

![Disk Encryption Set Creation with Key URI](https://raw.githubusercontent.com/PunitAnand-SRE/az-104-labs/main/06-Azure-Disk-Encryption-with-Customer-Managed-Keys-and-Key-Vault/screenshots/disk-encryption-set-creation-with-key-uri.png
)

**Key Concept**: DES supports auto-key rotation (disabled here for manual control). It uses the key URI (e.g., `https://keyvault69.vault.azure.net/keys/dataKey/...`) for secure reference.

### Step 5: Apply DES to Linux VM Disk
Encrypt an existing data disk using the DES.

- Navigate to **Disks > LinuxAppVM-disk1... > Encryption**.
- Encryption Type: Customer-managed.
- Disk Encryption Set: Select your DES.
- Save.

![Applying Encryption Set to LinuxAppVM Disk](https://raw.githubusercontent.com/PunitAnand-SRE/az-104-labs/main/06-Azure-Disk-Encryption-with-Customer-Managed-Keys-and-Key-Vault/screenshots/applying-encryption-set-to-linuxappvm-disk.png
)

**Key Concept**: Encryption is incremental—no VM restart needed for data disks. Azure handles wrap/unwrap operations via Key Vault.

### Step 6: Create Second Key for Windows VM (diskKey)
Repeat key creation for OS-specific isolation.

- In Key Vault > **Keys > Generate/Import**.
- Name: `diskKey`.
- Same specs as `dataKey`.
- Create.

![Creation of diskKey for Windows Encryption](https://raw.githubusercontent.com/PunitAnand-SRE/az-104-labs/main/06-Azure-Disk-Encryption-with-Customer-Managed-Keys-and-Key-Vault/screenshots/creation-of-diskkey-for-windows-encryption.png
)

### Step 7: Update Key Vault Access Configuration
Enable service access for Disk Encryption.

- In Key Vault > **Settings > Access configuration**.
- Permission Model: Azure role-based access control (recommended).
- Resource Access: Check `Azure Disk Encryption for volume encryption`.
- Save.

![Key Vault Access Configuration for Disk Encryption](https://raw.githubusercontent.com/PunitAnand-SRE/az-104-labs/main/06-Azure-Disk-Encryption-with-Customer-Managed-Keys-and-Key-Vault/screenshots/key-vault-access-configuration-for-disk-encryption.png
)

**Key Concept**: This grants the Azure Disk Encryption service principal permissions like `get`, `wrapKey`, and `unwrapKey` without broad vault access.

### Step 8: Encrypt Windows VM Using diskKey
Apply encryption to the OS disk via VM extension.

- Navigate to **Virtual Machines > AppVM > Extensions + applications > Add**.
- Extension: `Azure Disk Encryption`.
- Key Vault Resource ID: URI of `keyVault69`.
- Key Encryption Key URL: URI of `diskKey`.
- Create (VM may restart).

![Azure Disk Encryption Extension on AppVM](https://raw.githubusercontent.com/PunitAnand-SRE/az-104-labs/main/06-Azure-Disk-Encryption-with-Customer-Managed-Keys-and-Key-Vault/screenshots/azure-disk-encryption-extension-on-appvm.png
)

**Key Concept**: The extension runs BitLocker inside the guest OS. For OS disks, a restart is required, but data remains accessible post-encryption.

### Step 9: Verify Encryption
Connect to the Windows VM and confirm no disruption.

- RDP to `AppVM`.
- Open File Explorer: Drives (C:, D:, E:) should be accessible.
- PowerShell Check: `manage-bde -status C:` (shows "Protection Status: On").

![Verification of Encryption via RDP on AppVM](https://raw.githubusercontent.com/PunitAnand-SRE/az-104-labs/main/06-Azure-Disk-Encryption-with-Customer-Managed-Keys-and-Key-Vault/screenshots/verification-of-encryption-via-rdp-on-appvm.png
)

**Key Concept**: Encryption is transparent—users see normal files, but data is AES-256 encrypted at the block level.

## Verification and Testing
- **Key Usage Logs**: In Key Vault > **Events**, filter for "wrapKey" operations during encryption.
- **Disk Status**: In Disk > **Encryption**, confirm "Customer-managed" with your DES.
- **Cross-OS Test**: Repeat for Linux VM: SSH in and run `lsblk` or `cryptsetup status` to verify LUKS.

| Component | Status Check | Expected Outcome |
|-----------|--------------|------------------|
| Key Vault | Keys list | `dataKey` and `diskKey` enabled |
| DES | Associated resources | Linked to VM disks |
| VM Disks | Encryption type | Customer-managed via DES |
| VM Access | RDP/SSH | Full access, no errors |

## Cleanup
To avoid costs, delete resources in this order:
1. Disable encryption on VMs/disks.
2. Delete VM extensions.
3. Delete DES and keys (purge from soft-delete).
4. Delete Key Vault.

**Azure CLI Example**:
```bash
az keyvault delete --name keyVault69 --resource-group FirstVM
az group delete --name FirstVM --yes --no-wait
```

## Key Learnings and Best Practices
- **Least Privilege**: Use scoped RBAC roles; avoid legacy access policies.
- **Automation**: For production, use ARM templates or Terraform to deploy DES and keys.
- **Monitoring**: Enable Key Vault diagnostics to Log Analytics for auditing key access.
- **Cost Considerations**: Key Vault operations (~$0.03/10k), no extra VM/disk costs for encryption.
- **Limitations**: Not supported on Premium SSD v2 or Ultra disks yet; test in dev before prod.

This setup ensures data sovereignty and resilience. For extensions, explore integrating with Azure AD for key attestation or multi-region replication.

**Last Updated**: November 19, 2025  
**Author**: Punit Anand
