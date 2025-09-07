#  Lab Activity: Launching EC2 Instance & Enabling Web Server (Apache)

## Objective
- Launch an **Amazon EC2 instance**.
- Install and configure **Apache Web Server (httpd)**.
- Deploy a sample webpage and access it via **public IP**.

---

##  Prerequisites
- AWS Account
- Basic knowledge of Linux commands
- Security Group allowing **port 80 (HTTP)** inbound traffic

---

##  Steps

### 1. Launch EC2 Instance
- Go to **AWS Management Console → EC2 → Launch Instance**.
- Choose:
  - **AMI:** Amazon Linux 2
  - **Instance Type:** t2.micro (Free Tier)
  - **Key Pair:** Create or use an existing one
  - **Security Group:** Allow **HTTP (80)** and **SSH (22)**

Connect to the instance:
```bash
ssh -i "your-key.pem" ec2-user@<Public-IP>
```

---

### 2. Update Packages (Optional but Recommended)
```bash
sudo yum update -y
```

---

### 3. Install Apache Web Server
```bash
sudo yum install httpd -y
```

---

### 4. Start Apache Service
```bash
sudo systemctl start httpd
sudo systemctl status httpd
```

- Ensure status shows **"active (running)"**

---

### 5. Enable Apache on Boot
```bash
sudo systemctl enable httpd
```

---

### 6. Deploy a Sample Webpage
Navigate to Apache’s default directory:
```bash
cd /var/www/html
ls
```

Create an `index.html` file:
```bash
sudo vim index.html
```

Example content:
```html
<html>
  <head><title>My Web Server</title></head>
  <body>
    <h1>Hello from EC2 Apache Web Server!</h1>
  </body>
</html>
```

---

### 7. Test Web Server
Use **curl**:
```bash
curl http://<Public-IP>
```

Or access from browser:
```
http://<Public-IP>
```

---

### 8. Verify DNS Resolution (Optional)
```bash
nslookup <Public-IP>
```

---

### 9. History of Commands Used
```bash
history
```

---

## ✅ Expected Output
- Webpage displays **“Hello from EC2 Apache Web Server!”**
- Apache service runs successfully
- Public IP accessible via browser

---

## 🔑 Key Learnings
- How to launch and connect to EC2 instance
- How to install and configure Apache (httpd)
- Hosting static web content on AWS EC2



=====================================================================
# Launch a Simple Apache Web Server on Amazon EC2 (Lab)

Spin up an EC2 instance and serve a basic `index.html` over HTTP using Apache (`httpd`).  
This lab is beginner-friendly and takes ~15–20 minutes.

---

## What You’ll Learn
- Launch and SSH into an EC2 instance
- Install and run Apache (`httpd`)
- Serve a static web page from `/var/www/html`
- Verify from a browser using the instance’s public IP
- Troubleshoot port 80 / network issues

---

## Prerequisites
- **AWS account** with permission to create EC2 resources
- **Key pair (.pem)** to SSH into the instance
- **VPC/Subnet with Internet** (public subnet + Internet Gateway + route `0.0.0.0/0`)
- **Security Group** with at least:
  - Inbound **SSH (22)** from your IP
  - Inbound **HTTP (80)** from `0.0.0.0/0` (and `::/0` if you want IPv6)
- A terminal/SSH client (macOS/Linux: Terminal; Windows: PowerShell)

> 💡 If you forget to open port 80 now, you can add it later during **Troubleshooting**.

---

## Quick Architecture

```
Your Browser (HTTP) ──> Public Internet ──> EC2 (Public IP) ──> httpd ──> /var/www/html/index.html
```

---

## Steps

### 1) Launch EC2
**What & Why:** Create a small Linux VM to host the web server.

- AMI: **Amazon Linux 2** (or **Amazon Linux 2023**)
- Instance type: **t2.micro** or **t3.micro** (Free Tier friendly)
- Network: Public subnet with auto-assign public IP
- Security Group: Allow **SSH (22)** and **HTTP (80)**
- Key pair: Select or create one; download the `.pem`

### 2) Connect via SSH
**What & Why:** Access the instance to run setup commands.

```bash
# Mac/Linux
chmod 400 your-key.pem
ssh -i your-key.pem ec2-user@<EC2_PUBLIC_IP>

# Windows (PowerShell with OpenSSH)
ssh -i your-key.pem ec2-user@<EC2_PUBLIC_IP>
```

- **`ec2-user`** is the default user on Amazon Linux.
- Replace `<EC2_PUBLIC_IP>` with your instance’s public IP from the EC2 console.

### 3) Update the package index (Recommended)
**What & Why:** Get the latest package metadata before installing.

```bash
# Amazon Linux 2
sudo yum update -y

# Amazon Linux 2023 (uses dnf)
# sudo dnf update -y
```

### 4) Install Apache HTTP Server (`httpd`)
**What & Why:** Apache (`httpd`) will serve web content on port 80.

```bash
# Amazon Linux 2
sudo yum install -y httpd

# Amazon Linux 2023 (dnf)
# sudo dnf install -y httpd
```

### 5) Start and enable the service
**What & Why:** Start the server now and configure it to auto-start on reboot.

```bash
sudo systemctl start httpd
sudo systemctl enable httpd
sudo systemctl status httpd --no-pager
```

- `start`: runs the service immediately
- `enable`: ensures it starts after a reboot
- `status`: confirms it’s active (running)

### 6) Add your web page (`index.html`)
**What & Why:** Place your site’s home page in Apache’s default web root: `/var/www/html`.

```bash
cd /var/www/html

# Create a simple landing page
sudo tee /var/www/html/index.html > /dev/null <<'EOF'
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>EC2 Web Server</title>
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <style>body{font-family:Arial,Helvetica,sans-serif;margin:40px}h1{color:#2c3e50}</style>
</head>
<body>
  <h1>It works! 🚀</h1>
  <p>You are viewing this page from Apache (httpd) on an Amazon EC2 instance.</p>
</body>
</html>
EOF
```

- `tee` writes the file as root in one command.
- Default doc root is `/var/www/html` on Amazon Linux.

### 7) Test locally on the instance (optional but useful)
**What & Why:** Verify that `httpd` is serving content before testing from the internet.

```bash
curl -I http://localhost
curl http://localhost
```

- `-I` prints HTTP headers (expect `HTTP/1.1 200 OK`).
- The second command should print your HTML.

### 8) Browse from your machine
**What & Why:** Confirm public access works.

Open your browser and go to:

```
http://<EC2_PUBLIC_IP>
```

You should see **“It works! 🚀”**.

---

## Troubleshooting (Port 80 / Not Loading)

### A) Security Group doesn’t allow HTTP (80)
- Go to **EC2 Console → Instances → (your instance) → Security → Security groups**
- Edit **Inbound rules** → **Add rule**:
  - Type: **HTTP**
  - Port: **80**
  - Source: **0.0.0.0/0** (and **::/0** for IPv6 if needed)
- Save rules and refresh your browser.

### B) Apache not listening on 80
```bash
sudo systemctl status httpd --no-pager
sudo ss -lntp | grep ':80'      # should show httpd (listening)
sudo journalctl -u httpd --since "-10 min" --no-pager  # recent logs
```

If not running:
```bash
sudo systemctl start httpd
sudo systemctl enable httpd
```

### C) OS Firewall (if using another distro)
- Amazon Linux typically has no blocking firewall by default.
- On RHEL/CentOS with firewalld:
  ```bash
  sudo firewall-cmd --add-service=http --permanent
  sudo firewall-cmd --reload
  ```
- On Ubuntu with UFW:
  ```bash
  sudo ufw allow http
  sudo ufw reload
  ```

### D) Networking / Public Internet path
- Ensure the instance has a **Public IP** (or attach an **Elastic IP**).
- Subnet route table must have a **route to Internet Gateway** (`0.0.0.0/0 → igw-...`).
- Network ACLs should allow ephemeral traffic (default VPC NACLs are fine).

### E) File/Permissions
- Ensure the file exists and is readable:
  ```bash
  ls -l /var/www/html/index.html
  sudo chmod 644 /var/www/html/index.html
  sudo chmod 755 /var/www/html
  ```

---

## (Optional) Ubuntu/Debian Variant
If you launched **Ubuntu** instead of Amazon Linux:

```bash
sudo apt update -y
sudo apt install -y apache2
sudo systemctl start apache2
sudo systemctl enable apache2
echo "<h1>It works on Ubuntu!</h1>" | sudo tee /var/www/html/index.html
```

Service name is `apache2` (not `httpd`).

---

## Clean Up
To stop the web server and avoid charges:
```bash
sudo systemctl stop httpd
# Then stop or terminate the EC2 instance from the AWS Console when done
```

---

## FAQ
**Q: Do I need to reboot the instance after installing httpd?**  
A: No. Starting the service is enough. We enable it so future reboots automatically start it.

**Q: Why do I see my old page after editing `index.html`?**  
A: Browser caching. Hard refresh (`Ctrl+F5`) or clear cache.

**Q: My public IP changed after a stop/start.**  
A: That’s expected. Use an **Elastic IP** to keep a static IP.

---

