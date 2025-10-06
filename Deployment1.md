
# 🚀 Amazon Project Deployment Guide with Nginx Reverse Proxy & Docker 🐳

**Company:** Amazon (Demo)  
**Domain:** [https://eshwari.com](https://eshwari.com)  
**Technology:** Python (FastAPI) + React + Docker + Nginx Reverse Proxy + SSL  
**Author:** DevOps Team  

---

## 📜 Table of Contents

- [1. Introduction](#1-introduction)
- [2. Architecture Overview](#2-architecture-overview)
- [3. Project Structure](#3-project-structure)
- [4. Step-by-Step Deployment](#4-step-by-step-deployment)
- [5. Docker Setup](#5-docker-setup)
- [6. Nginx Reverse Proxy Configuration](#6-nginx-reverse-proxy-configuration)
- [7. SSL Configuration](#7-ssl-configuration)
- [8. Health Checks & Debugging](#8-health-checks--debugging)
- [9. Final Workflow Diagram](#9-final-workflow-diagram)

---

## 1. 📌 Introduction

This guide explains how to deploy a **production-ready full‑stack application** using:

- 🐳 Docker containers for frontend and backend
- 🌐 Nginx as a **reverse proxy** and SSL terminator
- 🔐 HTTPS with Let's Encrypt
- 🧰 FastAPI backend and React frontend

> This documentation uses dummy company **Amazon** and domain **eshwari.com** for demonstration.

---

## 2. 🏗️ Architecture Overview

```
[User Browser] ---> [Nginx Reverse Proxy] ---> [Frontend Container :3200]
                                   |
                                   ---> [Backend Container :8500] ---> [Database]
```

✅ Nginx listens on port `80`/`443` and forwards traffic:  
- `/` → Frontend container (React)  
- `/api/` → Backend container (FastAPI)

---

## 3. 📂 Project Structure

```
amazon-deployment/
├─ docker-compose.yml
├─ nginx/
│  ├─ nginx.conf
│  └─ conf.d/
│     └─ eshwari.conf
├─ frontend/
│  └─ Dockerfile
└─ backend/
   └─ Dockerfile
```

---

## 4. ⚙️ Step-by-Step Deployment

### ✅ Step 1: Connect to Server
```bash
ssh -i your-key.pem ubuntu@<server-ip>
sudo apt update && sudo apt upgrade -y
sudo apt install -y nginx docker.io docker-compose git certbot python3-certbot-nginx
```

### ✅ Step 2: Clone & Build
```bash
git clone https://github.com/amazon/eshwari-app.git
cd eshwari-app
docker-compose build
docker-compose up -d
```

### ✅ Step 3: Verify Containers
```bash
docker ps
```

---

## 5. 🐳 Docker Setup

### 📁 docker-compose.yml

```yaml
version: "3.8"

services:
  frontend:
    build: ./frontend
    container_name: eshwari_frontend
    expose:
      - "3200"
    networks:
      - webnet
    restart: unless-stopped

  backend:
    build: ./backend
    container_name: eshwari_backend
    expose:
      - "8500"
    networks:
      - webnet
    restart: unless-stopped

  nginx:
    image: nginx:1.27-alpine
    container_name: reverse_proxy
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
      - ./nginx/conf.d:/etc/nginx/conf.d:ro
      - ./certs:/etc/letsencrypt:ro
    depends_on:
      - frontend
      - backend
    networks:
      - webnet
    restart: unless-stopped

networks:
  webnet:
    driver: bridge
```

---

## 6. 🌐 Nginx Reverse Proxy Configuration

### 📁 nginx/conf.d/eshwari.conf

```nginx
server {
    listen 80;
    server_name eshwari.com www.eshwari.com;

    location / {
        proxy_pass http://frontend:3200;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    location /api/ {
        proxy_pass http://backend:8500;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### ✅ Test & Reload
```bash
sudo nginx -t
sudo systemctl reload nginx
```

---

## 7. 🔐 SSL Configuration (HTTPS)

Enable HTTPS using **Let's Encrypt**:

```bash
sudo certbot --nginx -d eshwari.com -d www.eshwari.com
sudo systemctl reload nginx
```

Your site is now live on:  
👉 `https://eshwari.com`  
👉 `https://www.eshwari.com`

---

## 8. 🧪 Health Checks & Debugging

### 📊 Check Nginx Status
```bash
sudo systemctl status nginx
sudo nginx -t
```

### 🐳 Check Container Logs
```bash
docker ps
docker logs eshwari_frontend
docker logs eshwari_backend
```

### 📁 Monitor Logs
```bash
sudo tail -f /var/log/nginx/access.log
sudo tail -f /var/log/nginx/error.log
```

---

## 9. 📊 Final Workflow Diagram (Textual)

```
🌍 User (Browser)
     |
     v
🔄 Nginx Reverse Proxy (Port 80/443)
     ├── /  → Frontend Container (Port 3200)
     └── /api → Backend Container (Port 8500)
                      |
                      └── 📦 Database (internal only)
```

---

## 🎉 Conclusion

✅ You have successfully deployed a full-stack project using:  
- **Nginx Reverse Proxy** for routing and SSL termination  
- **Docker containers** for backend and frontend  
- **Let's Encrypt** for HTTPS  
- **Zero downtime reload** for smooth updates  

This is the same architecture companies like **Amazon, Netflix, and Meta** use in real-world production 🚀.

---

💡 **Pro Tip:** Always test configuration before reloading:
```bash
sudo nginx -t && sudo systemctl reload nginx
```

---

🧑‍💻 **Author:** Amazon DevOps Team  
📅 Version: 1.0  
🌐 Domain: [https://eshwari.com](https://eshwari.com)

---

> ⚠️ **Note:** All details in this README are for educational/demo purposes only. Replace domains, keys, and credentials with real ones in production.
