# Lab 11: Zero-Downtime Deployment using Azure App Service Deployment Slots (Blue-Green Deployment Pattern)

[![Production After Swap – Version 1 Live](https://raw.githubusercontent.com/PunitAnand-SRE/az-104-labs/main/11-Zero-Downtime%20Deployment%20using%20Azure%20App%20Service%20Deployment%20Slots%20(Blue-Green%20Deployment%20Pattern)/screenshots/09-post-swap-verification-production-now-shows-version-1.png)](https://raw.githubusercontent.com/PunitAnand-SRE/az-104-labs/main/11-Zero-Downtime%20Deployment%20using%20Azure%20App%20Service%20Deployment%20Slots%20(Blue-Green%20Deployment%20Pattern)/screenshots/09-post-swap-verification-production-now-shows-version-1.png)

---

## Objective
Implement a **true production-grade blue-green deployment** with **zero downtime** and **instant rollback** using **Azure App Service Deployment Slots** — the same pattern used by Microsoft, Netflix, Adobe, and other enterprise-grade platforms.

---

## Why this lab matters for SRE / DevOps
- Guarantees **100% uptime** during releases  
- Supports **instant rollback (< 10s)**  
- Enables A/B testing, canary releases, feature flags  
- Required in regulated environments (PCI-DSS, HIPAA, SOC2)  
- Common real-world interview scenario for Azure/Azure DevOps roles

---

## Architecture Flow

```text
                ┌──────────────────────────────────────────────┐
                │               Production Slot                 │ ← User Traffic
                │ werapp-dev-cind.azurewebsites.net            │
                └──────────────────────────────────────────────┘
                                 ↑        ↔ Instant Swap (Zero Downtime)
                                 │
                ┌──────────────────────────────────────────────┐
                │                 Staging Slot                  │
                │ werapp-dev-cind-staging.azurewebsites.net    │
                └──────────────────────────────────────────────┘
Step-by-step walkthrough (with images)
1. Azure App Service created (Linux + PHP)
View the Web App overview (name werapp) and configuration details in the portal.



2. Deploy an initial (simple) PHP app to Production via SSH
SSH into the Web App console and create index.php in /home/site/wwwroot.

bash
Copy code
# sample commands used in the lab
cd /home/site/wwwroot
nano index.php
index.php (initial production version):

php
Copy code
<?php
echo "<h1>Hello from Azure Web Apps (PHP)!</h1>";
?>
Screenshot of SSH session and the deployed file:


Verify the production app is live by opening the web app URL in a browser:


3. Scale up to Standard S1 (required for deployment slots)
Deployment slots are not available on Free/Basic tiers — scale the App Service Plan to Standard S1.


4. Create a Staging deployment slot
Create a new slot named staging and clone production configuration (app settings, connection strings as needed).


Confirmation that the staging slot was created and its details:


5. Deploy a new app version (v1) to Staging and test
SSH into the staging slot (same steps as production) and update index.php to show Version 1.

index.php (staging - Version 1):

php
Copy code
<?php
echo "<h1>Web Application - Version 1</h1>";
echo "<p>This is a simple PHP application running on Azure App Service - Version 1</p>";
?>
After deployment, verify the staging site in the browser:


6. Perform the swap (Staging → Production) — zero downtime
From the App Service portal, perform a swap operation (staging → production). Azure pre-warms staging to avoid cold starts, then atomically swaps site content and configuration mappings.

Swap in progress / swap UI screenshot:


7. Post-swap verification
After swap completes, the Production URL now serves Version 1. The previous production content is now in staging — providing an immediate rollback point.



Instant rollback (career-saving scenario)
If Version 1 contains a critical bug after swap:

Open Deployment slots in the App Service blade.

Click Swap again (swap back).

The old, working version returns to Production in < 10 seconds.

No user impact, no complex rollback scripts — just swap again.

Key benefits demonstrated
Feature	Real-world advantage
Zero-downtime swap	Users don’t see errors or downtime during deploy
Instant rollback	One-click revert to previous working version
Pre-warmed instances	No cold-start latency after swap
Isolated testing URL	QA/UAT on staging without affecting production
Configuration isolation	Test different connection strings safely per slot
Traffic routing support	Canary/traffic split via App Gateway / Front Door

Azure features used
Azure App Service (Linux, PHP 8.4)

Deployment Slots + Swap operation

Standard S1 App Service Plan (required for slots)

Built-in SSH Console for file edits & quick tests

Zero-downtime traffic routing and pre-warming

AZ-104 exam relevance
Directly exercises: Deploy and configure Azure App Service — specifically Deployment Slots & Swap.

Tags
Azure • App Service • Deployment Slots • Blue-Green Deployment • Zero Downtime • DevOps • SRE • CI/CD • Rollback • AZ-104

Author
Punit Anand – Azure | DevOps | SRE
Turning labs into production-grade muscle, one deployment at a time.
