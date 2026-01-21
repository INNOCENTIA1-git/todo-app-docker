# CI/CD Pipeline Documentation

## Overview

This project uses GitHub Actions for automated CI/CD pipelines, ensuring every code change is built, tested, and deployed automatically.

## Workflows

### 1. Docker Build and Push (`docker-build-push.yml`)

**Triggers:**
- Push to `main` branch
- Pull requests to `main`
- Manual workflow dispatch

**Jobs:**

#### Build and Test
- Checks out code
- Sets up Docker Buildx
- Builds Docker image
- Runs container health test
- Uses GitHub cache for speed

#### Push to DockerHub
- Runs only on `main` branch
- Authenticates with DockerHub
- Builds and pushes image
- Tags: `latest`, `main-{sha}`

#### Push to AWS ECR
- Runs only on `main` branch
- Configures AWS credentials
- Authenticates with ECR
- Pushes to ECR repository
- Tags: `latest`, `{sha}`

#### Security Scan
- Scans image with Trivy
- Checks for vulnerabilities
- Uploads results to GitHub Security tab

**Duration:** ~3-5 minutes

---

### 2. Pull Request Checks (`pr-checks.yml`)

**Triggers:**
- Pull requests to `main`

**Jobs:**

#### Dockerfile Lint
- Uses Hadolint to check Dockerfile best practices
- Fails on warnings

#### Compose Validate
- Validates docker-compose.yml syntax
- Ensures configuration is valid

#### Code Quality
- Checks for trailing whitespace
- Identifies large files
- Ensures code standards

#### Image Size Check
- Builds image
- Measures size
- Alerts if > 300MB

**Duration:** ~2-3 minutes

---

### 3. Release (`release.yml`)

**Triggers:**
- Tags matching `v*.*.*` (e.g., v1.0.0)

**Jobs:**

#### Create GitHub Release
- Creates release notes
- Documents Docker pull commands
- Links to installation instructions

#### Build and Push Versioned
- Builds with version tags
- Pushes to DockerHub with semantic versioning:
  - `latest`
  - `v1.0.0`
  - `1.0.0`
  - `1` (major)
  - `1.0` (minor)
- Pushes to ECR with version tags

**Duration:** ~4-6 minutes

---

## GitHub Secrets Configuration

### Required Secrets

| Secret Name | Description | Example |
|-------------|-------------|---------|
| `DOCKERHUB_USERNAME` | DockerHub username | `innocentia15` |
| `DOCKERHUB_TOKEN` | DockerHub access token | `dckr_pat_xxx...` |
| `AWS_ACCOUNT_ID` | AWS account ID | `123456789012` |
| `AWS_ACCESS_KEY_ID` | AWS access key | `AKIA...` |
| `AWS_SECRET_ACCESS_KEY` | AWS secret key | `xxx...` |
| `AWS_REGION` | AWS region | `us-east-1` |

### Creating Secrets

1. Go to repository **Settings**
2. Navigate to **Secrets and variables** → **Actions**
3. Click **New repository secret**
4. Add each secret listed above

---

## Workflow Status Badges

Add these to your README.md to show pipeline status:

```markdown
![Docker Build](https://github.com/INNOCENTIA1-git/todo-app-docker/actions/workflows/docker-build-push.yml/badge.svg)
![PR Checks](https://github.com/INNOCENTIA1-git/todo-app-docker/actions/workflows/pr-checks.yml/badge.svg)
![Security Scan](https://github.com/INNOCENTIA1-git/todo-app-docker/actions/workflows/docker-build-push.yml/badge.svg?event=push)
```

---

## Development Workflow

### Making Changes

```bash
# 1. Create feature branch
git checkout -b feature/new-feature

# 2. Make changes
# Edit files...

# 3. Commit and push
git add .
git commit -m "Add new feature"
git push origin feature/new-feature

# 4. Create pull request on GitHub
# PR checks will run automatically

# 5. After review, merge to main
# Build and push workflow runs automatically
```

### Creating a Release

```bash
# 1. Update version in files
# 2. Commit changes
git add .
git commit -m "Bump version to 1.0.0"

# 3. Create and push tag
git tag v1.0.0
git push origin v1.0.0

# 4. Release workflow runs automatically
# Creates GitHub release and versioned images
```

---

## Build Optimization

### Caching Strategy

All workflows use GitHub Actions cache:

```yaml
cache-from: type=gha
cache-to: type=gha,mode=max
```

**Benefits:**
- 5-10x faster builds
- Reuses Docker layers
- Reduces build time from 3 min → 30 sec

### Build Performance

| Metric | First Build | Cached Build |
|--------|-------------|--------------|
| Duration | ~3-4 min | ~30-45 sec |
| Layers cached | 0% | ~80% |
| Data downloaded | ~500 MB | ~50 MB |

---

## Security Features

### Vulnerability Scanning

**Tool:** Trivy (Aqua Security)

**What it scans:**
- OS packages
- Application dependencies
- Known CVEs
- Misconfigurations

**Results:**
- Uploaded to GitHub Security tab
- Viewable under Security → Code scanning
- Alerts on critical/high vulnerabilities

### Secret Management

**Best Practices:**
- ✅ Never commit secrets to code
- ✅ Use GitHub Secrets for credentials
- ✅ Rotate tokens regularly
- ✅ Use minimal IAM permissions
- ✅ Enable 2FA on all accounts

---

## Monitoring and Logs

### Viewing Workflow Runs

1. Go to repository **Actions** tab
2. Click on workflow run
3. View job logs
4. Download artifacts if needed

### Troubleshooting Failed Builds

**Check:**
1. Workflow logs for error messages
2. Secrets are configured correctly
3. AWS/DockerHub credentials are valid
4. Dockerfile syntax is correct
5. Tests are passing locally

**Common Issues:**

| Error | Cause | Solution |
|-------|-------|----------|
| Authentication failed | Invalid token | Regenerate and update secret |
| Build timeout | Large image, slow network | Optimize Dockerfile, use cache |
| Test failure | Container not starting | Check ports, dependencies |
| Push denied | Insufficient permissions | Update IAM/DockerHub permissions |

---

## Performance Metrics

### Typical Workflow Times

**Full Pipeline (main branch):**
```
Build and Test:     2-3 minutes
Push to DockerHub:  1-2 minutes
Push to ECR:        1-2 minutes
Security Scan:      2-3 minutes
Total (parallel):   3-5 minutes
```

**Pull Request Checks:**
```
Dockerfile Lint:    30 seconds
Compose Validate:   20 seconds
Code Quality:       40 seconds
Image Size Check:   2-3 minutes
Total (parallel):   3 minutes
```

---

## Cost Considerations

### GitHub Actions Minutes

**Free tier:**
- 2,000 minutes/month for public repos
- Unlimited for public repos (as of 2024)

**This project usage:**
- ~5 minutes per push
- ~400 builds possible/month (if private)

### AWS ECR Costs

- Storage: $0.10/GB/month
- Current image: ~180MB = ~$0.018/month
- Essentially free for single project

---

## Future Enhancements

### Planned Additions

- [ ] Automated testing with Jest/Mocha
- [ ] Code coverage reporting
- [ ] Deployment to ECS/EKS
- [ ] Slack/Discord notifications
- [ ] Performance benchmarking
- [ ] Multi-architecture builds (ARM64)
- [ ] Integration testing
- [ ] Database migrations

### Advanced Features to Add

**Deployment Automation:**
```yaml
- name: Deploy to ECS
  run: |
    aws ecs update-service \
      --cluster todo-app-cluster \
      --service todo-app-service \
      --force-new-deployment
```

**Notifications:**
```yaml
- name: Notify on Slack
  uses: 8398a7/action-slack@v3
  with:
    status: ${{ job.status }}
    webhook_url: ${{ secrets.SLACK_WEBHOOK }}
```

**Multi-arch builds:**
```yaml
- name: Set up QEMU
  uses: docker/setup-qemu-action@v3

- name: Build multi-platform
  uses: docker/build-push-action@v5
  with:
    platforms: linux/amd64,linux/arm64
```

---

## Best Practices Implemented

### CI/CD Principles

✅ **Automated Testing** - Every build runs tests  
✅ **Fail Fast** - Quick feedback on issues  
✅ **Security First** - Vulnerability scanning  
✅ **Parallel Execution** - Faster pipelines  
✅ **Caching** - Optimized build times  
✅ **Semantic Versioning** - Clear release tracking  
✅ **Status Badges** - Visible build health  
✅ **Documentation** - Clear workflow docs  

### Docker Best Practices

✅ **Multi-stage builds** - Smaller images  
✅ **Layer caching** - Faster builds  
✅ **Security scanning** - Vulnerability detection  
✅ **Multi-registry** - DockerHub + ECR  
✅ **Semantic tagging** - Version management  

---

## Resources

- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [Docker Build Push Action](https://github.com/docker/build-push-action)
- [AWS ECR Action](https://github.com/aws-actions/amazon-ecr-login)
- [Trivy Scanner](https://github.com/aquasecurity/trivy)

---

**Last Updated:** January 2026  
**Maintained by:** INNOCENTIA1-git