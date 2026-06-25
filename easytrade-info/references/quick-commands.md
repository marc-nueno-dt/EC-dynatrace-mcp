# EasyTrade Quick Command Reference

## Local Development

```bash
# Clone repository
git clone https://github.com/Dynatrace/easytrade.git && cd easytrade

# Start all services with Docker Compose
docker compose up -d

# View logs
docker compose logs -f

# Stop services
docker compose down
```

## Building Services

```bash
# Build all services
docker compose build

# Build specific service
docker compose build accountservice

# Java service build
cd accountservice && ./gradlew build

# Node service build
cd frontend && npm install && npm run build

# Go service build
cd pricing-service && go mod tidy && go build .

# .NET service build
cd broker-service && dotnet build
```

## Vulnerability Scanning & Remediation

```bash
# Initial scan (from repo root)
snyk test --json --all-projects > vulnerabilities.json

# Scan specific service
cd accountservice && snyk test --json

# After fixing, verify all services
snyk test --json --all-projects

# Expected result: exit code 0, zero vulnerabilities
```

## Feature Flags & Problem Patterns

```bash
# Enable problem pattern (DbNotResponding example)
curl -X PUT "http://localhost/feature-flag-service/v1/flags/DbNotResponding" \
  -H "accept: application/json" \
  -d '{"enabled": true}'

# Disable problem pattern
curl -X PUT "http://localhost/feature-flag-service/v1/flags/DbNotResponding" \
  -H "accept: application/json" \
  -d '{"enabled": false}'

# List all feature flags
curl -X GET "http://localhost/feature-flag-service/v1/flags" \
  -H "accept: application/json"
```

## Authentication

```bash
# Login (default superuser)
curl -X POST "http://localhost/loginservice/v1/login" \
  -H "Content-Type: application/json" \
  -d '{"username": "demouser", "password": "demopass"}'

# Signup (new user)
curl -X POST "http://localhost/loginservice/v1/signup" \
  -H "Content-Type: application/json" \
  -d '{"username": "newuser", "password": "newpass"}'
```

## Kubernetes Deployment

```bash
# Deploy to Kubernetes
helm install easytrade oci://europe-docker.pkg.dev/dynatrace-demoability/helm/easytrade \
  --create-namespace --namespace easytrade

# Check deployment status
kubectl get pods -n easytrade

# Port forward to local
kubectl port-forward -n easytrade svc/easytrade 8080:80

# View logs
kubectl logs -n easytrade -l app=easytrade

# Uninstall
helm uninstall easytrade -n easytrade

# Delete namespace
kubectl delete namespace easytrade
```

## Testing & Validation

```bash
# Test Java service
cd accountservice && ./gradlew test

# Test Node service
cd frontend && npm test

# Test Go service
cd pricing-service && go test ./...

# Test .NET service
cd broker-service && dotnet test

# Health check
curl -i http://localhost/health || echo "Services still starting..."
```

## Common Endpoints

```bash
# Frontend
http://localhost/

# Account Service Swagger
http://localhost/accountservice/swagger-ui.html

# Feature Flag Service
http://localhost/feature-flag-service/v1/flags

# Broker Service Swagger  
http://localhost/broker-service/swagger-ui.html

# Pricing Service
http://localhost/pricing-service/

# Offer Service
http://localhost/offerservice/

# Credit Card Order Service
http://localhost/credit-card-order-service/

# Engine
http://localhost/engine/
```

## Dynatrace Integration

```bash
# Query problems via API (requires environment access)
curl -X GET "https://{environment-id}.live.dynatrace.com/api/v2/problems" \
  -H "Authorization: Api-Token {api-token}"

# Get entity ID
curl -X GET "https://{environment-id}.live.dynatrace.com/api/v2/entities" \
  -H "Authorization: Api-Token {api-token}"
```

## Cleanup & Reset

```bash
# Remove all containers and volumes
docker compose down -v

# Prune all Docker resources
docker system prune -a --volumes

# Reset service state
rm -rf ~/easytrade-data

# Clean build artifacts
find . -name "target" -type d -exec rm -rf {} +
find . -name "node_modules" -type d -exec rm -rf {} +
```
