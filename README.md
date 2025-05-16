# 🚀 Forward Proxy Using Squid (Ubuntu EC2 Setup)

This project demonstrates how to configure a **Forward Proxy Server** using **Squid** on an AWS EC2 Ubuntu instance. This proxy filters web access based on client IP and domains like Facebook or YouTube. You can expand this for logging, caching, or authentication later.

---

## 🧠 What is a Forward Proxy?

A **forward proxy** sits between **internal clients** and the **external internet**.

It is mainly used for:

- Controlling internet access (e.g., block social media)
- Caching frequently accessed sites
- Monitoring and logging internet activity
- Enhancing security and anonymity

---

## 🏗️ Setup Architecture

- **Client:** Your personal Ubuntu machine (or any internet-connected device)
- **Proxy Server:** Ubuntu EC2 instance running **Squid**
- **Flow:** Client request → EC2 Squid Proxy → Target Website (if allowed)

---

## ⚙️ Step-by-Step Setup

### ✅ 1. Install Squid on EC2

```bash
sudo apt update
sudo apt install squid -y
