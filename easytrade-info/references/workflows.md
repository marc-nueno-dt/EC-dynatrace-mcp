# EasyTrade Development Workflows

## Workflow 1: Setting Up Local Development Environment

### Prerequisites
- Docker v20.10.13+ with Compose plugin
- Git
- One of: Java 21+, Go 1.20+, Node.js 18+, .NET 8, C++ toolchain (depending on services you'll modify)

### Setup Steps

```bash
# 1. Clone the repository
git clone https://github.com/Dynatrace/easytrade.git
cd easytrade

# 2. Start all services
docker compose up -d

# 3. Wait for services to stabilize (2-3 minutes)
sleep 30 && curl -s http://localhost/ | head -20

# 4. Access the frontend
# Open browser to: http://localhost

# 5. Login with default credentials
# Username: demouser
# Password: demopass
```

### Verification
- [ ] All containers running: `docker compose ps`
- [ ] Frontend loads at http://localhost
- [ ] Login successful with demouser/demopass
- [ ] Can navigate to Deposit page (indicates backend connectivity)

---

## Workflow 2: Modifying a Java Service

### Example: Updating accountservice

```bash
# 1. Navigate to service
cd accountservice

# 2. Make code changes
# Edit src/main/java/com/example/Account.java

# 3. Build locally
./gradlew build

# 4. If successful, rebuild Docker image
cd ..
docker compose build accountservice

# 5. Restart the service
docker compose up -d accountservice

# 6. Verify logs
docker compose logs -f accountservice

# 7. Test endpoint
curl -X GET http://localhost/accountservice/api/accounts
```

### Troubleshooting
- Compilation errors: Run `./gradlew clean build`
- Container won't start: Check logs with `docker compose logs accountservice`
- Port conflicts: Ensure port 80 is not in use

---

## Workflow 3: Scanning for Vulnerabilities

### Complete Vulnerability Scan

```bash
# 1. Install Snyk CLI (if not already installed)
npm install -g snyk

# 2. Authenticate with Snyk
snyk auth

# 3. Run full repository scan
snyk test --json --all-projects > vulnerabilities.json

# 4. Review vulnerabilities by service
jq '.[] | {name: .packageManager, vulnerabilities: .vulnerabilities | length}' vulnerabilities.json

# 5. Group by technology for consistent fixes
# (Identify which services share vulnerable packages)

# 6. Apply fixes (see SKILL.md for technology-specific steps)

# 7. Verify fixes
snyk test --json --all-projects

# 8. Confirm exit code 0 before committing
echo $?  # Should print: 0
```

### By Technology Stack

**Java Services:**
```bash
cd accountservice && ./gradlew build && cd ..
# Repeat for all 6 Java services
snyk test --json --all-projects
```

**Node.js Services:**
```bash
cd frontend && npm install && npm run build && cd ..
cd offerservice && npm install && npm run build && cd ..
snyk test --json --all-projects
```

**Go Services:**
```bash
cd pricing-service && go mod tidy && go build . && cd ..
cd problem-operator && go mod tidy && go build . && cd ..
snyk test --json --all-projects
```

---

## Workflow 4: Testing Problem Patterns

### Enable a Problem Pattern

```bash
# 1. Start EasyTrade if not running
docker compose up -d

# 2. List available problem patterns
curl -s http://localhost/feature-flag-service/v1/flags | jq '.[] | .flagId'

# 3. Enable DbNotResponding pattern
curl -X PUT "http://localhost/feature-flag-service/v1/flags/DbNotResponding" \
  -H "accept: application/json" \
  -d '{"enabled": true}'

# 4. Monitor the effect
# Try to execute trades via frontend - should fail with DB error
# Observe logs: docker compose logs -f engine

# 5. Check Dynatrace for problem detection (~20 min)
# Navigate to: https://your-dynatrace-env.com

# 6. Disable the pattern
curl -X PUT "http://localhost/feature-flag-service/v1/flags/DbNotResponding" \
  -H "accept: application/json" \
  -d '{"enabled": false}'

# 7. Verify system recovers
# Trades should work again
```

### Testing Aggregator Slowdown

```bash
# Enable ErgoAggregatorSlowdown
curl -X PUT "http://localhost/feature-flag-service/v1/flags/ErgoAggregatorSlowdown" \
  -H "accept: application/json" \
  -d '{"enabled": true}'

# Monitor for:
# - Initial 150s slowdown in aggregator responses
# - Following 40% traffic reduction for 15-30 min
# - Service degradation in Dynatrace

# Watch pricing service logs
docker compose logs -f pricing-service

# After 30 minutes, disable
curl -X PUT "http://localhost/feature-flag-service/v1/flags/ErgoAggregatorSlowdown" \
  -H "accept: application/json" \
  -d '{"enabled": false}'
```

---

## Workflow 5: Deploying to Kubernetes

### Prerequisites
- `kubectl` configured with cluster access
- `helm` CLI installed
- Access to Dynatrace Playground environment (optional)

### Deployment Steps

```bash
# 1. Create namespace and install
helm install easytrade \
  oci://europe-docker.pkg.dev/dynatrace-demoability/helm/easytrade \
  --create-namespace \
  --namespace easytrade

# 2. Monitor deployment
kubectl get pods -n easytrade -w

# 3. Wait for all pods ready
kubectl wait --for=condition=ready pod \
  -l app=easytrade \
  -n easytrade \
  --timeout=300s

# 4. Expose the service (port-forward or ingress)
kubectl port-forward -n easytrade svc/easytrade 8080:80

# 5. Access application
# Open: http://localhost:8080

# 6. View logs from a pod
kubectl logs -n easytrade -l app=engine -f

# 7. Check resource usage
kubectl top pods -n easytrade

# 8. When done, uninstall
helm uninstall easytrade -n easytrade
kubectl delete namespace easytrade
```

### Troubleshooting Kubernetes Deployment
```bash
# Describe failing pod
kubectl describe pod <pod-name> -n easytrade

# Check events
kubectl get events -n easytrade --sort-by='.lastTimestamp'

# Stream logs from all pods
kubectl logs -n easytrade -l app=easytrade -f --all-containers

# Check resource requests vs available
kubectl describe nodes

# Restart a service
kubectl rollout restart deployment/<service-name> -n easytrade
```

---

## Workflow 6: Contributing a Fix or Feature

### Step-by-Step Process

```bash
# 1. Fork and clone
git clone https://github.com/YOUR-USERNAME/easytrade.git
cd easytrade

# 2. Create feature branch
git checkout -b feature/my-feature
# or for fixes
git checkout -b fix/bug-description

# 3. Make changes (example: Java service)
cd accountservice
# Edit src/main/java/...

# 4. Test locally
./gradlew test
./gradlew build

# 5. Commit with clear message
cd ..
git add accountservice/
git commit -m "feat: add new endpoint to account service"

# 6. Scan for vulnerabilities before pushing
snyk test --json --all-projects

# 7. Push to your fork
git push origin feature/my-feature

# 8. Create Pull Request on GitHub
# Include description, testing notes, and related issues

# 9. Address review feedback
git add .
git commit -m "address review feedback"
git push origin feature/my-feature
```

### PR Checklist
- [ ] All tests passing: `./gradlew test` (Java) or equivalent
- [ ] Code builds: `./gradlew build` (Java) or equivalent
- [ ] No vulnerabilities: `snyk test`
- [ ] Follows existing code style
- [ ] Includes tests for new functionality
- [ ] Updates relevant README files
- [ ] Documented any breaking changes
- [ ] Related issue referenced in PR description

---

## Workflow 7: Monitoring with Dynatrace

### Prerequisites
- Dynatrace environment access
- OneAgent or similar monitoring deployed
- MCP Server configured (optional, for VS Code integration)

### Monitoring Steps

```bash
# 1. Ensure services are running
docker compose up -d

# 2. Access Dynatrace environment
# Navigate to: https://your-environment.live.dynatrace.com

# 3. Check monitored entities
# Services → All services → Filter by "easytrade"

# 4. Create custom dashboard for EasyTrade
# Add tiles for:
# - Service response times
# - Apdex scores
# - Database queries
# - Problem count

# 5. Set up alerting for key metrics
# Example: Alert if broker-service response time > 1s

# 6. Configure business events
# Capture: Buy/Sell transactions, login attempts
# Via: capture rules in transaction settings

# 7. Monitor problem patterns
# Watch for problem detection when patterns enabled
```

### Common Dynatrace Queries (DQL)

```sql
-- Find all requests to EasyTrade services
fetch http_requests | filter origin CONTAINS "easytrade" | stats count()

-- Identify slowest services
fetch metrics | filter name == "service.response.time" 
  and tag("dt.entity.service") CONTAINS "easytrade"
  | stats avg(value) by tag("dt.entity.service")

-- Check error rates
fetch logs | filter service.name CONTAINS "easytrade" AND severity == "ERROR"
  | stats count() by service.name
```

---

## Workflow 8: Clean Up & Reset

### Remove All Data

```bash
# 1. Stop containers
docker compose down

# 2. Remove volumes (persisted data)
docker compose down -v

# 3. Remove images (optional, to force rebuild)
docker rmi $(docker images "*easytrade*" -q)

# 4. Remove local caches
rm -rf ~/.gradle/caches
rm -rf frontend/node_modules
rm -rf offerservice/node_modules
find . -name "target" -type d -exec rm -rf {} +

# 5. Start fresh
docker compose up -d

# 6. Verify clean state
docker compose ps
```

### Partial Clean Up

```bash
# Remove only one service's data
docker compose down accountservice -v

# Remove only build artifacts (no container cleanup)
./accountservice/gradlew clean
cd frontend && npm cache clean --force

# Prune Docker system (unused images, networks, volumes)
docker system prune -a --volumes
```
