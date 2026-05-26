# VoltEdge — Setup Guide

> Run the VoltEdge MVP locally or deploy to your own Azure subscription.

---

## 🖥️ Local Development (2 min)

### Prerequisites

| Tool | Version |
|------|---------|
| Python | 3.12+ |
| Git | any |
| Terminal | PowerShell / Bash |

### Steps

```powershell
# 1. Clone the repo
git clone https://github.com/Lula0002/VoltEdge.git
cd VoltEdge

# 2. Create virtual environment & install dependencies
python -m venv venv
.\venv\Scripts\Activate.ps1          # Windows
# source venv/bin/activate           # Mac / Linux

pip install -r src\requirements.txt

# 3. Start the server
cd src
uvicorn main:app --reload --port 8000
```

### Verify

| URL | What to expect |
|-----|----------------|
| http://localhost:8000/health | `{"status":"healthy"}` |
| http://localhost:8000/docs | Swagger UI with all 12 endpoints |

---

## ☁️ Deploy to Azure (1-2 hrs)

### Step 1 — Create Azure Web App

1. Go to [Azure Portal](https://portal.azure.com) → **App Services** → **Create** → **Web App**
2. Fill in:

   | Setting | Value |
   |---------|-------|
   | **Subscription** | Your active subscription |
   | **Resource Group** | Create new → `VoltEdge` |
   | **Name** | `voltedge-app` (or your own) |
   | **Runtime stack** | Python 3.12 |
   | **Region** | Germany West Central (or near you) |
   | **Sku** | **Free F1** (no cost) |

3. Click **Review + Create** → **Create**

### Step 2 — Set Startup Command

**App Service → Settings → Configuration → General Settings**

```
Startup Command: cd src && uvicorn main:app --host 0.0.0.0 --port 8000
```

Click **Save**.

### Step 3 — (Optional) Add MySQL Database

If you want to demonstrate MySQL support:

1. Create **Azure Database for MySQL Flexible Server** (Free tier B1ms)
2. Add firewall rule: "Allow public access from Azure services"
3. Create database named `voltedge`
4. In App Service → **Settings → Environment variables**, add:

   ```
   DATABASE_URL = mysql://user:password@your-server.mysql.database.azure.com:3306/voltedge
   ```

   > **No `DATABASE_URL`?** No problem — SQLite is used automatically (zero setup).

### Step 4 — Set up CI/CD

#### Option A: GitHub Actions (recommended)

The workflow already exists at `.github/workflows/main_voltedge-app.yml`.

1. In Azure Portal: **App Service → Deployment Center → GitHub**
2. Authorize GitHub → select repo `Lula0002/VoltEdge` → branch `main`
3. Azure generates a **publish profile secret** in your GitHub repo automatically

> **If deploying manually:**  
> 1. Go to **App Service → Overview → Get Publish Profile**  
> 2. In your GitHub repo: **Settings → Secrets and variables → Actions → New repository secret**  
> 3. Name: `AZUREAPPSERVICE_PUBLISHPROFILE_<YOUR_ID>`  
> 4. Update the workflow file with your secret name

#### Option B: Manual deploy via zip

```powershell
# From repo root (without venv/)
Compress-Archive -Path * -DestinationPath deploy.zip -Exclude "venv"
# Upload through Azure Portal: App Service → Deployment Center → ZIP Deploy
```

### Step 5 — Trigger First Deployment

- Push any change to `main`, **or**
- GitHub → **Actions** → **Deploy VoltEdge MVP to Azure Web App** → **Run workflow**

---

## ✅ Verification Checklist

Tick these off before the exam:

| # | Check | Status |
|---|-------|--------|
| 1 | `https://your-app.azurewebsites.net/health` returns 200 | ☐ |
| 2 | `https://your-app.azurewebsites.net/docs` shows Swagger UI | ☐ |
| 3 | Create a session via Swagger → rated → invoice generated | ☐ |
| 4 | GitHub Actions workflow shows green checkmark | ☐ |
| 5 | Tables created automatically (`init_db()` runs on startup) | ☐ |

---

## 🧪 Quick Test Flow (via Swagger)

```
POST /sessions          → { "session_id": "abc-123", "status": "started" }
PATCH /sessions/abc-123 → rate the session
  body: { "status": "rated" }
                          → invoice generated, anomalies checked
GET  /sessions/abc-123  → see full session with invoice
```

---

## 📦 Project Structure (quick overview)

```
VoltEdge/
├── src/
│   ├── main.py                    # FastAPI entry point
│   ├── session_service/
│   │   └── session_api.py         # Session CRUD + state machine
│   ├── billing_service/
│   │   └── billing_api.py         # Tariff rating + invoice generation
│   ├── analytics_service/
│   │   └── analytics_api.py       # ML anomaly detection
│   └── shared/
│       ├── database.py            # SQLite/MySQL with auto init_db()
│       ├── events.py              # Pydantic event models
│       └── models.py              # DB models & queries
├── .github/workflows/
│   └── main_voltedge-app.yml      # CI/CD pipeline
├── README.md                      # Full documentation
├── SETUP.md                       # ← You are here
└── requirements.txt
```

---

## ❓ Troubleshooting

| Problem | Solution |
|---------|----------|
| `uvicorn not found` | Activate venv first: `.\venv\Scripts\Activate.ps1` |
| Port 8000 in use | Use another port: `uvicorn main:app --reload --port 8001` |
| Azure deployment 403 | Subscription may be disabled → check billing in Azure Portal |
| MySQL connection refused | Check firewall rules in Azure Database for MySQL |
| Workflow fails on deploy | Verify publish profile secret name in GitHub matches workflow |
