# Developer Tools Guide: Git/GitHub, Docker, Linux & Postman
> **Target Audience:** 3rd Year CS/IT Students preparing for Backend & Dev Interviews  
> **Tools Covered:** Git & GitHub, Docker, Essential Linux Commands, Postman  
> **Language:** Simple English + Intuitive Hinglish (samajhne ke liye)

---

## 📑 Table of Contents
1. [Git & GitHub (Version Control & Collaboration)](#1-git--github-version-control--collaboration)
2. [Docker (Containerization & Deployment)](#2-docker-containerization--deployment)
3. [Linux Commands (The Backend Production OS)](#3-linux-commands-the-backend-production-os)
4. [Postman (API Testing & Debugging)](#4-postman-api-testing--debugging)

---

# 1. Git & GitHub (Version Control & Collaboration)

### 💡 The Mental Model (Hinglish Analogy)
> **Video Game Save Points Analogy:**  
> Jab tum GTA ya koi game khelte ho, mushkil mission se pehle tum game **"Save"** karte ho taaki agar mar jao toh wahan se wapas shuru kar sako.  
> - **Git** tumhare code ka time machine / save system hai. Har commit ek save point hai.  
> - **GitHub** Google Drive / Cloud hai jahan tum apni game saves (code) upload karte ho taaki doosre dost bhi sath mein khel sakein (collaborate kar sakein).

---

### ❓ Git vs GitHub (Common Confusion)
- **Git:** Ek **local tool/software** hai jo tumhare laptop par code changes track karta hai. Yeh offline bhi chalta hai.
- **GitHub:** Ek **cloud-based website/service** hai (owned by Microsoft) jahan Git repositories host hoti hain taaki teams code share kar sakein. (Alternates: GitLab, Bitbucket).

---

### 🔑 The 3 Stages of Git
Git mein code 3 stages se hokar commit hota hai:
```
[ Working Directory ] ──( git add )──► [ Staging Area ] ──( git commit )──► [ Local Repo ] ──( git push )──► [ GitHub (Remote) ]
  (Files you edited)                     (Ready to save)                      (Saved on laptop)                  (Saved on cloud)
```

1. **Working Directory:** Files jinhe tum VS Code mein edit kar rahe ho.
2. **Staging Area (`git add`):** Cart ya basket jisme tum chuni hui files daalte ho commit karne ke liye.
3. **Local Repository (`git commit`):** Tumhare laptop ke `.git` folder mein snapshot permanently save ho gaya.
4. **Remote Repository (`git push`):** Code GitHub cloud par upload ho gaya.

---

### 💻 Essential Git Commands Cheat Sheet
```bash
# 1. Setup & Starting
git init                           # Current folder ko git repo banata hai
git clone <url>                    # GitHub se existing project laptop par download karta hai

# 2. Daily Workflow
git status                         # Kaunsi files modify hui hain check karta hai
git add .                          # Saare changes ko staging area mein daalta hai
git commit -m "Add user login API" # Snapshot save karta hai descriptive message ke sath
git push origin main               # Local commits ko GitHub ke main branch par bhejta hai
git pull origin main               # GitHub se latest changes laptop par download karke merge karta hai

# 3. Branching (Team Collaboration)
git branch                         # List all branches
git checkout -b feature/auth       # Naya branch banakar uspe switch karta hai (Modern: git switch -c feature/auth)
git checkout main                  # Main branch par wapas jata hai

# 4. Undoing Mistakes (Interview Favorites!)
git checkout -- <file>             # Unstaged changes ko discard karta hai (wapas purana code)
git reset HEAD~1                   # Aakhri commit cancel karta hai, par code safe rehta hai
git revert <commit_hash>           # Naya commit banakar purane commit ke changes ko reverse karta hai (Safe for shared branches)
```

---

### 🛑 What is `.gitignore` and why is it CRITICAL?
Ek simple text file jo Git ko batati hai ki kaunsi files track **NAHI** karni hain:
- Passwords / Keys: `.env` (Never push API keys to GitHub!)
- Large Dependencies: `node_modules/`, `venv/`, `__pycache__/`
- Local DBs: `*.db`, `*.sqlite3`

---

# 2. Docker (Containerization & Deployment)

### 💡 The Mental Model (Hinglish Analogy)
> **The Shipping Container & House Moving Analogy:**  
> - **The Problem:** *"Bhai mere laptop par code chal raha tha, production server par chalaya toh crash ho gaya!"* (Kyun? Kyunki server par Python version alag tha, database driver missing tha, ya OS alag tha).  
> - **The Docker Solution:** Socho tum apna ghar shift kar rahe ho. Har item alag-alag pack karne ke bajaye, tum poora kamra (Code + Python version + Dependencies + OS libraries) ek **sealed shipping container** mein pack kar dete ho.  
> Yeh container chahe Windows par chale, Mac par chale, ya AWS Ubuntu server par—**har jagah EXACTLY same chalega!**

---

### 🔑 Core Concepts: Image vs Container
- **Docker Image:** Ek **Recipe / Blueprint** (Class in OOP). Yeh static file hoti hai jisme code aur dependencies ki frozen copy hoti hai.
- **Docker Container:** Ek **Running Instance** (Object in OOP). Image ko run karne par jo living, working process banta hai, use container kehte hain.

---

### ⚖️ Docker Container vs Virtual Machine (VM)
*Interviewers LOVE asking this difference!*

| Feature | Virtual Machine (VM) | Docker Container |
| :--- | :--- | :--- |
| **Architecture** | Includes Guest OS + Hypervisor | Shares Host OS Kernel (No Guest OS) |
| **Startup Time** | Minutes (boots full operating system) | Seconds or milliseconds |
| **Resource Usage**| Heavy (GBs of RAM & Disk per VM) | Extremely lightweight (MBs of RAM) |
| **Isolation** | Hardware-level isolation | Process-level isolation |

---

### 💻 Minimal Production Dockerfile (~10 Lines)
```dockerfile
# 1. Base lightweight Python image
FROM python:3.11-slim

# 2. Container ke andar working directory set karo
WORKDIR /app

# 3. Dependencies copy aur install karo (layer caching optimization)
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# 4. Baaki saara project code copy karo
COPY . .

# 5. Container start hote hi FastAPI uvicorn server run karo
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

### 🛠️ Daily Docker Commands
```bash
# Image build karna (current folder ke Dockerfile se)
docker build -t my-backend-app .

# Container run karna (-p: host_port:container_port, -d: background mode)
docker run -d -p 8000:8000 --name backend_container my-backend-app

# Running containers check karna
docker ps

# Logs dekhna (Debugging ke liye)
docker logs -f backend_container

# Container stop aur remove karna
docker stop backend_container && docker rm backend_container
```

---

# 3. Linux Commands (The Backend Production OS)

### 💡 The Mental Model (Hinglish Analogy)
> **The Real World Truth:**  
> Tum development Windows ya Mac par karte ho with fancy mouse clicks. Lekin 95%+ backend servers (AWS EC2, DigitalOcean, Google Cloud) bina mouse aur bina monitor ke hote hain (Headless Linux CLI).  
> Backend engineer ko terminal se server par files dekhna, logs check karna, aur crashing processes ko kill karna aana compulsory hai.

---

### 🔑 The 15 Must-Know Commands for Backend Engineers

#### 1. Navigation & File Inspection
```bash
pwd                       # Print Working Directory (Main kahan khada hoon?)
ls -la                    # List all files including hidden files (.env, .git) with permissions
cd /var/log               # Change directory
cat app.log               # Poori file terminal par print karta hai
less app.log              # Badi file ko page-by-page scroll karke padhne ke liye (Press 'q' to exit)
```

#### 2. Log Monitoring (Super High Interview Yield!)
```bash
# Real-time mein live incoming server logs dekhna
tail -f -n 100 app.log    # Aakhri 100 lines dikhayega aur nayi lines aate hi live scroll karega
```
> *Interview Hack:* Agar interviewer pooche: *"Server par error aaya hai, live kaise trace karoge?"*  
> Turant bolo: *"I will SSH into the server and use `tail -f` on the application log file."*

#### 3. Searching inside Files (`grep`)
```bash
grep -rn "ERROR" /var/log/app/   # /var/log/app folder ke andar saari files mein "ERROR" word search karo with line numbers
```

#### 4. Process Management (Server Troubleshooting)
```bash
ps aux                    # Server par chal rahe saare processes ki list dikhata hai
top  # or htop            # Task manager: CPU aur RAM consumption live dikhata hai
kill -9 <PID>             # Kisi stuck ya hung process ko force kill karta hai using Process ID
```

#### 5. Ports & Networking (Backend Debugging)
```bash
# Check karo kaunsa application port 8000 par listen kar raha hai
netstat -tulnp | grep 8000   # (or: ss -tulnp | grep 8000)

# Terminal se direct API call test karna
curl -I http://localhost:8000/health
```

#### 6. Permissions (`chmod`)
```bash
chmod 755 script.sh       # Owner ko Read/Write/Execute (7), doosron ko Read/Execute (5) permission deta hai
chmod 400 my-key.pem      # AWS SSH private key ko secure read-only permission deta hai
```

---

# 4. Postman (API Testing & Debugging)

### 💡 The Mental Model (Hinglish Analogy)
> **The Remote Control Analogy:**  
> Jab tak frontend team React UI ya mobile app nahi banati, tab tak backend API ko test kaise karoge? Browser mein sirf simple `GET` request chalti hai, browser URL bar se `POST`, headers ya JWT bearer token nahi bhej sakte.  
> **Postman ek universal remote control hai:** Tum method select karte ho (POST/PUT), headers lagate ho, JSON payload daalte ho, aur Send click karte hi status code aur response dekh lete ho.

---

### 🔑 Core Postman Features You Must Know

#### 1. Collections & Folders
Requests ko random chhodne ke bajaye organized rakhna:
- Collection: `E-Commerce API`
  - Folder: `Auth` (`POST /login`, `POST /register`)
  - Folder: `Products` (`GET /products`, `POST /products`)
  - Folder: `Orders` (`GET /orders`)

#### 2. Environments & Variables (`{{base_url}}`)
Kabhi bhi hardcoded URL (`http://localhost:8000/api/v1/users`) har request mein mat likho!
- Environment banao: **Local**, **Staging**, **Production**.
- Variable set karo: `base_url = http://localhost:8000`
- Har request mein likho: `{{base_url}}/api/v1/users`.  
  *Fayda:* Production par switch karne ke liye sirf dropdown se environment change karna hota hai, 50 requests edit nahi karni padti.

#### 3. Automatic Bearer Token Passing (Test Script Hack)
Login karne ke baad har request mein manually copy-paste token nahi karna padta.
Postman ke **Tests** tab mein login route par yeh 2 lines likh do:
```javascript
// Login ke response se token nikal kar environment variable mein save karta hai
var jsonData = pm.response.json();
pm.environment.set("jwt_token", jsonData.access_token);
```
Ab baaki saare protected routes par **Authorization Tab -> Type: Bearer Token -> Token: `{{jwt_token}}`** likh do. Saara auth flow automatically chalega!

---
*Proceed to `04_DevTools_Interview_Defense_And_QA.md` for the exact interview questions, cross-examination traps, and spoken scripts on these tools.*
