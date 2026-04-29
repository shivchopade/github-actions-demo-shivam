<p align="center">
  <img src="https://img.shields.io/badge/Go-1.26-00ADD8?style=for-the-badge&logo=go&logoColor=white" />
  <img src="https://img.shields.io/badge/MySQL-8.4-4479A1?style=for-the-badge&logo=mysql&logoColor=white" />
  <img src="https://img.shields.io/badge/Docker-Compose-2496ED?style=for-the-badge&logo=docker&logoColor=white" />
  <img src="https://img.shields.io/badge/Nginx-Alpine-009639?style=for-the-badge&logo=nginx&logoColor=white" />
  <img src="https://img.shields.io/badge/GitHub_Actions-CI%2FCD-2088FF?style=for-the-badge&logo=githubactions&logoColor=white" />
  <img src="https://img.shields.io/badge/AWS-EC2-FF9900?style=for-the-badge&logo=amazonec2&logoColor=white" />
</p>

<h1 align="center">⚡ SkillPulse</h1>

<p align="center">
  <strong>A modern, full-stack skill tracking dashboard with automated CI/CD deployment to AWS EC2.</strong>
</p>

<p align="center">
  Track your learning skills · Log practice sessions · Visualize progress<br/>
  Built with Go + MySQL + Vanilla JS · Deployed via GitHub Actions → DockerHub → EC2
</p>

---

## 📑 Table of Contents

- [Overview](#-overview)
- [Architecture](#-architecture)
- [Tech Stack](#-tech-stack)
- [Project Structure](#-project-structure)
- [Features](#-features)
- [Getting Started](#-getting-started)
  - [Prerequisites](#prerequisites)
  - [Local Development](#local-development)
  - [Environment Variables](#environment-variables)
- [Docker Setup](#-docker-setup)
- [CI/CD Pipeline](#-cicd-pipeline)
  - [CI — Continuous Integration](#ci--continuous-integration)
  - [CD — Continuous Deployment](#cd--continuous-deployment)
  - [GitHub Secrets Required](#github-secrets-required)
  - [Pipeline Flow Diagram](#pipeline-flow-diagram)
- [AWS EC2 Deployment](#-aws-ec2-deployment)
  - [EC2 Instance Setup](#ec2-instance-setup)
  - [Manual Deployment (First Time)](#manual-deployment-first-time)
  - [Verifying Deployment](#verifying-deployment)
- [API Reference](#-api-reference)
- [Database Schema](#-database-schema)
- [Frontend](#-frontend)
- [Nginx Configuration](#-nginx-configuration)
- [Troubleshooting](#-troubleshooting)
- [Contributing](#-contributing)
- [License](#-license)

---

## 🔍 Overview

**SkillPulse** is a full-stack web application that helps developers track their learning progress across multiple skills. It features a Go (Gin) REST API backend, a MySQL database, a responsive vanilla JavaScript frontend, and a complete CI/CD pipeline that automatically builds, pushes, and deploys to an AWS EC2 instance on every push to `main`.

**Live URL:** `http://13.60.252.3`

---

## 🏗 Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                        AWS EC2 Instance                         │
│                                                                 │
│  ┌──────────┐     ┌───────────────┐     ┌───────────────────┐  │
│  │  Nginx   │────▶│   Go Backend  │────▶│   MySQL 8.4       │  │
│  │ (Port 80)│     │   (Port 8080) │     │   (Port 3306)     │  │
│  │          │     │               │     │                   │  │
│  │ Static   │     │  Gin REST API │     │  skills table     │  │
│  │ Files    │     │  /api/*       │     │  learning_logs    │  │
│  └──────────┘     └───────────────┘     └───────────────────┘  │
│       │                                                         │
│       ▼                                                         │
│  ./frontend/          Docker Image:            Docker Volume:   │
│  (volume mount)       shiv9/githubactions      mysql_data       │
└─────────────────────────────────────────────────────────────────┘
         ▲
         │ Port 80
         │
    ┌────┴─────┐
    │  Browser  │
    │  Client   │
    └──────────┘
```

### How Requests Flow

1. **Browser** sends request to `http://<EC2-IP>/`
2. **Nginx** serves static frontend files (HTML, CSS, JS) for `/` routes
3. **Nginx** reverse-proxies `/api/*` requests to the **Go backend** on port 8080
4. **Go backend** queries **MySQL** database and returns JSON responses
5. **Frontend JS** renders data dynamically in the browser

---

## 🛠 Tech Stack

| Layer | Technology | Purpose |
|:------|:-----------|:--------|
| **Frontend** | HTML5, CSS3, Vanilla JavaScript | Responsive dashboard UI with dark/light theme |
| **Backend** | Go 1.26, Gin Framework | RESTful API server |
| **Database** | MySQL 8.4 | Persistent data storage |
| **Reverse Proxy** | Nginx Alpine | Serves static files + proxies API calls |
| **Containerization** | Docker, Docker Compose | Multi-container orchestration |
| **CI/CD** | GitHub Actions | Automated build, push, and deploy |
| **Registry** | Docker Hub | Container image registry |
| **Cloud** | AWS EC2 (Ubuntu) | Production hosting |
| **Fonts** | Google Fonts (Inter) | Modern typography |

---

## 📁 Project Structure

```
github-actions-demo-shivam/
│
├── .github/
│   └── workflows/
│       ├── ci.yml                 # CI: Build & push Docker image to DockerHub
│       └── cd.yml                 # CD: Deploy to EC2 via SSH
│
├── backend/
│   ├── Dockerfile                 # Multi-stage Go build (builder → alpine)
│   ├── go.mod                     # Go module definition
│   ├── main.go                    # Application entry point & route registration
│   ├── database/
│   │   └── db.go                  # MySQL connection with retry logic
│   ├── handlers/
│   │   ├── skills.go              # CRUD handlers for skills
│   │   ├── logs.go                # Learning log creation handler
│   │   └── dashboard.go           # Dashboard aggregation + health check
│   └── models/
│       └── skill.go               # Data models: Skill, LearningLog, Dashboard
│
├── frontend/
│   ├── index.html                 # Main SPA page (sidebar, stats, modals)
│   ├── css/
│   │   └── style.css              # Complete design system (light + dark themes)
│   └── js/
│       └── app.js                 # API integration, rendering, theme management
│
├── mysql/
│   └── init.sql                   # Database schema + seed data
│
├── nginx/
│   └── nginx.conf                 # Reverse proxy + static file serving config
│
├── docker-compose.yml             # Multi-service orchestration
├── .env                           # Environment variables for Docker Compose
└── README.md                      # This file
```

---

## ✨ Features

### Dashboard
- **Stats Overview** — Total skills, hours logged, total sessions, and top skill at a glance
- **Animated Cards** — Staggered fade-in animations with hover lift effects
- **Real-time Data** — Stats update immediately after adding skills or logging sessions

### Skills Management
- **Add Skills** — Create new skills with name, category, and target hours
- **Progress Tracking** — Visual progress bars showing logged hours vs target
- **Category Badges** — Color-coded labels (Programming, DevOps, Cloud, etc.)
- **Delete Skills** — Remove skills and all associated learning logs (cascade delete)

### Session Logging
- **Log Hours** — Record practice sessions with hours, date, and optional notes
- **Date Picker** — Defaults to today's date for quick logging

### UI/UX
- **Dark/Light Theme** — Toggle between modes; persisted in localStorage
- **Responsive Design** — Works on desktop, tablet, and mobile
- **Collapsible Sidebar** — Hamburger menu on mobile devices
- **Toast Notifications** — Success/error feedback for all user actions
- **Modern Blue Theme** — Premium design with Inter font, smooth gradients, and glass effects
- **SVG Icons** — Crisp vector icons throughout (no emoji dependencies)
- **No-cache Headers** — Nginx serves latest files without browser caching issues

### DevOps
- **Dockerized** — All services run in containers
- **CI/CD Pipeline** — Automatic build and deploy on every push to `main`
- **Multi-stage Build** — Optimized Docker image (~15MB) using Alpine
- **Health Check Endpoint** — `/health` for monitoring
- **Database Retry Logic** — Backend retries MySQL connection up to 30 times on startup

---

## 🚀 Getting Started

### Prerequisites

| Tool | Version | Purpose |
|:-----|:--------|:--------|
| [Docker](https://docs.docker.com/get-docker/) | 20.10+ | Container runtime |
| [Docker Compose](https://docs.docker.com/compose/install/) | v2+ | Multi-container orchestration |
| [Git](https://git-scm.com/) | 2.30+ | Version control |
| [Go](https://go.dev/) *(optional)* | 1.26+ | Only if running backend without Docker |

### Local Development

**1. Clone the repository**

```bash
git clone https://github.com/shivchopade/github-actions-demo-shivam.git
cd github-actions-demo-shivam
```

**2. Configure environment variables**

The project includes a `.env` file with default values. Modify if needed:

```bash
# .env (already provided with defaults)
MYSQL_ROOT_PASSWORD=rootpassword123
DB_HOST=db
DB_PORT=3306
DB_USER=skillpulse
DB_PASSWORD=skillpulse123
DB_NAME=skillpulse
DOCKERHUB_USERNAME=shiv9
```

**3. Start all services**

```bash
docker compose up -d
```

This starts three containers:
- `db` — MySQL 8.4 (initializes schema + seed data)
- `backend` — Go API server (pulled from DockerHub)
- `nginx` — Serves frontend + proxies API requests

**4. Access the application**

```
http://localhost
```

**5. Check service health**

```bash
# Container status
docker compose ps

# Backend health
curl http://localhost/health

# API test
curl http://localhost/api/dashboard
```

**6. Stop all services**

```bash
docker compose down          # Stop & remove containers (keeps data)
docker compose down -v       # Stop & remove containers AND data volumes
```

### Environment Variables

| Variable | Default | Used By | Description |
|:---------|:--------|:--------|:------------|
| `MYSQL_ROOT_PASSWORD` | `rootpassword123` | MySQL | Root password for MySQL |
| `DB_HOST` | `db` | Backend | MySQL hostname (Docker service name) |
| `DB_PORT` | `3306` | Backend | MySQL port |
| `DB_USER` | `skillpulse` | Backend, MySQL | Application database user |
| `DB_PASSWORD` | `skillpulse123` | Backend, MySQL | Application database password |
| `DB_NAME` | `skillpulse` | Backend, MySQL | Database name |
| `PORT` | `8080` | Backend | Go server port |
| `DOCKERHUB_USERNAME` | `shiv9` | Docker Compose | DockerHub username for image pull |

---

## 🐳 Docker Setup

### Services

```yaml
# docker-compose.yml defines 3 services:

db        → MySQL 8.4       → Port 3306 (internal)  → Persistent volume: mysql_data
backend   → Go API          → Port 8080 (internal)  → Image: shiv9/githubactions:latest
nginx     → Nginx Alpine    → Port 80 (exposed)     → Mounts: ./frontend, ./nginx/nginx.conf
```

### Backend Dockerfile (Multi-stage Build)

The backend uses a two-stage Docker build for minimal image size:

```dockerfile
# Stage 1: Build — compiles Go binary
FROM golang:1.26-alpine AS builder
WORKDIR /app
COPY . .
RUN go mod tidy && CGO_ENABLED=0 GOOS=linux go build -o skillpulse .

# Stage 2: Run — minimal Alpine image (~15MB)
FROM alpine:3.23
RUN apk --no-cache add ca-certificates
WORKDIR /app
COPY --from=builder /app/skillpulse .
EXPOSE 8080
CMD ["./skillpulse"]
```

### Key Docker Design Decisions

| Decision | Rationale |
|:---------|:----------|
| Backend as DockerHub image | Built in CI pipeline → consistent binary across environments |
| Frontend as volume mount | No build step needed → instant updates on `git pull` |
| MySQL init script | `init.sql` auto-executes on first container creation |
| Named volume (`mysql_data`) | Data persists across `docker compose down/up` |
| Nginx as reverse proxy | Single entry point on port 80 for both frontend + API |

---

## 🔄 CI/CD Pipeline

The project uses **two GitHub Actions workflows** that work together in a chain:

### CI — Continuous Integration

**File:** `.github/workflows/ci.yml`  
**Trigger:** Push to `main` branch

```
Push to main → Checkout → Setup Docker Buildx → Login DockerHub → Build & Push Image
```

| Step | Action | Details |
|:-----|:-------|:--------|
| 1 | Checkout code | `actions/checkout@v4` |
| 2 | Setup Docker Buildx | `docker/setup-buildx-action@v3` |
| 3 | Login to DockerHub | Uses `DOCKERHUB_TOKEN` secret |
| 4 | Build & Push | Builds `./backend` → pushes `shiv9/githubactions:latest` |

### CD — Continuous Deployment

**File:** `.github/workflows/cd.yml`  
**Trigger:** After CI workflow completes (`workflow_run`)

```
CI Completes → SSH into EC2 → git pull → docker pull → Smart Restart
```

| Step | Action | Details |
|:-----|:-------|:--------|
| 1 | SSH into EC2 | `appleboy/ssh-action@v1` |
| 2 | Clone/pull repo | Clones on first run, pulls latest on subsequent runs |
| 3 | Pull Docker image | Pulls `shiv9/githubactions:latest` from DockerHub |
| 4 | Smart restart | Only restarts backend if new image found; always restarts nginx |

**Smart Deployment Logic:**
```bash
# Only restarts backend if a new Docker image was downloaded
if grep -q "Downloaded newer image" /tmp/docker_pull.log; then
  docker compose up -d --no-deps backend    # Restart backend only
fi
docker compose restart nginx                 # Always restart nginx for frontend updates
```

### GitHub Secrets Required

Configure these in **GitHub → Repository → Settings → Secrets and Variables → Actions**:

| Secret Name | Description | Example |
|:------------|:------------|:--------|
| `DOCKERHUB_TOKEN` | DockerHub access token | `dckr_pat_xxxx` |
| `HOST` | EC2 public IP address | `13.60.252.3` |
| `EC2_USER` | SSH username for EC2 | `ubuntu` |
| `EC2_SSH_KEY` | Private SSH key (PEM content) | `-----BEGIN RSA PRIVATE KEY-----...` |

### Pipeline Flow Diagram

```
┌─────────────┐     ┌─────────────────────────────────────┐     ┌──────────────────────────────────┐
│  Developer   │     │         GitHub Actions CI            │     │       GitHub Actions CD           │
│             │     │                                     │     │                                  │
│  git push   │────▶│  1. Checkout code                   │     │                                  │
│  to main    │     │  2. Setup Docker Buildx             │     │                                  │
│             │     │  3. Login to DockerHub              │────▶│  1. SSH into EC2                 │
│             │     │  4. Build backend image             │     │  2. git pull origin main         │
│             │     │  5. Push shiv9/githubactions:latest │     │  3. docker pull latest image     │
│             │     │                                     │     │  4. Smart restart containers     │
└─────────────┘     └─────────────────────────────────────┘     └──────────────────────────────────┘
                                                                              │
                                                                              ▼
                                                                 ┌──────────────────────┐
                                                                 │  EC2 Instance         │
                                                                 │  http://13.60.252.3   │
                                                                 │                      │
                                                                 │  nginx  ← frontend/  │
                                                                 │  backend ← DockerHub │
                                                                 │  mysql  ← init.sql   │
                                                                 └──────────────────────┘
```

---

## ☁️ AWS EC2 Deployment

### EC2 Instance Setup

The EC2 instance needs the following software pre-installed:

```bash
# Update packages
sudo apt update && sudo apt upgrade -y

# Install Docker
sudo apt install -y docker.io
sudo systemctl enable docker
sudo systemctl start docker

# Install Docker Compose plugin
sudo apt install -y docker-compose-plugin

# Add ubuntu user to docker group (no sudo needed for docker commands)
sudo usermod -aG docker ubuntu
newgrp docker

# Install Git
sudo apt install -y git
```

### Security Group Rules

| Type | Port | Source | Purpose |
|:-----|:-----|:-------|:--------|
| SSH | 22 | Your IP / 0.0.0.0/0 | SSH access for deployment |
| HTTP | 80 | 0.0.0.0/0 | Web application access |
| MySQL | 3306 | Security Group self | Internal container communication |

> **Note:** Port 3306 only needs to be open within the security group (container-to-container). Do NOT expose it to the internet.

### Manual Deployment (First Time)

If deploying for the first time without CI/CD:

```bash
# SSH into EC2
ssh -i "your-key.pem" ubuntu@<EC2-PUBLIC-IP>

# Clone the repository
git clone https://github.com/shivchopade/github-actions-demo-shivam.git
cd github-actions-demo-shivam

# Start all services
docker compose up -d

# Verify
docker compose ps
curl http://localhost/health
```

### Verifying Deployment

```bash
# Check all containers are running
docker compose ps

# Expected output:
# NAME        SERVICE    STATUS
# ...-db-1    db         running
# ...-backend backend    running
# ...-nginx   nginx      running

# Check backend logs
docker compose logs backend

# Check if frontend files are mounted correctly
docker compose exec nginx ls /usr/share/nginx/html/

# Test API
curl http://localhost/api/dashboard

# Test health endpoint
curl http://localhost/health
# Expected: {"status":"healthy"}
```

---

## 📡 API Reference

Base URL: `http://<host>/api`

### Dashboard

| Method | Endpoint | Description |
|:-------|:---------|:------------|
| `GET` | `/api/dashboard` | Get dashboard statistics |

**Response:**
```json
{
  "total_skills": 5,
  "total_hours": 16.0,
  "total_logs": 9,
  "top_skill": "Docker"
}
```

### Skills

| Method | Endpoint | Description |
|:-------|:---------|:------------|
| `GET` | `/api/skills` | List all skills with total hours |
| `POST` | `/api/skills` | Create a new skill |
| `GET` | `/api/skills/:id` | Get skill details with learning logs |
| `DELETE` | `/api/skills/:id` | Delete a skill and all its logs |

**GET /api/skills — Response:**
```json
[
  {
    "id": 1,
    "name": "Docker",
    "category": "DevOps",
    "target_hours": 40,
    "total_hours": 6.5,
    "created_at": "2026-03-10T00:00:00Z"
  }
]
```

**POST /api/skills — Request:**
```json
{
  "name": "React",
  "category": "Programming",
  "target_hours": 50
}
```

**POST /api/skills — Response:**
```json
{
  "id": 6,
  "message": "Skill created"
}
```

**DELETE /api/skills/:id — Response:**
```json
{
  "message": "Skill deleted"
}
```

### Learning Logs

| Method | Endpoint | Description |
|:-------|:---------|:------------|
| `POST` | `/api/skills/:id/log` | Log a learning session for a skill |

**POST /api/skills/:id/log — Request:**
```json
{
  "hours": 1.5,
  "notes": "Learned Docker networking",
  "log_date": "2026-04-29"
}
```

**POST /api/skills/:id/log — Response:**
```json
{
  "id": 10,
  "message": "Learning session logged"
}
```

### Health Check

| Method | Endpoint | Description |
|:-------|:---------|:------------|
| `GET` | `/health` | Check backend + database health |

**Response (healthy):**
```json
{
  "status": "healthy"
}
```

**Response (unhealthy):**
```json
{
  "status": "unhealthy",
  "error": "connection refused"
}
```

---

## 🗄 Database Schema

**Database:** `skillpulse`  
**Engine:** MySQL 8.4

### `skills` Table

| Column | Type | Constraints | Description |
|:-------|:-----|:------------|:------------|
| `id` | `INT` | `PRIMARY KEY, AUTO_INCREMENT` | Unique skill identifier |
| `name` | `VARCHAR(100)` | `NOT NULL` | Skill name |
| `category` | `VARCHAR(50)` | `DEFAULT ''` | Category (DevOps, Programming, Cloud, etc.) |
| `target_hours` | `INT` | `DEFAULT 0` | Goal hours for this skill |
| `created_at` | `TIMESTAMP` | `DEFAULT CURRENT_TIMESTAMP` | Record creation time |

### `learning_logs` Table

| Column | Type | Constraints | Description |
|:-------|:-----|:------------|:------------|
| `id` | `INT` | `PRIMARY KEY, AUTO_INCREMENT` | Unique log identifier |
| `skill_id` | `INT` | `NOT NULL, FOREIGN KEY → skills(id)` | Associated skill |
| `hours` | `DECIMAL(4,1)` | `NOT NULL` | Hours spent in this session |
| `notes` | `TEXT` | | Optional session notes |
| `log_date` | `DATE` | `NOT NULL` | Date of the learning session |
| `created_at` | `TIMESTAMP` | `DEFAULT CURRENT_TIMESTAMP` | Record creation time |

### Relationships

```
skills (1) ──────< (N) learning_logs
         ON DELETE CASCADE
```

When a skill is deleted, all its associated learning logs are automatically deleted.

### Seed Data

The `mysql/init.sql` file pre-populates the database with:
- **5 skills:** Docker, Kubernetes, Go, Azure DevOps, Terraform
- **9 learning logs:** Sample sessions spread across multiple skills

---

## 🎨 Frontend

### Design System

The frontend uses a comprehensive CSS design system with **CSS custom properties (variables)** for both light and dark themes.

**Color Palette (Dark Theme):**

| Token | Value | Usage |
|:------|:------|:------|
| `--primary` | `#3b82f6` (Blue 500) | Buttons, active states, icons |
| `--bg` | `#0b1120` | Page background |
| `--surface` | `#111827` | Card backgrounds |
| `--text` | `#f8fafc` | Primary text |
| `--text-secondary` | `#94a3b8` | Secondary labels |

**Typography:** Inter (400, 500, 600, 700, 800) via Google Fonts

**Key UI Features:**
- CSS Grid for responsive stats (4 → 2 → 1 columns)
- `fadeInUp` animations with staggered delays
- Backdrop blur modal overlays
- Custom scrollbar styling
- No-cache nginx headers for instant updates

### Theme Toggle

Theme preference is:
1. Read from `localStorage` (if previously set)
2. Falls back to `prefers-color-scheme` media query
3. Stored in `localStorage` on toggle
4. Applied via `data-theme` attribute on `<html>`

---

## 🔧 Nginx Configuration

```nginx
server {
    listen 80;

    # Static frontend files
    location / {
        root /usr/share/nginx/html;
        index index.html;
        try_files $uri $uri/ /index.html;
        # No-cache headers for development
        add_header Cache-Control "no-cache, no-store, must-revalidate";
    }

    # API reverse proxy → Go backend
    location /api/ {
        proxy_pass http://backend:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }

    # Health check proxy
    location /health {
        proxy_pass http://backend:8080;
    }
}
```

---

## 🔍 Troubleshooting

### Frontend not updating after push?

```bash
# On EC2: Force-sync with GitHub
cd ~/github-actions-demo-shivam
git fetch origin main
git reset --hard origin/main
docker compose restart nginx
```

Then **hard-refresh** your browser: `Ctrl+Shift+R` (Windows/Linux) or `Cmd+Shift+R` (Mac)

### Backend can't connect to MySQL?

The backend has retry logic (30 attempts, 2s apart). Check:

```bash
# View backend logs
docker compose logs backend

# Verify MySQL is running
docker compose exec db mysqladmin -u skillpulse -pskillpulse123 ping
```

### Containers not starting?

```bash
# Check for port conflicts
sudo lsof -i :80
sudo lsof -i :3306

# Rebuild from scratch
docker compose down -v          # Remove containers AND volumes
docker compose up -d            # Fresh start with seed data
```

### GitHub Actions CD failing?

1. Verify all 4 secrets are set: `HOST`, `EC2_USER`, `EC2_SSH_KEY`, `DOCKERHUB_TOKEN`
2. Check EC2 security group allows SSH (port 22) from GitHub's IP ranges
3. Verify the SSH key format (should include `-----BEGIN RSA PRIVATE KEY-----`)
4. Check the Actions log for specific SSH error messages

### Docker Hub rate limiting?

```bash
# Login to DockerHub on EC2
docker login -u shiv9
```

---

## 🤝 Contributing

1. **Fork** the repository
2. **Create** a feature branch: `git checkout -b feature/my-feature`
3. **Commit** your changes: `git commit -m "feat: add my feature"`
4. **Push** to the branch: `git push origin feature/my-feature`
5. **Open** a Pull Request

### Coding Guidelines

- **Backend:** Follow standard Go conventions. Use `go fmt` before committing.
- **Frontend:** Use CSS custom properties for all colors. No inline styles.
- **Commits:** Follow [Conventional Commits](https://www.conventionalcommits.org/) format.

---

## 📄 License

This project is open source and available under the [MIT License](LICENSE).

---

<p align="center">
  Built with ❤️ by <a href="https://github.com/shivchopade">Shivam Chopade</a>
</p>
