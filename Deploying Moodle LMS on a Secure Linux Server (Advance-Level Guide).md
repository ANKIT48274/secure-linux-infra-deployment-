## 🚀 Deploying Moodle LMS on a Secure Linux Server (Advance-Level Guide)


> A complete guide to setting up Moodle on a Linux server with HTTPS, user and permission hardening, and integration with LDAP for secure authentication.

---


## 📌 Objective

This project aims to securely deploy [Moodle](https://moodle.org/) — a widely used open-source learning management system — on a Debian-based Linux server. The setup ensures HTTPS access via a domain, proper file permissions, database security, and scalable configuration for institutional or enterprise-level deployment.

---

## 🌐 Server Environment

| Component       | Details            |
| --------------- | ------------------ |
| **OS**          | Debian (or Ubuntu) |
| **Server IP**   | `192.168.29.237`   |
| **Domain Name** | `server.local`     |
| **Web Server**  | Apache             |
| **DB Server**   | MariaDB            |
| **PHP Version** | 7.4+               |
| **LDAP Auth**   | Integrated via PAM |

---

## 🛠 Step 1: Install Required Packages

```bash
sudo apt update
sudo apt install apache2 mariadb-server php php-mysql libapache2-mod-php php-xml \
php-gd php-curl php-intl php-zip php-soap php-mbstring php-bcmath php-cli git unzip wget -y
```

---

## 🛠 Step 2: Configure the Moodle Database

Log into MySQL as root and execute the following:

```sql
CREATE DATABASE moodle DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER 'moodleuser'@'localhost' IDENTIFIED BY '123';
GRANT ALL PRIVILEGES ON moodle.* TO 'moodleuser'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

---

## 🛠 Step 3: Download and Set Up Moodle

```bash
cd /var/www/html
wget https://download.moodle.org/download.php/direct/stable500/moodle-latest-500.zip -O moodle.zip
unzip moodle.zip
mv moodle /var/www/html/moodle

# Set correct permissions
chown -R www-data:www-data /var/www/html/moodle
chmod -R 755 /var/www/html/moodle

# Create moodledata directory
mkdir /var/www/moodledata
chown -R www-data:www-data /var/www/moodledata
chmod -R 770 /var/www/moodledata
```

---

## 🔐 Step 4: Enable HTTPS with Self-Signed Certificate

```bash
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
-keyout /etc/ssl/private/moodle.key \
-out /etc/ssl/certs/moodle.crt
```

---

## 🌐 Step 5: Configure Apache Virtual Host

```bash
nano /etc/apache2/sites-available/moodle.conf
```

Paste this configuration:

```apache
<VirtualHost *:443>
    ServerAdmin admin@server.local
    ServerName server.local
    DocumentRoot /var/www/html/moodle

    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/moodle.crt
    SSLCertificateKeyFile /etc/ssl/private/moodle.key

    <Directory /var/www/html/moodle>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/moodle_error.log
    CustomLog ${APACHE_LOG_DIR}/moodle_access.log combined
</VirtualHost>
```

---

## 🔁 Step 6: Enable the Moodle Site and SSL Module

```bash
a2enmod ssl
a2ensite moodle.conf
a2dissite 000-default.conf
systemctl reload apache2
```

---

## 🌍 Step 7: Final Installation via Web Interface

Open your browser:

```
https://192.168.29.237/moodle
OR
https://server.local/moodle
```

Follow the installer steps:

1. Select Language
2. Confirm web paths
3. Select **MySQLi**
4. Enter DB details (`moodle`, `moodleuser`, `123`)
5. Complete installation and admin setup

---

## 🔐 Advanced: Integrating LDAP for Centralized User Auth

To make Moodle use LDAP (optional but enterprise-ready):

1. Go to `Site administration > Plugins > Authentication > Manage authentication`

2. Enable **LDAP server**

3. Set:

   * **Host URL**: `ldap://192.168.29.237`
   * **Contexts**: `ou=People,dc=server,dc=local`
   * **Bind user**: `cn=admin,dc=server,dc=local`
   * **Bind password**: `ankit`

4. Map Moodle user fields to LDAP attributes like:

   * `Username → uid`
   * `First name → givenName`
   * `Surname → sn`
   * `Email → mail`

---

## ✅ Final Verification

* Login with LDAP users to Moodle
* Check HTTPS padlock
* Test access using:

  ```bash
  getent passwd ankit
  ssh ankit@192.168.29.237
  ```

---

## 📦 Future Enhancements

* Integrate **Jenkins** for CI/CD with Moodle plugins
* Use **Let's Encrypt** for production SSL
* Add **Fail2Ban** & **UFW** firewall rules
* Implement **Geo-blocking** and **REST API hardening**

---
