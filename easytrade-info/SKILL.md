---
name: EasyTrade Repository Guide
description: Comprehensive guide to the EasyTrade microservices application, including repository structure, service organization by technology, and vulnerability remediation processes.
---

# EasyTrade Repository Guide

## Project Overview

EasyTrade is a microservices-based demo application that simulates a stock brokerage platform. Users can buy and sell stocks with fake data, and prices follow a 24-hour cycle. The application is designed to showcase observability, microservices architecture, and integration with Dynatrace monitoring.

**Repository:** Dynatrace/easytrade  
**License:** Apache-2.0  
**Primary Use Case:** Demo application for monitoring, observability, and microservices patterns

---

## Repository Structure

The workspace root contains 19 services organized in the `src/` directory, grouped by technology stack and programming language.

### Service Organization by Technology

#### Java 21 / Spring Boot / Gradle (6 services)
- `accountservice`
- `contentcreator`
- `credit-card-order-service`
- `engine`
- `feature-flag-service`
- `third-party-service`

**Build Command:** `./gradlew build`

#### Go / Go Modules (3 services)
- `aggregator-service`
- `pricing-service`
- `problem-operator`

**Build Command:** `go build .`

#### TypeScript / Node.js / npm (3 services)
- `frontend`
- `loadgen`
- `offerservice`

**Build Command:** `npm run build`

#### C# / .NET 8 / NuGet (3 services)
- `broker-service`
- `loginservice`
- `manager`

**Build Command:** `dotnet build`

#### Python / Poetry (1 utility)
- `db/user-generator` (local utility script, not a deployed service)

#### Config Only — No Package Manifest (4 components)
- `calculationservice` (C++, built in Dockerfile)
- `frontendreverseproxy` (nginx)
- `rabbitmq` (message broker)
- `db` (MSSQL database)

---

## Service Endpoints

| Service | Port | Proxy Endpoint | Notes |
|---------|------|----------------|-------|
| Frontend | 80 | / | Main web application |
| Account Service | 80 | /accountservice | User account management |
| Aggregator Service | 80 | — | Internal service |
| Broker Service | 80 | /broker-service | Brokerage operations |
| Offer Service | 80 | /offerservice | Stock offers |
| Pricing Service | 80 | /pricing-service | Price calculations |
| Feature Flag Service | 80 | /feature-flag-service | Feature toggles |
| Credit Card Order Service | 80 | /credit-card-order-service | Payment processing |
| Login Service | 80 | /loginservice | Authentication |
| Engine | 80 | /engine | Core business logic |
| Third Party Service | 80 | /third-party-service | External integrations |

---

## Getting Started

### Docker Compose (Recommended for Local Development)

**Requirements:**
- Docker v20.10.13 or higher
- Docker Compose Plugin

```bash
# Run in foreground
docker compose up

# Run in background
docker compose up -d

# Access the application
# Visit: http://localhost or http://127.0.0.1
```

**Note:** Application may take several minutes to stabilize. Initial errors or missing data are normal.

### Kubernetes Deployment

**Requirements:**
- `helm`
- `kubectl`
- Configured `kubeconfig`

```bash
# Create namespace and deploy
helm install easytrade oci://europe-docker.pkg.dev/dynatrace-demoability/helm/easytrade \
  --create-namespace --namespace easytrade

# Uninstall
helm uninstall easytrade -n easytrade

# Delete namespace
kubectl delete namespace easytrade
```

### Default User Credentials

- **Superuser 1:** demouser / demopass
- **Superuser 2:** specialuser / specialpass

---

## Vulnerability Remediation Process

### Step 1: Scanning

Run `snyk test` on each service with a package manifest:

```bash
# Scan individual service
snyk test --json --all-projects

# Run from service directory or repository root
```

**Services that cannot be scanned (no manifest):**
- `calculationservice` (C++)
- `frontendreverseproxy` (nginx)
- `rabbitmq`

### Step 2: Group Findings by Technology

Services using the same technology stack often have identical vulnerable packages at identical versions. Group findings before applying fixes to ensure consistent patches across all affected services.

### Step 3: Apply Fixes by Technology Stack

#### Java / Gradle Services

Vulnerable dependencies that are not direct dependencies are pinned in `build.gradle` under a marked comment block:

```gradle
// -- not direct dependencies but need bumps to patch vulns
// -- can be removed once the parent packages upgrade
dependency_name:X.X.X
```

**Action:** Update version pins in the marked block across **all** affected `build.gradle` files in parallel.

**Affected services:** accountservice, contentcreator, credit-card-order-service, engine, feature-flag-service, third-party-service

#### Node.js / npm Services

1. Update the vulnerable package version constraint in `dependencies` in `package.json`
2. Pin transitive dependencies using the `overrides` field in `package.json`
3. Regenerate `package-lock.json`:
   ```bash
   npm install
   ```

**Affected services:** frontend, offerservice

#### Go Services

Go stdlib vulnerabilities are fixed by upgrading the Go toolchain version, not individual module dependencies. **Three files must be updated in sync:**

1. **`go.mod`** — bump the `go` directive
2. **`Dockerfile`** — bump both the image tag and the pinned digest
3. **`go.sum`** — regenerated automatically via `go mod tidy`

**To get the correct digest:**

```bash
docker pull golang:<new-version>-alpine3.23
docker inspect --format='{{index .RepoDigests 0}}' golang:<new-version>-alpine3.23
```

Both Go service Dockerfiles use the same base image, so one pull suffices for both.

**Affected services:** pricing-service, problem-operator

#### C# / .NET Services

Update vulnerable NuGet packages in the `.csproj` file or use the .NET CLI:

```bash
dotnet add package <PackageName> --version <NewVersion>
```

**Affected services:** broker-service, loginservice, manager

### Step 4: Verify Fixes

For each updated service, run the local build:

```bash
# Java/Gradle
cd <service-directory>
./gradlew build

# Node.js/npm
npm run build

# Go
go build .

# C#/.NET
dotnet build
```

Then re-scan from the repository root:

```bash
snyk test --json --all-projects
```

**Success Criteria:** All projects exit with status `0` and zero vulnerabilities before committing.

---

## Problem Patterns

EasyTrade supports 4 built-in problem patterns for testing and demonstration:

### DbNotResponding
- Effect: No new trades can be created; database throws errors
- Duration: ~20 minutes for problem detection in Dynatrace
- Use Case: Test database failure handling

### ErgoAggregatorSlowdown
- Effect: 2 aggregators receive slower responses, eventually stop sending requests
- Response Scenarios:
  - 15 min run: 150s slowdown, then 40% lower traffic for 15 min
  - 20 min run: 150s slowdown, then 40% lower traffic for 30 min
- Use Case: Test service degradation and recovery

### FactoryCrisis
- Effect: Factory stops producing cards; Third Party Service cannot process credit card orders
- Blocked Component: Credit Card Order Service
- Use Case: Test cascading failures

### HighCpuUsage
- Effect: Broker Service response time increases; CPU usage spikes
- Kubernetes Effect: CPU resource limit applied by Problem Operator, triggers throttling
- Use Case: Test resource constraint handling and CPU limiting

### Enable/Disable Problem Patterns

**Via REST API:**

```bash
curl -X PUT "http://{IP_ADDRESS}/feature-flag-service/v1/flags/{FEATURE_ID}/" \
  -H "accept: application/json" \
  -d '{"enabled": true}'
```

**Via Frontend:** Problem patterns can be toggled in the EasyTrade frontend UI

**Via Kubernetes CronJobs:** Automated problem pattern execution is available in the helm directory

---

## Data Types & Content Negotiation

EasyTrade network traffic uses REST requests with JSON as the default payload format. Some services support XML via content negotiation based on `Accept` and `Content-Type` headers.

### XML-Compatible Services

| Service | Accepted XML MIME Types |
|---------|-------------------------|
| LoginService | `application/xml`, `text/xml`, `application/*+xml` |
| CreditCardOrderService | `application/xml` |
| OfferService | `application/xml`, `text/xml` |
| PricingService | `application/xml` |

---

## Dynatrace Integration

### Business Events

EasyTrade is instrumented to showcase business events in two ways:

1. **Direct:** Using Dynatrace SDKs in code (JavaScript, Java, etc.)
2. **Indirect:** Capture rules configured in Dynatrace

**Configuration:** All Dynatrace configuration is managed via Monaco. See the `monaco/` directory for exported configurations and the README within.

### Local Dynatrace MCP Server

This repository includes a pre-configured Dynatrace MCP Server for VS Code:

**Requirements:**
- Access to Dynatrace Playground Environment
- GitHub Copilot access
- VS Code with MCP Server support

**Usage:**
1. Open Copilot Chat in VS Code
2. Switch to Agent mode
3. Example queries:
   - "Which environment am I connected to?"
   - "Are there any problems with my components on Dynatrace?"
   - "Are there any security vulnerabilities for my component?"

---

## Development Quick Reference

### Clone & Setup
```bash
git clone https://github.com/Dynatrace/easytrade.git
cd easytrade
```

### Run Locally
```bash
docker compose up -d
# Access: http://localhost
```

### Run Tests & Builds
```bash
# Java services
cd accountservice && ./gradlew test

# Node services
cd frontend && npm test

# Go services
cd pricing-service && go test ./...

# C# services
cd broker-service && dotnet test
```

### Scan for Vulnerabilities
```bash
snyk test --json --all-projects
```

### Check Service Swagger Docs
Visit individual service README files for Swagger endpoint URLs and API documentation.

---

## Repository Language Composition

- **TypeScript:** 40.1%
- **C#:** 20.1%
- **Java:** 18.6%
- **Go:** 7.2%
- **Python:** 5.0%
- **T-SQL:** 4.9%
- **Other:** 4.1%

---

## Contributing & License

- **Code of Conduct:** See CODE_OF_CONDUCT.md
- **Contributing Guide:** See CONTRIBUTING.md
- **Security Policy:** See SECURITY.md
- **License:** Apache-2.0

---

## References

- **GitHub Repository:** https://github.com/Dynatrace/easytrade
- **Dynatrace Docs:** https://docs.dynatrace.com
- **Business Events:** https://docs.dynatrace.com/docs/platform-modules/core-platform/business-analytics
- **Monaco:** https://github.com/Dynatrace/monaco
- **OpenKit:** https://github.com/Dynatrace/openkit-java
