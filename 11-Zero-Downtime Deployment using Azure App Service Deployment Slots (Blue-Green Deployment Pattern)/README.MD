# Lab 11: Zero-Downtime Deployment using Azure App Service Deployment Slots (Blue-Green Deployment Pattern)

[![Production After Swap – Version 1 Live](https://raw.githubusercontent.com/PunitAnand-SRE/az-104-labs/main/11-Zero-Downtime%20Deployment%20using%20Azure%20App%20Service%20Deployment%20Slots%20\(Blue-Green%20Deployment%20Pattern\)/screenshots/09-post-swap-verification-production-now-shows-version-1.png)](https://raw.githubusercontent.com/PunitAnand-SRE/az-104-labs/main/11-Zero-Downtime%20Deployment%20using%20Azure%20App%20Service%20Deployment%20Slots%20%28Blue-Green%20Deployment%20Pattern%29/screenshots/09-post-swap-verification-production-now-shows-version-1.png)

---

## Objective

Implement a **true blue-green deployment** with **zero downtime** and **instant rollback** using **Azure App Service Deployment Slots**, exactly how enterprise-grade releases are done.

---

## Why This Lab Is Critical for SRE/DevOps

* Ensures **100% uptime** during deployments
* **Rollback in < 10 seconds**
* Enables A/B, canary, feature flags
* Mandatory for PCI-DSS, HIPAA, SOC2
* One of the most frequently asked **Azure interview scenarios**

---

## Architecture Flow

```text
                ┌──────────────────────────────────────────┐
                │            Production Slot               │ ← User Traffic
                │  werapp-dev-cind.azurewebsites.net       │
                └──────────────────────────────────────────┘
                                   ↑         ↔ Instant Swap
                                   │
                ┌──────────────────────────────────────────┐
                │              Staging Slot                │
                │ werapp-dev-cind-staging.azurewebsites.net │
                └──────────────────────────────────────────┘
```

---

# Step-by-Step Execution

---

## **1. Azure App Service Created (Linux + PHP 8.4)**

**Screenshot:**
[https://raw.githubusercontent.com/PunitAnand-SRE/az-104-labs/main/11-Zero-Downtime%20Deployment%20using%20Azure%20App%20Service%20Deployment%20Slots%20(Blue-Green%20Deployment%20Pattern)/screenshots/01-webapp-overview-and-details.png](https://raw.githubusercontent.com/PunitAnand-SRE/az-104-labs/main/11-Zero-Downtime%20Deployment%20using%20Azure%20App%20Service%20Deployment%20Slots%20%28Blue-Green%20Deployment%20Pattern%29/screenshots/01-webapp-overview-and-details.png)

![Step 1](https://raw.githubusercontent.com/PunitAnand-SRE/az-104-labs/main/11-Zero-Downtime%20Deployment%20using%20Azure%20App%20Service%20Deployment%20Slots%20\(Blue-Green%20Deployment%20Pattern\)/screenshots/01-webapp-overview-and-details.png)

---

## **2. Deployed Initial Version to Production via SSH**

```bash
cd /home/site/wwwroot && nano index.php
# Content:
Hello from Azure Web Apps (PHP)!
```

**SSH Deployment Screenshot:**
[https://raw.githubusercontent.com/PunitAnand-SRE/az-104-labs/main/11-Zero-Downtime%20Deployment%20using%20Azure%20App%20Service%20Deployment%20Slots%20(Blue-Green%20Deployment%20Pattern)/screenshots/02-ssh-into-production-and-deploy-initial-php-app.png](https://raw.githubusercontent.com/PunitAnand-SRE/az-104-labs/main/11-Zero-Downtime%20Deployment%20using%20Azure%20App%20Service%20Deployment%20Slots%20%28Blue-Green%20Deployment%20Pattern%29/screenshots/02-ssh-into-production-and-deploy-initial-php-app.png)

![Step 2](https://raw.githubusercontent.com/PunitAnand-SRE/az-104-labs/main/11-Zero-Downtime%20Deployment%20using%20Azure%20App%20Service%20Deployment%20Slots%20\(Blue-Green%20Deployment%20Pattern\)/screenshots/02-ssh-into-production-and-deploy-initial-php-app.png)

**Production Live Verification:**
[https://raw.githubusercontent.com/PunitAnand-SRE/az-104-labs/main/11-Zero-Downtime%20Deployment%20using%20Azure%20App%20Service%20Deployment%20Slots%20(Blue-Green%20Deployment%20Pattern)/screenshots/03-production-app-live-verification.png](https://raw.githubusercontent.com/PunitAnand-SRE/az-104-labs/main/11-Zero-Downtime%20Deployment%20using%20Azure%20App%20Service%20Deployment%20Slots%20%28Blue-Green%20Deployment%20Pattern%29/screenshots/03-production-app-live-verification.png)

![Step 2 Live](https://raw.githubusercontent.com/PunitAnand-SRE/az-104-labs/main/11-Zero-Downtime%20Deployment%20using%20Azure%20App%20Service%20Deployment%20Slots%20\(Blue-Green%20Deployment%20Pattern\)/screenshots/03-production-app-live-verification.png)

---

## **3. Scaled Up to Standard S1 Tier (Required for Deployment Slots)**

**Screenshot:**
[https://raw.githubusercontent.com/PunitAnand-SRE/az-104-labs/main/11-Zero-Downtime%20Deployment%20using%20Azure%20App%20Service%20Deployment%20Slots%20(Blue-Green%20Deployment%20Pattern)/screenshots/04-scale-up-to-standard-s1-to-enable-slots.png](https://raw.githubusercontent.com/PunitAnand-SRE/az-104-labs/main/11-Zero-Downtime%20Deployment%20using%20Azure%20App%20Service%20Deployment%20Slots%20%28Blue-Green%20Deployment%20Pattern%29/screenshots/04-scale-up-to-standard-s1-to-enable-slots.png)

![Step 3](https://raw.githubusercontent.com/PunitAnand-SRE/az-104-labs/main/11-Zero-Downtime%20Deployment%20using%20Azure%20App%20Service%20Deployment%20Slots%20\(Blue-Green%20Deployment%20Pattern\)/screenshots/04-scale-up-to-standard-s1-to-enable-slots.png)

---

## **4. Created Staging Deployment Slot**

**Creating Slot Screenshot:**
[https://raw.githubusercontent.com/PunitAnand-SRE/az-104-labs/main/11-Zero-Downtime%20Deployment%20using%20Azure%20App%20Service%20Deployment%20Slots%20(Blue-Green%20Deployment%20Pattern)/screenshots/05-creating-staging-deployment-slot.png](https://raw.githubusercontent.com/PunitAnand-SRE/az-104-labs/main/11-Zero-Downtime%20Deployment%20using%20Azure%20App%20Service%20Deployment%20Slots%20%28Blue-Green%20Deployment%20Pattern%29/screenshots/05-creating-staging-deployment-slot.png)

![Step 4 Creating](https://raw.githubusercontent.com/PunitAnand-SRE/az-104-labs/main/11-Zero-Downtime%20Deployment%20using%20Azure%20App%20Service%20Deployment%20Slots%20\(Blue-Green%20Deployment%20Pattern\)/screenshots/05-creating-staging-deployment-slot.png)

**Staging Slot Ready Screenshot:**
[https://raw.githubusercontent.com/PunitAnand-SRE/az-104-labs/main/11-Zero-Downtime%20Deployment%20using%20Azure%20App%20Service%20Deployment%20Slots%20(Blue-Green%20Deployment%20Pattern)/screenshots/06-staging-slot-created-and-details.png](https://raw.githubusercontent.com/PunitAnand-SRE/az-104-labs/main/11-Zero-Downtime%20Deployment%20using%20Azure%20App%20Service%20Deployment%20Slots%20%28Blue-Green%20Deployment%20Pattern%29/screenshots/06-staging-slot-created-and-details.png)

![Step 4 Ready](https://raw.githubusercontent.com/PunitAnand-SRE/az-104-labs/main/11-Zero-Downtime%20Deployment%20using%20Azure%20App%20Service%20Deployment%20Slots%20\(Blue-Green%20Deployment%20Pattern\)/screenshots/06-staging-slot-created-and-details.png)

---

## **5. Deployed New Version (v1) to Staging & Tested**

```php
<h1>Web Application - Version 1</h1>
<p>This is a simple PHP application running on Azure App Service - Version 1</p>
```

**Staging v1 Screenshot:**
[https://raw.githubusercontent.com/PunitAnand-SRE/az-104-labs/main/11-Zero-Downtime%20Deployment%20using%20Azure%20App%20Service%20Deployment%20Slots%20(Blue-Green%20Deployment%20Pattern)/screenshots/07-staging-slot-php-app-deployed-and-tested.png](https://raw.githubusercontent.com/PunitAnand-SRE/az-104-labs/main/11-Zero-Downtime%20Deployment%20using%20Azure%20App%20Service%20Deployment%20Slots%20%28Blue-Green%20Deployment%20Pattern%29/screenshots/07-staging-slot-php-app-deployed-and-tested.png)

![Step 5](https://raw.githubusercontent.com/PunitAnand-SRE/az-104-labs/main/11-Zero-Downtime%20Deployment%20using%20Azure%20App%20Service%20Deployment%20Slots%20\(Blue-Green%20Deployment%20Pattern\)/screenshots/07-staging-slot-php-app-deployed-and-tested.png)

---

## **6. Executed Zero-Downtime Swap (Staging → Production)**

**Swap Screenshot:**
[https://raw.githubusercontent.com/PunitAnand-SRE/az-104-labs/main/11-Zero-Downtime%20Deployment%20using%20Azure%20App%20Service%20Deployment%20Slots%20(Blue-Green%20Deployment%20Pattern)/screenshots/08-performing-swap-between-production-and-staging.png](https://raw.githubusercontent.com/PunitAnand-SRE/az-104-labs/main/11-Zero-Downtime%20Deployment%20using%20Azure%20App%20Service%20Deployment%20Slots%20%28Blue-Green%20Deployment%20Pattern%29/screenshots/08-performing-swap-between-production-and-staging.png)

![Step 6](https://raw.githubusercontent.com/PunitAnand-SRE/az-104-labs/main/11-Zero-Downtime%20Deployment%20using%20Azure%20App%20Service%20Deployment%20Slots%20\(Blue-Green%20Deployment%20Pattern\)/screenshots/08-performing-swap-between-production-and-staging.png)

---

## **7. Final Result – Production Now Serves Version 1**

**Post-Swap Screenshot:**
[https://raw.githubusercontent.com/PunitAnand-SRE/az-104-labs/main/11-Zero-Downtime%20Deployment%20using%20Azure%20App%20Service%20Deployment%20Slots%20(Blue-Green%20Deployment%20Pattern)/screenshots/09-post-swap-verification-production-now-shows-version-1.png](https://raw.githubusercontent.com/PunitAnand-SRE/az-104-labs/main/11-Zero-Downtime%20Deployment%20using%20Azure%20App%20Service%20Deployment%20Slots%20%28Blue-Green%20Deployment%20Pattern%29/screenshots/09-post-swap-verification-production-now-shows-version-1.png)

![Step 7](https://raw.githubusercontent.com/PunitAnand-SRE/az-104-labs/main/11-Zero-Downtime%20Deployment%20using%20Azure%20App%20Service%20Deployment%20Slots%20\(Blue-Green%20Deployment%20Pattern\)/screenshots/09-post-swap-verification-production-now-shows-version-1.png)

---

# Instant Rollback (Real-World Scenario)

If Version 1 has a critical issue:
→ Go to **Deployment Slots → Swap**
→ Old version restored in **<10 seconds**
→ Zero downtime
→ Zero customer complaints

---

# Key Benefits

| Feature              | Real-World Advantage                    |
| -------------------- | --------------------------------------- |
| Zero-Downtime Swap   | Users never see errors                  |
| Instant Rollback     | One-click revert                        |
| Pre-warmed Instances | No cold starts                          |
| Isolated Testing URL | QA/UAT without touching production      |
| Config Isolation     | Test different connection strings       |
| Traffic Routing      | Canary via Front Door / Traffic Manager |

---

# Azure Features Used

* Azure App Service (Linux, PHP 8.4)
* Deployment Slots
* Swap Operation
* Standard S1 Tier
* Built-in SSH Console
* Zero-downtime traffic routing

---

# AZ-104 Exam Mapping

Covers:
**“Deploy and configure Azure App Service — Deployment Slots & Swap”**

---

# Tags

Azure • App Service • Deployment Slots • Blue-Green • Zero Downtime • DevOps • SRE • CI/CD • Rollback • Production • AZ-104

---

# Punit Anand – Azure | DevOps | SRE

Turning labs into production-grade muscle
