# 🚀 Task: Push Local Frontend & Backend Code to Azure DevOps Repository with Branches

This task is about learning how to **push local project files from VS Code (your laptop) into an Azure DevOps repository**.  
Two interns will work on different parts of the project:  

- ** developer 1 → Frontend code → `feature/frontend` branch**  
- ** developer 2 → Backend code → `feature/backend` branch**  

---

## 📌 Prerequisites
- [Git](https://git-scm.com/downloads) installed on your laptop.  
- Azure DevOps repo URL (provided by the organization).  
- VS Code with your project files ready.  

---

## 🛠️ Steps to Follow

### **1. Clone the Azure Repo**
```bash
git clone https://****url provided by organisation********
cd YourRepoName
```
📖 **Explanation:**  
This copies the remote repo from Azure DevOps into your laptop so you can start working on it.  

---

### **2. Create Your Folder**
- **Frontend developer:**
```bash
mkdir frontend
# Add/paste frontend source code files here
```

- **Backend developer:**
```bash
mkdir backend
# Add/paste backend source code files here
```

📖 **Explanation:**  
We create separate folders (`frontend` and `backend`) to keep the project organized.  

---

### **3. Create and Switch to Your Branch**
- **Frontend developer:**
```bash
git checkout -b feature/frontend
```

- **Backend developer:**
```bash
git checkout -b feature/backend
```

📖 **Explanation:**  
A branch allows each developer to work independently without affecting the `main` branch.  

---

### **4. Add Your Files to Git**
- **Frontend:**
```bash
git add frontend/
```

- **Backend:**
```bash
git add backend/
```

📖 **Explanation:**  
`git add` stages your files, meaning Git is now tracking them for the next commit.  

---

### **5. Commit Your Changes**
- **Frontend:**
```bash
git commit -m "Added frontend source code"
```

- **Backend:**
```bash
git commit -m "Added backend source code"
```

📖 **Explanation:**  
`git commit` takes a snapshot of your staged files with a descriptive message.  

---

### **6. Push Your Branch to Azure Repo**
- **Frontend:**
```bash
git push origin feature/frontend
```

- **Backend:**
```bash
git push origin feature/backend
```

📖 **Explanation:**  
`git push` uploads your branch and commits to the Azure DevOps repository.  

---

### **7. Verify in Azure DevOps**
1. Go to **Repos → Branches** in your Azure DevOps project.  
2. You should see:  
   - `feature/frontend`  
   - `feature/backend`  
3. From here, create a **Pull Request (PR)** to merge into `main`.  

---

## 📂 Final Project Structure
```
YourRepoName/
│
├── frontend/   # All frontend source code
│
├── backend/    # All backend source code
│
└── README.md   # This instruction file
```
# 🌱 Branching Strategy Explained

This document explains common Git branching strategies in both **layman’s terms** and **technical terms**.

---

## Common Branch Types

### **Main (or Master)**
- Always contains **stable, production-ready code**.  
- Nothing is pushed directly here; only merged after review.  

### **Develop**
- Contains the **latest integrated code** (all features merged but not yet released).  
- Acts as a **staging area** before moving to production.  

### **Feature Branches**
- Branch names: `feature/frontend`, `feature/backend`, `feature/login-api`.  
- Used for **new functionality** or tasks.  
- Created from `develop`, merged back into `develop`.  

### **Release Branches**
- Branch names: `release/v1.0`, `release/v2.0`.  
- Created from `develop` when preparing for a release.  
- Only bug fixes and refinements are made here.  
- Merged into both `main` and `develop` once released.  

### **Hotfix Branches**
- Branch names: `hotfix/critical-bug`.  
- Created from `main` if there’s an urgent issue in production.  
- Fixed quickly, then merged into both `main` and `develop`.  

---

## 3️⃣ Visual (GitFlow Style)
```
main  --------------------o-----------o-----> (stable releases)
                          \         /
develop -------------------o-------o--------> (integration branch)
                             \   /
feature/frontend -------------o o------------> (new features)
feature/backend  -------------o o------------> 
release/v1.0 -------------------o------------> (release prep)
hotfix/bugfix ---------------------o---------> (quick prod fix)
```

---

## ✅ In summary:
- **Layman:** Like writing a book → published version (`main`), draft (`develop`), personal chapters (`features`), proofreading (`release`), fixing typos (`hotfix`).  
- **Technical:** Controlled workflow where branches represent **code stability levels** and **collaboration steps**.  

---

✅ **End Goal:**  
- Both developers successfully push their code to separate branches.  
- Team can review and merge them into `main` via Pull Requests.  
