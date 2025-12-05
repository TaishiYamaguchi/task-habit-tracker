# Database Design

This document describes the database schema design for the Task & Habit Tracker application, including entity relationships, normalization approach, and future expansion considerations.

## Overview

The database follows a **normalized relational design** using PostgreSQL. The schema is designed to:
- Support the MVP features (user management, task tracking)
- Allow easy expansion for future features (habits, categories, teams)
- Maintain data integrity through constraints and foreign keys
- Enable efficient querying for common use cases

---

## Entity Relationship Diagram

### Phase 1: MVP Schema

```
┌─────────────────────────────────────────────────────────────────┐
│                         Users                                    │
├─────────────────────────────────────────────────────────────────┤
│ PK │ id              │ BIGSERIAL (auto-increment)               │
│    │ username        │ VARCHAR(50) NOT NULL UNIQUE               │
│    │ email           │ VARCHAR(255) NOT NULL UNIQUE              │
│    │ password_hash   │ VARCHAR(255) NOT NULL                     │
│    │ created_at      │ TIMESTAMP NOT NULL DEFAULT CURRENT        │
│    │ updated_at      │ TIMESTAMP NOT NULL DEFAULT CURRENT        │
└─────────────────────────────────────────────────────────────────┘
                              │
                              │ 1:N
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                         Tasks                                    │
├─────────────────────────────────────────────────────────────────┤
│ PK │ id              │ BIGSERIAL (auto-increment)               │
│ FK │ user_id         │ BIGINT NOT NULL → Users.id               │
│    │ title           │ VARCHAR(255) NOT NULL                     │
│    │ description     │ TEXT                                      │
│    │ status          │ VARCHAR(20) NOT NULL DEFAULT 'TODO'       │
│    │ due_date        │ DATE                                      │
│    │ completed_at    │ TIMESTAMP                                 │
│    │ created_at      │ TIMESTAMP NOT NULL DEFAULT CURRENT        │
│    │ updated_at      │ TIMESTAMP NOT NULL DEFAULT CURRENT        │
└─────────────────────────────────────────────────────────────────┘
```

---

## Table Definitions

### Users Table

The `users` table stores user account information.

```sql
CREATE TABLE users (
    id              BIGSERIAL PRIMARY KEY,
    username        VARCHAR(50) NOT NULL,
    email           VARCHAR(255) NOT NULL,
    password_hash   VARCHAR(255) NOT NULL,
    created_at      TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at      TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    
    CONSTRAINT uk_users_username UNIQUE (username),
    CONSTRAINT uk_users_email UNIQUE (email)
);

-- Index for login queries
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_username ON users(username);
```

#### Column Details

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `id` | BIGSERIAL | PRIMARY KEY | Auto-incrementing unique identifier |
| `username` | VARCHAR(50) | NOT NULL, UNIQUE | Display name for the user |
| `email` | VARCHAR(255) | NOT NULL, UNIQUE | User's email address for login |
| `password_hash` | VARCHAR(255) | NOT NULL | BCrypt hashed password |
| `created_at` | TIMESTAMP | NOT NULL, DEFAULT | Account creation timestamp |
| `updated_at` | TIMESTAMP | NOT NULL, DEFAULT | Last modification timestamp |

#### Design Decisions

- **BIGSERIAL for IDs**: Allows for large-scale growth
- **Separate username and email**: Supports login via email while having a display name
- **Password hashing**: BCrypt with cost factor for security
- **Timestamps**: Audit trail for all records

---

### Tasks Table

The `tasks` table stores user tasks with status tracking.

```sql
CREATE TABLE tasks (
    id              BIGSERIAL PRIMARY KEY,
    user_id         BIGINT NOT NULL,
    title           VARCHAR(255) NOT NULL,
    description     TEXT,
    status          VARCHAR(20) NOT NULL DEFAULT 'TODO',
    due_date        DATE,
    completed_at    TIMESTAMP,
    created_at      TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at      TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    
    CONSTRAINT fk_tasks_user FOREIGN KEY (user_id) 
        REFERENCES users(id) ON DELETE CASCADE,
    CONSTRAINT chk_tasks_status CHECK (status IN ('TODO', 'IN_PROGRESS', 'DONE'))
);

-- Indexes for common queries
CREATE INDEX idx_tasks_user_id ON tasks(user_id);
CREATE INDEX idx_tasks_user_status ON tasks(user_id, status);
CREATE INDEX idx_tasks_user_due_date ON tasks(user_id, due_date);
CREATE INDEX idx_tasks_user_created ON tasks(user_id, created_at DESC);
```

#### Column Details

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `id` | BIGSERIAL | PRIMARY KEY | Auto-incrementing unique identifier |
| `user_id` | BIGINT | NOT NULL, FK | Reference to owning user |
| `title` | VARCHAR(255) | NOT NULL | Task title |
| `description` | TEXT | NULLABLE | Optional detailed description |
| `status` | VARCHAR(20) | NOT NULL, CHECK | Task status (TODO, IN_PROGRESS, DONE) |
| `due_date` | DATE | NULLABLE | Optional due date |
| `completed_at` | TIMESTAMP | NULLABLE | When task was completed |
| `created_at` | TIMESTAMP | NOT NULL, DEFAULT | Task creation timestamp |
| `updated_at` | TIMESTAMP | NOT NULL, DEFAULT | Last modification timestamp |

#### Status Values

| Status | Description |
|--------|-------------|
| `TODO` | Task is pending |
| `IN_PROGRESS` | Task is being worked on |
| `DONE` | Task is completed |

#### Design Decisions

- **ON DELETE CASCADE**: When a user is deleted, their tasks are also deleted
- **VARCHAR for status**: Enum-like values with CHECK constraint for data integrity
- **Separate completed_at**: Tracks when task was marked done, distinct from updated_at
- **Composite indexes**: Optimized for filtering by user + status/date

---

## Normalization Approach

The schema follows **Third Normal Form (3NF)**:

### 1NF (First Normal Form)
✅ All columns contain atomic (indivisible) values
✅ Each row is unique (primary keys)
✅ No repeating groups

### 2NF (Second Normal Form)
✅ Meets 1NF requirements
✅ No partial dependencies (all non-key columns depend on entire primary key)

### 3NF (Third Normal Form)
✅ Meets 2NF requirements
✅ No transitive dependencies (non-key columns don't depend on other non-key columns)

---

## Future Schema Expansion

### Phase 2: Habits

```sql
-- Habits definition table
CREATE TABLE habits (
    id              BIGSERIAL PRIMARY KEY,
    user_id         BIGINT NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    name            VARCHAR(255) NOT NULL,
    description     TEXT,
    frequency       VARCHAR(20) NOT NULL DEFAULT 'DAILY',
    target_count    INT NOT NULL DEFAULT 1,
    color           VARCHAR(7),  -- Hex color for UI
    is_active       BOOLEAN NOT NULL DEFAULT TRUE,
    created_at      TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at      TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

-- Daily habit completion records
CREATE TABLE habit_records (
    id              BIGSERIAL PRIMARY KEY,
    habit_id        BIGINT NOT NULL REFERENCES habits(id) ON DELETE CASCADE,
    record_date     DATE NOT NULL,
    count           INT NOT NULL DEFAULT 1,
    notes           TEXT,
    created_at      TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    
    CONSTRAINT uk_habit_records UNIQUE (habit_id, record_date)
);

CREATE INDEX idx_habits_user_id ON habits(user_id);
CREATE INDEX idx_habit_records_habit_date ON habit_records(habit_id, record_date);
```

### Phase 2: Categories/Tags

```sql
-- Categories table
CREATE TABLE categories (
    id              BIGSERIAL PRIMARY KEY,
    user_id         BIGINT NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    name            VARCHAR(100) NOT NULL,
    color           VARCHAR(7),
    created_at      TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    
    CONSTRAINT uk_categories_user_name UNIQUE (user_id, name)
);

-- Many-to-many relationship for tasks
CREATE TABLE task_categories (
    task_id         BIGINT NOT NULL REFERENCES tasks(id) ON DELETE CASCADE,
    category_id     BIGINT NOT NULL REFERENCES categories(id) ON DELETE CASCADE,
    
    PRIMARY KEY (task_id, category_id)
);

CREATE INDEX idx_categories_user_id ON categories(user_id);
CREATE INDEX idx_task_categories_task ON task_categories(task_id);
CREATE INDEX idx_task_categories_category ON task_categories(category_id);
```

### Phase 3: Team Features

```sql
-- Teams table
CREATE TABLE teams (
    id              BIGSERIAL PRIMARY KEY,
    name            VARCHAR(255) NOT NULL,
    description     TEXT,
    owner_id        BIGINT NOT NULL REFERENCES users(id),
    created_at      TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at      TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

-- Team membership
CREATE TABLE team_members (
    team_id         BIGINT NOT NULL REFERENCES teams(id) ON DELETE CASCADE,
    user_id         BIGINT NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    role            VARCHAR(20) NOT NULL DEFAULT 'MEMBER',
    joined_at       TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    
    PRIMARY KEY (team_id, user_id),
    CONSTRAINT chk_team_members_role CHECK (role IN ('OWNER', 'ADMIN', 'MEMBER'))
);

-- Modify tasks to support team ownership (migration)
ALTER TABLE tasks ADD COLUMN team_id BIGINT REFERENCES teams(id);
CREATE INDEX idx_tasks_team_id ON tasks(team_id);
```

### Phase 3: File Attachments

```sql
-- Attachments table
CREATE TABLE attachments (
    id              BIGSERIAL PRIMARY KEY,
    task_id         BIGINT NOT NULL REFERENCES tasks(id) ON DELETE CASCADE,
    filename        VARCHAR(255) NOT NULL,
    file_path       VARCHAR(500) NOT NULL,
    file_size       BIGINT NOT NULL,
    mime_type       VARCHAR(100) NOT NULL,
    uploaded_by     BIGINT NOT NULL REFERENCES users(id),
    created_at      TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_attachments_task_id ON attachments(task_id);
```

---

## Database Migration Strategy

### Using Flyway

Migration files are stored in `src/main/resources/db/migration/`:

```
db/migration/
├── V1__create_users_table.sql
├── V2__create_tasks_table.sql
├── V3__add_habits_tables.sql
└── V4__add_categories_tables.sql
```

#### Naming Convention
- `V{version}__{description}.sql` for versioned migrations
- `R__{description}.sql` for repeatable migrations

### Migration Best Practices

1. **Never modify existing migrations** - Always create new ones
2. **Test migrations** - Run against copy of production data
3. **Backward compatible** - Support rolling deployments
4. **Small, focused changes** - One logical change per migration

---

## Query Patterns

### Common Queries

#### Get user's tasks for today
```sql
SELECT * FROM tasks 
WHERE user_id = :userId 
  AND (due_date = CURRENT_DATE OR (due_date IS NULL AND status != 'DONE'))
ORDER BY due_date NULLS LAST, created_at DESC;
```

#### Get task completion statistics
```sql
SELECT 
    COUNT(*) FILTER (WHERE status = 'DONE') as completed,
    COUNT(*) FILTER (WHERE status != 'DONE') as pending,
    COUNT(*) as total
FROM tasks 
WHERE user_id = :userId
  AND created_at >= :startDate;
```

#### Search tasks by title
```sql
SELECT * FROM tasks 
WHERE user_id = :userId 
  AND title ILIKE '%' || :searchTerm || '%'
ORDER BY created_at DESC;
```

---

## Performance Considerations

### Indexing Strategy

| Index | Purpose | Query Pattern |
|-------|---------|---------------|
| `idx_users_email` | Login lookup | WHERE email = ? |
| `idx_tasks_user_id` | User's tasks | WHERE user_id = ? |
| `idx_tasks_user_status` | Status filtering | WHERE user_id = ? AND status = ? |
| `idx_tasks_user_due_date` | Due date queries | WHERE user_id = ? AND due_date = ? |

### Future Optimizations

1. **Partitioning**: Consider date-based partitioning for habit_records
2. **Archiving**: Move old completed tasks to archive table
3. **Full-text search**: PostgreSQL full-text search for task search
4. **Materialized views**: For dashboard statistics

---

## Data Integrity Rules

### Business Rules Enforced by Database

| Rule | Implementation |
|------|----------------|
| User must have unique email | UNIQUE constraint |
| Task must belong to a user | NOT NULL + FK constraint |
| Valid task status values | CHECK constraint |
| Delete user cascades to tasks | ON DELETE CASCADE |

### Application-Level Validation

| Rule | Implementation |
|------|----------------|
| Email format validation | Jakarta Validation |
| Password strength | Service layer |
| Title length limits | DTO validation |

---

## Related Documentation

- [Architecture Overview](./architecture.md) - System architecture
- [Technology Stack](./tech-stack.md) - Database technology choices
- [Features Roadmap](./ja/features-roadmap.md) - Phased development plan
