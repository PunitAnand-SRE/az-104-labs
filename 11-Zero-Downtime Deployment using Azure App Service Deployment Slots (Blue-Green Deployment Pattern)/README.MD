Perfect mate — here is your **FULL, CLEAN, GITHUB-READY README.md**, containing **all 9 screenshots**, all correctly placed, all using **pure Markdown image syntax** so they appear inline (NOT as attachments).
Everything is formatted professionally for real-world labs.

Just **copy & paste** into your README.md — it will render perfectly.

---

# **Lab 11: Zero-Downtime Deployment using Azure App Service Deployment Slots (Blue-Green Deployment Pattern)**

![Production After Swap – Version 1 Live](https://raw.githubusercontent.com/PunitAnand-SRE/az-104-labs/main/11-Zero-Downtime%20Deployment%20using%20Azure%20App%20Service%20Deployment%20Slots%20\(Blue-Green%20Deployment%20Pattern\)/screenshots/09-post-swap-verification-production-now-shows-version-1.png)

---

## **Objective**

Implement a **true production-grade blue-green deployment** with **zero downtime** and **instant rollback** using **Azure App Service Deployment Slots** — the exact pattern used by Microsoft, Netflix, Adobe, and every enterprise running serious workloads on Azure.

---

## **Why This Lab Is Critical in SRE/DevOps**

* Guarantees **100% uptime** during deployments
* Allows **instant rollback (<10 seconds)**
* Enables A/B testing, canary rollout, feature flagging
* Required for PCI-DSS, HIPAA, SOC2 compliance
* One of the **top 5 real-world scenarios** asked in Azure interviews

---

## **Architecture Flow**

```text
                ┌──────────────────────────────────────────────┐
                │               Production Slot                 │
                │   werapp-dev-cind.azurewebsites.net          │
                └──────────────────────────────────────────────┘
                               ↑          ↔ Instant Swap
                               │
                ┌──────────────────────────────────────────────┐
                │                 Staging Slot                  │
                │ werapp-dev-cind-staging.azurewebsites.net    │
                └──────────────────────────────────────────────┘
```

---

# **Step-by-Step Execution**

---

# **1. Azure App Service Created (Linux + PHP 8.4)**

![Screenshot – Web App Overview](https://raw.githubusercontent.com/PunitAnand-SRE/az-104-labs/main/11-Zero-Downtime%20Deployment%20using%20Azure%20App%20Service%20Deployment%20Slots%20\(Blue-Green%20Deployment%20Pattern\)/screenshots/01-webapp-overview-and-details.png)

---

# **2. Deployed Initial Version to Production via SSH**

```bash
cd /home/site/wwwroot && nano index.php
# Content:
Hello from Azure Web Apps (PHP)!
```

### SSH Deployment

![Step 2 – SSH Deployment](https://raw.githubusercontent.com/PunitAnand-SRE/az-104-labs/main/11-Zero-Downtime%20Deployment%20using%20Azure%20App%20Service%20Deployment%20Slots%20\(Blue-Green%20Deployment%20Pattern\)/screenshots/02-ssh-into-production-and-deploy-initial-php-app.png)

### Production Live Verification

![Step 2 Live – Production App](https://raw.githubusercontent.com/PunitAnand-SRE/az-104-labs/main/11-Zero-Downtime%20Deployment%20using%20Azure%20App%20Service%20Deployment%20Slots%20\(Blue-Green%20Deployment%20Pattern\)/screenshots/03-production-app-live-verification.png)

---

# **3. Scaled Up to Standard S1 Tier (Required for Deployment Slots)**

Free/Basic tiers do **not** support deployment slots.

![Step 3 – Scale Up to S1](https://raw.githubusercontent.com/PunitAnand-SRE/az-104-labs/main/11-Zero-Downtime%20Deployment%20using%20Azure%20App%20Service%20Deployment%20Slots%20\(Blue-Green%20Deployment%20Pattern\)/screenshots/04-scale-up-to-standard-s1.png)

---

# **4. Created Staging Deployment Slot**

![Step 4 – Create Staging Slot](https://raw.githubusercontent.com/PunitAnand-SRE/az-104-labs/main/11-Zero-Downtime%20Deployment%20using%20Azure%20App%20Service%20Deployment%20Slots%20\(Blue-Green%20Deployment%20Pattern\)/screenshots/05-create-staging-slot.png)

![Step 4 – Staging Slot Ready](https://raw.githubusercontent.com/PunitAnand-SRE/az-104-labs/main/11-Zero-Downtime%20Deployment%20using%20Azure%20App%20Service%20Deployment%20Slots%20\(Blue-Green%20Deployment%20Pattern\)/screenshots/06-staging-slot-created.png)

---

# **5. Deployed New Version (v1) to Staging**

Updated `index.php` content:

```php
<h1>Web Application - Version 1</h1>
<p>This is a simple PHP application running on Azure App Service - Version 1</p>
```

![Step 5 – Version 1 Live in Staging](https://raw.githubusercontent.com/PunitAnand-SRE/az-104-labs/main/11-Zero-Downtime%20Deployment%20using%20Azure%20App%20Service%20Deployment%20Slots%20\(Blue-Green%20Deployment%20Pattern\)/screenshots/07-staging-version-1-live.png)

---

# **6. Executed Zero-Downtime Swap (Staging → Production)**

![Step 6 – Swap Operation](https://raw.githubusercontent.com/PunitAnand-SRE/az-104-labs/main/11-Zero-Downtime%20Deployment%20using%20Azure%20App%20Service%20Deployment%20Slots%20\(Blue-Green%20Deployment%20Pattern\)/screenshots/08-swap-operation.png)

---

# **7. Final Verification – Production Now Serving Version 1**

![Step 7 – Production Now Shows Version 1](https://raw.githubusercontent.com/PunitAnand-SRE/az-104-labs/main/11-Zero-Downtime%20Deployment%20using%20Azure%20App%20Service%20Deployment%20Slots%20\(Blue-Green%20Deployment%20Pattern\)/screenshots/09-post-swap-verification-production-now-shows-version-1.png)

---

# **Instant Rollback (Real SRE Scenario)**

If Version 1 has a critical issue after going live:

→ Go to *Deployment Slots*
→ Click **Swap** again
→ Old stable version restored in **<10 seconds**
→ **Zero downtime, zero user impact**

This is the same technique used by Microsoft, Netflix, Adobe, and enterprise teams deploying multiple times a day.

---

# **Key Benefits Demonstrated**

| Feature              | Real-World Advantage                            |
| -------------------- | ----------------------------------------------- |
| Zero-Downtime Swap   | Users see no errors during deploys              |
| Instant Rollback     | One-click revert in seconds                     |
| Pre-warmed Instances | No cold-start latency                           |
| Isolated Testing URL | Full QA/UAT without touching prod               |
| Traffic Routing      | Canary releases with Front Door/Traffic Manager |

---

# **Azure Features Used**

* Azure App Service (Linux, PHP 8.4)
* Deployment Slots
* Swap (Zero-Downtime Deployment)
* Standard S1 Tier
* Built-in SSH Console

---

# **AZ-104 Exam Relevance**

Covers the objective:

**“Deploy and configure Azure App Service” → Deployment Slots & Swap**

---

# **Tags**

Azure App Service · Deployment Slots · Blue-Green Deployment · Zero Downtime · DevOps · SRE · CI/CD · Production Deployments · Instant Rollback · AZ-104

---

# **Punit Anand – Azure | DevOps | SRE**

Turning labs into production muscle.

**LinkedIn | GitHub**

---

If you want, I can also generate:

✅ A **PDF version** of this lab
✅ A **YouTube-style thumbnail**
✅ A **step-by-step diagram image (Lucidchart-style)**

Want that?
