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
