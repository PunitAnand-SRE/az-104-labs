# Lab 11: Zero-Downtime Deployment using Azure App Service Deployment Slots (Blue-Green Deployment Pattern)

[![Production After Swap](https://raw.githubusercontent.com/PunitAnand-SRE/az-104-labs/main/11-Zero-Downtime%20Deployment%20using%20Azure%20App%20Service%20Deployment%20Slots%20(Blue-Green%20Deployment%20Pattern)/screenshots/09-post-swap-verification-production-now-shows-version-1.png)](https://raw.githubusercontent.com/PunitAnand-SRE/az-104-labs/main/11-Zero-Downtime%20Deployment%20using%20Azure%20App%20Service%20Deployment%20Slots%20(Blue-Green%20Deployment%20Pattern)/screenshots/09-post-swap-verification-production-now-shows-version-1.png)

## Objective
Implement a **real-world, production-grade, zero-downtime deployment** using **Azure App Service Deployment Slots** — the exact blue-green deployment pattern used by Microsoft, Netflix, Adobe, and nearly every enterprise on Azure.

## Why This is Critical in Production & Interviews
- 100% uptime during deployments
- Instant rollback in under 10 seconds
- Supports A/B testing, canary releases, feature toggles
- Required for PCI-DSS, HIPAA, SOC2 compliance
- One of the **top 5 most-asked DevOps/Azure scenarios** in interviews

## Architecture Overview
User Traffic ---> Production Slot (werapp-dev-cind.azurewebsites.net)
↑                       ↔ Instant Swap (Zero Downtime)
Staging Slot (werapp-dev-cind-staging.azurewebsites.net)
text## Step-by-Step Execution

### 1. Created Azure App Service (Linux + PHP 8.4)
![Web App Overview](https://raw.githubusercontent.com/PunitAnand-SRE/az-104-labs/main/11-Zero-Downtime%20Deployment%20using%20Azure%20App%20Service%20Deployment%20Slots%20(Blue-Green%20Deployment%20Pattern)/screenshots/01-webapp-overview-and-details.png)

### 2. Deployed Initial Version to Production via SSH
```bash
cd /home/site/wwwroot && nano index.php
Content: "Hello from Azure Web Apps (PHP)!"
SSH into Production
Production Live – Initial Version
3. Scaled Up to Standard S1 Tier
Deployment Slots are not available in Free/Basic tiers
Scale Up to S1
4. Created Staging Deployment Slot
Name: staging | Cloned configuration from production
Creating Staging Slot
Staging Slot Created
5. Deployed New Version (v1) to Staging Slot & Tested
Updated index.php → "Web Application - Version 1"
Staging – Version 1 Live
6. Performed Zero-Downtime Swap
No restart | No dropped requests | No DNS changes
Swap Operation
7. Verified Production Now Serves Version 1
Old version instantly moved to staging (ready for rollback)
Final Production – Version 1 Live
Key Benefits Demonstrated

































FeatureReal-World AdvantageZero-Downtime SwapUsers never experience errors or downtimeInstant RollbackOne-click revert in under 10 secondsPre-warmed InstancesNew version fully warmed up before receiving trafficIsolated Testing URLQA/stakeholders validate without affecting productionConfiguration IsolationTest different connection strings / app settings safelyTraffic Routing SupportEnable canary releases with Azure Front Door / Traffic Manager
Real-World Rollback Scenario (Career-Saving Moment)
If Version 1 has a critical bug after going live:

Go to Deployment Slots
Click Swap again
Old stable version restored instantly
→ Zero user impact. No 2 AM calls.

This is how companies deploy multiple times per day safely.
Azure Features Used

Azure App Service (Linux, PHP 8.4)
Deployment Slots + Swap
Standard Tier (S1)
Built-in SSH Console
Zero-downtime traffic routing

AZ-104 Exam Relevance
Directly maps to: "Deploy and configure Azure App Service" → Deployment Slots & Swap operations
Tags
Azure App Service Deployment Slots Blue-Green Deployment Zero Downtime DevOps SRE CI/CD Rollback Production Deployment AZ-104

Punit Anand – Azure | DevOps | SRE
Building production-grade cloud expertise, one lab at a time
LinkedIn | GitHub
