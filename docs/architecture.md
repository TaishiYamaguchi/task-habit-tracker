# System Architecture

This document describes the overall architecture of the Task & Habit Tracker application, including system components, design patterns, and key architectural decisions.

## Architecture Overview

The application follows a **layered architecture** pattern with clear separation of concerns between frontend and backend components.

```
┌──────────────────────────────────────────────────────────────────┐
│                         Client Layer                             │
│  ┌────────────────────────────────────────────────────────────┐  │
│  │                    React SPA (TypeScript)                   │  │
│  │  Components → Hooks → Services → API Client (Axios)        │  │
│  └────────────────────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────────────────┘
                              │ HTTP/REST
                              ▼
┌──────────────────────────────────────────────────────────────────┐
│                        Backend Layer                             │
│  ┌────────────────────────────────────────────────────────────┐  │
│  │                  Spring Boot Application                    │  │
│  │  Controller → Service → Repository → Entity                │  │
│  └────────────────────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────────────────┘
                              │ JDBC
                              ▼
┌──────────────────────────────────────────────────────────────────┐
│                        Data Layer                                │
│  ┌────────────────────────────────────────────────────────────┐  │
│  │                      PostgreSQL                             │  │
│  └────────────────────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────────────────┘
```

---

## Backend Architecture

### Layered Architecture

The backend follows a traditional **layered architecture** pattern:

```
┌─────────────────────────────────────────────────────────┐
│                   Presentation Layer                     │
│  ┌─────────────────────────────────────────────────────┐ │
│  │  REST Controllers                                    │ │
│  │  • Handle HTTP requests/responses                   │ │
│  │  • Input validation                                 │ │
│  │  • DTO transformation                               │ │
│  └─────────────────────────────────────────────────────┘ │
├─────────────────────────────────────────────────────────┤
│                    Business Layer                        │
│  ┌─────────────────────────────────────────────────────┐ │
│  │  Service Classes                                     │ │
│  │  • Business logic                                   │ │
│  │  • Transaction management                           │ │
│  │  • Orchestration                                    │ │
│  └─────────────────────────────────────────────────────┘ │
├─────────────────────────────────────────────────────────┤
│                   Persistence Layer                      │
│  ┌─────────────────────────────────────────────────────┐ │
│  │  Repositories (Spring Data JPA)                     │ │
│  │  • Data access abstraction                          │ │
│  │  • Query methods                                    │ │
│  │  • Entity management                                │ │
│  └─────────────────────────────────────────────────────┘ │
├─────────────────────────────────────────────────────────┤
│                    Domain Layer                          │
│  ┌─────────────────────────────────────────────────────┐ │
│  │  Entities & Value Objects                           │ │
│  │  • Domain models                                    │ │
│  │  • JPA annotations                                  │ │
│  └─────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────┘
```

### Package Structure

```
src/main/java/com/taskhabittracker/
├── TaskHabitTrackerApplication.java
├── config/
│   ├── SecurityConfig.java
│   ├── WebConfig.java
│   └── OpenApiConfig.java
├── controller/
│   ├── AuthController.java
│   ├── TaskController.java
│   └── UserController.java
├── dto/
│   ├── request/
│   │   ├── LoginRequest.java
│   │   └── TaskCreateRequest.java
│   └── response/
│       ├── TaskResponse.java
│       └── UserResponse.java
├── entity/
│   ├── User.java
│   └── Task.java
├── exception/
│   ├── GlobalExceptionHandler.java
│   ├── ResourceNotFoundException.java
│   └── UnauthorizedException.java
├── repository/
│   ├── TaskRepository.java
│   └── UserRepository.java
├── security/
│   ├── JwtAuthenticationFilter.java
│   ├── JwtTokenProvider.java
│   └── UserPrincipal.java
└── service/
    ├── AuthService.java
    ├── TaskService.java
    └── UserService.java
```

### Key Design Patterns

| Pattern | Usage |
|---------|-------|
| **Repository Pattern** | Data access abstraction via Spring Data JPA |
| **DTO Pattern** | Separate API contracts from domain models |
| **Dependency Injection** | Constructor-based injection for testability |
| **Builder Pattern** | Fluent object construction (via Lombok) |

---

## Frontend Architecture

### Component-Based Architecture

The frontend follows a **component-based architecture** with clear separation:

```
┌─────────────────────────────────────────────────────────┐
│                      Pages                               │
│  • Route-level components                               │
│  • Data fetching orchestration                          │
│  • Layout composition                                   │
├─────────────────────────────────────────────────────────┤
│                    Components                            │
│  ┌───────────────┐  ┌───────────────┐  ┌─────────────┐ │
│  │   Features    │  │    Shared     │  │     UI      │ │
│  │ Domain-specific│  │  Reusable    │  │  Base       │ │
│  │  components   │  │  components   │  │ components  │ │
│  └───────────────┘  └───────────────┘  └─────────────┘ │
├─────────────────────────────────────────────────────────┤
│                      Hooks                               │
│  • Custom hooks for business logic                      │
│  • API integration hooks (TanStack Query)               │
│  • Form handling hooks                                  │
├─────────────────────────────────────────────────────────┤
│                     Services                             │
│  • API client configuration                             │
│  • HTTP interceptors                                    │
│  • Authentication service                               │
├─────────────────────────────────────────────────────────┤
│                      Types                               │
│  • TypeScript interfaces                                │
│  • API response types                                   │
│  • Shared constants                                     │
└─────────────────────────────────────────────────────────┘
```

### Directory Structure

```
src/
├── App.tsx
├── main.tsx
├── api/
│   ├── client.ts          # Axios instance configuration
│   ├── auth.ts            # Authentication API calls
│   └── tasks.ts           # Tasks API calls
├── components/
│   ├── features/
│   │   ├── tasks/
│   │   │   ├── TaskList.tsx
│   │   │   ├── TaskItem.tsx
│   │   │   └── TaskForm.tsx
│   │   └── auth/
│   │       ├── LoginForm.tsx
│   │       └── RegisterForm.tsx
│   ├── shared/
│   │   ├── Header.tsx
│   │   ├── Sidebar.tsx
│   │   └── Layout.tsx
│   └── ui/
│       ├── Button.tsx
│       ├── Input.tsx
│       └── Card.tsx
├── hooks/
│   ├── useAuth.ts
│   ├── useTasks.ts
│   └── useLocalStorage.ts
├── pages/
│   ├── HomePage.tsx
│   ├── LoginPage.tsx
│   ├── RegisterPage.tsx
│   └── TasksPage.tsx
├── types/
│   ├── auth.ts
│   ├── task.ts
│   └── api.ts
└── utils/
    ├── constants.ts
    └── helpers.ts
```

---

## Authentication Flow

### JWT-Based Authentication

```
┌─────────┐                    ┌─────────┐                    ┌────────┐
│ Client  │                    │ Backend │                    │   DB   │
└────┬────┘                    └────┬────┘                    └───┬────┘
     │                              │                              │
     │  1. POST /api/auth/login     │                              │
     │  {username, password}        │                              │
     │ ─────────────────────────────▶                              │
     │                              │  2. Verify credentials       │
     │                              │ ─────────────────────────────▶
     │                              │                              │
     │                              │  3. User data                │
     │                              │ ◀─────────────────────────────
     │                              │                              │
     │  4. JWT Token                │                              │
     │ ◀─────────────────────────────                              │
     │                              │                              │
     │  5. Request with Bearer token│                              │
     │  Authorization: Bearer <JWT> │                              │
     │ ─────────────────────────────▶                              │
     │                              │  6. Validate & extract user  │
     │                              │                              │
     │  7. Protected resource       │                              │
     │ ◀─────────────────────────────                              │
     │                              │                              │
```

### Token Structure

```json
{
  "header": {
    "alg": "HS512",
    "typ": "JWT"
  },
  "payload": {
    "sub": "user_id",
    "username": "john_doe",
    "iat": 1699900000,
    "exp": 1699986400
  }
}
```

### Security Considerations

- **Token Storage**: Store JWT in memory or httpOnly cookies (not localStorage)
- **Token Expiration**: Short-lived access tokens (24 hours initially)
- **HTTPS**: All communication over HTTPS in production
- **CORS**: Strict CORS configuration for API endpoints

---

## API Design Principles

### RESTful API Guidelines

| Principle | Description |
|-----------|-------------|
| **Resource-Based URLs** | `/api/tasks`, `/api/users` |
| **HTTP Methods** | GET (read), POST (create), PUT (update), DELETE (remove) |
| **Stateless** | Each request contains all necessary information |
| **JSON** | Request/response bodies in JSON format |
| **HTTP Status Codes** | Appropriate status codes for responses |

### API Endpoint Structure

```
Base URL: /api

Authentication:
  POST   /api/auth/register     # User registration
  POST   /api/auth/login        # User login
  POST   /api/auth/refresh      # Refresh token

Tasks:
  GET    /api/tasks             # List all tasks for user
  GET    /api/tasks/:id         # Get single task
  POST   /api/tasks             # Create task
  PUT    /api/tasks/:id         # Update task
  DELETE /api/tasks/:id         # Delete task
  PATCH  /api/tasks/:id/toggle  # Toggle completion status

Users:
  GET    /api/users/me          # Get current user profile
  PUT    /api/users/me          # Update current user profile
```

### Standard Response Format

**Success Response:**
```json
{
  "success": true,
  "data": { ... },
  "message": "Operation successful"
}
```

**Error Response:**
```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid input",
    "details": [
      { "field": "email", "message": "Invalid email format" }
    ]
  }
}
```

---

## Docker Container Composition

### Development Environment

```yaml
# docker-compose.yml structure
services:
  backend:
    build: ./backend
    ports: ["8080:8080"]
    environment:
      - SPRING_PROFILES_ACTIVE=dev
      - DB_HOST=database
    depends_on:
      - database

  frontend:
    build: ./frontend
    ports: ["3000:3000"]
    environment:
      - VITE_API_URL=http://localhost:8080

  database:
    image: postgres:15-alpine
    ports: ["5432:5432"]
    volumes:
      - postgres_data:/var/lib/postgresql/data
    environment:
      - POSTGRES_DB=task_habit_tracker
      - POSTGRES_USER=${DB_USER}
      - POSTGRES_PASSWORD=${DB_PASSWORD}

volumes:
  postgres_data:
```

### Container Communication

```
┌─────────────────────────────────────────────────────────────┐
│                    Docker Network                            │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐  │
│  │   Frontend   │───▶│   Backend    │───▶│   Database   │  │
│  │   :3000      │    │   :8080      │    │   :5432      │  │
│  └──────────────┘    └──────────────┘    └──────────────┘  │
│         │                   │                              │
└─────────│───────────────────│──────────────────────────────┘
          │                   │
    ┌─────▼─────┐      ┌─────▼─────┐
    │  Browser  │      │ API Tools │
    │ localhost │      │ (Postman) │
    └───────────┘      └───────────┘
```

---

## Development Environment Setup

### Prerequisites

1. **Java 17+** - Backend development
2. **Node.js 18+** - Frontend development
3. **Docker & Docker Compose** - Container orchestration
4. **IDE** - IntelliJ IDEA (recommended) or VS Code

### Quick Start Flow

```bash
# 1. Clone repository
git clone <repository-url>
cd task-habit-tracker

# 2. Copy environment variables
cp .env.example .env

# 3. Start all services
docker-compose up -d

# 4. Access application
# Frontend: http://localhost:3000
# Backend API: http://localhost:8080
# API Docs: http://localhost:8080/swagger-ui.html
```

### Development Workflow

```
┌─────────────────────────────────────────────────────────────┐
│                   Development Workflow                       │
│                                                              │
│  1. Feature Branch    ─▶  2. Local Development               │
│     (git checkout)        (docker-compose up)                │
│                                                              │
│  3. Write Tests       ─▶  4. Implementation                  │
│     (TDD approach)        (Iterative)                        │
│                                                              │
│  5. Run Tests         ─▶  6. Code Review                     │
│     (All pass)            (PR submission)                    │
│                                                              │
│  7. Merge             ─▶  8. Deploy (future)                 │
│     (to develop)          (CI/CD)                            │
└─────────────────────────────────────────────────────────────┘
```

---

## Future Architecture Considerations

### Phase 2+ Enhancements

| Feature | Architecture Impact |
|---------|---------------------|
| **Habit Tracking** | New entity, service, and API endpoints |
| **Calendar View** | Frontend component, date-based queries |
| **Notifications** | WebSocket or Push notification service |
| **File Attachments** | Object storage integration (S3-compatible) |
| **Team Features** | Multi-tenancy considerations |

### Scalability Path

1. **Database**: Connection pooling, query optimization
2. **Caching**: Redis for session and query caching
3. **API**: Rate limiting, pagination
4. **Frontend**: Code splitting, lazy loading

---

## Related Documentation

- [Technology Stack](./tech-stack.md) - Detailed technology choices
- [Database Design](./database-design.md) - Data model and schema
- [Features Roadmap](./ja/features-roadmap.md) - Development phases
- [ADR: Tech Stack Selection](./adr/0001-tech-stack-selection.md) - Technology decisions
