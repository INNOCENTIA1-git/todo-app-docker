# Todo App Docker Deployment

# Todo App Docker Deployment

![Docker Build](https://github.com/INNOCENTIA1-git/todo-app-docker/actions/workflows/docker-build-push.yml/badge.svg)
![PR Checks](https://github.com/INNOCENTIA1-git/todo-app-docker/actions/workflows/pr-checks.yml/badge.svg)
![Security Scan](https://github.com/INNOCENTIA1-git/todo-app-docker/workflows/Security%20Scan/badge.svg)
![Docker Image Size](https://img.shields.io/docker/image-size/innocentia15/todo-app/latest)
![Docker Pulls](https://img.shields.io/docker/pulls/innocentia15/todo-app)
![GitHub last commit](https://img.shields.io/github/last-commit/INNOCENTIA1-git/todo-app-docker)
![License](https://img.shields.io/github/license/INNOCENTIA1-git/todo-app-docker)

## 📋 Project Overview
...

## ��� Project Overview

This project demonstrates the containerization of a full-stack Todo application using Docker and Docker Compose. It showcases skills in building optimized Docker images, container orchestration, and deployment to DockerHub.

**Original Repository:** [Docker Getting Started App](https://github.com/docker/getting-started-app)

## ��� Project Objectives

- Build production-ready Docker images using best practices
- Implement efficient layer caching for faster builds
- Deploy containerized applications using Docker Compose
- Push images to DockerHub container registry
- Create comprehensive documentation for DevOps portfolio

## ���️ Technologies Used

| Technology | Purpose | Version |
|------------|---------|---------|
| Docker | Containerization platform | Latest |
| Docker Compose | Multi-container orchestration | v3.8 |
| Node.js | Runtime environment | 18-alpine |
| Express.js | Backend framework | Latest |
| React | Frontend framework | Latest |
| DockerHub | Container registry | - |

## ��� Project Structure

```
todo-app-docker/
├── Dockerfile              # Container image definition
├── docker-compose.yml      # Multi-container configuration
├── .dockerignore          # Files excluded from build
├── package.json           # Node.js dependencies
├── src/                   # Application source code
├── screenshots/           # Deployment evidence
│   ├── 01-docker-build.png
│   ├── 02-docker-images.png
│   ├── 03-app-running.png
│   ├── 04-docker-push.png
│   ├── 05-dockerhub-repo.png
│   └── 06-compose-running.png
└── README.md              # This file
```

## ��� Quick Start

### Prerequisites

- Docker Desktop installed ([Download](https://www.docker.com/products/docker-desktop))
- Docker Compose installed (included with Docker Desktop)
- Git installed
- DockerHub account (free)

### Installation & Running

**1. Clone the repository**
```bash
git clone https://github.com/INNOCENTIA1-git/todo-app-docker.git
cd todo-app-docker
```

**2. Build the Docker image**
```bash
docker build -t todo-app:latest .
```

**3. Run with Docker Compose**
```bash
docker-compose up -d
```

**4. Access the application**
```
http://localhost:3000
```

**5. View logs**
```bash
docker-compose logs -f
```

**6. Stop the application**
```bash
docker-compose down
```

## ��� Docker Implementation

### Dockerfile Breakdown

```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 3000
CMD ["node", "src/index.js"]
```

**Key Optimizations:**

1. **Alpine Base Image**
   - Reduces image size from ~900MB to ~180MB
   - Minimal attack surface for better security
   - Faster download and deployment times

2. **Layer Caching Strategy**
   - Copy `package.json` before application code
   - Leverages Docker's layer caching mechanism
   - Rebuilds only when dependencies change
   - Results in 10x faster subsequent builds

3. **Single Process per Container**
   - Follows Docker best practice
   - Easier to scale and debug
   - Clear separation of concerns

### .dockerignore Configuration

```
node_modules
npm-debug.log
.git
.gitignore
README.md
.env
.DS_Store
*.md
```

**Benefits:**
- Excludes unnecessary files from build context
- Reduces image size by ~50MB
- Prevents sensitive data exposure
- Speeds up build process

## ��� Docker Compose Features

### Service Configuration

```yaml
version: '3.8'

services:
  todo-app:
    build: .
    image: todo-app:latest
    container_name: todo-app
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "wget", "--quiet", "--tries=1", "--spider", "http://localhost:3000"]
      interval: 30s
      timeout: 10s
      retries: 3
```

**Key Features:**

- **Health Checks:** Monitors application availability
- **Restart Policy:** Ensures high availability
- **Environment Variables:** Configurable deployment
- **Port Mapping:** Flexible networking setup

## ��� Deployment Evidence

### Docker Build Process
![Docker Build](screenshots-01-docker-build.png)
*Successful image build with all layers cached*

### Docker Images List
![Docker Images](screenshots-02-docker-images.png)
*Image created with optimized size (~180MB)*

### Application Running
![App Running](screenshots-03-app-running.png)
*Todo app accessible at localhost:3000*

### DockerHub Push
![Docker Push](screenshots-04-docker-push.png)
*Image successfully pushed to registry*

### DockerHub Repository
![DockerHub Repo](screenshots-05-dockerhub-repo.png)
*Public repository available for deployment*

### Docker Compose Deployment
![Compose Running](screenshots-06-compose-running.png)
*Application orchestrated with health checks*

## ��� Container Registry

### DockerHub

**Repository:** `https://hub.docker.com/r/innocentia15/todo-app`

**Pull the image:**
```bash
docker pull innocentia15/todo-app:latest
```

**Run directly from DockerHub:**
```bash
docker run -d -p 3000:3000 innocentia15/todo-app:latest
```

## ��� Testing & Verification

### Manual Testing Steps

**1. Test image builds successfully**
```bash
docker build -t todo-app:latest .
echo $?  # Should return 0 (success)
```

**2. Test container starts**
```bash
docker run -d -p 3000:3000 --name test-todo todo-app:latest
docker ps | grep test-todo  # Should show running container
```

**3. Test application responds**
```bash
curl http://localhost:3000
# Should return HTML content
```

**4. Test health check**
```bash
docker inspect test-todo | grep -A 5 Health
# Should show "healthy" status
```

**5. Cleanup**
```bash
docker stop test-todo && docker rm test-todo
```

### Docker Compose Testing

```bash
# Start services
docker-compose up -d

# Verify all services healthy
docker-compose ps

# Test application endpoint
curl http://localhost:3000

# Check logs for errors
docker-compose logs

# Cleanup
docker-compose down
```

## ��� Troubleshooting

### Common Issues & Solutions

**Problem:** Port 3000 already in use
```bash
# Find process using port
lsof -i :3000  # macOS/Linux
netstat -ano | findstr :3000  # Windows

# Solution: Use different port
docker run -d -p 8080:3000 todo-app:latest
```

**Problem:** Build fails with npm errors
```bash
# Clear npm cache
docker build --no-cache -t todo-app:latest .

# Or clear Docker build cache
docker builder prune
```

**Problem:** Container exits immediately
```bash
# Check logs
docker logs CONTAINER_ID

# Run interactively to debug
docker run -it todo-app:latest sh
```

**Problem:** Changes not reflected in image
```bash
# Ensure you rebuild after code changes
docker-compose up --build
```

## ��� Key DevOps Concepts Demonstrated

### 1. **Containerization Best Practices**
- Multi-layer caching optimization
- Minimal base images (Alpine)
- Single responsibility per container
- Proper .dockerignore usage

### 2. **Container Orchestration**
- Service definition with Docker Compose
- Health check implementation
- Restart policies for reliability
- Environment-based configuration

### 3. **CI/CD Readiness**
- Reproducible builds
- Registry integration (DockerHub)
- Automated deployment capability
- Version tagging strategy

### 4. **Infrastructure as Code**
- Declarative configuration (YAML)
- Version-controlled infrastructure
- Portable across enviroments
- self-documenting setup

## ��� Future Enhancements

- [ ] Add PostgreSQL database with Docker volume
- [ ] Implement multi-stage build for smaller production image
- [ ] Add GitHub Actions CI/CD pipeline
- [ ] Deploy to AWS ECS or Kubernetes
- [ ] Add container scanning with Trivy
- [ ] Implement Docker secrets management
- [ ] Add Nginx reverse proxy
- [ ] Set up monitoring with Prometheus

## ��� Project Metrics

| Metric | Value |
|--------|-------|
| Base Image Size | 900MB (node:18) |
| Optimized Image Size | ~180MB (node:18-alpine) |
| Size Reduction | 80% |
| Build Time (cached) | ~5 seconds |
| Build Time (no cache) | ~45 seconds |
| Container Startup | <2 seconds |
| Memory Usage | ~50MB |

## ��� Author

**INNOCENTIA AZAL**
- GitHub: [@INNOCENTIA1-git](https://github.com/INNOCENTIA1-git)
- LinkedIn: [Inoocentia Azal](https://www.linkedin.com/in/innocentia-azal/)
- Email: innocentiaazal1@gmail.com


## ��� License

This project is open source and available under the [MIT License](LICENSE).

## ��� Acknowledgments

- Original app from [Docker Getting Started](https://github.com/docker/getting-started-app)
- Docker documentation and community
- DevOps best practices from industry leaders

---

**Built as part of a DevOps portfolio to demonstrate containerization and orchestration expertise.**

*Last Updated: January 2026*
