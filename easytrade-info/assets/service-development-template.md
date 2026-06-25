# EasyTrade Service Development Template

Use this template when creating a new service or modifying an existing EasyTrade service.

## Service Overview

**Service Name:** [Service Name]  
**Language:** [Java/Go/TypeScript/C#/Other]  
**Framework:** [Spring Boot/Go Modules/Node.js/ASP.NET Core/Other]  
**Port:** 80  
**Proxy Endpoint:** /[service-name]  
**Main Responsibility:** [Describe what this service does]  

## File Structure

```
service-name/
├── build.gradle          # Java/Gradle services
├── package.json          # Node.js services
├── go.mod               # Go services
├── *.csproj             # .NET services
├── Dockerfile           # Container definition
├── README.md            # Service documentation
├── src/
│   ├── main/            # Production code
│   └── test/            # Test code
└── [language-specific folders]
```

## Implementation Checklist

### Setup
- [ ] Language toolchain installed locally
- [ ] Dependencies/packages installed
- [ ] Build succeeds: `./gradlew build` / `npm run build` / `go build .` / `dotnet build`
- [ ] Tests pass locally

### Code
- [ ] Main service logic implemented
- [ ] API endpoints documented (Swagger/OpenAPI)
- [ ] Error handling implemented
- [ ] Logging configured
- [ ] Configuration externalized (environment variables, config files)

### Security & Quality
- [ ] No hardcoded secrets
- [ ] Input validation on all endpoints
- [ ] SQL injection protection (if database access)
- [ ] CORS configured if needed
- [ ] No known vulnerabilities: `snyk test`

### Docker & Deployment
- [ ] Dockerfile created and tested
- [ ] Docker image builds: `docker build -t [service-name] .`
- [ ] Container runs successfully
- [ ] Health check endpoint implemented
- [ ] Environment variables documented

### Integration
- [ ] Service registered in `docker-compose.yaml`
- [ ] Routing configured in nginx/proxy
- [ ] Database migrations (if applicable)
- [ ] Service-to-service communication working
- [ ] RabbitMQ integration (if async required)

### Documentation
- [ ] README.md completed with:
  - Service description
  - API endpoints / Swagger URL
  - Configuration options
  - Local development instructions
  - Deployment notes
- [ ] Code commented where necessary
- [ ] Team informed of changes

### Testing
- [ ] Unit tests written and passing
- [ ] Integration tests (if applicable)
- [ ] Manual testing in Docker Compose
- [ ] Load testing (with loadgen service)
- [ ] Problem pattern compatibility tested

---

## Build & Run Commands

### [Language] Service Build & Run

Replace `[service-name]` and `[language]` markers below.

#### Java (Gradle)
```bash
cd [service-name]
./gradlew clean build
./gradlew bootRun                    # Local development
docker build -t [service-name] .
docker compose up -d [service-name]
```

#### Node.js
```bash
cd [service-name]
npm install
npm run build                        # If applicable
npm start                           # Local development
npm test
docker build -t [service-name] .
docker compose up -d [service-name]
```

#### Go
```bash
cd [service-name]
go mod tidy
go build .
./[service-name]                    # Local development
go test ./...
docker build -t [service-name] .
docker compose up -d [service-name]
```

#### C# (.NET)
```bash
cd [service-name]
dotnet restore
dotnet build
dotnet run                          # Local development
dotnet test
docker build -t [service-name] .
docker compose up -d [service-name]
```

---

## Vulnerability Remediation Template

### Scanning
```bash
# Individual service scan
cd [service-name]
snyk test --json > vulnerabilities.json

# All services scan (from repo root)
snyk test --json --all-projects
```

### Technology-Specific Fix Steps

#### If Java Service (build.gradle)
1. [ ] Open `build.gradle`
2. [ ] Locate the comment block:
   ```gradle
   // -- not direct dependencies but need bumps to patch vulns
   // -- can be removed once the parent packages upgrade
   ```
3. [ ] Update vulnerable dependency versions
4. [ ] Test: `./gradlew clean build`
5. [ ] Re-scan: `snyk test --json`

#### If Node.js Service (package.json)
1. [ ] Open `package.json`
2. [ ] Update vulnerable package in `dependencies`
3. [ ] Add/update transitive deps in `overrides` field
4. [ ] Regenerate lock file: `npm install`
5. [ ] Test: `npm run build`
6. [ ] Re-scan: `snyk test --json`

#### If Go Service
1. [ ] Open `go.mod`
2. [ ] Bump `go` directive version
3. [ ] Update `Dockerfile` image tag and digest
4. [ ] Regenerate: `go mod tidy`
5. [ ] Test: `go build .`
6. [ ] Re-scan: `snyk test --json`

#### If C# Service
1. [ ] Open `.csproj` file
2. [ ] Update vulnerable NuGet package version
3. [ ] Test: `dotnet build`
4. [ ] Re-scan: `snyk test --json`

### Verification
```bash
# Test build
[language-specific build command]

# Scan again
snyk test --json

# Verify exit code 0, zero vulnerabilities
echo $?  # Should output: 0
```

---

## API Endpoint Template

### REST Endpoint Definition

```
Method:     POST
Endpoint:   /[service-name]/v1/[resource]
Headers:    Content-Type: application/json (default)
            Accept: application/json (or application/xml for supported services)
Auth:       Bearer [token] (if applicable)

Request Body:
{
  "field1": "value1",
  "field2": 123
}

Response (200 OK):
{
  "id": "uuid",
  "field1": "value1",
  "field2": 123,
  "timestamp": "2026-06-25T10:30:00Z"
}

Error Response (400 Bad Request):
{
  "error": "Invalid input",
  "details": "field1 is required"
}
```

### Swagger/OpenAPI Documentation

Add to your service's Spring Boot / .NET / Go service:

```yaml
openapi: 3.0.0
info:
  title: [Service Name] API
  version: v1
paths:
  /[service-name]/v1/[resource]:
    post:
      summary: Create [resource]
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/[ResourceRequest]'
      responses:
        '200':
          description: Successfully created
        '400':
          description: Invalid input
```

---

## Debugging Checklist

### Service Won't Start

- [ ] Check Docker logs: `docker compose logs [service-name]`
- [ ] Verify dependencies installed: `[language package manager] install`
- [ ] Check port 80 available: `lsof -i :80`
- [ ] Verify configuration/environment variables set
- [ ] Check database connectivity (if applicable)

### Service Crashes

- [ ] Review application logs: `docker compose logs [service-name]`
- [ ] Check memory usage: `docker stats [service-name]`
- [ ] Verify health check passing: `curl http://localhost/[service-name]/health`
- [ ] Check external service connectivity

### Slow Response Times

- [ ] Profile service: `[language profiling tool]`
- [ ] Check database query performance
- [ ] Verify no network timeouts
- [ ] Monitor CPU/Memory: `docker stats`
- [ ] Check RabbitMQ queue depth (if async)

### Integration Issues

- [ ] Verify service registered in docker-compose.yaml
- [ ] Check service-to-service network connectivity
- [ ] Verify DNS resolution: `docker exec [container] nslookup [service]`
- [ ] Check firewall/proxy rules
- [ ] Review authentication/authorization

---

## Docker Compose Integration

Add to `docker-compose.yaml`:

```yaml
version: '3.8'
services:
  [service-name]:
    build:
      context: .
      dockerfile: [service-name]/Dockerfile
    ports:
      - "80"
    environment:
      - LOG_LEVEL=INFO
      - DATABASE_URL=jdbc:mssql://db:1433
      - RABBITMQ_HOST=rabbitmq
    depends_on:
      - db
      - rabbitmq
    networks:
      - easytrade
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost/health"]
      interval: 30s
      timeout: 10s
      retries: 3

networks:
  easytrade:
    driver: bridge
```

---

## Environment Variables Template

Create `.env` or document in README:

```bash
# Database
DB_HOST=db
DB_PORT=1433
DB_NAME=easytrade
DB_USER=sa
DB_PASSWORD=YourPassword123!

# RabbitMQ
RABBITMQ_HOST=rabbitmq
RABBITMQ_PORT=5672
RABBITMQ_USER=guest
RABBITMQ_PASSWORD=guest

# Service Configuration
LOG_LEVEL=INFO
SERVICE_PORT=8080
ENV=development

# External Services
DYNATRACE_URL=https://your-env.live.dynatrace.com
DYNATRACE_API_TOKEN=dt0c01.xxxxx
```

---

## Testing Template

### Unit Tests

```[language]
// Example: Java JUnit5
@Test
void testCreateResource() {
    // Arrange
    ResourceRequest request = new ResourceRequest("test");
    
    // Act
    ResourceResponse response = service.create(request);
    
    // Assert
    assertNotNull(response.getId());
    assertEquals("test", response.getName());
}
```

### Integration Tests

```bash
# Start services in test environment
docker compose -f docker-compose.test.yaml up -d

# Run integration tests
./gradlew test --tests IntegrationTest

# Cleanup
docker compose -f docker-compose.test.yaml down
```

### Load Testing

```bash
# Using loadgen service
docker compose up -d loadgen

# Monitor service performance
docker stats [service-name]

# Check Dynatrace for response times and errors
```

