# 🛡️ Nginx Hardening Case Study on Ubuntu 22.04

## 📌 Overview
This case study demonstrates how to harden an Nginx web server running on Ubuntu 22.04. We walk through best practices for OS-level and application-level hardening, using industry-standard tools and configurations to significantly reduce the attack surface.

Target audience: DevOps Engineers, System Administrators, and Security Professionals.

---

## 🧭 Roadmap

### 1. Initial Setup
- Install Ubuntu 22.04
- Install Nginx
- Update system packages

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install nginx ufw fail2ban lynis auditd -y
```

---

### 2. OS Hardening

#### 🔹 Disable Unnecessary Services
```bash
systemctl list-unit-files | grep enabled
sudo systemctl disable bluetooth.service
sudo systemctl disable avahi-daemon.service
```

#### 🔹 sysctl Hardening
```bash
sudo nano /etc/sysctl.conf

# Add the following:
net.ipv4.ip_forward = 0
net.ipv4.conf.all.accept_redirects = 0
net.ipv4.conf.all.send_redirects = 0
net.ipv4.conf.all.rp_filter = 1

# Apply settings:
sudo sysctl -p
```

#### 🔹 UFW Firewall Configuration
```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 'Nginx Full'
sudo ufw enable
```

---

### 3. Network Hardening

#### 🔹 Install and Configure Fail2Ban
```bash
sudo systemctl enable fail2ban
sudo systemctl start fail2ban
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
```

---

### 4. Nginx Application Hardening

#### 🔹 Hide Server Version
```bash
sudo nano /etc/nginx/nginx.conf

# Inside http block:
server_tokens off;
```

#### 🔹 Restrict HTTP Methods
```nginx
location / {
    limit_except GET POST {
        deny all;
    }
}
```

#### 🔹 Enable HTTPS with Let's Encrypt
```bash
sudo apt install certbot python3-certbot-nginx -y
sudo certbot --nginx
```

#### 🔹 Add Security Headers
```nginx
add_header X-Frame-Options DENY;
add_header X-Content-Type-Options nosniff;
add_header X-XSS-Protection "1; mode=block";
add_header Strict-Transport-Security "max-age=63072000; includeSubDomains; preload" always;
```

#### 🔹 Enable ModSecurity
```bash
sudo apt install libnginx-mod-security
sudo nano /etc/nginx/modsec/main.conf

# Activate in nginx.conf:
modsecurity on;
modsecurity_rules_file /etc/nginx/modsec/main.conf;
```

---

### 5. Auditing & Monitoring

#### 🔹 Run Lynis Security Audit
```bash
sudo lynis audit system
```

#### 🔹 Enable auditd
```bash
sudo systemctl enable auditd
sudo systemctl start auditd
```

---

## ✅ Summary Table

| Area             | Tool/Configuration                |
|------------------|------------------------------------|
| OS Hardening     | `ufw`, `sysctl`, `auditd`, `systemctl` |
| Network Security | `fail2ban`, `UFW`                 |
| Nginx Hardening  | `HTTPS`, `ModSecurity`, security headers |
| Auditing         | `Lynis`, `auditd`, log rotation    |

---

## 🔚 Conclusion
By following this roadmap, you can significantly enhance the security posture of your Nginx server. Regular audits, updates, and the use of layered security practices ensure your infrastructure remains resilient against evolving threats.

---

> 🧠 _Security is not a one-time effort—it’s an ongoing process._
