# Technology Stack

This document provides a comprehensive overview of the technologies used in the Task & Habit Tracker project. Each technology choice is documented with rationale to help developers understand the project's technical foundation.

## Overview

The project follows a modern full-stack architecture with:
- **Backend**: Java/Spring Boot REST API
- **Frontend**: React/TypeScript SPA
- **Database**: PostgreSQL
- **Infrastructure**: Docker containerization

---

## Backend Technologies

### Core Framework

| Technology | Version | Purpose |
|------------|---------|---------|
| **Java** | 17+ (LTS) | Primary backend language |
| **Spring Boot** | 3.x | Application framework with auto-configuration |
| **Spring Security** | 6.x | Authentication and authorization |
| **Spring Data JPA** | 3.x | Database access abstraction |

#### Why Spring Boot?
- **Productivity**: Auto-configuration reduces boilerplate code
- **Ecosystem**: Extensive library support and community
- **Production-Ready**: Built-in monitoring, health checks, and externalized configuration
- **Industry Standard**: Widely used in enterprise applications

### Supporting Libraries

| Technology | Purpose |
|------------|---------|
| **Lombok** | Reduces boilerplate code (getters, setters, builders) |
| **MapStruct** | Type-safe object mapping between layers |
| **Jakarta Validation** | Request validation annotations |

### Database

| Technology | Purpose |
|------------|---------|
| **PostgreSQL** | Primary relational database |
| **Flyway** or **Liquibase** | Database migration management |
| **HikariCP** | Connection pooling (Spring Boot default) |

#### Why PostgreSQL?
- **Reliability**: ACID-compliant with excellent data integrity
- **Features**: Advanced features like JSON support, full-text search
- **Performance**: Handles concurrent workloads efficiently
- **Community**: Strong community support and documentation

### API Documentation

| Technology | Purpose |
|------------|---------|
| **SpringDoc OpenAPI** | OpenAPI 3.0 specification generation |
| **Swagger UI** | Interactive API documentation |

---

## Frontend Technologies

### Core Framework

| Technology | Version | Purpose |
|------------|---------|---------|
| **React** | 18.x | UI component library |
| **TypeScript** | 5.x | Type-safe JavaScript |
| **Vite** | 5.x | Build tool and dev server |

#### Why React?
- **Component Model**: Reusable, composable UI components
- **Ecosystem**: Extensive library ecosystem
- **Performance**: Virtual DOM and efficient updates
- **Developer Experience**: Excellent tooling and debugging

#### Why TypeScript?
- **Type Safety**: Catch errors at compile time
- **IDE Support**: Better autocomplete and refactoring
- **Documentation**: Types serve as inline documentation
- **Maintainability**: Easier to refactor and maintain code

### Routing & State Management

| Technology | Purpose |
|------------|---------|
| **React Router** | Client-side routing |
| **TanStack Query** (React Query) | Server state management and caching |

#### Why TanStack Query?
- **Caching**: Automatic caching and background updates
- **Loading States**: Built-in loading and error states
- **Optimistic Updates**: Better UX with optimistic mutations
- **DevTools**: Excellent debugging tools

### HTTP Client

| Technology | Purpose |
|------------|---------|
| **Axios** | HTTP client for API calls |

### Styling

| Technology | Purpose |
|------------|---------|
| **Tailwind CSS** or **Material-UI (MUI)** | UI styling framework |

> **Decision Pending**: Final choice between Tailwind CSS (utility-first) and MUI (component library) will be made at the start of frontend development based on design requirements.

---

## Infrastructure

### Containerization

| Technology | Purpose |
|------------|---------|
| **Docker** | Application containerization |
| **Docker Compose** | Multi-container orchestration |

#### Container Architecture
```
┌─────────────────────────────────────────┐
│              Docker Compose             │
├─────────────┬─────────────┬─────────────┤
│   Frontend  │   Backend   │  Database   │
│   (React)   │ (Spring)    │ (PostgreSQL)│
│   :3000     │   :8080     │   :5432     │
└─────────────┴─────────────┴─────────────┘
```

#### Why Docker?
- **Consistency**: Same environment across development, testing, and production
- **Isolation**: Isolated dependencies and configurations
- **Portability**: Easy to share and deploy
- **Learning**: Valuable DevOps skill

---

## Testing Technologies

### Backend Testing

| Technology | Purpose |
|------------|---------|
| **JUnit 5** | Unit and integration testing framework |
| **Mockito** | Mocking framework |
| **AssertJ** | Fluent assertions |
| **Testcontainers** | Docker-based integration testing |

#### Why Testcontainers?
- **Realistic Tests**: Test against real PostgreSQL instance
- **Isolation**: Each test gets clean database state
- **CI-Friendly**: Works well in CI/CD pipelines

### Frontend Testing

| Technology | Purpose |
|------------|---------|
| **Vitest** | Unit testing framework (Vite-native) |
| **React Testing Library** | Component testing |
| **MSW** (Mock Service Worker) | API mocking |

---

## Quality & Documentation Tools

### Code Quality

| Technology | Purpose |
|------------|---------|
| **ESLint** | JavaScript/TypeScript linting |
| **Prettier** | Code formatting |
| **Checkstyle** | Java code style checking |
| **SpotBugs** | Java static analysis |

### Documentation

| Technology | Purpose |
|------------|---------|
| **SpringDoc OpenAPI** | API documentation |
| **Architecture Decision Records (ADR)** | Design decision documentation |

---

## CI/CD (Future Consideration)

| Technology | Purpose |
|------------|---------|
| **GitHub Actions** | CI/CD automation |

### Planned CI/CD Pipeline
1. **Build**: Compile and package application
2. **Test**: Run unit and integration tests
3. **Lint**: Check code style and quality
4. **Security Scan**: Dependency vulnerability scanning
5. **Deploy**: (Future) Automated deployment

---

## Version Compatibility Matrix

| Component | Minimum Version | Recommended Version |
|-----------|-----------------|---------------------|
| Java | 17 | 21 (latest LTS) |
| Node.js | 18 | 20 (LTS) |
| Docker | 20.10 | Latest stable |
| Docker Compose | 2.0 | Latest stable |

---

## Related Documentation

- [Architecture Overview](./architecture.md) - System architecture details
- [Database Design](./database-design.md) - Schema and data model
- [ADR: Tech Stack Selection](./adr/0001-tech-stack-selection.md) - Decision rationale

---

## Technology Updates Policy

- **Security Patches**: Apply within 1 week of release
- **Minor Updates**: Evaluate monthly
- **Major Updates**: Evaluate during planning phases
- **Dependencies**: Use tools like Dependabot for automated updates
