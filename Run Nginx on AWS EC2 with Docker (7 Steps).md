# 🚀 From Zero to Live: Run Nginx on AWS EC2 with Docker (7 Steps)

This lab demonstrates how to set up and run an **Nginx web server** on an **Amazon Linux EC2 instance** using Docker.  
It includes all steps — from installing Docker to cleaning up.

---

## 0) Prep – Security Group + SSH

- **Allow inbound rules in EC2 Security Group:**
  - `TCP 80` (HTTP) → from your IP or `0.0.0.0/0` for demo
  - (Optional) `TCP 8080` for custom port test
- **SSH into EC2 instance:**
```bash
ssh -i /path/to/key.pem ec2-user@<EC2_PUBLIC_IP>
```

**General Example:**  
Opening shop’s main door (HTTP) and staff entry (SSH).  

**Technical Takeaway:**  
Inbound rules decide accessibility; use **public IP** for browser access.

---

## 1) Install Docker Engine (Amazon Linux)

```bash
sudo dnf update -y
sudo dnf install -y docker
sudo systemctl enable --now docker
sudo usermod -aG docker $USER
newgrp docker
docker version
```

- `dnf install docker` → Installs Docker Engine + CLI  
- `systemctl enable --now` → Starts Docker now and on boot  
- `usermod + newgrp` → Run Docker without `sudo`  

**General Example:**  
Install the kitchen and get keys to use it.  

**Technical Takeaway:**  
Check `docker version` → both **client & server** confirm Docker daemon is running.

---

## 2) First Run: Hello World

```bash
docker run --rm hello-world
```

- Validates Docker Hub connectivity.  
- `--rm` removes container after exit.

**General Example:**  
Flip a light switch to confirm electricity.  

**Technical Takeaway:**  
Ensures Docker daemon can pull and run containers.

---

## 3) Pull a Real Image (Nginx) & List Images

```bash
docker pull nginx:alpine
docker images
```

- `nginx:alpine` → lightweight, fast.  
- `docker images` → lists cached images.

**General Example:**  
Download a compact recipe book.  

**Technical Takeaway:**  
Shows image ID, size, and repo for later use.

---

## 4) Run & Access a Web Server

Start Nginx in background:
```bash
docker run -d --name web1 -p 80:80 nginx:alpine
```

Verify running container:
```bash
docker ps
```

Test inside EC2:
```bash
curl -I localhost
curl localhost
```

Test from browser:  
👉 `http://<EC2_PUBLIC_IP>/`

**General Example:**  
Park a food truck (container), open the serving window at stall #80 (port 80).  

**Technical Takeaway:**  
- `-d` → detached mode  
- `-p 80:80` → maps host port to container port  
- `curl -I` → health check headers only  
- `curl` → fetch page content  

---

## 5) Look Inside: Logs, Inspect, Shell

**Logs:**
```bash
docker logs web1
docker logs -f web1   # follow mode
```

**Inspect details (JSON):**
```bash
docker inspect web1 | head -n 40
```

**Open shell inside container:**
```bash
docker exec -it web1 sh
cd /usr/share/nginx/html
ls -l
echo "Hello from EC2 + Docker" > hello.html
exit
```

Now hit:  
👉 `http://<EC2_PUBLIC_IP>/hello.html`

**General Example:**  
- Logs = CCTV footage of the shop  
- Inspect = X-ray of the truck  
- Exec = step into the truck’s kitchen  

**Technical Takeaway:**  
Logs confirm requests/errors, Inspect reveals config, Exec enables debugging.

---

## 6) Stop, Start, Restart, Status

```bash
docker stop web1         # graceful stop
docker ps                # check running containers
docker ps -a             # all (running + stopped)
docker start web1        # start again
docker restart web1      # restart in one step
```

**General Example:**  
Close the stall, reopen later.  

**Technical Takeaway:**  
Container lifecycle ≠ image lifecycle. `-a` shows stopped ones too.

---

## 7) Remove Container & Image (Cleanup)

```bash
docker stop web1
docker rm web1                 # remove container
docker rmi nginx:alpine        # remove image
docker system df               # check disk usage
docker system prune -f         # clean unused
docker system prune -a -f      # aggressive clean (all unused images)
```

**General Example:**  
Clean the kitchen and free pantry space.  

**Technical Takeaway:**  
Remove **containers first**, then images. Prune to reclaim space.

---

## ✅ Bonus Tips

- **Make your page default `/`:**
```bash
docker exec -it web1 sh -c 'cd /usr/share/nginx/html && mv index.html index.html.bak && mv hello.html index.html'
```

- **Persistent edits with bind mount:**
```bash
mkdir -p ~/site
echo "Hello from EC2 + Docker (persistent)" > ~/site/index.html
docker rm -f web1
docker run -d --name web1 -p 80:80 -v ~/site:/usr/share/nginx/html:ro nginx:alpine
```

---

## 📌 Conclusion
In 7 steps you:  
- Installed Docker  
- Ran containers (hello-world, nginx)  
- Exposed a web server  
- Explored logs, inspect, and shell  
- Stopped, restarted, and removed resources  

This is the foundation of **Docker + AWS EC2 labs** for developers and DevOps engineers.

---

### 🔖 Tags
`#docker #aws #ec2 #nginx #devops #containers #linux #cloud`
