
# 🛡️ Nginx Hardening Case Study on Ubuntu

## 📌 Overview
This case study walks through best practices to **harden an Nginx web server** running on **Ubuntu 22.04**. We cover both **OS-level** and **application-level** hardening using industry-standard tools and configurations, reducing the server’s attack surface and strengthening security.

> **Target Audience:** DevOps Engineers, System Administrators, and Security Professionals.

---

## 🧭 Roadmap

### 1. Initial Setup

- Install Ubuntu 22.04
- Install Nginx and essential security tools
- Update system packages

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install nginx ufw fail2ban lynis auditd curl gnupg2 -y
```

---

### 2. OS Hardening

#### 🔹 Disable Unnecessary Services

Check and disable services that are not needed:

```bash
systemctl list-unit-files --state=enabled
sudo systemctl disable bluetooth.service
sudo systemctl disable avahi-daemon.service
```

---

#### 🔹 Kernel Parameter Hardening (sysctl)

Edit `/etc/sysctl.conf`:

```bash
sudo nano /etc/sysctl.conf
```

Add the following lines:

```bash
# IP Spoofing Protection
net.ipv4.conf.all.rp_filter = 1
net.ipv4.conf.default.rp_filter = 1

# Disable IP Forwarding
net.ipv4.ip_forward = 0

# Ignore ICMP Redirects
net.ipv4.conf.all.accept_redirects = 0
net.ipv4.conf.default.accept_redirects = 0

# Ignore Send Redirects
net.ipv4.conf.all.send_redirects = 0
net.ipv4.conf.default.send_redirects = 0
```

Apply changes:

```bash
sudo sysctl -p
```

---

#### 🔹 Firewall (UFW) Configuration

Configure UFW (Uncomplicated Firewall):

```bash
sudo ufw allow OpenSSH
sudo ufw allow 'Nginx Full'
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw enable
sudo ufw status verbose
```

---

### 3. Network Hardening

#### 🔹 Install and Configure Fail2Ban

Install and start Fail2Ban:

```bash
sudo systemctl enable fail2ban
sudo systemctl start fail2ban
```

Copy and edit the jail file:

```bash
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
sudo nano /etc/fail2ban/jail.local
```

Example SSH protection configuration:

```ini
[sshd]
enabled = true
port    = ssh
filter  = sshd
logpath = /var/log/auth.log
maxretry = 5
bantime = 3600
findtime = 600
```

Restart Fail2Ban:

```bash
sudo systemctl restart fail2ban
sudo fail2ban-client status sshd
```

---

### 4. Nginx Application Hardening

#### 🔹 Hide Nginx Version

Edit `nginx.conf`:

```bash
sudo nano /etc/nginx/nginx.conf
```

Inside the `http` block, add:

```nginx
server_tokens off;
```

Restart Nginx:

```bash
sudo systemctl reload nginx
```

---

#### 🔹 Restrict HTTP Methods

Limit HTTP methods to only GET and POST:

```nginx
location / {
    limit_except GET POST {
        deny all;
    }
}
```

---

#### 🔹 Enable HTTPS with Let's Encrypt

Install Certbot:

```bash
sudo apt install certbot python3-certbot-nginx -y
```

Obtain and install SSL certificates:

```bash
sudo certbot --nginx
```

Auto-renewal test:

```bash
sudo certbot renew --dry-run
```

---

#### 🔹 Add HTTP Security Headers

Edit your server block:

```nginx
add_header X-Frame-Options "DENY";
add_header X-Content-Type-Options "nosniff";
add_header X-XSS-Protection "1; mode=block";
add_header Strict-Transport-Security "max-age=63072000; includeSubDomains; preload" always;
add_header Referrer-Policy "no-referrer-when-downgrade";
add_header Content-Security-Policy "default-src 'self';";
```

---

#### 🔹 Enable ModSecurity (WAF)

Install ModSecurity module:

```bash
sudo apt install libnginx-mod-security
```

Create a basic ModSecurity configuration:

```bash
sudo mkdir -p /etc/nginx/modsec
sudo nano /etc/nginx/modsec/main.conf
```

Example content for `main.conf`:

```bash
SecRuleEngine On
```

Activate ModSecurity in your Nginx config:

```nginx
modsecurity on;
modsecurity_rules_file /etc/nginx/modsec/main.conf;
```

Restart Nginx:

```bash
sudo systemctl reload nginx
```

---

### 5. Auditing & Monitoring

#### 🔹 Run Lynis Security Audit

Install and run Lynis:

```bash
sudo lynis audit system
```

Check report under `/var/log/lynis-report.dat`.

---

#### 🔹 Enable Auditd

Auditd helps log important system events:

```bash
sudo systemctl enable auditd
sudo systemctl start auditd
```

Optional: Configure custom audit rules:

```bash
sudo nano /etc/audit/rules.d/audit.rules
```

---

## ✅ Summary Table

| Category           | Tool/Configuration                           |
|--------------------|----------------------------------------------|
| OS Hardening       | `ufw`, `sysctl`, `auditd`, disabling services |
| Network Hardening  | `fail2ban`, firewall rules                   |
| Nginx Hardening    | `HTTPS`, `ModSecurity`, `Security Headers`   |
| Auditing           | `Lynis`, `auditd`                            |

---

## 🔚 Conclusion

By applying these best practices, you can **substantially reduce the risk** to your Nginx server and strengthen its defenses against real-world attacks. Security hardening should always be treated as an **ongoing, iterative process** that evolves with new threats.

> 🧠 _Security is a journey, not a destination._

---

# 📋 Notes
- Always **keep your server and packages updated**.
- Perform **periodic security audits**.
- Implement **automated backup strategies**.
- Monitor server logs regularly.

---

# ✨ License
This case study is released under the [MIT License](https://opensource.org/licenses/MIT).
