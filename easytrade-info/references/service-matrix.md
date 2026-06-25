# EasyTrade Service Matrix

## Complete Service Overview

| Service | Language | Framework | Port | Endpoint | Purpose |
|---------|----------|-----------|------|----------|---------|
| **Frontend** | TypeScript | Vite/Vue | 80 | `/` | Web UI, user interface |
| **Frontend Reverse Proxy** | Nginx | Nginx | 80 | — | Proxy routing |
| **Account Service** | Java | Spring Boot | 80 | `/accountservice` | User account management |
| **Aggregator Service** | Go | Go Modules | 80 | — | Data aggregation |
| **Broker Service** | C# | .NET 8 | 80 | `/broker-service` | Brokerage operations |
| **Calculation Service** | C++ | Native | 80 | — | Complex calculations |
| **Content Creator** | Java | Spring Boot | 80 | — | Content generation |
| **Credit Card Order Service** | Java | Spring Boot | 80 | `/credit-card-order-service` | Payment processing |
| **Database** | T-SQL | MSSQL | 80 | — | Data persistence |
| **Engine** | Java | Spring Boot | 80 | `/engine` | Core business logic |
| **Feature Flag Service** | Java | Spring Boot | 80 | `/feature-flag-service` | Feature toggles & problem patterns |
| **Loadgen** | TypeScript | Node.js | — | — | Load generation for testing |
| **Login Service** | C# | .NET 8 | 80 | `/loginservice` | Authentication |
| **Manager** | C# | .NET 8 | 80 | `/manager` | Application management |
| **Offer Service** | TypeScript | Node.js | 80 | `/offerservice` | Stock offer generation |
| **Pricing Service** | Go | Go Modules | 80 | `/pricing-service` | Price calculations |
| **Problem Operator** | Go | Go Modules | — | — | Kubernetes problem injection |
| **RabbitMQ** | Erlang | RabbitMQ | 80 | — | Message broker |
| **Third Party Service** | Java | Spring Boot | 80 | `/third-party-service` | External integrations |

## Technology Stack Summary

### By Language
- **Java (6):** accountservice, contentcreator, credit-card-order-service, engine, feature-flag-service, third-party-service
- **TypeScript (3):** frontend, loadgen, offerservice
- **C# (3):** broker-service, loginservice, manager
- **Go (3):** aggregator-service, pricing-service, problem-operator
- **C++ (1):** calculationservice
- **T-SQL (1):** db
- **Erlang (1):** rabbitmq

### By Framework
- **Spring Boot (6 Java):** All Java microservices
- **Vite + Vue (1 TypeScript):** frontend
- **Node.js (2 TypeScript):** loadgen, offerservice
- **.NET 8 (3 C#):** broker-service, loginservice, manager
- **Native (1 C++):** calculationservice
- **Go Modules (3 Go):** aggregator-service, pricing-service, problem-operator

### By Infrastructure
- **Docker Compose:** All services
- **Kubernetes/Helm:** All services deployable
- **RabbitMQ:** Message queue
- **MSSQL:** Database

## Build Matrix

| Technology | Build Command | Service Count | Services |
|------------|---------------|---------------|----------|
| Java/Gradle | `./gradlew build` | 6 | accountservice, contentcreator, credit-card-order-service, engine, feature-flag-service, third-party-service |
| Node.js/npm | `npm run build` | 2 | frontend, offerservice |
| Go | `go build .` | 3 | aggregator-service, pricing-service, problem-operator |
| .NET 8 | `dotnet build` | 3 | broker-service, loginservice, manager |
| C++ | Build via Dockerfile | 1 | calculationservice |
| No package manifest | Manual/Docker | 4 | frontendreverseproxy, rabbitmq, db, loadgen* |

*loadgen has package.json but is a utility, not a deployed service

## Data Flow & Dependencies

```
Frontend (Vue) → Frontend Reverse Proxy (nginx)
                 ↓
        ┌────────┴────────┬────────┬─────────┐
        ↓                 ↓        ↓         ↓
   Login Service    Account Service  Manager  Broker Service
        ↓                 ↓                   ↓
   Authentication   User Data         Credit Card Orders
                                              ↓
                                     Third Party Service
                                     Credit Card Factory
                                              ↓
        ┌────────────────────────────────────┴────────┐
        ↓                                              ↓
   Engine (Core Logic)                        Content Creator
        ↓                                       ↓
   Pricing Service                      (Content Generation)
        ↓
   Aggregator Service
        ↓
   Offer Service
        
   Feature Flag Service ← Problem Patterns (feature toggles)
   
   Database (MSSQL) ← All services
   RabbitMQ ← Message bus for async operations
```

## Problem Patterns & Affected Services

| Pattern | Primary Service | Affected Services | Duration |
|---------|-----------------|------------------|----------|
| **DbNotResponding** | Database | All services | ~20 min |
| **ErgoAggregatorSlowdown** | Aggregator Service | Pricing Service, Offer Service | 15-20 min |
| **FactoryCrisis** | Third Party Service | Credit Card Order Service | Variable |
| **HighCpuUsage** | Broker Service | Broker Service, Problem Operator (K8s) | Variable |

## Network Communication

| Source | Destination | Protocol | Purpose |
|--------|-------------|----------|---------|
| Frontend | Reverse Proxy | HTTP/REST | User interface |
| Reverse Proxy | All Services | HTTP/REST | Route requests |
| Broker Service | Third Party Service | HTTP/REST | Card processing |
| Engine | Pricing Service | HTTP/REST | Price lookups |
| Engine | Aggregator Service | HTTP/REST | Data aggregation |
| All Services | Database | TCP/SQL | Data persistence |
| Services | RabbitMQ | AMQP | Async messaging |
| Feature Flag Service | All Services | HTTP/REST | Feature toggles |

## XML Compatibility

The following services accept XML payloads via content negotiation:

| Service | Accepted MIME Types |
|---------|-------------------|
| **LoginService** | `application/xml`, `text/xml`, `application/*+xml` |
| **CreditCardOrderService** | `application/xml` |
| **OfferService** | `application/xml`, `text/xml` |
| **PricingService** | `application/xml` |

## Health & Monitoring

| Service | Health Endpoint | Metrics Endpoint | Status |
|---------|-----------------|-----------------|--------|
| Frontend | — | — | Web UI |
| Account Service | `/actuator/health` | `/actuator/metrics` | Spring Boot |
| Broker Service | `/health` | `/metrics` | .NET 8 |
| Pricing Service | — | — | Go service |
| All Services | — | — | Observable by Dynatrace |
