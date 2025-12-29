# 🧪 Lab 16 – Azure NSG Two-Tier Architecture: Secure Web-to-Database Access

---

## 📌 Lab Objective

This lab demonstrates how to design and secure a **two-tier architecture** in Microsoft Azure using **Network Security Groups (NSGs)**, ensuring:

* 🌐 **Public users can access only the Web VM (HTTP)**
* 🛢️ **Database VM is completely isolated from the Internet**
* 🔐 **Only the Web VM can communicate with the Database VM**
* 🚫 **Outbound Internet access is blocked from both VMs**
* 🧠 **NSG rule priority, protocol filtering, and stateful behavior** are clearly understood

This setup reflects **real-world enterprise security architecture**, not just theory.

---

## 🏗️ Architecture Overview

```
Internet
   |
   |  HTTP (80)
   v
[ Web VM ]
   |
   |  MySQL (3306) — Private IP only
   v
[ DB VM ]
```

### Key Design Principles Used

* **Least Privilege Access**
* **Defense in Depth**
* **Zero Trust Networking**
* **Private IP–only backend communication**

---

## 🔍 Key Azure Concepts Used

### 🔹 Network Security Group (NSG)

An NSG acts as a **stateful virtual firewall** that controls:

* Inbound traffic to a resource
* Outbound traffic from a resource

Rules are evaluated based on:

1. **Priority** (lower number = higher priority)
2. **Protocol**
3. **Source & Destination**
4. **Port**
5. **Action (Allow / Deny)**

---

### 🔹 Stateful Nature of NSGs

If **inbound traffic is allowed**, the **return traffic is automatically allowed**, even if outbound rules are restrictive.

> This behavior is critical in understanding why application traffic works while Internet access is blocked.

---

## 🧩 Lab Components

| Component                     | Purpose                         |
| ----------------------------- | ------------------------------- |
| Web VM (`vm-dev-cind-web-01`) | Public-facing web tier          |
| DB VM (`vm-dev-cind-db-01`)   | Private database tier           |
| Web NSG                       | Controls Internet → Web traffic |
| DB NSG                        | Controls Web → DB traffic       |

---

## 🧪 Step-by-Step Implementation & Validation

---

## 1️⃣ Install and Verify MySQL on Database VM

Great catch 👍
You’re right — for a **top-tier lab**, readers should see **exact commands + explanation**.
Below is an **enhanced, drop-in replacement section** you can paste into your README that upgrades the lab from *good* to **professional / interview-ready / enterprise-grade**.

---

## 🔧 1️⃣ Install and Configure MySQL on Database VM (vm-dev-cind-db-01)

In this step, we prepare the **database tier** by installing MySQL, securing it, and configuring it to accept connections **only over the private network**.

---

### 🔹 Step 1: Update the Package Index

```bash
sudo apt update
```

**Why this is required:**

* Fetches the latest package metadata from Ubuntu repositories
* Ensures we install the **latest stable MySQL version**
* Prevents dependency and compatibility issues

---

### 🔹 Step 2: Install MySQL Server

```bash
sudo apt install -y mysql-server
```

**What this does:**

* Installs the MySQL Server binaries
* Creates the MySQL system user
* Sets up default configuration files
* Starts the MySQL service automatically

The `-y` flag ensures non-interactive installation (best practice for automation).

---

### 🔹 Step 3: Enable MySQL Service at Boot

```bash
sudo systemctl enable mysql
```

**Why this matters:**

* Ensures MySQL starts automatically after VM reboot
* Critical for production workloads and HA scenarios

---

### 🔹 Step 4: Configure MySQL to Listen on Private IP Only

By default, MySQL listens on `127.0.0.1` (localhost).
For a **two-tier architecture**, it must listen on the **DB VM private IP** so that the Web VM can connect.

Edit the MySQL configuration file:

```bash
sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf
```

Update the following line:

```ini
bind-address = 10.0.1.4
```

> `10.0.1.4` is the **private IP address** of `vm-dev-cind-db-01`.

**Security Advantage:**

* Database is accessible only within the VNet
* No public exposure
* Fully controlled by NSG rules

---

### 🔹 Step 5: Restart MySQL Service

```bash
sudo systemctl restart mysql
```

**Why restart is required:**

* Applies the updated `bind-address`
* Reloads MySQL configuration safely

---

### 🔹 Step 6: Verify MySQL Service Status

```bash
sudo systemctl status mysql
```

You should see:

* `Active: active (running)`

This confirms MySQL is operational and ready.

---

### 🔹 Step 7: Verify Local MySQL Login

```bash
sudo mysql
```

Expected result:

* Successful login to MySQL shell
* Confirms MySQL server is working correctly

Exit MySQL:

```sql
EXIT;
```

---

### 📸 Screenshot – MySQL Installed and Running on DB VM

![MySQL Installed and Running on DB VM](https://raw.githubusercontent.com/PunitAnand-SRE/az-104-labs/main/16-azure-nsg-two-tier-architecture-web-to-db-secure-access/screenshots/01-mysql-installed-and-running-on-db-vm.png)

---

## 🧠 Why This Configuration Is Enterprise-Grade

| Aspect                | Benefit                     |
| --------------------- | --------------------------- |
| Private IP binding    | No public database exposure |
| NSG-controlled access | Only Web VM can connect     |
| Service auto-start    | Production reliability      |
| Explicit validation   | Operational confidence      |

---

## 🔑 Key Takeaway

> **A secure two-tier architecture is not just about NSG rules —
> it starts with correctly configuring the application and database services themselves.**

This step ensures the database is:

* Installed correctly
* Securely configured
* Ready for controlled access from the web tier

---

## 2️⃣ Secure Database VM – Block Internet, Allow Web VM Only

### NSG Strategy (DB VM)

* ❌ **Deny MySQL (3306) from Internet**
* ✅ **Allow MySQL (3306) only from Web VM private IP**
* ❌ Deny unwanted port ranges
* ❌ Deny Internet outbound traffic

### Why This Matters

* Database servers **must never be Internet-accessible**
* Only the application tier should reach the database
* Prevents brute force, scanning, and data exfiltration risks

### 📸 Screenshot – DB NSG Rules (Internet Denied, Web VM Allowed)

![DB NSG Deny Internet Allow Web Only](https://raw.githubusercontent.com/PunitAnand-SRE/az-104-labs/main/16-azure-nsg-two-tier-architecture-web-to-db-secure-access/screenshots/02-db-nsg-deny-mysql-from-internet-allow-web-vm-only.png)

---

## 3️⃣ Secure Web VM – Allow HTTP, Block Everything Else

### NSG Strategy (Web VM)

* ✅ Allow **HTTP (80)** from Internet
* ❌ Deny unnecessary inbound port ranges
* ❌ Deny outbound Internet access
* ✅ Allow VNet traffic (default rule)

### Why This Matters

* Web servers should expose **only required ports**
* Reduces attack surface
* Prevents outbound malware callbacks or data leaks

### 📸 Screenshot – Web NSG with Controlled Internet Exposure

![Web NSG HTTP Allowed Deny Others](https://raw.githubusercontent.com/PunitAnand-SRE/az-104-labs/main/16-azure-nsg-two-tier-architecture-web-to-db-secure-access/screenshots/03-web-nsg-allow-http-from-internet-and-deny-unwanted-ports.png)

---

## 4️⃣ Validate Secure Web → DB Communication

From the **Web VM**, a MySQL client connection is made using the **DB VM private IP**:

```bash
mysql -h 10.0.1.4 -u <username> -p
```

### Why This Works

* NSG allows MySQL from Web VM → DB VM
* Both VMs are in the same VNet
* NSGs are **stateful**, so response traffic is allowed

### Why This Is Secure

* No public IP involved
* No Internet exposure
* Traffic stays entirely inside Azure backbone

### 📸 Screenshot – Successful Web VM to DB VM Connection

![Web VM Connects to DB VM](https://raw.githubusercontent.com/PunitAnand-SRE/az-104-labs/main/16-azure-nsg-two-tier-architecture-web-to-db-secure-access/screenshots/04-web-vm-connects-to-db-vm-over-mysql-private-ip.png)

---

## 🚫 Outbound Internet Blocking – Security Hardening

### What Was Enforced

* ❌ No outbound Internet access from Web VM
* ❌ No outbound Internet access from DB VM

### Why This Is Critical

* Prevents:

  * Malware downloads
  * Unauthorized updates
  * Data exfiltration
* Enforces **controlled update pipelines** via bastion, proxy, or NAT (enterprise best practice)

---

## 🧠 Key Learnings from This Lab

* NSGs are **stateful**, not stateless firewalls
* Rule **priority always overrides rule order**
* Backend services should **never face the Internet**
* Private IP communication is the **most secure pattern**
* Azure default NSG rules are helpful — but must be **understood, not blindly trusted**

---

## 🎯 Real-World Use Cases

* Production Web + Database applications
* PCI-DSS compliant environments
* Banking & finance workloads
* Zero Trust cloud architectures
* AZ-104 & AZ-500 exam scenarios

---

## ✅ Final Outcome

✔ Internet users can access the Web application
✔ Database is completely isolated
✔ Only Web VM can reach Database
✔ Outbound Internet is blocked
✔ Unnecessary ports are denied
✔ Enterprise-grade security posture achieved

---

## 📌 Conclusion

This lab represents a **realistic, production-ready Azure security design**, not just an academic exercise.
It combines **networking fundamentals, security best practices, and practical validation**, making it ideal for:

* Azure administrators
* Cloud engineers
* SREs
* Security-focused roles

