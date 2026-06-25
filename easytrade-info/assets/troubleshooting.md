# EasyTrade Troubleshooting & Quick Checklist

## Pre-Flight Checklist

Before starting development or deployment:

- [ ] Docker daemon running: `docker info`
- [ ] Docker Compose installed: `docker compose version`
- [ ] Sufficient disk space: `df -h`
- [ ] Port 80 available: `lsof -i :80` (should be empty or only Docker services)
- [ ] Git configured: `git config --list | grep user`
- [ ] Repository cloned: `ls -la .git`
- [ ] Services defined: `ls -la docker-compose.yaml`

---

## Quick Troubleshooting Decision Tree

### "Services won't start"

1. **Check Docker status:**
   ```bash
   docker ps
   docker compose ps
   ```

2. **If containers exiting:**
   ```bash
   docker compose logs --tail=50 [service-name]
   ```

3. **If port conflict:**
   ```bash
   lsof -i :80
   # Kill conflicting service or use docker-compose port override
   ```

4. **If volume/permission issue:**
   ```bash
   docker compose down -v
   docker compose up -d
   ```

5. **Last resort — full reset:**
   ```bash
   docker compose down -v
   docker system prune -a --volumes
   docker compose build --no-cache
   docker compose up -d
   ```

---

### "Frontend not loading"

1. **Is it actually running?**
   ```bash
   curl -I http://localhost/
   ```

2. **Check reverse proxy logs:**
   ```bash
   docker compose logs frontendreverseproxy
   ```

3. **Verify backend services ready:**
   ```bash
   docker compose logs | grep -i "started\|error"
   ```

4. **Browser cache issue:**
   - Clear browser cache (Ctrl+Shift+Delete / Cmd+Shift+Delete)
   - Try incognito/private window

5. **Network issue:**
   ```bash
   docker exec frontend curl -I http://frontendreverseproxy
   ```

---

### "Cannot login to application"

1. **Check Login Service:**
   ```bash
   docker compose logs loginservice
   ```

2. **Verify credentials:**
   - Default: `demouser` / `demopass`
   - Or: `specialuser` / `specialpass`

3. **Check database connection:**
   ```bash
   docker compose logs db
   ```

4. **Try user signup first:**
   - Click "Sign up" on login page
   - Create new user
   - Return to login page
   - Attempt login with new credentials

5. **Reset database:**
   ```bash
   docker compose down -v
   docker compose up -d
   # Wait 2-3 minutes for DB migration
   ```

---

### "Build fails with compilation error"

**Java service:**
```bash
cd [service-name]
./gradlew clean build --debug
# Check error in output
```

**Node.js service:**
```bash
cd [service-name]
npm ci  # Use ci instead of install
npm run build
```

**Go service:**
```bash
cd [service-name]
go mod tidy
go mod verify
go build -v
```

**C# service:**
```bash
cd [service-name]
dotnet clean
dotnet restore
dotnet build -v
```

---

### "Service responding slowly"

1. **Check service logs:**
   ```bash
   docker compose logs -f [service-name] | grep -i "timeout\|slow\|error"
   ```

2. **Monitor resource usage:**
   ```bash
   docker stats [service-name]
   # Watch CPU, Memory, Network I/O
   ```

3. **Check database query performance:**
   ```bash
   docker exec db sqlcmd -S localhost -U sa -P YourPassword123! \
     -Q "SELECT COUNT(*) FROM sys.dm_exec_requests"
   ```

4. **Check if problem pattern enabled:**
   ```bash
   curl -s http://localhost/feature-flag-service/v1/flags | \
     jq '.[] | {flagId, enabled}'
   ```

5. **Network connectivity issue:**
   ```bash
   docker exec [service-name] ping aggregator-service
   ```

---

### "Vulnerability scan shows issues"

1. **Run scan:**
   ```bash
   snyk test --json --all-projects
   ```

2. **Identify affected services:**
   ```bash
   snyk test --json --all-projects | jq '.[] | {packageManager, vulnerabilities: .vulnerabilities | length}'
   ```

3. **For Java services:**
   ```bash
   cd [service]
   nano build.gradle
   # Edit vulnerability pin block
   ./gradlew build
   ```

4. **For Node services:**
   ```bash
   cd [service]
   nano package.json
   # Update dependencies and overrides
   npm install
   ```

5. **For Go services:**
   ```bash
   cd [service]
   nano go.mod
   # Bump go directive
   go mod tidy
   ```

6. **Verify fix:**
   ```bash
   snyk test --json --all-projects | grep -i "vulnerabilities"
   # Should show 0 for all projects
   ```

---

## Common Error Messages & Solutions

### "Cannot connect to Docker daemon"
```bash
# Solution: Start Docker
sudo systemctl start docker  # Linux
open -a Docker             # macOS
```

### "Port 80 is already allocated"
```bash
# Solution: Find and stop conflicting service
lsof -i :80
kill -9 <PID>

# Or: Use alternate port
docker compose -p easytrade up -d
```

### "Database connection failed"
```bash
# Solution: Wait for DB to initialize
docker compose logs db | tail -20
# Wait 30-60 seconds, then retry

# If still failing:
docker compose down -v
docker compose up -d db
sleep 60
docker compose up -d
```

### "Out of disk space"
```bash
# Solution: Clean up Docker
docker system prune -a --volumes

# Check space
df -h /var/lib/docker
```

### "Gradle wrapper permission denied"
```bash
# Solution: Make executable
chmod +x accountservice/gradlew
# Then retry build
```

### "npm ERR! 404 Not Found"
```bash
# Solution: Clear npm cache
npm cache clean --force
npm install
```

### "go: updates to go.sum needed"
```bash
# Solution: Tidy dependencies
go mod tidy
go mod verify
```

### "FATAL: role 'sa' does not exist"
```bash
# Solution: Reset database
docker compose down -v
docker compose up -d db
sleep 120  # Wait for initialization
docker compose up -d
```

---

## Performance Optimization Checklist

For slow applications:

- [ ] Check if problem patterns enabled (disable them)
- [ ] Verify database indices: `docker exec db sqlcmd ... < check_indices.sql`
- [ ] Monitor memory usage: `docker stats`
- [ ] Check network latency between services: `docker exec [service] ping [other-service]`
- [ ] Review application logs for warnings/errors
- [ ] Check RabbitMQ queue depth: `docker exec rabbitmq rabbitmqctl list_queues`
- [ ] Profile service with language-specific tools
- [ ] Enable query logging in database

---

## Deployment Checklist

Before deploying to production:

- [ ] All tests passing: `./gradlew test` (or equivalent)
- [ ] No vulnerabilities: `snyk test --json --all-projects`
- [ ] Exit code 0: `echo $?`
- [ ] Docker images built and tagged correctly
- [ ] All services started successfully: `docker compose ps`
- [ ] Frontend loads: `curl -I http://localhost/`
- [ ] Login works with test credentials
- [ ] Database migrations completed
- [ ] Problem patterns disabled
- [ ] Logs reviewed for errors
- [ ] Dynatrace monitoring active (if applicable)
- [ ] Backup of current production state
- [ ] Rollback plan documented

---

## Useful Commands Quick Reference

```bash
# Logs & Monitoring
docker compose logs [service]              # View service logs
docker compose logs -f                    # Follow all logs
docker stats                               # Monitor resource usage
docker compose top [service]               # View process info

# Service Management
docker compose ps                          # List running services
docker compose up -d [service]             # Start service
docker compose down [service]              # Stop service
docker compose restart [service]           # Restart service
docker compose logs [service] | tail -50   # Last 50 lines

# Network & Connectivity
docker exec [service] ping [other-service] # Test connectivity
docker exec [service] curl http://other-service/health  # Check health
docker network ls                          # List networks
docker network inspect easytrade           # View network details

# Data & Database
docker compose exec db sqlcmd              # Access database CLI
docker volume ls                           # List volumes
docker volume rm [volume]                  # Remove volume
docker compose down -v                     # Remove all volumes

# Building
docker compose build                       # Build all services
docker compose build --no-cache            # Rebuild from scratch
docker compose pull                        # Update base images

# Cleanup
docker system prune                        # Remove unused images/containers
docker system prune -a --volumes           # Aggressive cleanup
docker rmi [image]                         # Remove specific image
```

---

## Getting Help

1. **Check service README:**
   ```bash
   cat [service-name]/README.md
   ```

2. **Search GitHub issues:**
   ```
   https://github.com/Dynatrace/easytrade/issues
   ```

3. **Review AGENTS.md (repository guide):**
   ```bash
   cat AGENTS.md
   ```

4. **Check this skill's SKILL.md:**
   ```bash
   cat easytrade-info/SKILL.md
   ```

5. **Enable debug logging:**
   ```bash
   export DEBUG=*
   docker compose up  # See debug output
   ```

