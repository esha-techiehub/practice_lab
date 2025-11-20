
# ✨ MongoDB Admin User Recovery Guide  
## **Fixing Authentication Failures in the OrionData Engine Deployment**

---

# 📌 **Overview**
During the deployment of the **OrionData Engine**, backend services were unable to authenticate with MongoDB, resulting in repeated failures:

```
MongoServerError: Authentication failed.
```

A deep inspection revealed that the MongoDB data volume **contained no admin user**, and the database was created with **MongoDB 7**, while the production runtime was upgraded to **MongoDB 8**.

This guide visually walks through how the issue was diagnosed, fixed, and verified.  
Perfect for DevOps documentation, runbooks, or onboarding!

---

# 🎯 **Problem Summary**

## ❗ The Backend Could Not Connect to MongoDB  
```mermaid
flowchart LR
A[Backend Service] -->|Connect using DB_URL| B[(MongoDB)]
B -->|Auth Check| C{User Exists?}
C -->|No ❌| D[Authentication Failed]
C -->|Yes ✅| E[Successful Startup]
```

---

# 🚨 **Root Cause**
### ⚡ Version Mismatch
- MongoDB data was created using **MongoDB 7.x**.
- Production was running **MongoDB 8.x**.
- MongoDB 8 detected an old **featureCompatibilityVersion (FCV 7.0)** and refused proper startup.

### ⚠ Missing Admin User
- No administrative user existed in `admin.system.users`.
- Backend could not authenticate with its configured credentials.

---

# 🛠️ **Recovery Steps (Visual + Commands)**

---

# **1️⃣ Start Temporary MongoDB 7 Recovery Server**

### Why?
MongoDB 8 cannot read databases initialized with FCV 7, so we mount the volume using MongoDB 7 which is compatible.

```sh
docker run -d --rm   --name mongo-recovery   -v oriondb_data:/var/lib/mongo   mongo:7   mongod --dbpath /var/lib/mongo --bind_ip_all --noauth
```

### Visual:
```mermaid
flowchart LR
A[MongoDB 8] -->|Cannot Open| B((Data Volume))
B --> C[Mongo Recovery (Mongo 7)]
C -->|No Auth| D[Temporary Access Enabled]
```

---

# **2️⃣ Create New Admin User (`sysadmin`)**

```sh
docker run --rm --link mongo-recovery:mongo   mongo:7 mongosh "mongodb://mongo:27017/admin"   --eval 'db.createUser({user:"sysadmin", pwd:"SecureP@ss2025", roles:[{role:"root", db:"admin"}]})'
```

### Verify:
```sh
docker run --rm --link mongo-recovery:mongo   mongo:7 mongosh "mongodb://mongo:27017/admin"   --eval 'db.getUsers()'
```

### Visual:
```mermaid
sequenceDiagram
    participant M as MongoDB (Recovery Mode)
    participant U as Admin User
    participant D as DevOps Engineer

    D->>M: Create admin user "sysadmin"
    M->>U: Store user in admin DB
    D->>M: Verify users
    M-->>D: Returns user list ✔
```

---

# **3️⃣ Stop Recovery Server**
```sh
docker stop mongo-recovery
```

---

# **4️⃣ Update Environment Variables**

### Correct:
```
DB_URL=mongodb://sysadmin:SecureP%40ss2025@orion-mongo:27017/orion_data?authSource=admin
DB_ROOT_PASSWORD=SecureP@ss2025
```

🎉 Percent encoding required only in URLs  
🔐 Raw password used for actual DB authentication

---

# **5️⃣ Restart Production Stack**
```sh
docker compose -f docker-compose.prod.yml up -d --build
```

---

# 🎉 **Results**

## Before vs After
```mermaid
flowchart LR
A1[❌ Backend Unhealthy] --> B1[🛠 Fix Applied]
B1 --> C1[✅ Backend Healthy]
C1 --> D1[🚀 Deployment Operational]
```

| Component | Before ❌ | After ✅ |
|----------|-----------|-----------|
| MongoDB Auth | Failed | Working |
| Admin User | Missing | Restored |
| Backend Health | Unhealthy | Stable |
| Data Integrity | At Risk | Safe |

---

# 🔮 **Recommended Next Steps**
- 🆙 Perform proper FCV upgrade (7 → 8)
- 🛡 Add automatic user-provisioning script
- 📊 Add alerts for auth failures
- ♻ Schedule periodic DB backups

---

# 🏁 **Summary**
You successfully restored MongoDB operations by using a **version-matched recovery container**, recreating the missing admin user, updating environment variables, and relaunching the stack.

This ensured:
✔ Authentication success  
✔ Backend stability  
✔ Zero data loss

---

# 🎉 **Done!**
This README is ready for GitHub, wikis, or onboarding docs.

