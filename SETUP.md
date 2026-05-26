# VoltEdge — Deploy to Azure

> Guide til at deploye VoltEdge MVP til jeres eget Azure-abonnement.

---

## ☁️ Deploy to Azure (1-2 timer)

### Step 1 — Opret Azure Web App

1. Gå til [Azure Portal](https://portal.azure.com) → **App Services** → **Create** → **Web App**
2. Udfyld:

   | Felt | Værdi |
   |------|-------|
   | **Subscription** | Jeres aktive subscription |
   | **Resource Group** | Opret ny → `VoltEdge` |
   | **Name** | Vælg et unikt navn, fx `voltedge-app-jeresnavn` |
   | **Runtime stack** | Python 3.12 |
   | **Region** | Vælg en tæt på jer |
   | **Sku** | **Free F1** (gratis) |

3. Klik **Review + Create** → **Create**

### Step 2 — Sæt Startup Command

**App Service → Settings → Configuration → General Settings**

Sæt **Startup Command** til:

```
cd src && uvicorn main:app --host 0.0.0.0 --port 8000
```

Klik **Save**.

### Step 3 — Sæt CI/CD op

1. **App Service → Deployment Center → GitHub**
2. Authorize GitHub → vælg `Lula0002/VoltEdge` repo → `main` branch
3. Azure opretter automatisk et **publish profile secret** i jeres GitHub repo

### Step 4 — Trigger første deploy

- Push en ændring til `main`, **eller**
- GitHub → **Actions** → **Deploy VoltEdge MVP to Azure Web App** → **Run workflow**

### Step 5 — Bekræft at det virker

Åbn i browseren:

```
https://jeres-app-navn.azurewebsites.net/docs
```

Her skulle Swagger UI gerne vise alle endpoints.

---

## 🖥️ Local Development (faldeback)

Hvis I vil teste lokalt først:

<details>
<summary>🍏 Mac</summary>

```bash
git clone https://github.com/Lula0002/VoltEdge.git
cd VoltEdge
python3 -m venv venv
source venv/bin/activate
pip install -r src/requirements.txt
cd src
uvicorn main:app --reload --port 8000
```
</details>

<details>
<summary>🪟 Windows</summary>

```powershell
git clone https://github.com/Lula0002/VoltEdge.git
cd VoltEdge
python -m venv venv
.\venv\Scripts\Activate.ps1
pip install -r src\requirements.txt
cd src
python -m uvicorn main:app --reload --port 8000
```
</details>

Åbn http://localhost:8000/docs for Swagger UI.

---

## ✅ Tjekliste til eksamen

| # | Tjek | Status |
|---|------|--------|
| 1 | Swagger UI vises på jeres Azure URL (`/docs`) | ☐ |
| 2 | POST `/sessions/start` → opretter session | ☐ |
| 3 | POST `/{id}/authorize` → godkender | ☐ |
| 4 | POST `/{id}/start-charging` → starter opladning | ☐ |
| 5 | POST `/{id}/complete` → afslutter session | ☐ |
| 6 | POST `/billing/rate` → beregner pris | ☐ |
| 7 | POST `/billing/invoice` → genererer faktura | ☐ |
| 8 | GitHub Actions workflow viser grønt ✅ | ☐ |
| 9 | Database oprettes automatisk via `init_db()` | ☐ |
| 10 | Genstart app'en → data overlever | ☐ |

---

## 🧪 Happy Path (test via Swagger)

Kør disse 6 kald i rækkefølge:

```
POST /sessions/start                  → session_id modtages
POST /sessions/{id}/authorize         → { "authorization_code": "AUTH123" }
POST /sessions/{id}/start-charging    → { "meter_start_kwh": 100 }
POST /sessions/{id}/complete          → { "meter_stop_kwh": 150, "end_reason": "normal" }
POST /billing/rate                    → { "session_id": "..." , "energy_kwh": 50 }
POST /billing/invoice                 → { "session_id": "..." , "total_amount": 75.0 }
```

---

## ❓ Troubleshooting

| Problem | Løsning |
|---------|---------|
| `uvicorn not found` | Aktivér venv: `source venv/bin/activate` (Mac) / `.\venv\Scripts\Activate.ps1` (Windows) |
| Port 8000 i brug | Brug anden port: `uvicorn main:app --reload --port 8001` |
| Azure deploy fejler 403 | Tjek at subscription er aktiv → Azure Portal → Subscriptions |
| Workflow fejler | Tjek at publish profile secret findes i GitHub repo Settings → Secrets |
