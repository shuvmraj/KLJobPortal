# KL Job Portal

A modern full-stack job portal application built with Java Spring Boot backend and React frontend, designed for seamless CI/CD deployment to Azure.

## 📋 Project Overview

KL Job Portal is a monorepo containing:
- **Backend**: Spring Boot REST API (Java 21)
- **Frontend**: React 18 + Vite SPA (Node.js 20)

Both services are containerized and deployed to Azure App Service with automated CI/CD pipelines.

## 🏗️ Architecture

```
KLJobPortal (Monorepo)
├── Backend/
│   └── JobPortal/          # Spring Boot application
│       ├── src/
│       │   ├── main/       # Application code
│       │   └── test/       # Unit tests
│       ├── pom.xml         # Maven dependencies
│       └── Dockerfile      # Container build
│
├── Frontend/
│   └── vite-project/       # React application
│       ├── src/            # React components
│       ├── package.json    # npm dependencies
│       └── Dockerfile      # Container build
│
├── azure-pipelines-infra.yml    # Infrastructure provisioning
├── azure-pipelines.yml          # CI/CD deployment pipeline
└── .azure/
    └── pipeline-setup.md        # Azure setup guide
```

## 🚀 Quick Start

### Prerequisites

- **Local Development**:
  - Java 21+ (for backend)
  - Node.js 20+ (for frontend)
  - Maven 3.9+ (for building backend)

- **Azure Deployment**:
  - Azure subscription
  - Azure DevOps organization
  - Service Principal with appropriate RBAC roles

### Local Development

#### Backend Setup

```bash
cd Backend/JobPortal

# Build the application
mvn clean package

# Run the application
mvn spring-boot:run
```

Backend runs on `http://localhost:8080`

#### Frontend Setup

```bash
cd Frontend/vite-project

# Install dependencies
npm install

# Start development server
npm run dev
```

Frontend runs on `http://localhost:5173` (Vite default)

### Environment Variables

#### Backend (application.properties)

```properties
server.port=8080
spring.profiles.active=dev
server.servlet.context-path=/api
```

#### Frontend (.env)

```env
VITE_API_URL=http://localhost:8080/api
VITE_APP_NAME=KL Job Portal
```

## 🐳 Docker & Containerization

Both services are containerized using multi-stage builds for optimal performance.

### Build Docker Images

```bash
# Build backend image
docker build -t jobportal-api:latest Backend/JobPortal/

# Build frontend image
docker build -t jobportal-web:latest Frontend/vite-project/

# Run containers
docker run -p 8080:8080 jobportal-api:latest
docker run -p 3000:3000 jobportal-web:latest
```

### Docker Compose (Optional)

To run both services together:

```bash
docker-compose up -d
```

## 🔄 CI/CD Pipeline

### Pipeline Stages

1. **CI (Continuous Integration)**
   - Builds backend with Maven
   - Builds frontend with npm
   - Runs unit tests (if configured)
   - Caches dependencies for faster builds

2. **CD (Continuous Deployment)**
   - **Dev**: Auto-deploys on every commit to `main` or `develop`
   - **Staging**: Auto-deploys commits to `main` (with approval)
   - **Production**: Manual approval required

### Triggers

| Branch | Trigger | Stages |
|--------|---------|--------|
| `main` | Push | CI → Dev → Staging → Prod* |
| `develop` | Push | CI → Dev |
| `*` | Pull Request | CI only (no deployment) |

\* Production requires manual approval

### Pipeline Files

- **[azure-pipelines-infra.yml](azure-pipelines-infra.yml)**: Provisions Azure resources (Container Registry, App Service)
- **[azure-pipelines.yml](azure-pipelines.yml)**: Builds, tests, and deploys applications

## 📦 Deployment to Azure

### Step 1: Prepare Azure Subscription

```bash
# Login to Azure
az login

# Create resource group
az group create --name klrg --location eastus
```

### Step 2: Set Up Azure DevOps

1. Create project at https://dev.azure.com
2. Create service connection with Service Principal
3. Push code to Azure Repos

### Step 3: Create Service Principal

```bash
SUBSCRIPTION_ID="your-subscription-id"
SERVICE_PRINCIPAL_NAME="KLJobPortal-CI-CD"
RESOURCE_GROUP="klrg"

az ad sp create-for-rbac \
  --name $SERVICE_PRINCIPAL_NAME \
  --role contributor \
  --scopes /subscriptions/$SUBSCRIPTION_ID/resourceGroups/$RESOURCE_GROUP
```

Save the output for Azure DevOps Service Connection configuration.

### Step 4: Configure Pipeline Variables

In Azure DevOps → Pipelines → Edit → Variables:

```
REGISTRY_NAME: acrklXXXXX (your ACR name)
REGISTRY_USERNAME: username from ACR
REGISTRY_PASSWORD: password from ACR (mark as secret)
RESOURCE_GROUP: klrg
RESOURCE_TOKEN: klXXXX (5-char lowercase token)
```

### Step 5: Run Pipelines

1. **Infrastructure Pipeline** (one-time):
   ```
   az devops admin banner list (verify in ADO)
   ```
   Creates: Container Registry, App Service Plan, Web Apps

2. **CI/CD Pipeline** (on every push):
   - Commits to `develop` → deploys to Dev
   - Commits to `main` → deploys to Dev, then Staging (with approval)

### Access Deployed Applications

After successful deployment:

- **Backend API**: `https://jobportal-api-klXXXX.azurewebsites.net`
- **Frontend**: `https://jobportal-web-klXXXX.azurewebsites.net`

## 📊 Monitoring & Logs

### Azure Portal

Monitor deployed applications:
1. Go to Azure Portal → Resource Groups → `klrg`
2. Check App Service metrics, logs, and health
3. View Container Registry images and builds

### Azure DevOps

Monitor pipeline runs:
1. Go to Pipelines → View pipeline runs
2. Check job logs for build and deployment details
3. View deployment history per environment

## 🔧 Configuration

### Backend Configuration

#### application.properties (Development)
```properties
server.port=8080
spring.profiles.active=dev
```

#### application-prod.properties (Production)
```properties
server.port=8080
spring.profiles.active=prod
```

### Frontend Configuration

Create `.env` files for each environment:

```env
# .env.development
VITE_API_URL=http://localhost:8080/api

# .env.production
VITE_API_URL=https://jobportal-api-klXXXX.azurewebsites.net/api
```

## 🛠️ Troubleshooting

### Build Failures

**Maven Build Fails**:
```bash
# Clear Maven cache
rm -rf ~/.m2/repository

# Rebuild
mvn clean package
```

**npm Build Fails**:
```bash
# Clear npm cache
npm cache clean --force

# Reinstall dependencies
npm install
```

### Pipeline Issues

| Issue | Solution |
|-------|----------|
| Service Connection fails | Verify Service Principal credentials in Azure AD |
| ACR push fails | Check Container Registry credentials and RBAC roles |
| App deployment timeout | Increase deployment timeout in pipeline settings |
| Health check fails | Verify app is responding on correct port and health endpoint |

### Access Issues

**Cannot access deployed app**:
1. Check App Service status in Azure Portal
2. Verify firewall/network rules
3. Check application logs in App Service → Log stream
4. Verify environment variables and connection strings

## 📝 Development Workflow

### Local Development

```bash
# 1. Create feature branch
git checkout -b feature/new-feature

# 2. Make changes and test locally
cd Backend/JobPortal && mvn clean package
cd Frontend/vite-project && npm run build

# 3. Commit and push
git add .
git commit -m "Add new feature"
git push origin feature/new-feature

# 4. Create Pull Request
# Pipeline runs CI (no deployment)

# 5. Merge to develop/main
# Pipeline runs CI → Deploy to Dev
```

### Release Workflow

```bash
# Merge develop → main
# Pipeline: CI → Deploy Dev → Deploy Staging → Deploy Prod (approval)
```

## 📚 Additional Resources

- [Azure DevOps Documentation](https://docs.microsoft.com/azure/devops)
- [Spring Boot Documentation](https://spring.io/projects/spring-boot)
- [React Documentation](https://react.dev)
- [Vite Documentation](https://vitejs.dev)
- [Docker Documentation](https://docs.docker.com)

## 📄 License

This project is licensed under the MIT License.

## 👥 Support

For issues or questions:
1. Check [.azure/pipeline-setup.md](.azure/pipeline-setup.md) for setup help
2. Review Azure DevOps pipeline logs
3. Check Azure Application Insights for runtime errors
4. Open an issue in the repository

---

**Last Updated**: January 2026
**Version**: 1.0.0
