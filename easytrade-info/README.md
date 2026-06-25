# EasyTrade Info Skill

A comprehensive skill providing complete documentation, guides, and workflows for the EasyTrade microservices application.

## Contents

### SKILL.md
The main skill file containing:
- **Project Overview:** What EasyTrade is and its purpose
- **Repository Structure:** 19 services organized by technology (Java, Go, TypeScript, C#)
- **Service Endpoints:** Port mappings and proxy routes
- **Getting Started:** Docker Compose and Kubernetes deployment instructions
- **Vulnerability Remediation Process:** Detailed steps for scanning and fixing vulnerabilities by tech stack
- **Problem Patterns:** 4 built-in patterns for testing system behavior
- **Dynatrace Integration:** Business events, monitoring, and MCP Server setup

### References/
Detailed reference materials for common tasks:

#### quick-commands.md
- Essential command reference for development
- Building services by technology
- Vulnerability scanning commands
- Feature flag and problem pattern management
- Authentication and testing commands
- Kubernetes operations
- Cleanup procedures

#### service-matrix.md
- Complete service overview table
- Technology stack summary
- Build matrix with commands
- Data flow diagrams
- Network communication patterns
- XML compatibility matrix
- Health and monitoring endpoints

#### workflows.md
Step-by-step workflows for 8 common scenarios:
1. Local development environment setup
2. Modifying Java services
3. Scanning for vulnerabilities
4. Testing problem patterns
5. Kubernetes deployment
6. Contributing code (PR workflow)
7. Monitoring with Dynatrace
8. Cleanup and reset procedures

### Assets/
Practical templates and checklists:

#### service-development-template.md
Template for creating or modifying services:
- Service overview structure
- File structure template
- Implementation checklist (setup, code, security, Docker, integration, documentation)
- Build and run commands by language
- Vulnerability remediation steps
- API endpoint definition template
- Docker Compose integration example
- Environment variables template
- Testing templates

#### troubleshooting.md
Comprehensive troubleshooting guide:
- Pre-flight checklist
- Decision tree for common issues
- Quick troubleshooting for: startup failures, frontend issues, login problems, build errors, slow responses, vulnerabilities
- Common error messages and solutions
- Performance optimization checklist
- Deployment pre-flight checklist
- Quick command reference
- Getting help resources

## How to Use This Skill

### For New Developers
1. Start with **SKILL.md** → "Getting Started" section
2. Follow the Docker Compose setup in **workflows.md** → "Workflow 1"
3. Refer to **service-matrix.md** to understand service organization
4. Use **troubleshooting.md** if you encounter issues

### For Modifying a Service
1. Identify service technology in **service-matrix.md**
2. Use **assets/service-development-template.md** for implementation guidelines
3. Follow language-specific build commands from **quick-commands.md**
4. Run vulnerability scan using steps in **SKILL.md** → "Vulnerability Remediation"

### For Fixing Vulnerabilities
1. Review **SKILL.md** → "Vulnerability Remediation Process"
2. Run scan: `snyk test --json --all-projects`
3. Follow technology-specific steps:
   - **Java:** Update `build.gradle` pins
   - **Node:** Update `package.json` dependencies and overrides
   - **Go:** Bump Go version in `go.mod` and `Dockerfile`
   - **C#:** Update NuGet packages
4. Verify: Run scan again, expect exit code 0

### For Kubernetes Deployment
1. Check prerequisites in **workflows.md** → "Workflow 5"
2. Use Helm commands from **SKILL.md** → "Kubernetes Deployment"
3. Monitor with kubectl commands from **quick-commands.md**
4. Troubleshoot using **assets/troubleshooting.md**

### For Testing Problem Patterns
1. Read pattern descriptions in **SKILL.md** → "Problem Patterns"
2. Follow testing steps in **workflows.md** → "Workflow 4"
3. Use curl commands from **references/quick-commands.md** → "Feature Flags & Problem Patterns"
4. Monitor in Dynatrace or via logs

### For Troubleshooting
1. Check **assets/troubleshooting.md** decision tree
2. Run suggested diagnostic commands
3. Follow solutions for your specific error
4. Refer to **quick-commands.md** for command syntax

---

## Technology Stack Reference

| Tech | Services | Build Command |
|------|----------|---------------|
| Java 21/Gradle | 6 services | `./gradlew build` |
| Go | 3 services | `go build .` |
| TypeScript/npm | 3 services | `npm run build` |
| C#/.NET 8 | 3 services | `dotnet build` |
| C++ | 1 service | Via Dockerfile |

---

## Key Endpoints

- **Frontend:** http://localhost/
- **Feature Flags:** http://localhost/feature-flag-service/v1/flags
- **Default User:** demouser / demopass

---

## Quick Start Command

```bash
# Clone, start, and access
git clone https://github.com/Dynatrace/easytrade.git
cd easytrade
docker compose up -d
# Wait 2-3 minutes for services to stabilize
open http://localhost  # or navigate in browser
```

---

## File Organization

```
easytrade-info/
├── SKILL.md                              # Main skill documentation
├── README.md                             # This file
├── references/
│   ├── quick-commands.md                # Essential commands
│   ├── service-matrix.md                # Service overview & tech stack
│   └── workflows.md                     # 8 step-by-step workflows
└── assets/
    ├── service-development-template.md  # Template for new/modified services
    └── troubleshooting.md               # Troubleshooting guide & checklists
```

---

## External Resources

- **GitHub Repository:** https://github.com/Dynatrace/easytrade
- **Dynatrace Documentation:** https://docs.dynatrace.com
- **Snyk Security:** https://snyk.io
- **Docker Compose:** https://docs.docker.com/compose/
- **Kubernetes:** https://kubernetes.io/docs/

---

## Tips

- **Always run `snyk test --json --all-projects` before committing** — no vulnerabilities is a requirement
- **Services take 2-3 minutes to stabilize** — initial errors/missing data are normal
- **Vulnerability fixes should be applied across all affected services simultaneously** — consistency prevents future issues
- **Docker Compose V1 will not work** — ensure you have Compose Plugin installed
- **Problem patterns are for testing only** — disable before sharing environments

---

## Support

For issues, questions, or contributions:
1. Check the troubleshooting section first
2. Review the relevant workflow for your task
3. Consult the GitHub repository: https://github.com/Dynatrace/easytrade
4. Create an issue with detailed error messages and logs

