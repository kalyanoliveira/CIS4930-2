# CIS4930 - Faculty Management Dashboard

A comprehensive faculty management system with React frontend and FastAPI backend.

## Project Structure

```
CIS4930/
├── frontend/          # React faculty management dashboard
│   ├── src/
│   │   ├── components/    # Faculty manager components
│   │   ├── App.jsx        # Main application
│   │   └── index.css      # Global styles
│   └── package.json       # Frontend dependencies
├── backend/           # FastAPI faculty management API
│   ├── main.py           # FastAPI app with faculty CRUD routes
│   ├── requirements.txt  # Python dependencies
│   ├── Dockerfile        # Backend container
│   └── .env             # Database configuration
├── docker-compose.yml # Container orchestration
├── Jenkinsfile        # CI/CD pipeline with health check
└── README.md
```

## Backend API (FastAPI)

### Faculty Management Endpoints

**GET /faculty**
- Returns list of all faculty members
- Response: `[{"id": 1, "name": "John Doe", "department": "Computer Science", "email": "john@university.edu"}, ...]`

**POST /faculty**
- Add a new faculty member
- Request body: `{"name": "string", "department": "string", "email": "string"}`
- Response: Created faculty object with ID

**DELETE /faculty/{id}**
- Remove a faculty member by ID
- Response: `{"message": "Faculty member deleted successfully"}`

**GET /health**
- Health check endpoint for Jenkins verification
- Response: `{"status": "ok"}`

### Database
- **SQLite** with SQLAlchemy ORM
- **Faculty table** with fields: id, name, department, email
- Auto-creates tables on startup

## Quick Start

### Backend Only
```bash
cd backend
pip install -r requirements.txt
uvicorn main:app --reload
# API available at http://localhost:8003
# Health check: http://localhost:8003/health
```

### Full Stack (Docker)
```bash
# Build and run all services
docker compose up --build

# Backend API: http://localhost:8003
# Frontend Dashboard: http://localhost:80
```

### Frontend Only
```bash
cd frontend
yarn install
yarn dev
# Dashboard at http://localhost:5174
```

## Testing the API

```bash
# Health check
curl http://localhost:8003/health

# Get all faculty
curl http://localhost:8003/faculty

# Add a faculty member
curl -X POST http://localhost:8003/faculty \
  -H "Content-Type: application/json" \
  -d '{"name": "Dr. Jane Smith", "department": "Computer Science", "email": "jane@university.edu"}'

# Delete a faculty member
curl -X DELETE http://localhost:8003/faculty/1
```

## CI/CD Pipeline (Complete)

The Jenkins pipeline automates the entire deployment process with four stages:

### Pipeline Stages

1. **Checkout** (Kalyan)
   - Clones latest code from GitHub repository
   - Command: `checkout scm`
   - Ensures pipeline builds from most recent commit

2. **Build** (Caio)
   - Builds Docker images for frontend and backend
   - Command: `docker compose build`
   - Uses layer caching for faster builds
   - Timeout: 15 minutes

3. **Verify** (Sofia)
   - Starts backend container and validates health
   - Command: `docker compose up -d --wait backend`
   - Checks `/health` endpoint using Python urllib
   - Timeout: 3 minutes with automatic retries

4. **Deploy** (Kalyan)
   - Deploys full application stack
   - Commands: `docker compose down` then `docker compose up -d --wait`
   - Waits for all services to be healthy
   - Timeout: 3 minutes

### Automated Deployment

- **GitHub webhook** triggers pipeline on every push to main
- **Webhook URL**: `http://3.137.158.102:8080/github-webhook/`
- No manual intervention required
- Automatic cleanup on failure

### Health Check Integration

```yaml
backend:
  healthcheck:
    test: ["CMD", "python3", "-c", "import urllib.request; urllib.request.urlopen('http://localhost:8003/health')"]
    interval: 10s
    timeout: 5s
    retries: 5
    start_period: 40s
```

- Uses Python's built-in urllib (no curl dependency needed)
- 40-second grace period for pip dependencies to install
- Checks endpoint every 10 seconds
- Frontend waits for backend health before starting

## Technology Stack

**Backend:**
- **FastAPI** - Modern Python web framework
- **SQLAlchemy** - ORM for database operations
- **Pydantic** - Data validation
- **Uvicorn** - ASGI server
- **SQLite** - Lightweight database

**Frontend:**
- **React 18** - Component-based UI library
- **Vite** - Fast build tool
- **Tailwind CSS** - Utility-first styling
- **PostCSS & Autoprefixer** - CSS processing

**Infrastructure:**
- **Docker & Docker Compose** - Containerization
- **Jenkins** - CI/CD automation
- **Nginx** - Frontend web server
- **AWS EC2** - Cloud hosting
- **GitHub** - Version control

## Production Deployment

**Live Application:** http://3.137.158.102

**Access Points:**
- **Frontend**: http://3.137.158.102 (port 80)
- **Backend API**: http://3.137.158.102:8003
- **Jenkins Dashboard**: http://3.137.158.102:8080

## Team Members

- **Caio** - Infrastructure & Containerization
  - EC2 setup, Docker installation, Dockerfile creation
  - Jenkins installation and configuration
  - Build stage implementation

- **Sofia** - Application Development & Verification
  - FastAPI backend development
  - React frontend with CRUD functionality
  - Verify stage with health checks

- **Kalyan** - CI/CD Pipeline & Documentation
  - Checkout and Deploy stage implementation
  - docker-compose.yml orchestration
  - GitHub webhook configuration
  - Project documentation