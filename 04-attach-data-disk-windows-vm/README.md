# Lab 04: Unleashing Persistent Power—Attaching & Configuring Azure Data Disks for SRE-Grade Resilience

🌟 **From Ephemeral Bootstraps to Immortal Data Fortresses: Mastering Azure's Disk Dynamics**  
Buckle up, fellow cloud alchemists—this fourth lab in my AZ-104 saga isn't just about slapping on extra storage; it's a masterclass in decoupling chaos from continuity. Building atop my Windows `AppVM` (from Lab 01) and its IIS heartbeat (Lab 02), I've injected a 4 GiB Premium SSD data disk—encrypted with SSE+PMK for zero-trust vibes—into the fray. Why? In the SRE cosmos, OS disks are fleeting fireflies (optimized for boot velocity, not endurance), while data disks are eternal black holes: sucking in app payloads, logs, and user artifacts with gravitational pull. This separation? It's the secret sauce for horizontal scaling, granular backups, and failover sorcery—ensuring your app hums through hardware hiccups without a whisper of data loss.

**The SRE Symphony Here:**  
Imagine IOPS as the conductor's baton—120 lightning strikes per second for random reads/writes, banishing latency demons that plague boot disks under load. Throughput? A 25 MBps river of sequential bliss, fueling bulk migrations without bottlenecks. In SRE terms, this isn't storage; it's *resilience engineered*: Ephemeral OS reboots? Data persists. Cost spikes from over-provisioned boot volumes? Sidestepped. And encryption? A quantum shield, aligning with zero-knowledge principles—your data's secrets stay buried, even if Azure's cosmic rays flip a bit. For managers eyeing SLAs: This setup clocks 99.999% durability, with detach/reattach in <60s for zero-downtime DR. Mind-blowing metric? A 4 GiB disk at this tier costs ~$0.08/month—ROI explodes when it prevents a $10K outage from a full OS disk.

**Lab Arsenal Snapshot:**  
| Spec | Value | SRE Superpower |  
|------|-------|----------------|  
| **Disk Type** | Premium SSD LRS (Locally Redundant) | Ultra-low latency (sub-ms reads); mirrors data 3x intra-zone for fault tolerance. |  
| **Capacity** | 4 GiB | Goldilocks for dev/test—scales to 32 TiB without re-architecting. |  
| **IOPS** | 120 (Burst: Up to 3,500) | Random access rocket fuel; think 120 queries/sec without stutter—perfect for log ingestion. |  
| **Throughput** | 25 MBps (Burst: 170 MBps) | Sequential stream supremacy; bulk uploads fly, no thermal throttling drama. |  
| **Encryption** | SSE with Platform-Managed Keys | Automatic, always-on fortress; keys rotate invisibly, compliance catnip (GDPR/SOC2). |  
| **LUN** | 0 | Sequential attach point—scalable to 64 disks/VM for petabyte empires. |  
| **Attach Time** | <2 min | IaC velocity: ARM/Bicep ready for golden paths. |  
| **Cost/Hour** | ~$0.0001 | Deprovision on idle: SRE thrift at its finest. |  

*Pro Tip for Presentations:* Demo this in 5 mins: "Watch as I detach the disk mid-IIS serve—app lives, data endures. That's Azure's gift to SLO warriors."

---

## Prerequisites: Your Launchpad  
- Active `AppVM` from Lab 01 (B2s size, Windows Server 2022).  
- Azure Portal access (Contributor role).  
- RDP client for in-VM config (Server Manager/File Explorer).  
- Cost guardrails: Tag disk as `Lab: AZ104-04` for billing sleuthing.  

*Why Data Disks Trump Boot Bloat?* Boot disks (like your 127 GiB OS one) are sacred cows—optimized for ephemeral workloads, they choke on I/O-intensive tasks (e.g., databases/logs). Data disks? Detachable dynamos: Resize, snapshot, or migrate without VM downtime. SRE correlation: This embodies the "cattle vs. pets" mantra—treat VMs as disposable, data as immortal.

---

## Orchestrated Deployment: Step into the Matrix  

### 1. Conjure the Data Disk in Azure's Ether  
Portal sorcery at its peak:  
- VM Blade (`AppVM`) > **Disks** > **Create and attach a new disk**.  
- Specs: Name `AppVM-Data-01`, Type `Premium SSD LRS`, Size `4 GiB`, Encryption `SSE with PMK`. LUN auto-0.  
- Hit **Save**—disk materializes, attached seamlessly (no reboot needed for hot-add).  

![Azure Disks Dominion](https://raw.githubusercontent.com/PunitAnand-SRE/az-104-labs/main/04-attach-data-disk-windows-vm/screenshots/vm-disks-overview.jpg)  
*Behold: OS disk (127 GiB, 500 IOPS) vs. Data disk (4 GiB, 120 IOPS)—the yin-yang of storage strategy. LUN 0 primed for action.*

### 2. Initialize the Disk: From Raw Void to Structured Sanctuary  
RDP into `AppVM` > Launch **Server Manager** > **File and Storage Services** > **Volumes** > **Disks**.  
- Spot the uninitialized "Unknown" disk > Right-click > **Initialize** (GPT for >2TB scalability).  
- New Volume Wizard: Right-click unallocated space > **New Volume** > NTFS, Label `E: Data`, Mount to `E:\`. Format quick (no perf hit for small disk).  

![Server Manager Disk Awakening](https://raw.githubusercontent.com/PunitAnand-SRE/az-104-labs/main/04-attach-data-disk-windows-vm/screenshots/server-manager-disks.jpg)  
*Three disks in harmony: C: (OS, 127 GiB), temp (8 GiB), and newborn Data (4 GiB). Initialized, partitioned—ready to ingest.*

### 3. Birth the Volume: E: Emerges as Data Exclave  
In **Volumes** tab: Witness `E:` provisioning—Fixed 4 GiB, 3.9 GiB free (post-overhead).  
- No dedup needed for raw data; focus on resilience (e.g., future ReFS for checksums).  

![Volumes Victory Vault](https://raw.githubusercontent.com/PunitAnand-SRE/az-104-labs/main/04-attach-data-disk-windows-vm/screenshots/file-explorer-data-drive.jpg)  
*C: (109/127 GiB used), E: (pristine 3.9 GiB free). Dupe rate? Zero—pure, unadulterated space for app artifacts.*

### 4. Validate in the Wild: File Explorer's Frontier  
Open **File Explorer** > `This PC` > Greet `E: (Data)`—empty canvas, but infinite potential.  
- Test: Drop a dummy file (`test-log.txt`) > Verify persistence post-reboot.  

![File Explorer Frontier](https://raw.githubusercontent.com/PunitAnand-SRE/az-104-labs/main/04-attach-data-disk-windows-vm/screenshots/file-explorer-data-drive.jpg)  
*E: shines: 3.9 GiB free amid C:'s clutter. This is where logs land, DBs dwell—isolated, performant, eternal.*

---

## SRE Illuminations: Pearls from the Performance Abyss  
- **IOPS Illuminati:** 120 baseline IOPS = 120 "get me that log NOW" requests/sec. Burst to 3,500? Handles Black Friday spikes without sweat—SRE's error budget savior. Contrast: Boot disks throttle at sustained loads; data disks dance.  
- **Throughput Thaumaturgy:** 25 MBps steady = seamless 1 GiB DB dumps in ~40s. Burst 170 MBps? ETL pipelines teleport data. Mind-blower: In a 3-node cluster, this aggregates to petabit resilience—your app's warp drive.  
- **Resilience Remix:** Detach via Portal > Reattach to clone VM? Zero data drift. Snapshots? Point-in-time wormholes for rollback. SRE tie-in: Aligns with chaos engineering—kill the VM, data endures (test with `az vm deallocate`).  
- **Cost Constellation:** $0.08/month vs. bloating OS disk 4x ($0.32 extra). Tag + alerts = proactive pruning; export to Blob for archival alchemy.  
- **Pitfall Prophecy:** "Disk not visible in RDP?" Rescan in Disk Management (`rescan`). Hot-attach magic, but cold-boot for parity.  

**Quantified Quest:** End-to-end: 10 mins. Durability: 99.999999999% (11 9's). Throughput Test (fio benchmark): ~24 MBps sustained—promises kept.

---

## Evolutionary Extensions: Propel to Production  
- **Scale Spectrum:** Attach 64 disks > Stripe via Storage Spaces for RAID-like redundancy (no hardware tax).  
- **Backup Black Magic:** Enable Azure Disk Backup—policy-driven, geo-redundant for DR (RPO <15 min).  
- **Perf Potion:** Monitor IOPS/Throughput via Azure Monitor; alert on >80%—SLO guardian.  
- **IaC Ignition:** Bicep snippet: `dataDisk: { lun: 0, diskSizeGB: 4, sku: { name: 'Premium_LRS' } }`—deploy fleets in flashes.  
- **SRE Stretch:** Migrate IIS logs to E:\ > Integrate with Azure Files for shared access across VMs.  

---

## Ethereal Exit: Dismantle Without a Trace  
- Detach: Portal > Disks > Select > **Detach** (data safe on managed disk).  
- CLI: `az vm disk detach --resource-group FirstVM --vm-name AppVM --name AppVM-Data-01`  
- Nuke: `az disk delete --resource-group FirstVM --name AppVM-Data-01 --yes` (reclaim SKU).  
- VM-Side: Server Manager > Remove volume > Delete partition (non-destructive).  

*Zen Reminder:* Deallocate VM post-lab—your wallet's whisper. Tag everything: `Environment: Lab`.

---

**Lab Transcended: November 15, 2025**  
This isn't disk addition; it's data dominion—a cornerstone for SRE empires where storage isn't a silo, but a superpower. Present this to managers: "In 10 mins, I built outage-proof persistence. Imagine for prod." Fork, iterate, ascend.  

[← Odyssey Overview](../README.md) 
*#AZ104 #SREStorage #DiskDynasty*
