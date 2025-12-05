# ADR-0001: Technology Stack Selection

**Status**: Accepted

**Date**: 2024-01-01

**Deciders**: Project Owner

---

## Context

This is a practical learning project to build a modern web application from scratch. The primary goals are:

1. **Learning**: Gain hands-on experience with modern web development technologies
2. **Portfolio**: Create a production-quality project demonstrating full-stack skills
3. **Practical Use**: Build a functional task and habit tracking application

The technology stack needs to:
- Support a full-stack web application (frontend, backend, database)
- Be widely used in the industry (transferable skills)
- Have good documentation and community support
- Enable containerized development and deployment
- Support best practices for testing, security, and maintainability

## Decision

We will use the following technology stack:

### Backend
- **Java 17+** with **Spring Boot 3.x** as the application framework
- **Spring Security** with **JWT** for authentication
- **Spring Data JPA** for data access
- **PostgreSQL** as the database
- **Flyway** for database migrations

### Frontend
- **React 18.x** as the UI library
- **TypeScript** for type-safe JavaScript
- **Vite** as the build tool
- **TanStack Query** for server state management
- **Tailwind CSS** or **Material-UI** for styling (final decision deferred)

### Infrastructure
- **Docker** and **Docker Compose** for containerization

### Testing
- **JUnit 5** + **Mockito** + **Testcontainers** for backend
- **Vitest** + **React Testing Library** for frontend

## Consequences

### Positive Consequences

1. **Industry Relevance**
   - Java/Spring is one of the most demanded backend stacks
   - React/TypeScript dominates frontend development
   - Docker skills are essential for modern development

2. **Learning Value**
   - Comprehensive exposure to enterprise patterns
   - Experience with statically-typed languages on both ends
   - Understanding of containerization concepts

3. **Ecosystem Benefits**
   - Extensive library ecosystem for both platforms
   - Strong community support and documentation
   - Mature tooling and IDE support

4. **Production Readiness**
   - Battle-tested technologies used in large-scale applications
   - Built-in support for security, monitoring, and scalability

### Negative Consequences

1. **Learning Curve**
   - Spring's complexity can be overwhelming initially
   - Multiple new concepts to learn simultaneously
   - TypeScript adds complexity to JavaScript

2. **Setup Overhead**
   - More initial configuration compared to simpler stacks
   - Docker adds another layer of complexity

3. **Resource Requirements**
   - JVM applications have higher memory footprint
   - Development machine needs Docker support

### Neutral Consequences

1. **Not the "latest" stack** - Chose stability over cutting-edge (e.g., not using Rust, Go, or Deno)
2. **Opinionated choices** - Could have used Node.js/Express or Python/FastAPI as alternatives

## Alternatives Considered

### Alternative 1: Node.js + Express Backend

**Pros:**
- Same language (JavaScript/TypeScript) for full stack
- Faster to get started
- Lighter weight than JVM

**Cons:**
- Less structured than Spring (more decisions to make)
- TypeScript support less integrated than Java's type system
- Want to specifically learn Java/Spring

**Why not chosen:** The explicit goal is to learn Java/Spring ecosystem, which is highly valued in enterprise environments.

### Alternative 2: Python + FastAPI Backend

**Pros:**
- Simpler syntax
- Great for rapid development
- Good async support

**Cons:**
- Different paradigm from enterprise Java
- Less structured approach to large applications
- Want to specifically learn Java/Spring

**Why not chosen:** While Python is excellent, the project goals specifically include learning Java and Spring Boot.

### Alternative 3: Next.js Full-Stack

**Pros:**
- Unified React-based full stack
- Server-side rendering built-in
- Simpler deployment (Vercel)

**Cons:**
- Less separation between frontend and backend
- Would miss learning traditional REST API design
- Want explicit frontend/backend separation for learning

**Why not chosen:** The goal is to learn separate frontend and backend development patterns, which are still common in enterprise settings.

### Alternative 4: MySQL or MongoDB instead of PostgreSQL

**Pros:**
- MySQL: Very widely used, simpler
- MongoDB: Flexible schema, good for rapid iteration

**Cons:**
- MySQL: Fewer advanced features than PostgreSQL
- MongoDB: Different paradigm, less applicable to relational data

**Why not chosen:** PostgreSQL offers the best balance of features, reliability, and learning value for a relational application.

## References

- [Spring Boot Documentation](https://docs.spring.io/spring-boot/docs/current/reference/html/)
- [React Documentation](https://react.dev/)
- [TypeScript Documentation](https://www.typescriptlang.org/docs/)
- [PostgreSQL Documentation](https://www.postgresql.org/docs/)
- [Docker Documentation](https://docs.docker.com/)

---

## Notes

### Future Considerations

- **CI/CD**: GitHub Actions will be added when needed
- **Cloud Deployment**: AWS/GCP deployment is out of scope initially but the Docker setup enables this
- **Mobile**: React Native could be considered for Phase 4+

### Version Selection Rationale

- **Java 17+**: LTS version with modern features (records, pattern matching)
- **Spring Boot 3.x**: Latest major version with Jakarta EE namespace
- **React 18.x**: Current stable version with concurrent features
- **PostgreSQL 15+**: Latest stable with performance improvements
