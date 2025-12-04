# **Lab 11: Zero-Downtime Deployment using Azure App Service Deployment Slots (Blue-Green Deployment Pattern)**

![Production After Swap – Version 1 Live](https://raw.githubusercontent.com/PunitAnand-SRE/az-104-labs/main/11-Zero-Downtime%20Deployment%20using%20Azure%20App%20Service%20Deployment%20Slots%20\(Blue-Green%20Deployment%20Pattern\)/screenshots/09-post-swap-verification-production-now-shows-version-1.png)

---

## **Objective**

Implement a **zero-downtime, enterprise-grade blue-green deployment** using **Azure App Service Deployment Slots**, ensuring:

* Continuous availability during deployments
* Instant rollback capability
* Isolation of staging/testing from production
* No user impact even during major version upgrades

This is exactly how **Netflix, Microsoft, Walmart, and Adobe** deploy safely at scale.

---

## **Why This Lab Matters for SRE/DevOps Roles**

* Ensures *always-on* production systems
* Reduces deployment risk
* Enables safe experimentation & canary testing
* Eliminates 2 AM outages due to bad deployments
* One of the **top Azure interview scenarios** (SRE + DevOps)

---

## **Architecture Diagram**

```text
                ┌──────────────────────────────────────────────┐
                │               Production Slot                 │
                │   werapp-dev-cind.azurewebsites.net          │
                └──────────────────────────────────────────────┘
                               ↑          ↔ Zero-Downtime Swap
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

![Web App Overview Screenshot](https://raw.githubusercontent.com/PunitAnand-SRE/az-104-labs/main/11-Zero-Downtime%20Deployment%20using%20Azure%20App%20Service%20Deployment%20Slots%20\(Blue-Green%20Deployment%20Pattern\)/screenshots/01-webapp-overview-and-details.png)

### **What This Screenshot Shows**

* App Service created on **Linux** runtime
* **PHP 8.4** selected
* Default **production slot** automatically generated
* App URL assigned by Azure

### **Why This Matters**

This forms the foundation of the entire blue-green deployment architecture.
Azure App Service's slot capability is available **only in Standard tier or above**, which we will configure next.

---

# **2. Deployed Initial Version to Production via SSH**

### **Code Deployed**

```bash
cd /home/site/wwwroot && nano index.php
# Content:
Hello from Azure Web Apps (PHP)!
```

### **SSH Deployment Screenshot**

![SSH Deployment Screenshot](https://raw.githubusercontent.com/PunitAnand-SRE/az-104-labs/main/11-Zero-Downtime%20Deployment%20using%20Azure%20App%20Service%20Deployment%20Slots%20\(Blue-Green%20Deployment%20Pattern\)/screenshots/02-ssh-into-production-and-deploy-initial-php-app.png)

**What it shows:**

* Using built-in App Service **SSH console**
* Creating and updating the initial PHP application
* This becomes the **Baseline Version (V0)**

---

### **Production Live Verification Screenshot**

![Production Live Screenshot](https://raw.githubusercontent.com/PunitAnand-SRE/az-104-labs/main/11-Zero-Downtime%20Deployment%20using%20Azure%20App%20Service%20Deployment%20Slots%20\(Blue-Green%20Deployment%20Pattern\)/screenshots/03-production-app-live-verification.png)

**What it shows:**

* Production slot serving the initial version
* Confirms deployment success
* This is the version that will remain live until we swap

---

# **3. Scaled Up to Standard S1 Tier (Required for Deployment Slots)**

![Scale Up Screenshot](https://raw.githubusercontent.com/PunitAnand-SRE/az-104-labs/main/11-Zero-Downtime%20Deployment%20using%20Azure%20App%20Service%20Deployment%20Slots%20\(Blue-Green%20Deployment%20Pattern\)/screenshots/04-scale-up-to-standard-s1.png)

### **What This Screenshot Shows**

* App Service is being scaled from **Basic → Standard S1**
* Deployment slots are only available from **S1 tier onwards**

### **Why This Matters**

Without Standard tier, blue-green deployments are not possible.

---

# **4. Created Staging Deployment Slot**

### **Creating the Slot**

![Create Staging Slot](https://raw.githubusercontent.com/PunitAnand-SRE/az-104-labs/main/11-Zero-Downtime%20Deployment%20using%20Azure%20App%20Service%20Deployment%20Slots%20\(Blue-Green%20Deployment%20Pattern\)/screenshots/05-create-staging-slot.png)

### **Staging Slot Ready**

![Staging Slot Ready](https://raw.githubusercontent.com/PunitAnand-SRE/az-104-labs/main/11-Zero-Downtime%20Deployment%20using%20Azure%20App%20Service%20Deployment%20Slots%20\(Blue-Green%20Deployment%20Pattern\)/screenshots/06-staging-slot-created.png)

### **Why This Matters**

* Staging slot acts as a **parallel environment**
* Enables safe testing without touching production
* Both slots have **identical infrastructure**

This forms the classic **blue-green deployment pattern**.

---

# **5. Deployed New Version (v1) to Staging**

### **Updated PHP Application**

```php
<h1>Web Application - Version 1</h1>
<p>This is a simple PHP application running on Azure App Service - Version 1</p>
```

### **Screenshot – New Version in Staging**

![Staging Slot Version 1 Live](https://raw.githubusercontent.com/PunitAnand-SRE/az-104-labs/main/11-Zero-Downtime%20Deployment%20using%20Azure%20App%20Service%20Deployment%20Slots%20\(Blue-Green%20Deployment%20Pattern\)/screenshots/07-staging-version-1-live.png)

### **What This Screenshot Shows**

* “Version 1” deployed ONLY to the staging slot
* Allows us to validate new release safely
* Confirms staging slot isolation

This version will soon replace production.

---

# **6. Executed Zero-Downtime Swap (Staging → Production)**

![Swap Operation Screenshot](https://raw.githubusercontent.com/PunitAnand-SRE/az-104-labs/main/11-Zero-Downtime%20Deployment%20using%20Azure%20App%20Service%20Deployment%20Slots%20\(Blue-Green%20Deployment%20Pattern\)/screenshots/08-swap-operation.png)

### **What This Screenshot Shows**

* Azure is performing an **atomic swap** between slots
* Production gets pre-warmed staging instance
* No cold start
* No downtime
* No dropped connections

This is Azure’s **most powerful deployment feature**.

---

# **7. Final Verification – Production Now Serving Version 1**

![Post Swap – Production Version 1](https://raw.githubusercontent.com/PunitAnand-SRE/az-104-labs/main/11-Zero-Downtime%20Deployment%20using%20Azure%20App%20Service%20Deployment%20Slots%20\(Blue-Green%20Deployment%20Pattern\)/screenshots/09-post-swap-verification-production-now-shows-version-1.png)

### **What This Screenshot Shows**

* Production is now upgraded to **Version 1**
* Staging now contains the **old production version**
* Instant rollback is now possible

This confirms a **successful zero-downtime release**.

---

# **Instant Rollback (The SRE Superpower)**

If Version 1 fails after going live:

→ Go to **Deployment Slots**
→ Click **Swap**
→ Production instantly reverts to the old (stable) version
→ Rollback completes in **<10 seconds**
→ No downtime, no outage, no user impact

This is why **enterprises deploy multiple times per day** confidently.

---

# **Key Benefits Demonstrated**

| Feature                 | Real SRE Benefit                                        |
| ----------------------- | ------------------------------------------------------- |
| Zero-Downtime Swap      | No errors during deployment                             |
| Instant Rollback        | Revert instantly to last stable version                 |
| Pre-Warmed Instances    | No cold startup delay                                   |
| Isolated Testing URL    | Full QA/UAT without touching prod                       |
| Traffic Routing         | Canary rollouts via Azure Front Door or Traffic Manager |
| Configuration Isolation | Different connection strings per slot                   |

---

# **Azure Services Used**

* **Azure App Service (Linux)**
* **PHP 8.4 Runtime**
* **Deployment Slots**
* **Zero-Downtime Swap**
* **Standard S1 Tier**
* **Built-in SSH Console**

---

# **AZ-104 Exam Relevance**

This lab directly maps to the AZ-104 objective:

> **Deploy and configure Azure App Service Web Apps
> (including Deployment Slots & Swap)**

---

# **Author – Punit Anand (Azure | DevOps | SRE)**

Building real-world SRE labs that match how enterprise production works.
