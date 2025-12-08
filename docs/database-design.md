# データベース設計 / Database Design

このドキュメントでは、Task Habit Trackerのデータベース設計について説明します。

This document describes the database design for the Task Habit Tracker application.

## 📋 目次 / Table of Contents

- [概要 / Overview](#概要--overview)
- [Phase 1: MVP スキーマ](#phase-1-mvp-スキーマ)
- [Phase 2: 拡張スキーマ](#phase-2-拡張スキーマ)
- [ER図 / ER Diagram](#er図--er-diagram)
- [正規化と設計方針](#正規化と設計方針)
- [インデックス戦略](#インデックス戦略)
- [マイグレーション戦略](#マイグレーション戦略)

---

## 🎯 概要 / Overview

### データベース管理システム / Database Management System

**PostgreSQL 15+**

**選定理由 / Reasons for Selection**:
- ACID準拠の信頼性 / ACID compliance for reliability
- 豊富なデータ型サポート / Rich data type support
- 優れたパフォーマンス / Excellent performance
- JSON型のサポート（将来の拡張用） / JSON type support for future extensions
- 無償で商用利用可能 / Free for commercial use

### 設計原則 / Design Principles

1. **正規化** / Normalization
   - 第3正規形（3NF）を基本とする
   - パフォーマンスが必要な箇所のみ非正規化を検討

2. **拡張性** / Extensibility
   - 将来の機能追加を考慮した柔軟な設計
   - マイグレーションを前提とした段階的な構築

3. **パフォーマンス** / Performance
   - 適切なインデックスの設定
   - クエリパフォーマンスの最適化

4. **データ整合性** / Data Integrity
   - 外部キー制約による参照整合性
   - NOT NULL制約による必須項目の保証
   - CHECK制約によるデータ検証

---

## 📊 Phase 1: MVP スキーマ

### users テーブル

**目的 / Purpose**: ユーザー情報の管理 / Manage user information

```sql
CREATE TABLE users (
    id BIGSERIAL PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    username VARCHAR(100) NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- インデックス / Indexes
CREATE UNIQUE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_created_at ON users(created_at);
```

**カラム説明 / Column Descriptions**:

| カラム名 / Column | 型 / Type | 制約 / Constraints | 説明 / Description |
|------------------|-----------|-------------------|-------------------|
| id | BIGSERIAL | PRIMARY KEY | ユーザーの一意識別子 / Unique user identifier |
| email | VARCHAR(255) | UNIQUE, NOT NULL | ログイン用メールアドレス / Email for login |
| username | VARCHAR(100) | NOT NULL | 表示名 / Display name |
| password_hash | VARCHAR(255) | NOT NULL | BCryptハッシュ化パスワード / BCrypt hashed password |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | 作成日時 / Creation timestamp |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | 更新日時 / Last update timestamp |

### tasks テーブル

**目的 / Purpose**: タスク情報の管理 / Manage task information

```sql
CREATE TABLE tasks (
    id BIGSERIAL PRIMARY KEY,
    user_id BIGINT NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    title VARCHAR(200) NOT NULL,
    description TEXT,
    due_date DATE,
    priority VARCHAR(20) DEFAULT 'MEDIUM',
    completed BOOLEAN DEFAULT FALSE,
    completed_at TIMESTAMP,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    CONSTRAINT chk_priority CHECK (priority IN ('LOW', 'MEDIUM', 'HIGH'))
);

-- インデックス / Indexes
CREATE INDEX idx_tasks_user_id ON tasks(user_id);
CREATE INDEX idx_tasks_due_date ON tasks(due_date);
CREATE INDEX idx_tasks_completed ON tasks(completed);
CREATE INDEX idx_tasks_user_completed ON tasks(user_id, completed);
CREATE INDEX idx_tasks_user_due_date ON tasks(user_id, due_date);
```

**カラム説明 / Column Descriptions**:

| カラム名 / Column | 型 / Type | 制約 / Constraints | 説明 / Description |
|------------------|-----------|-------------------|-------------------|
| id | BIGSERIAL | PRIMARY KEY | タスクの一意識別子 / Unique task identifier |
| user_id | BIGINT | FK, NOT NULL | タスクの所有者 / Task owner |
| title | VARCHAR(200) | NOT NULL | タスクのタイトル / Task title |
| description | TEXT | NULL | タスクの詳細説明 / Detailed description |
| due_date | DATE | NULL | 期日 / Due date |
| priority | VARCHAR(20) | DEFAULT 'MEDIUM' | 優先度（LOW, MEDIUM, HIGH） / Priority level |
| completed | BOOLEAN | DEFAULT FALSE | 完了フラグ / Completion flag |
| completed_at | TIMESTAMP | NULL | 完了日時 / Completion timestamp |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | 作成日時 / Creation timestamp |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | 更新日時 / Last update timestamp |

### サンプルデータ / Sample Data

```sql
-- ユーザーの作成 / Create users
INSERT INTO users (email, username, password_hash) VALUES
('john@example.com', 'John Doe', '$2a$10$...'),
('jane@example.com', 'Jane Smith', '$2a$10$...');

-- タスクの作成 / Create tasks
INSERT INTO tasks (user_id, title, description, due_date, priority, completed) VALUES
(1, 'プロジェクト資料を作成', 'Phase 1の完了報告資料', '2024-12-15', 'HIGH', false),
(1, '週次ミーティング準備', 'アジェンダと資料の準備', '2024-12-10', 'MEDIUM', false),
(1, '買い物リスト作成', '週末の買い物', '2024-12-08', 'LOW', true);
```

---

## 🚀 Phase 2: 拡張スキーマ

### habits テーブル

**目的 / Purpose**: 習慣情報の管理 / Manage habit information

```sql
CREATE TABLE habits (
    id BIGSERIAL PRIMARY KEY,
    user_id BIGINT NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    title VARCHAR(200) NOT NULL,
    description TEXT,
    frequency VARCHAR(20) DEFAULT 'DAILY',
    target_days TEXT,
    icon VARCHAR(50),
    color VARCHAR(7),
    archived BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    CONSTRAINT chk_frequency CHECK (frequency IN ('DAILY', 'WEEKLY', 'CUSTOM'))
);

-- インデックス / Indexes
CREATE INDEX idx_habits_user_id ON habits(user_id);
CREATE INDEX idx_habits_archived ON habits(archived);
CREATE INDEX idx_habits_user_archived ON habits(user_id, archived);
```

**カラム説明 / Column Descriptions**:

| カラム名 / Column | 型 / Type | 制約 / Constraints | 説明 / Description |
|------------------|-----------|-------------------|-------------------|
| id | BIGSERIAL | PRIMARY KEY | 習慣の一意識別子 / Unique habit identifier |
| user_id | BIGINT | FK, NOT NULL | 習慣の所有者 / Habit owner |
| title | VARCHAR(200) | NOT NULL | 習慣のタイトル / Habit title |
| description | TEXT | NULL | 習慣の説明 / Habit description |
| frequency | VARCHAR(20) | DEFAULT 'DAILY' | 頻度（DAILY, WEEKLY, CUSTOM） / Frequency |
| target_days | TEXT | NULL | 曜日指定（例: "1,3,5"） / Target weekdays |
| icon | VARCHAR(50) | NULL | アイコン名 / Icon name |
| color | VARCHAR(7) | NULL | カラーコード（例: #FF5733） / Color code |
| archived | BOOLEAN | DEFAULT FALSE | アーカイブフラグ / Archive flag |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | 作成日時 / Creation timestamp |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | 更新日時 / Last update timestamp |

### habit_logs テーブル

**目的 / Purpose**: 習慣の達成記録 / Track habit completion

```sql
CREATE TABLE habit_logs (
    id BIGSERIAL PRIMARY KEY,
    habit_id BIGINT NOT NULL REFERENCES habits(id) ON DELETE CASCADE,
    log_date DATE NOT NULL,
    completed BOOLEAN DEFAULT FALSE,
    note TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    UNIQUE(habit_id, log_date)
);

-- インデックス / Indexes
CREATE INDEX idx_habit_logs_habit_id ON habit_logs(habit_id);
CREATE INDEX idx_habit_logs_date ON habit_logs(log_date);
CREATE INDEX idx_habit_logs_completed ON habit_logs(completed);
CREATE INDEX idx_habit_logs_habit_date ON habit_logs(habit_id, log_date);
```

**カラム説明 / Column Descriptions**:

| カラム名 / Column | 型 / Type | 制約 / Constraints | 説明 / Description |
|------------------|-----------|-------------------|-------------------|
| id | BIGSERIAL | PRIMARY KEY | ログの一意識別子 / Unique log identifier |
| habit_id | BIGINT | FK, NOT NULL | 対象の習慣 / Target habit |
| log_date | DATE | NOT NULL | 記録日 / Log date |
| completed | BOOLEAN | DEFAULT FALSE | 達成フラグ / Completion flag |
| note | TEXT | NULL | メモ / Note |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | 作成日時 / Creation timestamp |

### categories テーブル

**目的 / Purpose**: タスクのカテゴリ管理 / Manage task categories

```sql
CREATE TABLE categories (
    id BIGSERIAL PRIMARY KEY,
    user_id BIGINT NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    name VARCHAR(100) NOT NULL,
    color VARCHAR(7),
    icon VARCHAR(50),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    UNIQUE(user_id, name)
);

-- インデックス / Indexes
CREATE INDEX idx_categories_user_id ON categories(user_id);
```

**カラム説明 / Column Descriptions**:

| カラム名 / Column | 型 / Type | 制約 / Constraints | 説明 / Description |
|------------------|-----------|-------------------|-------------------|
| id | BIGSERIAL | PRIMARY KEY | カテゴリの一意識別子 / Unique category identifier |
| user_id | BIGINT | FK, NOT NULL | カテゴリの所有者 / Category owner |
| name | VARCHAR(100) | NOT NULL | カテゴリ名 / Category name |
| color | VARCHAR(7) | NULL | カラーコード / Color code |
| icon | VARCHAR(50) | NULL | アイコン名 / Icon name |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | 作成日時 / Creation timestamp |

### tags テーブル

**目的 / Purpose**: タスクのタグ管理 / Manage task tags

```sql
CREATE TABLE tags (
    id BIGSERIAL PRIMARY KEY,
    user_id BIGINT NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    name VARCHAR(50) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    UNIQUE(user_id, name)
);

-- インデックス / Indexes
CREATE INDEX idx_tags_user_id ON tags(user_id);
CREATE INDEX idx_tags_name ON tags(name);
```

### task_tags テーブル（中間テーブル）

**目的 / Purpose**: タスクとタグの多対多関係 / Many-to-many relationship between tasks and tags

```sql
CREATE TABLE task_tags (
    task_id BIGINT NOT NULL REFERENCES tasks(id) ON DELETE CASCADE,
    tag_id BIGINT NOT NULL REFERENCES tags(id) ON DELETE CASCADE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    PRIMARY KEY (task_id, tag_id)
);

-- インデックス / Indexes
CREATE INDEX idx_task_tags_task_id ON task_tags(task_id);
CREATE INDEX idx_task_tags_tag_id ON task_tags(tag_id);
```

### Phase 2 でのtasksテーブルの拡張

```sql
-- tasks テーブルにカテゴリカラムを追加
ALTER TABLE tasks ADD COLUMN category_id BIGINT REFERENCES categories(id) ON DELETE SET NULL;
CREATE INDEX idx_tasks_category_id ON tasks(category_id);
```

---

## 📐 ER図 / ER Diagram

### Phase 1: MVP

```
┌─────────────────┐
│     users       │
├─────────────────┤
│ * id (PK)       │
│   email         │
│   username      │
│   password_hash │
│   created_at    │
│   updated_at    │
└─────────────────┘
         │
         │ 1
         │
         │ N
         ↓
┌─────────────────┐
│     tasks       │
├─────────────────┤
│ * id (PK)       │
│ * user_id (FK)  │
│   title         │
│   description   │
│   due_date      │
│   priority      │
│   completed     │
│   completed_at  │
│   created_at    │
│   updated_at    │
└─────────────────┘
```

### Phase 2: 拡張版

```
                    ┌─────────────────┐
                    │     users       │
                    ├─────────────────┤
                    │ * id (PK)       │
                    │   email         │
                    │   username      │
                    │   password_hash │
                    │   created_at    │
                    │   updated_at    │
                    └─────────────────┘
                    ↙       ↓        ↘
                   1        1         1
                  ↙         ↓          ↘
                N          N            N
    ┌──────────────┐ ┌─────────────┐ ┌──────────────┐
    │   habits     │ │   tasks     │ │ categories   │
    ├──────────────┤ ├─────────────┤ ├──────────────┤
    │ * id (PK)    │ │ * id (PK)   │ │ * id (PK)    │
    │ * user_id(FK)│ │ *user_id(FK)│ │ * user_id(FK)│
    │   title      │ │   title     │ │   name       │
    │   frequency  │ │   ...       │ │   color      │
    │   ...        │ │   ...       │ │   icon       │
    └──────────────┘ └─────────────┘ └──────────────┘
         │                  │ N              ↑
         │ 1                │                │ 1
         │                  │ N              │
         │ N                ↓                │
         ↓          ┌──────────────┐         │
    ┌──────────────┐│  task_tags   │         │
    │ habit_logs   ││              │         │
    ├──────────────┤│ * task_id(FK)│         │
    │ * id (PK)    ││ * tag_id (FK)│    (category_id)
    │ *habit_id(FK)│└──────────────┘
    │   log_date   │         ↑
    │   completed  │         │ N
    │   note       │         │
    └──────────────┘         │ 1
                             │
                    ┌──────────────┐
                    │     tags     │
                    ├──────────────┤
                    │ * id (PK)    │
                    │ * user_id(FK)│
                    │   name       │
                    └──────────────┘
```

---

## 🎯 正規化と設計方針

### 正規化レベル / Normalization Level

**第3正規形（3NF）を達成**:

1. **第1正規形（1NF）**
   - すべてのカラムがアトミック（分割不可能）な値を持つ
   - 繰り返しグループがない

2. **第2正規形（2NF）**
   - 1NFを満たし、非キー属性が主キー全体に完全関数従属
   - 部分関数従属がない

3. **第3正規形（3NF）**
   - 2NFを満たし、非キー属性が他の非キー属性に推移的に従属しない

### 正規化の例 / Normalization Example

**非正規化の例（悪い設計）**:
```sql
-- アンチパターン: タスクテーブルにユーザー名を含める
CREATE TABLE tasks_bad (
    id BIGSERIAL PRIMARY KEY,
    user_email VARCHAR(255),
    user_name VARCHAR(100),  -- 冗長: ユーザー情報の重複
    title VARCHAR(200),
    ...
);
```

**正規化された設計（良い設計）**:
```sql
-- usersテーブルとtasksテーブルを分離
-- tasksテーブルはuser_idで参照
```

### 意図的な非正規化

**パフォーマンス最適化のための非正規化**（将来的に検討）:

```sql
-- 統計情報をキャッシュするテーブル
CREATE TABLE user_statistics (
    user_id BIGINT PRIMARY KEY REFERENCES users(id),
    total_tasks INTEGER DEFAULT 0,
    completed_tasks INTEGER DEFAULT 0,
    total_habits INTEGER DEFAULT 0,
    current_streak INTEGER DEFAULT 0,
    last_calculated_at TIMESTAMP,
    
    -- 非正規化: 頻繁にアクセスされる集計データをキャッシュ
    -- 定期的に再計算して同期を保つ
);
```

---

## 🚀 インデックス戦略

### インデックスの原則 / Index Principles

1. **WHERE句で頻繁に使用されるカラム**
2. **JOIN条件に使用されるカラム**
3. **ORDER BY句で使用されるカラム**
4. **外部キー**

### 主要なクエリとインデックス / Key Queries and Indexes

**クエリ1: ユーザーの未完了タスクを取得**
```sql
SELECT * FROM tasks 
WHERE user_id = ? AND completed = false 
ORDER BY due_date ASC;

-- 対応インデックス
CREATE INDEX idx_tasks_user_completed ON tasks(user_id, completed);
CREATE INDEX idx_tasks_due_date ON tasks(due_date);
```

**クエリ2: 特定期間の習慣ログを取得**
```sql
SELECT * FROM habit_logs 
WHERE habit_id = ? AND log_date BETWEEN ? AND ?;

-- 対応インデックス
CREATE INDEX idx_habit_logs_habit_date ON habit_logs(habit_id, log_date);
```

**クエリ3: カテゴリでタスクをフィルタ**
```sql
SELECT * FROM tasks 
WHERE user_id = ? AND category_id = ? AND completed = false;

-- 対応インデックス
CREATE INDEX idx_tasks_user_category_completed ON tasks(user_id, category_id, completed);
```

### インデックスのメンテナンス / Index Maintenance

```sql
-- インデックスの使用状況を確認
SELECT 
    schemaname,
    tablename,
    indexname,
    idx_scan,
    idx_tup_read,
    idx_tup_fetch
FROM pg_stat_user_indexes
ORDER BY idx_scan;

-- 未使用のインデックスを削除
-- (定期的にモニタリングして判断)
```

---

## 🔄 マイグレーション戦略

### マイグレーションツール

**Flyway または Liquibase** を使用（推奨: Flyway）

### バージョン管理

```
db/migration/
├── V1__create_users_table.sql
├── V2__create_tasks_table.sql
├── V3__add_indexes_to_tasks.sql
├── V4__create_habits_table.sql
├── V5__create_habit_logs_table.sql
└── V6__create_categories_and_tags.sql
```

### マイグレーションの例

**V1__create_users_table.sql**:
```sql
CREATE TABLE users (
    id BIGSERIAL PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    username VARCHAR(100) NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE UNIQUE INDEX idx_users_email ON users(email);
```

**V2__create_tasks_table.sql**:
```sql
CREATE TABLE tasks (
    id BIGSERIAL PRIMARY KEY,
    user_id BIGINT NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    title VARCHAR(200) NOT NULL,
    description TEXT,
    due_date DATE,
    priority VARCHAR(20) DEFAULT 'MEDIUM',
    completed BOOLEAN DEFAULT FALSE,
    completed_at TIMESTAMP,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    CONSTRAINT chk_priority CHECK (priority IN ('LOW', 'MEDIUM', 'HIGH'))
);

CREATE INDEX idx_tasks_user_id ON tasks(user_id);
CREATE INDEX idx_tasks_due_date ON tasks(due_date);
CREATE INDEX idx_tasks_completed ON tasks(completed);
```

### ロールバック戦略

```sql
-- マイグレーションのロールバック用スクリプト
-- U1__rollback_create_users_table.sql
DROP TABLE IF EXISTS users CASCADE;

-- U2__rollback_create_tasks_table.sql
DROP TABLE IF EXISTS tasks CASCADE;
```

---

## 📈 将来の拡張計画

### Phase 3: 高度な機能

**teams テーブル**（チーム機能用）:
```sql
CREATE TABLE teams (
    id BIGSERIAL PRIMARY KEY,
    name VARCHAR(200) NOT NULL,
    description TEXT,
    created_by BIGINT REFERENCES users(id),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE team_members (
    team_id BIGINT REFERENCES teams(id),
    user_id BIGINT REFERENCES users(id),
    role VARCHAR(20),
    joined_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (team_id, user_id)
);
```

**attachments テーブル**（ファイル添付用）:
```sql
CREATE TABLE attachments (
    id BIGSERIAL PRIMARY KEY,
    task_id BIGINT REFERENCES tasks(id) ON DELETE CASCADE,
    file_name VARCHAR(255) NOT NULL,
    file_size BIGINT,
    file_type VARCHAR(100),
    storage_url TEXT NOT NULL,
    uploaded_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

---

このデータベース設計は、プロジェクトの進化に応じて更新されます。

This database design will be updated as the project evolves.
