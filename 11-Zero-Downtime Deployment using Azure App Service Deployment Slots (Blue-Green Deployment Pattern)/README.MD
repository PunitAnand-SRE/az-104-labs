# Lab 11: Zero-Downtime Deployment using Azure App Service Deployment Slots (Blue-Green Deployment Pattern)

[![Production After Swap](https://raw.githubusercontent.com/PunitAnand-SRE/az-104-labs/main/11-Zero-Downtime%20Deployment%20using%20Azure%20App%20Service%20Deployment%20Slots%20(Blue-Green%20Deployment%20Pattern)/screenshots/09-post-swap-verification-production-now-shows-version-1.png)](https://raw.githubusercontent.com/PunitAnand-SRE/az-104-labs/main/11-Zero-Downtime%20Deployment%20using%20Azure%20App%20Service%20Deployment%20Slots%20(Blue-Green%20Deployment%20Pattern)/screenshots/09-post-swap-verification-production-now-shows-version-1.png)

## 🎯 Objective  
Implement a **production-grade, zero-downtime deployment strategy** using **Azure App Service Deployment Slots** — the same pattern used by Microsoft, Netflix, Adobe, and other enterprise workloads.

This is a **true blue-green deployment**.

---

## 🚀 Why This Lab is Critical for SRE/DevOps Roles
- Ensures **100% uptime** during deployments  
- Enables **instant rollback** (under 10 seconds)  
- Supports canary releases, A/B testing, feature flags  
- Required in compliance environments (PCI-DSS, HIPAA, SOC2)  
- One of the **top 5 most asked scenarios** in Azure DevOps/SRE interviews  

---

## 🏗️ Architecture Flow

```

User Traffic ---> Production Slot (werapp-dev-cind.azurewebsites.net)
↑                     ↔ (Instant Swap – Zero Downtime)
|
Staging Slot (werapp-dev-cind-staging.azurewebsites.net)

````

---

## 🔧 Step-by-Step Execution

### **1. Azure App Service Created (Linux + PHP 8.4)**
![Web App Overview](https://raw.githubusercontent.com/PunitAnand-SRE/az-104-labs/main/11-Zero-Downtime%20Deployment%20using%20Azure%20App%20Service%20Deployment%20Slots%20(Blue-Green%20Deployment%20Pattern)/screenshots/01-webapp-overview-and-details.png)

---

### **2. Deployed Initial Version to Production via SSH**

```bash
cd /home/site/wwwroot && nano index.php
# Content:
Hello from Azure Web Apps (PHP)!
````

Production now serves the **initial version**.

---

### **3. Scaled Up to Standard S1 Tier**

> Required because **deployment slots are not available** in Free/Basic tiers.

---

### **4. Created Staging Deployment Slot**

* Name: `staging`
* Config cloned from production

Staging slot created successfully.

---

### **5. Deployed New Version (v1) to Staging & Tested**

Updated `index.php`:

```php
Web Application - Version 1
```

Staging now serves **Version 1**.

---

### **6. Executed Zero-Downtime Swap (Staging → Production)**

Azure pre-warms the staging app to eliminate cold starts.

Swap completed — no downtime.

---

### **7. Verified Production Now Serves Version 1**

Old version automatically moved to staging → **instant rollback available**.

---

## ♻️ Instant Rollback Demo (Real Scenario)

If Version 1 has an issue:

1. Go to Deployment Slots
2. Click **Swap** again
3. Old stable version restored in **< 10 seconds**
4. **No user impact**

---

## ⭐ Key Benefits Proven

| Feature              | Real-World Advantage                        |
| -------------------- | ------------------------------------------- |
| Zero-Downtime Swap   | No user disruption                          |
| Instant Rollback     | One-click revert                            |
| Pre-warmed Instances | No cold-start latency                       |
| Isolated Testing URL | QA/UAT without affecting production         |
| AutoSwap + Traffic % | Canary releases with Front Door/App Gateway |

---

## 🔧 Azure Features Used

* Azure App Service (Linux, PHP 8.4)
* Deployment Slots + Swap
* Standard S1 Tier
* Built-in SSH Console
* Zero-downtime traffic routing

---

## 📘 AZ-104 Exam Relevance

Directly covers:

**“Deploy and configure Azure App Service” → Deployment Slots & Swap**

---

## 🏷️ Tags

Azure • App Service • Deployment Slots • Blue-Green Deployment • Zero Downtime • DevOps • SRE • CI/CD • Production Deployment • Rollback • AZ-104

---

## ✍️ Author

**Punit Anand – Azure | DevOps | SRE**
Turning labs into production muscle
