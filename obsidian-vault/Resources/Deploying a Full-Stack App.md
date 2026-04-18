---
tags: [learning, deployment, reference, fly-io, vercel, devops]
created: 2026-04-18
applies-to: [VulnBrief, any FastAPI + Next.js project]
---

# Deploying a Full-Stack Web App — Complete Reference Guide

> Written during VulnBrief deployment (April 2026). Covers the full process from zero to live — what every tool is, why we chose it, and how to repeat this for any project.

---

## What We Built and Where It Lives

VulnBrief has two parts:

```
┌─────────────────────────────────┐
│  Frontend (Next.js)             │  ← What users see in their browser
│  Hosted on: Vercel              │
└──────────────┬──────────────────┘
               │ API calls
┌──────────────▼──────────────────┐
│  Backend (FastAPI + Python)     │  ← The brain — fetches data, scores CVEs
│  Hosted on: Fly.io              │
└──────────────┬──────────────────┘
               │ reads/writes
┌──────────────▼──────────────────┐
│  Database (PostgreSQL)          │  ← Stores all CVE data permanently
│  Hosted on: Fly.io (managed)    │
└─────────────────────────────────┘
```

Every modern web app has this shape: a frontend the user interacts with, a backend that does the work, and a database that stores everything. Understanding this split is the foundation of everything else.

---

## The Tools — What They Are and Why We Used Them

### Git
**What it is:** Version control. Tracks every change to your code and lets you go back in time.
**Why we used it:** Industry standard. Every deployment platform pulls code from Git. Without it you cannot deploy.
**Key commands:**
```bash
git clone <url>        # copy a repo to your machine
git add <file>         # stage a change
git commit -m "msg"    # save a snapshot
git push               # send changes to GitHub
git pull               # get latest changes from GitHub
```

---

### GitHub
**What it is:** Cloud storage for Git repositories. The place your code lives online.
**Why we used it:** Both Fly.io and Vercel connect directly to GitHub to pull your code. It is also your backup — if your laptop dies, the code is safe.
**Key concept — Personal Access Token (PAT):** A password that lets other systems (or scripts) push to your repo on your behalf. Always regenerate after use and never leave in code.

---

### Windows Terminal + PowerShell
**What it is:** The command line on Windows. You type commands, the computer executes them.
**Why we used PowerShell over Git Bash:** PowerShell is the native Windows terminal. Fly CLI and some installers only work correctly in PowerShell. Git Bash is fine for git commands but inconsistent for everything else.
**Rule of thumb:** Use PowerShell for Fly, Vercel, npm commands. Git Bash is fine for git only.

---

### Node.js and npm
**What it is:** Node.js is a JavaScript runtime. npm is its package manager — used to install JavaScript tools and libraries.
**Why we needed it:** The Vercel CLI is a Node.js application. The frontend (Next.js) also runs on Node.js. Even though the backend is Python, the frontend toolchain is JavaScript.
**Key commands:**
```bash
node --version         # check Node is installed
npm --version          # check npm is installed
npm install -g <tool>  # install a tool globally (available anywhere)
npm install            # install project dependencies
npm run build          # build the project for production
```

---

### Fly.io
**What it is:** A cloud hosting platform. You give it a Dockerfile and it runs your app anywhere in the world.
**Why we chose it over AWS/GCP/Azure:**
- Much simpler to set up — no IAM roles, no VPCs, no load balancer configuration
- Free tier is genuinely useful for getting started
- Pricing is transparent: ~$3–5/month for a small app
- Built-in managed PostgreSQL (handles backups, scaling, connection pooling)
- CLI-first workflow fits how developers actually work

**Key concepts:**
- **App:** Your backend running in Fly's infrastructure
- **Machine:** The virtual server running your app
- **Region:** The physical location of the server (lhr = London)
- **Secrets:** Environment variables stored securely — API keys, database passwords. Never put these in your code.
- **fly.toml:** The configuration file that tells Fly how to run your app

**Key commands:**
```bash
fly auth login                    # log in to your Fly account
fly launch --no-deploy            # register the app without deploying yet
fly secrets set KEY="value"       # store a secret environment variable
fly deploy                        # build and deploy your app
fly logs                          # see live logs from your running app
fly status                        # check if your app is running
fly postgres create               # create a managed PostgreSQL database
fly postgres connect --app <name> # open a database shell
```

**fly.toml explained:**
```toml
app = "vulnbrief-api"        # the name of your app on Fly
primary_region = "lhr"       # London — pick the region closest to your users

[http_service]
  internal_port = 8000       # the port your app listens on inside the container
  force_https = true         # always redirect HTTP to HTTPS

  auto_stop_machines = true  # shut down when no traffic (saves money)
  auto_start_machines = true # wake up automatically when a request comes in
  min_machines_running = 0   # allow full scale-to-zero

[[vm]]
  memory = "512mb"           # RAM — increase if app crashes with out-of-memory
  cpu_kind = "shared"        # shared CPU — cheapest option, fine for low traffic
```

---

### Vercel
**What it is:** A hosting platform specifically optimised for Next.js (and other frontend frameworks).
**Why we chose it:**
- Built by the same team that makes Next.js — perfect compatibility
- Free tier covers most small projects
- Automatic deployments: every push to GitHub main branch auto-deploys
- Global CDN built in — your site loads fast from anywhere
- Zero configuration for Next.js — it just works

**Key concepts:**
- **Project:** Your frontend app on Vercel
- **Deployment:** A specific version of your app that was deployed
- **Environment Variables:** Same concept as Fly secrets — configuration your app needs at runtime
- **Production URL:** The live URL of your app (e.g. vulnbrief.vercel.app)
- **Preview Deployments:** Every pull request gets its own temporary URL for testing

**Key commands:**
```bash
vercel login           # log in to Vercel
vercel                 # deploy from the current directory (interactive)
vercel --prod          # deploy to production
vercel env add         # add an environment variable
vercel logs            # view deployment logs
```

---

### Docker and Dockerfile
**What it is:** A way to package your entire application — code, dependencies, runtime — into a single portable container.
**Why it matters for deployment:** Fly.io uses Docker under the hood. When you run `fly deploy`, it reads your Dockerfile, builds a container image, and runs it on their servers. This guarantees your app runs exactly the same in production as it does locally.

**Our Dockerfile explained:**
```dockerfile
FROM python:3.12-slim          # start from an official Python image

WORKDIR /app                   # all commands run inside /app

COPY requirements.txt .        # copy dependency list first (layer caching)
RUN pip install -r requirements.txt  # install Python packages

COPY . .                       # copy all your code

EXPOSE 8000                    # document that the app uses port 8000
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
# start the FastAPI server when the container runs
```

---

### PostgreSQL
**What it is:** A relational database. Stores your data in tables with rows and columns — structured, reliable, queryable with SQL.
**Why we chose it:** Industry standard. Fast. Handles everything from small apps to huge enterprises. Fly.io manages it for us — no manual setup, automatic backups.
**Scale-to-zero:** We answered Y to "scale to zero after one hour" — this means the database shuts down after an hour of no activity and restarts on the first query. Saves money on a dev/staging setup. For production with real users you would say N.

---

### Environment Variables and Secrets
**What they are:** Configuration values your app reads at runtime — things that change between environments (dev vs production) or that must never be in your code (API keys, passwords).

**The rule:** Never put secrets in code or commit them to Git. Use:
- `fly secrets set` for Fly.io (backend)
- Vercel dashboard Environment Variables for frontend
- `.env` file locally (and add `.env` to `.gitignore` so it never gets committed)

**Our secrets:**
| Secret | What it is | Where it's used |
|---|---|---|
| `DATABASE_URL` | PostgreSQL connection string from Fly | Backend reads/writes CVE data |
| `ANTHROPIC_API_KEY` | Your Claude API key | Backend generates CVE summaries |
| `NEXT_PUBLIC_API_URL` | The Fly.io backend URL | Frontend knows where to send API calls |

---

## Full Deployment Process — Step by Step

### One-time setup (Windows)

```
1. Install Git            → git-scm.com/download/win
2. Install Node.js LTS    → nodejs.org
3. Install Fly CLI        → PowerShell: iwr https://fly.io/install.ps1 -useb | iex
4. Install Vercel CLI     → PowerShell: npm install -g vercel
5. Install VSCode         → code.visualstudio.com
6. Create Fly.io account  → fly.io (sign up with GitHub)
7. Create Vercel account  → vercel.com (sign up with GitHub)
```

### Per-project deployment

#### Backend (Fly.io)

```bash
# 1. Clone your repo
git clone https://github.com/<you>/<repo>.git
cd <repo>/backend

# 2. Create the database (first time only)
fly postgres create --name <app>-db --region lhr
# → Save the DATABASE_URL it prints

# 3. Register the app
fly launch --no-deploy
# → Use existing fly.toml: Y
# → Tweak settings: N

# 4. Set secrets
fly secrets set DATABASE_URL="<connection string>"
fly secrets set ANTHROPIC_API_KEY="<your key>"
# Add any other environment variables your app needs

# 5. Deploy
fly deploy
# → Builds Docker image, pushes to Fly, starts the app
# → Takes 2-3 minutes first time

# 6. Verify
fly status
fly logs
# Visit: https://<app-name>.fly.dev/health
```

#### Frontend (Vercel)

```bash
# From the frontend directory
cd ../frontend

# 1. Login and deploy
vercel

# Follow prompts:
# → Link to existing project or create new: create new
# → Project name: vulnbrief
# → Directory: ./ (current)

# 2. Set environment variable
vercel env add NEXT_PUBLIC_API_URL
# → Enter value: https://<your-fly-app>.fly.dev
# → Select: Production + Preview + Development

# 3. Redeploy with the env var active
vercel --prod
```

---

## After Deployment — Seeding Data

A fresh deployment has an empty database. You need to run the ingestion jobs to populate it:

```bash
# SSH into your Fly app
fly ssh console --app vulnbrief-api

# Run ingestion scripts
python -m app.ingest.kev      # CISA Known Exploited Vulnerabilities
python -m app.ingest.epss     # EPSS exploitation probability scores
python -m app.ingest.nvd      # NVD CVE data (takes longer)
```

---

## Troubleshooting

| Problem | What to check |
|---|---|
| `fly deploy` fails | Run `fly logs` to see the error |
| App crashes on startup | Check `DATABASE_URL` secret is set correctly |
| Frontend shows no data | Check `NEXT_PUBLIC_API_URL` points to the correct Fly URL |
| 500 errors from API | Run `fly logs --app vulnbrief-api` for stack trace |
| Database connection refused | Check the Fly Postgres app is running: `fly status --app vulnbrief-db` |
| Slow cold start | App scaled to zero — first request wakes it up, takes ~3s |

---

## Cost Reference

| Service | Free tier | Paid |
|---|---|---|
| Fly.io (app) | 3 shared VMs free | ~$2–5/month per app |
| Fly.io (Postgres) | Scale-to-zero = nearly free | ~$3/month always-on |
| Vercel (frontend) | 100GB bandwidth, unlimited deploys | $20/month Pro |
| Anthropic API | Pay per use | ~$0.003 per 1K tokens |

**Realistic monthly cost for a small production app: £5–15/month.**

---

## Key Lessons

1. **Separate your concerns** — frontend, backend, and database are always three separate things. Deploy them separately.
2. **Secrets are not config** — never put API keys or passwords in code. Always use environment variables.
3. **Docker makes deployment repeatable** — if it runs locally in Docker, it runs on Fly.io.
4. **Scale to zero saves money** — dev/staging apps don't need to run 24/7. Cold starts are acceptable when you're not serving real users yet.
5. **Vercel + Fly.io is the fastest path from code to live URL** — for early-stage products this stack beats AWS by weeks of setup time.
6. **Always check logs first** — when something breaks, `fly logs` tells you what happened before you start guessing.

---

## Related

- [[VulnBrief - Product Plan]] — the product this deployment serves
- [[VulnBrief - Competitive Intelligence]] — market context
- [[Project Glasswing]] — the research program behind the product
