# Coordinates API - Kubernetes Deployment

A FastAPI application that manages geographic coordinates with PostgreSQL backend, deployed using Kubernetes StatefulSet.

## Overview

This project demonstrates:
- FastAPI REST API with PostgreSQL integration
- Docker containerization

## Prerequisites

- Python 3.11+
- Docker
- Kubernetes cluster (e.g., Minikube)
- kubectl configured

## Files in This Project

```
.
├── main.py                  # FastAPI application
├── requirements.txt         # Python dependencies
├── Dockerfile              # Container definition (you'll write this)
├── .env                     # Environment variables (not committed)
├── .env.example            # Environment template
├── .gitignore              # Git ignore file
├── .dockerignore           # Docker ignore file
└── README.md               # This file

```

## Local Development

### Setup Virtual Environment

```bash
python -m venv .venv
source .venv/bin/activate  # On Windows: .venv\Scripts\activate
pip install -r requirements.txt
```

### Configure Environment

Copy `.env.example` to `.env` and update values:

```bash
cp .env.example .env
```

Edit `.env` with your database credentials:

```env
POSTGRES_HOST=localhost
POSTGRES_PORT=5432
POSTGRES_DB=coordinates_db
POSTGRES_USER=postgres
POSTGRES_PASSWORD=your_password
```

### Run Application

```bash
python main.py
```

Application runs on `http://localhost:8000`

## API Endpoints

### Health Check
```bash
GET /
```

Response:
```json
{
  "status": "ok",
  "message": "Coordinates API is running"
}
```

### Add Coordinate
```bash
POST /coordinates
Content-Type: application/json

{
  "latitude": 40.7128,
  "longitude": -74.0060,
  "name": "New York City"
}
```

Response (201 Created):
```json
{
  "id": 1,
  "latitude": 40.7128,
  "longitude": -74.0060,
  "name": "New York City"
}
```

### Get All Coordinates
```bash
GET /coordinates
```

Response:
```json
[
  {
    "id": 1,
    "latitude": 40.7128,
    "longitude": -74.0060,
    "name": "New York City"
  }
]
```

### Delete Coordinate
```bash
DELETE /coordinates/{id}
```


## Kubernetes Deployment

### Step 1: Write the Dockerfile

Complete the `Dockerfile` in the root directory. It should:
- Use python:3.11-slim as base image
- Copy all files with `COPY . .`
- Install dependencies from requirements.txt
- Expose port 8000
- Run `python main.py`

### Step 2: Create Docker Image

```bash
docker build -t coordinates-api:v1 .
```

Push to your registry (Docker Hub example):
```bash
docker push yourusername/coordinates-api:v1
```

