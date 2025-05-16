🚀 Forward Proxy Using Squid (Ubuntu EC2 Setup)
This guide explains how to set up a Forward Proxy Server using Squid on an AWS EC2 Ubuntu instance. It includes basic site blocking, client IP-based access, and real-world troubleshooting.

🧠 What is a Forward Proxy?
A forward proxy acts as a middleman between a client (e.g., your browser or curl) and the internet. It routes outbound traffic and can:

Control internet access

Cache frequently accessed content

Enforce content filtering (e.g., block Facebook, YouTube)

Log user traffic for auditing

🏗️ Setup Architecture
Client: Your personal Ubuntu machine or another system

Proxy Server: AWS EC2 Ubuntu instance running Squid

Client’s internet requests → Forward Proxy (Squid) → Destination website

⚙️ Step-by-Step Setup
✅ 1. Install Squid on EC2 (Ubuntu)
bash
Copy
Edit
sudo apt update
sudo apt install squid -y
✅ 2. Edit Squid Configuration
bash
Copy
Edit
sudo vim /etc/squid/squid.conf
Modify/Add the following at the top of the file:

squid
Copy
Edit
# Allow your client machine to use the proxy
acl allowed_client src <your-local-machine-public-IP>

# Block specific domains
acl blocked_sites dstdomain .facebook.com .www.facebook.com

# Enforce rules in order
http_access deny blocked_sites
http_access allow allowed_client
http_access deny all
🔁 Order matters – http_access deny blocked_sites should come before allowing the client.

✅ 3. Restart Squid
bash
Copy
Edit
sudo systemctl restart squid
Or if there's an issue:

bash
Copy
Edit
sudo squid -k parse  # Check for syntax errors
🧪 Testing the Proxy
From the client machine (your local PC):
bash
Copy
Edit
curl -v -x http://<EC2-IP>:3128 http://example.com     # ✅ Should work
curl -v -x http://<EC2-IP>:3128 http://facebook.com    # ❌ Should return 403 Forbidden
On the proxy server:
bash
Copy
Edit
sudo tail -f /var/log/squid/access.log
Check for entries like:

log
Copy
Edit
TCP_DENIED/403 GET http://facebook.com/
TCP_MISS/200 GET http://example.com/
🧰 Problems & Troubleshooting
Issue	Cause	Solution
Squid fails to start	Invalid config syntax	Run sudo squid -k parse to identify line errors
Proxy doesn't block sites	Rule order incorrect	Place http_access deny blocked_sites before allow rule
Request hangs using curl	Client IP not allowed	Make sure correct IP is set in allowed_client ACL
Block rule not matched	ACL domain missing www	Use both .facebook.com and .www.facebook.com
Still getting 301 (redirect)	Site not fully blocked	Try blocking by IP or more subdomains if needed

📌 Key Learnings
Squid ACLs are powerful but order-sensitive

Always test using curl and check logs in /var/log/squid/access.log

Use squid -k parse to prevent crashes during config changes

You can build on this to add authentication, logging, caching, or SSL inspection

🧱 Optional Enhancements
Use a file to block multiple domains:

squid
Copy
Edit
acl blocked_sites dstdomain "/etc/squid/blocked_sites.txt"
http_access deny blocked_sites
Contents of /etc/squid/blocked_sites.txt:

Copy
Edit
.facebook.com
.youtube.com
.instagram.com
