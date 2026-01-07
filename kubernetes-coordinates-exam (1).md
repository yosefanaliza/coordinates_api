# Kubernetes Deployment + StatefulSet Exam: Coordinates API

## Story

After Yosef's heroic assignment in Rafah, where he successfully infiltrated enemy territory under heavy fire and extracted critical intelligence, he was immediately recruited by the Mossad for his exceptional field skills. His new mission: Operation Coordinate Strike.

For months, Yosef worked undercover, piecing together fragments of intelligence from intercepted communications, surveillance footage, and informant networks. Through dangerous reconnaissance missions across hostile territories, he finally cracked the cell's communication patterns and identified their safe houses.

Now, the final phase begins. Yosef has been tasked with building a secure, distributed coordinate tracking system that will allow Mossad operatives in the field to input real-time GPS coordinates of enemy positions. The system must be resilient - if one component fails, the mission cannot be compromised. The coordinates collected will be used to coordinate a simultaneous multi-site operation to neutralize the threats to the state.

Your mission: Build the infrastructure that will support Operation Coordinate Strike.

---

## Repository Setup

**You MUST download and work from this repository:**

```
https://github.com/yosefanaliza/coordinates_api
```

**Download as ZIP** and extract it to your working directory.

**What's included in the repository:**
- `main.py` (the FastAPI application)
- `requirements.txt` (Python dependencies)
- `.env.example` (environment variable template)
- `.gitignore` (Git ignore file)
- `.dockerignore` (Docker ignore file)
- `README.md` (repository documentation)
- **NO Dockerfile** (you must create it)
- **NO k8s/ directory** (you must create it)

---

## Expected Repository Structure

Your final repository should look like this:

```
coordinates-k8s/
├── Dockerfile
├── main.py
├── requirements.txt
├── .env.example
├── .gitignore
├── .dockerignore
├── README.md
├── k8s/
│   ├── postgres-statefulset.yaml
│   ├── postgres-service.yaml
│   ├── api-deployment.yaml
│   └── api-service.yaml
└── screenshots/
    ├── kubectl-get-all.png
    ├── api-response.png
    └── cluster-ui.png
```

**Important:**
- Keep all original files from the downloaded repository
- Add your `Dockerfile` at the root
- Create the `k8s/` directory with all 4 YAML files
- Create the `screenshots/` directory with verification images
- Update the `README.md` with deployment instructions

---

## Deliverables Checklist

You must create these **exact files** with these **exact names**:

```
✅ Dockerfile (at repository root)
✅ k8s/api-deployment.yaml
✅ k8s/api-service.yaml
✅ k8s/postgres-statefulset.yaml
✅ k8s/postgres-service.yaml
```

---

## Git Workflow Requirements

You must use Git branching throughout this exam. Each component must be developed on its own feature branch.

**Required branch structure:**

```
main (final working application)
├── feature/dockerfile
├── feature/postgres-statefulset
├── feature/postgres-service
├── feature/api-deployment
└── feature/api-service
```

**Workflow:**
1. Start with the base codebase from the repository
2. Create a feature branch for each component
3. Develop the component on its feature branch
4. Merge the completed component into `main`
5. Continue with the next component

**Branch names you must use:**
- `feature/dockerfile` - for the Dockerfile
- `feature/postgres-statefulset` - for `k8s/postgres-statefulset.yaml`
- `feature/postgres-service` - for `k8s/postgres-service.yaml`
- `feature/api-deployment` - for `k8s/api-deployment.yaml`
- `feature/api-service` - for `k8s/api-service.yaml`

Your final `main` branch must contain the complete, working application with all components integrated.

---

## Part 1: Docker Requirements

### 1.1 Create the Dockerfile

Create `Dockerfile` at the repository root with these **exact requirements:**

| Requirement | Value |
|------------|-------|
| Base image | `python:3.11-slim` |
| Working directory | `/app` |
| Copy location | Copy all files into `/app` |
| Dependencies | Install from `requirements.txt` |
| Exposed port | `8000` |
| Run command | `python main.py` |

**The application MUST live at path `/app` inside the container.**

### 1.2 Build and Push Image

**Your image tag MUST be:** `<dockerhub_username>/coordinates_api:v1`

Replace `<dockerhub_username>` with your actual Docker Hub username.

**Commands to run:**

```bash
docker build -t <dockerhub_username>/coordinates_api:v1 .
docker push <dockerhub_username>/coordinates_api:v1
```

---

## Part 2: Kubernetes Requirements

### 2.1 PostgreSQL StatefulSet

**File:** `k8s/postgres-statefulset.yaml`

**Requirements:**

| Field | Value |
|-------|-------|
| Kind | `StatefulSet` |
| Name | `postgres` |
| Replicas | `3` |
| Container image | `postgres:15` |
| Container port | `5432` |
| serviceName | `postgres-svc` |

**Environment variables for Postgres container:**
- `POSTGRES_DB` = `coordinates_db`
- `POSTGRES_USER` = `postgres`
- `POSTGRES_PASSWORD` = `postgres`

**volumeClaimTemplates required:**
- Name: `postgres-storage`
- accessModes: `["ReadWriteOnce"]`
- storage: `1Gi`
- mountPath: `/var/lib/postgresql/data`

---

### 2.2 PostgreSQL Service (Headless)

**File:** `k8s/postgres-service.yaml`

**Requirements:**

| Field | Value |
|-------|-------|
| Kind | `Service` |
| Name | `postgres-svc` |
| Type | `ClusterIP` |
| clusterIP | `None` (headless service) |
| Port | `5432` |
| Selector | Must match StatefulSet labels |

**This is a headless service** (clusterIP: None) required for StatefulSets.

---

### 2.3 API Deployment

**File:** `k8s/api-deployment.yaml`

**Requirements:**

| Field | Value |
|-------|-------|
| Kind | `Deployment` |
| Name | `coordinates-api` |
| Replicas | `3` |
| Container image | `<dockerhub_username>/coordinates_api:v1` |
| Container port | `8000` |

**Environment Variables (REQUIRED):**

The API container MUST have these environment variables:

| Variable | Value |
|----------|-------|
| `POSTGRES_HOST` | `postgres-svc` |
| `POSTGRES_PORT` | `5432` |
| `POSTGRES_DB` | `coordinates_db` |
| `POSTGRES_USER` | `postgres` |
| `POSTGRES_PASSWORD` | `postgres` |

**Note:** You may use ConfigMap or Secret for these values, but:
- ConfigMap/Secret files must be in `k8s/` directory
- Must be referenced correctly in Deployment

---

### 2.4 API Service (NodePort)

**File:** `k8s/api-service.yaml`

**Requirements:**

| Field | Value |
|-------|-------|
| Kind | `Service` |
| Name | `coordinates-api-svc` |
| Type | `NodePort` |
| Port (service) | `8000` |
| targetPort | `8000` |
| nodePort | `30080` (must be explicitly set) |
| Selector | Must match Deployment labels |

**Port mapping explanation:**
- External traffic hits any cluster node on port `30080`
- Routes to service on port `8000`
- Forwards to pod container port `8000`

---

## Part 3: Verification

### 3.1 Expected Resources

After deploying all resources, verify with:

```bash
kubectl get all
```

**Expected resources:**
- 1 StatefulSet (postgres) with 3 pods
- 1 Deployment (coordinates-api) with 3 pods
- 2 Services (postgres-svc, coordinates-api-svc)

**Check pods are running:**

```bash
kubectl get pods
```

All pods should show `STATUS: Running` and `READY: 1/1` (or 3/3).

### 3.2 Test the Application

**Access the API:**

```bash
curl http://<node-ip>:30080/
```

Or visit in browser: `http://localhost:30080/` (if using local cluster)

**Expected response:** JSON response from the API indicating it's running.

---

## Common Kubernetes Mistakes

- ❌ Wrong service name for POSTGRES_HOST (must be `postgres-svc`)
- ❌ Wrong selector labels (Service must match Deployment/StatefulSet)
- ❌ Forgot NodePort `30080` in api-service
- ❌ Wrong image name (must match what you pushed to Docker Hub)
- ❌ Postgres service not headless (forgot `clusterIP: None`)
- ❌ Wrong container port (must be `8000` for API, `5432` for Postgres)
- ❌ Forgot volumeClaimTemplates in StatefulSet
- ❌ Environment variables incorrect or missing

---

## Submission

Create a **GitHub repository** containing:
- `Dockerfile`
- `k8s/` directory with all 4 YAML files
- `screenshots/` directory with:
  - Screenshot of `kubectl get all` showing all resources
  - Screenshot of the response from calling the `/coordinates` endpoint via curl (after posting a few coordinates)
  - Screenshot of cluster UI dashboard (Minikube dashboard or Docker Desktop Kubernetes UI) showing deployed resources
- `README.md` with:
  - Your full name
  - Your student ID
  - Your class name

**Repository naming:** `coordinates-k8s`

**Submit the GitHub repository URL.**

---

## Grading Standards

**Total Points: 90**

**Feature Branches (25 points):**
- 5 points per feature branch (must use exact branch names specified)
- All 5 feature branches must be visible in repository history
- Branches: `feature/dockerfile`, `feature/postgres-statefulset`, `feature/postgres-service`, `feature/api-deployment`, `feature/api-service`

**Implementation (50 points):**
- Dockerfile correct and working
- PostgreSQL StatefulSet with 3 replicas and volumeClaimTemplates
- PostgreSQL headless Service
- API Deployment with 3 replicas
- API NodePort Service on port 30080
- Application accessible and functional

**Screenshots (15 points):**
- Screenshot of `kubectl get all` showing all resources (5 points)
- Screenshot of `/coordinates` endpoint response after posting coordinates (5 points)
- Screenshot of cluster UI dashboard showing deployed resources (5 points)
  - Minikube dashboard or Docker Desktop Kubernetes UI

**The primary evaluation will be on your `main` branch, which must contain the final, integrated, working application.**

**Note:** This exam is worth 90 points. There is also a separate 10-point multiple choice test (not included here).

---

**Good luck! 🎯**
