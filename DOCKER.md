# Docker Setup Guide

## Overview
This project uses Docker and Docker Compose to containerize the Multi-Agent AI Travel System, including the Streamlit frontend and PostgreSQL database.

## Prerequisites
- [Docker](https://docs.docker.com/get-docker/) (v20.10+)
- [Docker Compose](https://docs.docker.com/compose/install/) (v1.29+)

## Quick Start

### 1. Configure Environment Variables
Create a `.env` file in the project root with your API keys:

```bash
cp .env.example .env
```

Edit `.env` and add your actual API keys:
```
GROQ_API_KEY=your_actual_groq_key
TAVILY_API_KEY=your_actual_tavily_key
AVIATIONSTACK_API_KEY=your_actual_aviationstack_key
```

### 2. Build and Run with Docker Compose

**Start all services:**
```bash
docker-compose up --build
```

The `--build` flag rebuilds the image if you made code changes.

**Run in detached mode (background):**
```bash
docker-compose up -d --build
```

### 3. Access the Application

- **Streamlit Frontend**: http://localhost:8501
- **PostgreSQL**: localhost:5432 (if connecting from host machine)

## Useful Docker Compose Commands

### View logs
```bash
# All services
docker-compose logs -f

# Specific service
docker-compose logs -f frontend
docker-compose logs -f postgres
```

### Stop services
```bash
docker-compose down
```

### Remove volumes (deletes database data)
```bash
docker-compose down -v
```

### Rebuild without cache
```bash
docker-compose up --build --no-cache
```

### Enter a container shell
```bash
# Frontend container
docker-compose exec frontend bash

# PostgreSQL container
docker-compose exec postgres psql -U postgres -d langgraph_memory
```

## Service Details

### Frontend Service
- **Image**: Built from Dockerfile (Python 3.12 + dependencies)
- **Port**: 8501
- **Depends On**: PostgreSQL (waits for health check)
- **Volumes**: Current directory mounted to /app (live code updates)
- **Restart Policy**: unless-stopped

### PostgreSQL Service
- **Image**: postgres:16-alpine
- **Port**: 5432
- **Database**: langgraph_memory
- **User**: postgres
- **Password**: Bmw1250GS@#&
- **Volume**: postgres_data (persistent storage)
- **Health Check**: Enabled

## Troubleshooting

### Port Already in Use
If port 8501 or 5432 is already in use:

```bash
# Change in docker-compose.yml
ports:
  - "8502:8501"    # External:Internal
  - "5433:5432"
```

### Database Connection Issues
- Ensure PostgreSQL is healthy: `docker-compose logs postgres`
- Check DATABASE_URL in docker-compose.yml is correct
- Password special chars are URL-encoded: `@=%40`, `#=%23`, `&=%26`

### Rebuilding Everything from Scratch
```bash
docker-compose down -v
docker-compose up --build
```

### Checking Container Health
```bash
docker-compose ps
```

Look for "Up" status and "(healthy)" for postgres.

## Production Deployment Notes

For production deployments, consider:

1. **Security**:
   - Use secrets management instead of .env
   - Change default PostgreSQL password
   - Use environment-specific configurations

2. **Performance**:
   - Add resource limits in docker-compose.yml
   - Use separate databases for each environment
   - Implement proper logging aggregation

3. **Scaling**:
   - Consider using Kubernetes or container orchestration
   - Add load balancing for multiple frontend instances
   - Optimize database connection pooling

## File Structure

```
.
├── Dockerfile              # Frontend service image definition
├── docker-compose.yml      # Services orchestration
├── .dockerignore          # Files to exclude from Docker build
├── .env                   # Environment variables (create from .env.example)
├── .env.example           # Template for environment variables
├── frontend.py            # Streamlit application
├── main.py               # LangGraph orchestration
├── requirements.txt      # Python dependencies
├── tools/               # Utility tools
│   ├── flight_tool.py
│   └── tavily_tool.py
└── DOCKER.md            # This file
```

## Development Workflow

### Local Development with Live Reload
```bash
docker-compose up
```

Code changes are reflected immediately since the project directory is mounted as a volume.

### Rebuilding After Dependency Changes
```bash
docker-compose up --build
```

This rebuilds the image if requirements.txt was modified.

### Testing Before Deployment
```bash
# Test all services start correctly
docker-compose up --build

# Verify frontend is accessible
curl http://localhost:8501/_stcore/health

# Verify database connection
docker-compose exec frontend python -c "import psycopg; print('Database OK')"
```

## Additional Resources

- [Docker Documentation](https://docs.docker.com/)
- [Docker Compose Documentation](https://docs.docker.com/compose/)
- [Streamlit in Docker](https://docs.streamlit.io/deploy/tutorials/docker)
- [PostgreSQL Docker Image](https://hub.docker.com/_/postgres)
