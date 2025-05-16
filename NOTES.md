# 🧭 Squid Forward Proxy Setup on AWS EC2

This guide walks you through setting up a **forward proxy server** using **Squid** on an **Ubuntu EC2 instance**, allowing your local machine to route HTTP traffic through it and block specific domains like Facebook.

---

## ✅ 1. Install Squid on EC2 (Ubuntu)

```bash
sudo apt update
sudo apt install squid -y
```

---

## ✅ 2. Edit Squid Configuration

Open the Squid configuration file:

```bash
sudo vim /etc/squid/squid.conf
```

Modify or add the following at the **top of the file**:

```squid
# Allow your client machine to use the proxy
acl allowed_client src <your-local-machine-public-IP>

# Block specific domains
acl blocked_sites dstdomain .facebook.com .www.facebook.com

# Enforce rules in order
http_access deny blocked_sites
http_access allow allowed_client
http_access deny all
```

> 🔁 **Order matters** – `http_access deny blocked_sites` must come **before** the allow rule.

---

## ✅ 3. Restart Squid

Restart the Squid service to apply changes:

```bash
sudo systemctl restart squid
```

Or, check for configuration syntax issues first:

```bash
sudo squid -k parse
```

---

## 🧪 Testing the Proxy

### From your client machine (local PC):

```bash
# ✅ Should work
curl -v -x http://<EC2-PUBLIC-IP>:3128 http://example.com

# ❌ Should be blocked (403 Forbidden)
curl -v -x http://<EC2-PUBLIC-IP>:3128 http://facebook.com
```

### On the proxy server (EC2):

To check access logs:

```bash
sudo tail -f /var/log/squid/access.log
```

Example entries:

```
TCP_DENIED/403 GET http://facebook.com/
TCP_MISS/200 GET http://example.com/
```

---

## 🧰 Problems & Troubleshooting

| **Issue**                     | **Cause**                        | **Solution**                                                        |
|------------------------------|----------------------------------|---------------------------------------------------------------------|
| Squid fails to start         | Invalid config syntax            | Run `sudo squid -k parse` to identify line errors                   |
| Proxy doesn't block sites    | Rule order incorrect             | Place `http_access deny blocked_sites` **before** the allow rule    |
| Request hangs using curl     | Client IP not allowed            | Make sure your public IP is in the `allowed_client` ACL             |
| Block rule not matched       | ACL domain missing `www`         | Use both `.facebook.com` and `.www.facebook.com`                    |
| Still getting 301 redirect   | Site not fully blocked           | Try blocking by IP or more subdomains if needed                     |

---

## 📌 Key Learnings

- 🔁 **Squid ACLs are order-sensitive** – `deny` rules should come **before** `allow`.
- 🧪 Always test with `curl` and monitor with `tail -f /var/log/squid/access.log`.
- 🛠️ Use `sudo squid -k parse` before restarting to avoid syntax issues.
- 🚀 Extendable: Add caching, authentication, SSL inspection, or access logs.

---

## 🧱 Optional Enhancements

### 📁 Use an external file to block multiple domains

In `/etc/squid/squid.conf`:

```squid
acl blocked_sites dstdomain "/etc/squid/blocked_sites.txt"
http_access deny blocked_sites
```

Create and populate `/etc/squid/blocked_sites.txt`:

```text
.facebook.com
.youtube.com
.instagram.com
```

Then restart Squid:

```bash
sudo systemctl restart squid
```

---

## 🖼️ Architecture Overview

```text
[Client PC] --> [Squid Proxy (EC2)] --> [Target Website]

- Blocks specified sites like Facebook
- Allows only allowed IPs
```

---

## 👩‍💻 Author

**Vaishnavi** – DevOps & Cloud Enthusiast 🚀  
Passionate about Cloud, Automation, and Building Scalable Infrastructure
