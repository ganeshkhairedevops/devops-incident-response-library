# Incident Report: Nginx SSL Setup Issue on App Server 3

## Overview

During application preparation on **App Server 3 (stapp03)** , the production support team identified that **SSL was not correctly configured on Nginx**, causing HTTPS requests from the jump host to fail.

The objective was to configure Nginx with a **self-signed SSL certificate**, deploy a test webpage, and ensure it was reachable from the jump host.

---

# Infrastructure Details

| Server    | IP            | Role               |
| --------- | ------------- | ------------------ |
| stapp03   | 172.16.238.12 | Application Server |
| jump_host | Dynamic       | Access server      |

---

# Problem

When testing HTTPS connectivity from the jump host:

```bash
curl -Ik https://172.16.238.12
```

The connection initially failed due to one or more of the following issues:

* SSL certificate not deployed correctly
* Nginx configuration missing SSL server block
* Nginx service not running
* Firewall blocking port **443**
* Default nginx page still present instead of required test page

---

# Root Cause

The root cause was identified as:

1. **SSL certificate and key were not properly deployed in Nginx configuration**
2. **Firewall rules did not allow incoming traffic on port 443**
3. Nginx configuration needed validation and restart

---

# Solution

## 1. Install and Enable Nginx

```bash
sudo yum install nginx -y
sudo systemctl enable nginx
sudo systemctl start nginx
```

---

## 2. Move SSL Certificate and Key

Create SSL directory:

```bash
sudo mkdir -p /etc/nginx/ssl
```

Move files:

```bash
sudo mv /tmp/sslcert.crt /etc/nginx/ssl/
sudo mv /tmp/sslcert.key /etc/nginx/ssl/
```

---

## 3. Configure Nginx for HTTPS

Edit nginx configuration:

```bash
sudo vi /etc/nginx/nginx.conf
```

Add SSL server block inside the `http {}` section:

```nginx
server {
    listen 443 ssl;
    server_name _;

    ssl_certificate /etc/nginx/ssl/sslcert.crt;
    ssl_certificate_key /etc/nginx/ssl/sslcert.key;

    root /usr/share/nginx/html;
    index index.html;
}
```

---

## 4. Create Web Page

```bash
echo "Welcome!" | sudo tee /usr/share/nginx/html/index.html
```

---

## 5. Validate Nginx Configuration

```bash
sudo nginx -t
```

Expected output:

```
syntax is ok
test is successful
```

---

## 6. Restart Nginx

```bash
sudo systemctl restart nginx
```

---

## 7. Allow HTTPS in Firewall

```bash
sudo iptables -I INPUT -p tcp --dport 443 -j ACCEPT
sudo iptables-save | sudo tee /etc/sysconfig/iptables
```

---

# Verification

## Check Nginx Listening Port

```bash
sudo ss -tulnp | grep 443
```

Expected output:

```
LISTEN 0 511 0.0.0.0:443 nginx
```

---

## Test from Jump Host

```bash
curl -k https://172.16.238.12
```

Expected output:

```
Welcome!
```

Check headers:

```bash
curl -Ik https://172.16.238.12
```

Expected:

```
HTTP/1.1 200 OK
Server: nginx
```

---

# Result

The issue was successfully resolved:

* Nginx installed and configured
* SSL certificate deployed
* HTTPS enabled on port **443**
* Web page successfully served
* Connectivity verified from jump host

---

# Lessons Learned

* Always validate nginx configuration using `nginx -t` before restarting.
* Ensure firewall rules allow application ports.
* For self-signed certificates, use `curl -k` for testing.
* Document infrastructure fixes for future troubleshooting.

---

# Status

✅ **Resolved**
