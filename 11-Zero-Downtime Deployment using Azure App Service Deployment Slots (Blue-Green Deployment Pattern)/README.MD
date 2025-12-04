# Lab 11: Zero-Downtime Deployment using Azure App Service Deployment Slots (Blue-Green Deployment Pattern)

[![Production After Swap – Version 1 Live](https://raw.githubusercontent.com/PunitAnand-SRE/az-104-labs/main/11-Zero-Downtime%20Deployment%20using%20Azure%20App%20Service%20Deployment%20Slots%20(Blue-Green%20Deployment%20Pattern)/screenshots/09-post-swap-verification-production-now-shows-version-1.png)](https://raw.githubusercontent.com/PunitAnand-SRE/az-104-labs/main/11-Zero-Downtime%20Deployment%20using%20Azure%20App%20Service%20Deployment%20Slots%20(Blue-Green%20Deployment%20Pattern)/screenshots/09-post-swap-verification-production-now-shows-version-1.png)

---

## Objective  
Implement a **true production-grade blue-green deployment** with **zero downtime** and **instant rollback** using **Azure App Service Deployment Slots** — the exact pattern used by Microsoft, Netflix, Adobe, and every enterprise running critical workloads on Azure.

---

## Why This Lab is Pure Gold for SRE/DevOps Roles
- Guarantees **100% uptime** during releases  
- Enables **instant rollback in <10 seconds**  
- Supports A/B testing, canary releases, feature flags  
- Required for PCI-DSS, HIPAA, SOC2 compliance  
- One of the **top 5 most-asked real-world scenarios** in Azure interviews  

---

## Architecture Flow

```text
                ┌──────────────────────────────────────────────┐
                │               Production Slot                 │ ← User Traffic
                │     werapp-dev-cind.azurewebsites.net        │
                └──────────────────────────────────────────────┘
                                 ↑        ↔ Instant Swap (Zero Downtime)
                                 │
                ┌──────────────────────────────────────────────┐
                │                  Staging Slot                 │
                │  werapp-dev-cind-staging.azurewebsites.net   │
                └──────────────────────────────────────────────┘
````

---

# Step-by-Step Execution

---

## 1. Azure App Service Created (Linux + PHP 8.4)

**Web App Overview**

![Web App Overview](https://raw.githubusercontent.com/PunitAnand-SRE/az-104-labs/main/11-Zero-Downtime%20Deployment%20using%20Azure%20App%20Service%20Deployment%20Slots%20\(Blue-Green%20Deployment%20Pattern\)/screenshots/01-webapp-overview-and-details.png)

---

## 2. Deployed Initial Version to Production via SSH

```bash
cd /home/site/wwwroot && nano index.php

# Content:
Hello from Azure Web Apps (PHP)!
```

* SSH Deployment
* Production Live – Initial Version

---

## 3. Scaled Up to Standard S1 Tier (Required for Deployment Slots)

* Upgraded from Free → Standard S1
* Deployment slots require **Standard** tier or above

---

## 4. Created Staging Deployment Slot

* Name: `staging`
* Settings cloned from production

Staging Slot Ready.

---

## 5. Deployed New Version (v1) to Staging & Tested

Updated PHP Code:

```php
<h1>Web Application - Version 1</h1>
<p>This is a simple PHP application running on Azure App Service - Version 1</p>
```

**Staging – Version 1 Live**

---

## 6. Executed Zero-Downtime Swap

* Staging → Production
* Azure pre-warms the staging instance
* Swap completed with **zero downtime**

---

## 7. Final Result – Production Now Serves Version 1

Old version automatically moved to staging → **ready for instant rollback**.

![Production After Swap – Version 1 Live](https://raw.githubusercontent.com/PunitAnand-SRE/az-104-labs/main/11-Zero-Downtime%20Deployment%20using%20Azure%20App%20Service%20Deployment%20Slots%20\(Blue-Green%20Deployment%20Pattern\)/screenshots/09-post-swap-verification-production-now-shows-version-1.png)

---

# Instant Rollback (Career-Saving Scenario)

If Version 1 has a critical bug after going live:

1. Go to **Deployment Slots**
2. Click **Swap** again
3. Old version restored in **<10 seconds**
4. Zero user impact
5. No 2 AM panic

This is how companies deploy **multiple times per day** safely.

---

# Key Benefits Demonstrated

| Feature                 | Real-World Advantage                                   |
| ----------------------- | ------------------------------------------------------ |
| Zero-Downtime Swap      | Users never see errors or outages                      |
| Instant Rollback        | One-click revert in seconds                            |
| Pre-warmed Instances    | No cold-start latency                                  |
| Isolated Testing URL    | Full QA/UAT without affecting production               |
| Configuration Isolation | Test connection strings safely                         |
| Traffic Routing Support | Canary releases via Azure Front Door / Traffic Manager |

---

# Azure Features Used

* Azure App Service (Linux, PHP 8.4)
* Deployment Slots + Swap operation
* Standard S1 Tier
* Built-in SSH Console
* Zero-downtime traffic routing

---

# AZ-104 Exam Relevance

This lab directly covers:

**"Deploy and configure Azure App Service" → Deployment Slots & Swap**

---

# Tags

Azure • App Service • Deployment Slots • Blue-Green Deployment • Zero Downtime • DevOps • SRE • CI/CD • Rollback • Production Deployment • AZ-104

---

# Author

**Punit Anand – Azure | DevOps | SRE**
Turning labs into production-grade muscle, one deployment at a time.
