# Task & Habit Tracker

A full-stack web application for managing tasks and tracking daily habits. Built as a practical learning project to demonstrate modern web development best practices.

## Overview

Task & Habit Tracker is a personal productivity application that helps users:
- Manage daily tasks with status tracking
- Build and maintain healthy habits with streak tracking
- Visualize progress through calendars and heatmaps

This project serves as a comprehensive learning platform for modern full-stack development, covering Java/Spring Boot, React/TypeScript, PostgreSQL, and Docker.

## Features

### Current (Phase 1 - MVP)
- [ ] User registration and authentication (JWT)
- [ ] Task CRUD operations
- [ ] Task completion toggle
- [ ] Daily task view

### Planned (Phase 2)
- [ ] Habit tracking with daily records
- [ ] Calendar view with heatmap visualization
- [ ] Categories and tags
- [ ] Search and filtering

### Future (Phase 3+)
- [ ] Weather API integration for suggestions
- [ ] Reminder notifications
- [ ] Team collaboration features
- [ ] File attachments

## Technology Stack

| Layer | Technologies |
|-------|-------------|
| **Backend** | Java 17+, Spring Boot 3.x, Spring Security, Spring Data JPA |
| **Frontend** | React 18, TypeScript, Vite, TanStack Query |
| **Database** | PostgreSQL 15+ |
| **Infrastructure** | Docker, Docker Compose |
| **Testing** | JUnit 5, Testcontainers, Vitest, React Testing Library |

For detailed technology information, see [docs/tech-stack.md](docs/tech-stack.md).

## Quick Start

> 🚧 **Note**: The application is currently in initial development. Quick start instructions will be updated as the project progresses.

### Prerequisites
- Java 17 or higher
- Node.js 18 or higher
- Docker and Docker Compose
- Git

### Setup

```bash
# Clone the repository
git clone https://github.com/TaishiYamaguchi/task-habit-tracker.git
cd task-habit-tracker

# Copy environment variables
cp .env.example .env

# Start all services with Docker Compose
docker-compose up -d

# Access the application
# Frontend: http://localhost:3000
# Backend API: http://localhost:8080
# API Docs: http://localhost:8080/swagger-ui.html
```

## Project Structure

```
task-habit-tracker/
├── backend/                 # Spring Boot application (TBD)
├── frontend/                # React application (TBD)
├── docs/
│   ├── tech-stack.md        # Technology stack details
│   ├── architecture.md      # System architecture
│   ├── database-design.md   # Database schema design
│   ├── adr/                 # Architecture Decision Records
│   │   ├── 0000-template.md
│   │   └── 0001-tech-stack-selection.md
│   └── ja/                  # Japanese documentation
│       ├── project-background.md
│       ├── development-guideline.md
│       └── features-roadmap.md
├── .github/
│   └── PULL_REQUEST_TEMPLATE.md
├── .env.example             # Environment variable template
├── .gitignore
└── README.md
```

## Documentation

### English
- [Technology Stack](docs/tech-stack.md) - Comprehensive technology choices and rationale
- [Architecture](docs/architecture.md) - System design and component structure
- [Database Design](docs/database-design.md) - Schema design and data model
- [ADR: Tech Stack Selection](docs/adr/0001-tech-stack-selection.md) - Technology decision record

### 日本語 (Japanese)
- [プロジェクト背景](docs/ja/project-background.md) - 開発動機と学習目標
- [開発ガイドライン](docs/ja/development-guideline.md) - コーディング規約と品質基準
- [機能ロードマップ](docs/ja/features-roadmap.md) - フェーズ別開発計画

## Development

### Code Quality
- Follows [Conventional Commits](https://www.conventionalcommits.org/) for commit messages
- Uses feature branch workflow (main → develop → feature/*)
- Includes comprehensive testing (unit, integration)
- Maintains up-to-date documentation

### Contributing
1. Create a feature branch from `develop`
2. Make your changes
3. Ensure all tests pass
4. Submit a Pull Request using the PR template
5. Address review feedback

See [Development Guidelines](docs/ja/development-guideline.md) for detailed coding standards.

## License

This project is for educational purposes.

## Acknowledgments

This project was created as a practical learning exercise for modern web development best practices.