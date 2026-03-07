# 🧠 Project – Nginx Load Balancer Configuration

## 📌 Task Overview

The Production Support team noticed that traffic to one of their websites has been increasing day by day. Due to this increase in traffic, the website performance started degrading.

To solve this issue, the team decided to deploy the application on a **High Availability architecture** using a **Load Balancer** in the **Datacenter**.

Most of the migration was completed, but the **Load Balancer (LBR) configuration** was still pending.

The task was to configure **Nginx as a Load Balancer** on the server **lb01** so that traffic could be distributed across multiple application servers.

---

# 🏗 Infrastructure Details

| Server Name | IP Address    | Hostname                          | User   | Purpose              |
| ----------- | ------------- | --------------------------------- | ------ | -------------------- |
| app01     | 172.16.238.10 | app01.os.corp.com   | tony   | Application Server 1 |
| app02     | 172.16.238.11 | app02.os.corp.com   | steve  | Application Server 2 |
| app03     | 172.16.238.12 | app03.os.corp.com   | banner | Application Server 3 |
| lb01      | 172.16.238.14 | lb01.os.corp.com    | loki   | Load Balancer Server |
| jump_host   | Dynamic       | jump_host.os.corp.com | thor   | Jump Host            |

---

# 🎯 Objective

Configure the **Nginx Load Balancer** with the following requirements:

* Install **nginx** on the LBR server if not installed.
* Configure load balancing using **all application servers**.
* Modify only the main configuration file:

```
/etc/nginx/nginx.conf
```

* Do **not modify Apache port configuration** on the application servers.
* Ensure Apache service is **running on all application servers**.
* Test the setup using:

```
curl http://lb01:80
```

---

# ⚙️ Step 1 – Login to Load Balancer Server

```
ssh loki@lb01
```

---

# 📦 Step 2 – Install Nginx

Check if nginx is installed:

```
nginx -v
```

If not installed:

```
sudo yum install nginx -y
```

---

# ⚙️ Step 3 – Configure Nginx Load Balancer

Edit the configuration file:

```
sudo vi /etc/nginx/nginx.conf
```

Add the following configuration inside the **http block**.

```
upstream backend_servers {
    server stapp01:80;
    server stapp02:80;
    server stapp03:80;
}

server {
    listen 80;

    location / {
        proxy_pass http://backend_servers;
    }
}
```
Check Nginx Configuration
```
sudo nginx -t
```
Expected output:
```
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

---

# 🔄 Step 4 – Restart Nginx

```
sudo systemctl restart nginx
```

Verify nginx status:

```
sudo systemctl status nginx
```

---

# ❌ Problem Encountered

After configuring nginx and running the test command:

```
curl http://stlb01
```

The following error appeared:

```
502 Bad Gateway
```

---

# 🔍 Troubleshooting

To identify the issue, backend connectivity was tested from the load balancer.

```
curl app01
```

Output:

```
curl: (7) Failed to connect to stapp01 port 80: Connection refused
```
Login to app01 server
```
ssh tony@app01
```
check Apache service status,
further inspection of Apache service showed:

```
sudo systemctl status httpd
```

Output:

```
Server configured, listening on: port 6400
```

This revealed that **Apache was running on port 6400 instead of port 80**.

---

# 🧠 Root Cause

The Nginx configuration was forwarding traffic to:

```
app01:80
app02:80
app03:80
```

However, the Apache web servers were listening on:

```
port 6400
```

This mismatch caused the **502 Bad Gateway error**.

---

# ✅ Solution

Update the Nginx upstream configuration to match the Apache port.
On lb01 server /etc/nginx/nginx.conf
```
upstream backend_servers {
    server stapp01:6400;
    server stapp02:6400;
    server stapp03:6400;
}
```
After updating this is the final `nginx.conf` file
```
upstream backend_servers {
    server stapp01:6400;
    server stapp02:6400;
    server stapp03:6400;
}

server {
    listen 80;

    location / {
        proxy_pass http://backend_servers;
    }
}
```

Restart nginx:

```
sudo systemctl restart nginx
```

---

# 🧪 Verification

Test the load balancer:

```
curl http://stlb01:80
```

Output:

```
Welcome to Corp !
```

This confirms that:

* Nginx is successfully receiving requests.
* Traffic is forwarded to backend application servers.
* The application servers are responding correctly.

---

# 🔎 Optional Load Balancing Test

Run multiple requests to observe load distribution:

```
for i in {1..10}; do curl http://stlb01; echo; done
```

---

# 🧠 DevOps Concepts Demonstrated

* Nginx Reverse Proxy
* Load Balancing Configuration
* Backend Service Debugging
* High Availability Architecture
* Apache HTTP Server Management
* Troubleshooting 502 Errors

---

# 🏁 Conclusion

The Nginx Load Balancer was successfully configured on **lb01**, enabling traffic distribution across **three application servers**. After correcting the backend port configuration, the system began functioning correctly, improving scalability and availability of the application.

---
