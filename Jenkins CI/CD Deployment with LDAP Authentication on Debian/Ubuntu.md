## 🚀 Jenkins CI/CD Deployment with LDAP Authentication on Debian/Ubuntu


> 🧩 Enterprise-ready Jenkins setup with secure LDAP integration for centralized authentication. Ideal for DevOps teams seeking scalable, secure, and maintainable CI/CD environments.

---

## 🔍 Project Overview

This project provides a comprehensive guide and deployment scripts to:

* Install Jenkins on a Debian-based server
* Integrate with LDAP for centralized user authentication
* Harden the Jenkins service for enterprise-grade usage
* Enable secure remote access
* Automate key steps using scripts

---

## 📌 Why This Setup?

🔒 **LDAP Integration**
Streamlines user management across multiple services with centralized credentials.

🔁 **Continuous Integration**
Enables teams to automatically build, test, and deploy software.

🌐 **Accessible Web Interface**
Runs on `http://your-server:8080`, easy to use and monitor remotely.

📦 **Extensible via Plugins**
Jenkins offers 1500+ plugins for security, SCM (Git), testing, and containers.

---

## 🧱 Architecture Diagram

```plaintext
                 +-------------------+
                 |  LDAP Server      |
                 | dc=server,dc=local|
                 +--------+----------+
                          |
                    LDAP Auth
                          |
           +--------------v-------------+
           |        Jenkins Server      |
           |  Port 8080 (Web Interface) |
           | LDAP Realm Configured      |
           +-------------+--------------+
                         |
             Admin via Web or CLI
```

---

## 💻 System Requirements

| Component    | Version / Value                  |
| ------------ | -------------------------------- |
| OS           | Debian 11 / Ubuntu 20.04+        |
| Java         | OpenJDK 17                       |
| Jenkins      | LTS Version (Debian Repo)        |
| LDAP Server  | OpenLDAP (LDAP://192.168.29.237) |
| Jenkins Port | 8080                             |
| Firewall     | UFW                              |
| RAM          | 2 GB Minimum                     |
| Disk Space   | 10 GB+                           |

---

## 🧰 Installation Guide

### ✅ Step 1: Install Java (OpenJDK 17)

```bash
sudo apt update
sudo apt install openjdk-17-jdk -y
java -version
```

---

### ✅ Step 2: Add Jenkins Repository and GPG Key

```bash
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee \
/usr/share/keyrings/jenkins-keyring.asc > /dev/null

echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
/etc/apt/sources.list.d/jenkins.list > /dev/null
```

---

### ✅ Step 3: Install Jenkins

```bash
sudo apt update
sudo apt install jenkins -y
```

---

### ✅ Step 4: Start & Enable Jenkins Service

```bash
sudo systemctl start jenkins
sudo systemctl enable jenkins
```

🔍 Check if it’s running:

```bash
sudo systemctl status jenkins
```

---

### ✅ Step 5: Configure UFW (Optional)

```bash
sudo ufw allow 8080
sudo ufw enable
sudo ufw reload
```

---

## 🌐 Access Jenkins Dashboard

Open your browser:

```bash
http://192.168.29.237:8080
OR
http://server.local:8080
```

💡 On first login, you’ll need the initial admin password:

```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

---

## 🔐 LDAP Authentication Integration (Advanced Setup)

### 🎯 Prerequisites

* LDAP Server (e.g., OpenLDAP)
* Base DN: `dc=server,dc=local`
* Admin DN: `cn=admin,dc=server,dc=local`
* LDAP Users located at: `ou=People,dc=server,dc=local`

---

### 🛠 Step-by-Step via Jenkins UI

1. Open Jenkins → **Manage Jenkins**
2. Go to **Configure Global Security**
3. Under **Security Realm**, choose **LDAP**
4. Fill in details:

| Field            | Value                         |
| ---------------- | ----------------------------- |
| Server           | `ldap://192.168.29.237:389`   |
| Root DN          | `dc=server,dc=local`          |
| User Search Base | `ou=People`                   |
| User Filter      | `uid={0}`                     |
| Manager DN       | `cn=admin,dc=server,dc=local` |
| Manager Password | `admin` or the real password  |

✅ Optional: Use group-based role strategies with `Group Search Base`.

5. Save and Logout.
6. Login with an LDAP user (e.g., `ankit`).

---

## 🧪 Testing LDAP Integration

```bash
# From Jenkins login page
Username: ankit
Password: ankit
```

If authentication succeeds, integration is successful. You can now assign roles to LDAP users.

---

## 🔄 Automation Script (install\_jenkins.sh)

```bash
#!/bin/bash

echo "Installing Java 17..."
sudo apt update
sudo apt install openjdk-17-jdk -y

echo "Adding Jenkins Repo..."
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee \
/usr/share/keyrings/jenkins-keyring.asc > /dev/null

echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
/etc/apt/sources.list.d/jenkins.list > /dev/null

echo "Installing Jenkins..."
sudo apt update
sudo apt install jenkins -y

echo "Enabling Jenkins Service..."
sudo systemctl enable jenkins
sudo systemctl start jenkins

echo "Opening firewall..."
sudo ufw allow 8080
sudo ufw reload

echo "Installation complete. Access Jenkins at: http://$(hostname -I | awk '{print $1}'):8080"
```

---

## 🔒 Security Hardening Recommendations

* Use HTTPS reverse proxy (e.g., Nginx with SSL)
* Configure Role-based Access Control (RBAC)
* Enable Audit Trail plugin
* Rotate LDAP bind credentials regularly
* Disable CLI over Remoting
* Setup scheduled backups of `/var/lib/jenkins`

---

## 📦 Useful Plugins

* 🔐 **LDAP Plugin** — Authentication
* 💼 **Role Strategy Plugin** — Granular permissions
* 🧪 **Pipeline Plugin** — CI/CD scripting
* 📊 **Monitoring Plugin** — JVM health and memory
* 🔄 **Backup Plugin** — Periodic Jenkins backups

---

## 🧠 Conclusion

With this setup, Jenkins is deployed in a scalable, secure, and centralized way using LDAP authentication — making it suitable for enterprise DevOps pipelines. LDAP ensures compliance with organizational policies and improves user management across systems.

---
