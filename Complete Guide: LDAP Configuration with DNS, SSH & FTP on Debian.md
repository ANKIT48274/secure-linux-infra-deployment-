 🛡️ Complete Guide: LDAP Configuration with DNS, SSH & FTP on Debian

In this guide, we’ll walk through setting up an LDAP server with client authentication over a secure Debian infrastructure, complete with DNS resolution, SSH login, and FTP access.

---

## 🌐 Prerequisites

* **Domain:** `server.local`
* **LDAP Server IP:** `192.168.29.237`
* **LDAP Client IP:** `192.168.29.22`
* Use static IP addresses on both server and client.

---

## 📌 Step 1: Update Hostname

On the LDAP server:

```bash
hostnamectl set-hostname ns.server.local
reboot
```
![image](https://github.com/user-attachments/assets/53027b90-74f7-4332-a217-967629332816)


---

# 🔧 DNS Configuration

DNS is essential for resolving hostnames like `server.local` and `ldap.server.local`.

### 1️⃣ Install DNS Packages

```bash
apt install bind9 dnsutils
```

### 2️⃣ Define the DNS Zone

Edit:

```bash
vim /etc/bind/named.conf.local
```
![image](https://github.com/user-attachments/assets/cd4b3e25-c2e4-4661-a9c6-d80e11fb2fbb)



### 3️⃣ Create Forward Zone File

```bash
mkdir /etc/bind/zones
vim /etc/bind/zones/forward.server.local
```

![image](https://github.com/user-attachments/assets/2fc0624a-f23d-4b64-8090-485b68a80dfc)


```bash
server.local. IN A 192.168.29.237
ldap.server.local. IN A 192.168.29.237
```

### 4️⃣ Update `/etc/hosts` (Optional)

```bash
vim /etc/hosts
```

![image](https://github.com/user-attachments/assets/37c49d02-235a-4e23-a4e1-d3bf11729fbd)


```bash
192.168.29.237 server.local ldap.server.local
```

### 5️⃣ Restart DNS Service

```bash
systemctl restart bind9
systemctl enable named.service
```
![image](https://github.com/user-attachments/assets/27791f11-4703-46a4-b8da-d6eae19b5425)


### 6️⃣ Test DNS Resolution

```bash
nslookup server.local 192.168.29.237
nslookup ldap.server.local 192.168.29.237
```
![image](https://github.com/user-attachments/assets/565f2b85-9f72-42c0-8102-2a0a5283f8d2)

### 7️⃣ Configure `resolv.conf`

```bash
vim /etc/resolv.conf
```

![image](https://github.com/user-attachments/assets/76db7437-3da2-421f-ba95-49ad8c8e29f9)

```bash
nameserver 192.168.29.237
```

✅ **DNS is ready!**

---

# 📂 LDAP Server Setup

### 1️⃣ Install Required Packages

```bash
apt install slapd ldap-utils
```
![image](https://github.com/user-attachments/assets/d10c837c-22e8-4ff4-9e20-8a47fc708a5c)
![image](https://github.com/user-attachments/assets/0543f07d-f05f-4cb9-b3e2-ed70ca523c8b)


### 2️⃣ Configure LDAP Server

```bash
dpkg-reconfigure slapd
```
![image](https://github.com/user-attachments/assets/298d4460-5273-424a-aeb7-06325dd92139)
![image](https://github.com/user-attachments/assets/4cb798aa-724c-405b-9c4a-5d9378a7f3e6)
![image](https://github.com/user-attachments/assets/276bac5c-b5fc-4b05-abe3-00d62f173f47)
![image](https://github.com/user-attachments/assets/06ff33b4-371b-4f69-9a9d-e2221165f090)
![image](https://github.com/user-attachments/assets/0f391e31-ee84-476e-bbbc-99c62da75390)


Sample answers:
* Omit OpenLDAP server configuration? → `No`
* DNS Domain Name → `server.local`
* Organization name → `server`
* Administrator Password → 123
* Confirm Password → 123
* Do you want the database removed when slapd is purged? → `No`
 * Move old database? → `Yes`

### 3️⃣ Edit LDAP Config

``

vim /etc/ldap/ldap.conf
![image](https://github.com/user-attachments/assets/b0c3dd3a-d9b3-49a1-a890-6381c85744cb)

`

```bash
BASE dc=server,dc=local
URI ldap://ldap.server.local
```

---

## ✅ LDAP Sanity Checks

Run:

```bash
ldapsearch -Q -LLL -Y EXTERNAL -H ldapi:/// -b cn=config dn
ldapsearch -x -LLL -H ldap:/// -b dc=server,dc=local dn
ldapwhoami -x
ldapwhoami -x -D cn=admin,dc=server,dc=local -W
ldapwhoami -Y EXTERNAL -H ldapi:/// -Q
```

---

### 4️⃣ Create Users Using `users.ldif`

Generate password:

```bash
slappasswd
```

Sample hashed output:

```
{SSHA}D88BroZNxE34q7ERv9fq6VVDSQ98xYWS
```

Create and edit:

```bash
vim users.ldif
```
![image](https://github.com/user-attachments/assets/f4310e9e-fa19-4b66-813a-b448c0044d08)


Add organizational units and users.

### 5️⃣ Add Users to LDAP

```bash
ldapadd -x -D cn=admin,dc=server,dc=local -W -f users.ldif
```

Verify:

```bash
ldapsearch -x -LLL -b dc=server,dc=local '(uid=ankit)' cn gidNumber
```

### 6️⃣ Set LDAP Password for Users

```bash
ldappasswd -x -D cn=admin,dc=server,dc=local -W -S uid=ankit,ou=people,dc=server,dc=local
```

---

## 🔐 PAM & NSS Configuration

### 7️⃣ Edit PAM Auth

```bash
vim /etc/pam.d/common-auth
```

Add:

```bash
auth sufficient pam_ldap.so
```
![image](https://github.com/user-attachments/assets/465ff88f-c2cc-48a3-b445-d84943e93bf2)


### 8️⃣ Enable Home Dir Creation

```bash
vim /etc/pam.d/common-session
```

Add:

```bash
session required pam_mkhomedir.so skel=/etc/skel/ umask=0022
```
![image](https://github.com/user-attachments/assets/d77fcd41-0129-424a-811d-e4eaa3662772)


### 9️⃣ Install NSS & PAM Modules

```bash
apt install nslcd libpam-ldapd
```
![image](https://github.com/user-attachments/assets/126c969c-7f74-4de5-a2a6-4a8b17962a6a)

![image](https://github.com/user-attachments/assets/319bf108-85a4-4a64-9602-e32197114562)
![image](https://github.com/user-attachments/assets/04356f83-e47f-4735-ae81-e629ce4e9d75)

During setup, use:

* URI: `ldap://ldap.server.local`
* Base DN: `dc=server,dc=local`

### 🔟 Configure Admin Binding

```bash
vim /etc/nslcd.conf

```
![image](https://github.com/user-attachments/assets/3a18a17b-16a0-4aa8-8420-19330f3a7bf3)


Add:

```bash
binddn cn=admin,dc=server,dc=local
bindpw 123
```

Restart:

```bash
systemctl restart nslcd.service
```

✅ **LDAP authentication is now active!**

---

## 🧪 Test LDAP User

```bash
getent passwd ankit
id ankit

```
![image](https://github.com/user-attachments/assets/70b9d4cf-0bfd-4fac-97dd-2b9fa00b26c3)

![image](https://github.com/user-attachments/assets/73aee1f7-2d4e-4cf6-a25e-5dfec9fc9965)

---

# 🖥️ LDAP Client Configuration

### 1️⃣ Install LDAP Client Tools

```bash
apt install libnss-ldapd libpam-ldapd ldap-utils
```
![image](https://github.com/user-attachments/assets/6e095cdf-8836-4561-971d-7517180eaa74)
![image](https://github.com/user-attachments/assets/c242b42a-90ac-48cf-9e9c-a07805a1775b)


### 2️⃣ Enable Home Directory Creation

```bash
vim /etc/pam.d/common-session
```

Add:

```bash
session required pam_mkhomedir.so skel=/etc/skel/ umask=0022
```
![image](https://github.com/user-attachments/assets/9d56a0de-7e2d-4377-b118-36067d0ed128)


### 3️⃣ Login as LDAP User

```bash
su - ankit
```
![image](https://github.com/user-attachments/assets/d3f79b1d-5b61-48a3-8e49-2d2a5fece0e7)


If `/home/ankit` is created — success!

---

# 🧩 Enable SSH Access for LDAP Users

### 1️⃣ SSHD Config Update

```bash
vim /etc/ssh/sshd_config
```

Ensure:

```bash
UsePAM yes
```

### 2️⃣ Restart SSH

```bash
systemctl restart sshd
```

### 3️⃣ SSH Login Test

```bash
ssh ankit@192.168.29.237
```
![image](https://github.com/user-attachments/assets/549394ed-f1ab-4e18-be93-821576a8fcf7)


---

# 🌐 FTP Integration with LDAP

### 1️⃣ Install FTP Server

```bash
apt install vsftpd
```

### 2️⃣ Configure PAM for vsftpd

```bash
vim /etc/pam.d/vsftpd
```

Add:

```bash
auth required pam_ldap.so
account required pam_ldap.so
session required pam_loginuid.so
```
![image](https://github.com/user-attachments/assets/2685a528-3170-4bd2-9395-413f96d0ae3e)


### 3️⃣ vsftpd Main Config

```bash
vim /etc/vsftpd.conf
```

Set:

```bash
local_enable=YES
write_enable=YES
pam_service_name=vsftpd
```

### 4️⃣ Restart FTP Service

```bash
systemctl restart vsftpd
```

### 5️⃣ FTP Client Install

```bash
apt install ftp
```

### 6️⃣ Test FTP Login (Client Side)

```bash
ftp 192.168.29.237
```

Login with `ankit` (LDAP user)
![image](https://github.com/user-attachments/assets/118c1a58-ccb8-4cb8-a5ca-4b24b21f965a)


---

## 🎉 Conclusion

With this configuration:

* ✅ LDAP users are authenticated system-wide
* ✅ DNS resolves correctly
* ✅ SSH access is LDAP-enabled
* ✅ FTP works securely via LDAP

Now your Debian infrastructure is centralized, secure, and efficient.

---
