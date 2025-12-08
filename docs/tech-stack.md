# 技術スタック詳細 / Tech Stack Details

このドキュメントでは、Task Habit Trackerプロジェクトで使用する技術スタックの詳細を説明します。

This document provides detailed information about the tech stack used in the Task Habit Tracker project.

## 📋 目次 / Table of Contents

- [バックエンド / Backend](#バックエンド--backend)
- [フロントエンド / Frontend](#フロントエンド--frontend)
- [データベース / Database](#データベース--database)
- [インフラ / Infrastructure](#インフラ--infrastructure)
- [開発ツール / Development Tools](#開発ツール--development-tools)
- [テスト / Testing](#テスト--testing)

---

## 🔧 バックエンド / Backend

### Java 17+

**選定理由 / Reasons for Selection**:
- エンタープライズ開発で広く使用されている / Widely used in enterprise development
- 型安全性が高く、大規模開発に適している / Strong type safety, suitable for large-scale development
- 豊富なライブラリとフレームワーク / Rich ecosystem of libraries and frameworks
- 長期サポート（LTS）バージョン / Long-term support (LTS) version

**バージョン / Version**: Java 17 (LTS)

**主な機能 / Key Features**:
- Records（データクラスの簡潔な記述）
- Pattern Matching
- Text Blocks
- Sealed Classes

### Spring Boot 3.x

**選定理由 / Reasons for Selection**:
- Javaで最も人気のあるWebフレームワーク / Most popular web framework for Java
- 設定より規約（Convention over Configuration）で迅速な開発 / Convention over Configuration for rapid development
- Spring Ecosystemの豊富な機能 / Rich Spring Ecosystem features
- マイクロサービスアーキテクチャへの対応 / Supports microservices architecture

**主要モジュール / Main Modules**:

#### Spring Web
- RESTful APIの構築 / Building RESTful APIs
- @RestController、@RequestMapping / REST controllers and request mapping
- JSONシリアライゼーション（Jackson） / JSON serialization with Jackson

```java
@RestController
@RequestMapping("/api/tasks")
public class TaskController {
    
    @GetMapping
    public ResponseEntity<List<Task>> getAllTasks() {
        // Implementation
    }
    
    @PostMapping
    public ResponseEntity<Task> createTask(@Valid @RequestBody TaskRequest request) {
        // Implementation
    }
}
```

#### Spring Data JPA
- データベースアクセスの抽象化 / Database access abstraction
- リポジトリパターンの実装 / Repository pattern implementation
- クエリの自動生成 / Automatic query generation

```java
@Repository
public interface TaskRepository extends JpaRepository<Task, Long> {
    List<Task> findByUserIdAndCompleted(Long userId, Boolean completed);
    
    @Query("SELECT t FROM Task t WHERE t.userId = :userId AND t.dueDate = :date")
    List<Task> findTasksDueOn(@Param("userId") Long userId, @Param("date") LocalDate date);
}
```

#### Spring Security
- 認証・認可の実装 / Authentication and authorization
- セキュリティ設定の集約 / Centralized security configuration
- JWT統合 / JWT integration

### Spring Security + JWT

**認証フロー / Authentication Flow**:
1. ユーザーがログイン情報を送信 / User submits login credentials
2. サーバーが認証を検証 / Server validates credentials
3. JWTトークンを生成して返却 / Generate and return JWT token
4. クライアントは以降のリクエストでトークンを送信 / Client sends token with subsequent requests
5. サーバーでトークンを検証 / Server validates token

**実装例 / Implementation Example**:
```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {
    
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .csrf().disable()
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/auth/**").permitAll()
                .anyRequest().authenticated()
            )
            .sessionManagement(session -> session
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS)
            )
            .addFilterBefore(jwtAuthFilter, UsernamePasswordAuthenticationFilter.class);
        
        return http.build();
    }
}
```

**セキュリティ対策 / Security Measures**:
- BCryptによるパスワードハッシュ化 / Password hashing with BCrypt
- JWTトークンの有効期限管理 / JWT token expiration management
- CORS設定 / CORS configuration
- CSRF対策 / CSRF protection

### PostgreSQL

**選定理由 / Reasons for Selection**:
- オープンソースで強力なRDBMS / Open-source powerful RDBMS
- ACID特性の保証 / ACID compliance
- 豊富なデータ型とインデックス / Rich data types and indexing options
- JSONサポート（将来の拡張用） / JSON support for future extensions

**バージョン / Version**: PostgreSQL 15+

**使用する機能 / Features Used**:
- トランザクション管理 / Transaction management
- 外部キー制約 / Foreign key constraints
- インデックス最適化 / Index optimization
- フルテキスト検索（Phase 2以降） / Full-text search (Phase 2+)

### Maven

**ビルドツール / Build Tool**:
- 依存関係管理 / Dependency management
- ビルドライフサイクル / Build lifecycle
- プラグインエコシステム / Plugin ecosystem

**主要な依存関係 / Key Dependencies**:
```xml
<dependencies>
    <!-- Spring Boot Starter -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    
    <!-- Spring Security -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-security</artifactId>
    </dependency>
    
    <!-- Spring Data JPA -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
    
    <!-- PostgreSQL Driver -->
    <dependency>
        <groupId>org.postgresql</groupId>
        <artifactId>postgresql</artifactId>
    </dependency>
    
    <!-- JWT -->
    <dependency>
        <groupId>io.jsonwebtoken</groupId>
        <artifactId>jjwt-api</artifactId>
    </dependency>
</dependencies>
```

---

## 🎨 フロントエンド / Frontend

### React 18+

**選定理由 / Reasons for Selection**:
- コンポーネントベースの開発 / Component-based development
- 豊富なエコシステム / Rich ecosystem
- パフォーマンス最適化（Virtual DOM） / Performance optimization with Virtual DOM
- 大規模なコミュニティサポート / Large community support

**主要機能 / Key Features**:
- Hooks（useState, useEffect, useContext など）
- Concurrent Features
- Suspense（データフェッチング）
- Error Boundaries

**コンポーネント例 / Component Example**:
```typescript
import React, { useState, useEffect } from 'react';

interface Task {
  id: number;
  title: string;
  completed: boolean;
}

const TaskList: React.FC = () => {
  const [tasks, setTasks] = useState<Task[]>([]);
  
  useEffect(() => {
    fetchTasks();
  }, []);
  
  const fetchTasks = async () => {
    const response = await fetch('/api/tasks');
    const data = await response.json();
    setTasks(data);
  };
  
  return (
    <div className="task-list">
      {tasks.map(task => (
        <TaskItem key={task.id} task={task} />
      ))}
    </div>
  );
};
```

### TypeScript

**選定理由 / Reasons for Selection**:
- 型安全性によるバグの早期発見 / Type safety for early bug detection
- IDEの補完機能向上 / Better IDE autocomplete
- リファクタリングの容易さ / Easier refactoring
- 大規模開発での保守性向上 / Better maintainability in large projects

**型定義例 / Type Definition Example**:
```typescript
// API Response Types
interface ApiResponse<T> {
  data: T;
  message?: string;
  error?: string;
}

interface Task {
  id: number;
  title: string;
  description?: string;
  dueDate?: string;
  priority: 'LOW' | 'MEDIUM' | 'HIGH';
  completed: boolean;
  createdAt: string;
  updatedAt: string;
}

interface CreateTaskRequest {
  title: string;
  description?: string;
  dueDate?: string;
  priority: 'LOW' | 'MEDIUM' | 'HIGH';
}
```

### Tailwind CSS

**選定理由 / Reasons for Selection**:
- ユーティリティファーストのアプローチ / Utility-first approach
- カスタマイズ性が高い / Highly customizable
- バンドルサイズの最適化（未使用CSSの削除） / Optimized bundle size with purging
- レスポンシブデザインが簡単 / Easy responsive design

**使用例 / Usage Example**:
```tsx
<div className="container mx-auto px-4">
  <div className="bg-white rounded-lg shadow-md p-6 hover:shadow-lg transition-shadow">
    <h2 className="text-2xl font-bold text-gray-800 mb-4">
      タスク一覧
    </h2>
    <button className="bg-blue-500 hover:bg-blue-600 text-white font-medium py-2 px-4 rounded">
      新規作成
    </button>
  </div>
</div>
```

**テーマ設定 / Theme Configuration**:
```javascript
// tailwind.config.js
module.exports = {
  theme: {
    extend: {
      colors: {
        primary: {
          50: '#f0f9ff',
          500: '#3b82f6',
          900: '#1e3a8a',
        },
      },
    },
  },
};
```

### 状態管理 / State Management

**検討中のオプション / Options Under Consideration**:

1. **React Context API**（シンプルな状態管理用）
   - 軽量で学習コストが低い / Lightweight with low learning curve
   - 小〜中規模アプリに適している / Suitable for small to medium apps

2. **Zustand**（推奨）
   - シンプルで使いやすい / Simple and easy to use
   - Boilerplateが少ない / Minimal boilerplate
   - TypeScript対応 / TypeScript support

```typescript
// Zustand ストアの例
import create from 'zustand';

interface TaskStore {
  tasks: Task[];
  fetchTasks: () => Promise<void>;
  addTask: (task: Task) => void;
  updateTask: (id: number, updates: Partial<Task>) => void;
  deleteTask: (id: number) => void;
}

const useTaskStore = create<TaskStore>((set) => ({
  tasks: [],
  fetchTasks: async () => {
    const response = await fetch('/api/tasks');
    const data = await response.json();
    set({ tasks: data });
  },
  addTask: (task) => set((state) => ({ tasks: [...state.tasks, task] })),
  updateTask: (id, updates) => set((state) => ({
    tasks: state.tasks.map(t => t.id === id ? { ...t, ...updates } : t)
  })),
  deleteTask: (id) => set((state) => ({
    tasks: state.tasks.filter(t => t.id !== id)
  })),
}));
```

### ルーティング / Routing

**React Router v6**:
- クライアントサイドルーティング / Client-side routing
- ネストされたルート / Nested routes
- 保護されたルート / Protected routes

```tsx
import { BrowserRouter, Routes, Route, Navigate } from 'react-router-dom';

function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/login" element={<LoginPage />} />
        <Route path="/register" element={<RegisterPage />} />
        <Route path="/" element={<PrivateRoute><Dashboard /></PrivateRoute>}>
          <Route index element={<TaskList />} />
          <Route path="tasks/:id" element={<TaskDetail />} />
          <Route path="habits" element={<HabitList />} />
        </Route>
      </Routes>
    </BrowserRouter>
  );
}
```

---

## 🗄️ データベース / Database

### PostgreSQL 15

**データ型の活用 / Data Types Used**:
- INTEGER, BIGINT: 数値データ
- VARCHAR, TEXT: 文字列データ
- TIMESTAMP: 日時データ
- BOOLEAN: 真偽値
- JSON/JSONB: 柔軟なデータ構造（将来の拡張用）

**パフォーマンス最適化 / Performance Optimization**:
```sql
-- インデックスの作成
CREATE INDEX idx_tasks_user_id ON tasks(user_id);
CREATE INDEX idx_tasks_due_date ON tasks(due_date);
CREATE INDEX idx_tasks_completed ON tasks(completed);
CREATE INDEX idx_habit_logs_date ON habit_logs(habit_id, log_date);

-- 複合インデックス
CREATE INDEX idx_tasks_user_completed ON tasks(user_id, completed);
```

**バックアップ戦略 / Backup Strategy**:
- 日次自動バックアップ / Daily automated backups
- Point-in-time Recovery / ポイントインタイムリカバリ
- レプリケーション（本番環境） / Replication (production)

---

## 🐳 インフラ / Infrastructure

### Docker

**選定理由 / Reasons for Selection**:
- 環境の一貫性 / Environment consistency
- ポータビリティ / Portability
- 簡単なセットアップ / Easy setup
- マイクロサービスへの移行が容易 / Easy migration to microservices

**Dockerfile例 / Dockerfile Example**:
```dockerfile
# Backend Dockerfile
FROM eclipse-temurin:17-jdk-alpine AS build
WORKDIR /app
COPY pom.xml .
COPY src ./src
RUN mvn clean package -DskipTests

FROM eclipse-temurin:17-jre-alpine
WORKDIR /app
COPY --from=build /app/target/*.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

```dockerfile
# Frontend Dockerfile
FROM node:18-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM nginx:alpine
COPY --from=build /app/dist /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 3000
CMD ["nginx", "-g", "daemon off;"]
```

### Docker Compose

**開発環境の構築 / Development Environment Setup**:
- すべてのサービスを一括管理 / Manage all services together
- ネットワークの自動構成 / Automatic network configuration
- ボリュームマウント / Volume mounting

### Swagger/OpenAPI

**API ドキュメント / API Documentation**:
- 自動生成されるAPIドキュメント / Auto-generated API documentation
- インタラクティブなテスト / Interactive testing
- クライアントコードの生成（将来的に） / Client code generation (future)

**設定例 / Configuration Example**:
```java
@Configuration
@OpenAPIDefinition(
    info = @Info(
        title = "Task Habit Tracker API",
        version = "1.0",
        description = "タスク・習慣管理アプリケーションのAPI"
    )
)
public class OpenApiConfig {
    
    @Bean
    public OpenAPI customOpenAPI() {
        return new OpenAPI()
            .components(new Components()
                .addSecuritySchemes("bearer-jwt", 
                    new SecurityScheme()
                        .type(SecurityScheme.Type.HTTP)
                        .scheme("bearer")
                        .bearerFormat("JWT")
                )
            );
    }
}
```

---

## 🛠️ 開発ツール / Development Tools

### IDE

**推奨 / Recommended**:
- **IntelliJ IDEA**（バックエンド） / for Backend
- **VS Code**（フロントエンド） / for Frontend

### VS Code 拡張機能 / Extensions

**必須 / Essential**:
- ESLint
- Prettier
- TypeScript
- Tailwind CSS IntelliSense
- GitLens

**推奨 / Recommended**:
- REST Client
- Docker
- Thunder Client（API テスト）

### Git

**バージョン管理 / Version Control**:
- Git Flow ベースのブランチ戦略
- Conventional Commits
- PR レビュープロセス

---

## 🧪 テスト / Testing

### バックエンドテスト / Backend Testing

#### JUnit 5
```java
@SpringBootTest
@Transactional
class TaskServiceTest {
    
    @Autowired
    private TaskService taskService;
    
    @MockBean
    private TaskRepository taskRepository;
    
    @Test
    @DisplayName("タスク作成が成功すること")
    void createTask_Success() {
        // Given
        TaskRequest request = new TaskRequest("Test Task");
        Task expectedTask = new Task(1L, "Test Task");
        when(taskRepository.save(any())).thenReturn(expectedTask);
        
        // When
        Task result = taskService.createTask(request);
        
        // Then
        assertNotNull(result);
        assertEquals("Test Task", result.getTitle());
        verify(taskRepository).save(any());
    }
}
```

#### Mockito
- モックオブジェクトの作成 / Creating mock objects
- 依存関係の分離 / Isolating dependencies
- ユニットテストの高速化 / Speeding up unit tests

#### REST Assured（API テスト）
```java
@SpringBootTest(webEnvironment = WebEnvironment.RANDOM_PORT)
class TaskApiTest {
    
    @LocalServerPort
    private int port;
    
    @Test
    void getTasks_ReturnsTaskList() {
        given()
            .port(port)
            .header("Authorization", "Bearer " + token)
        .when()
            .get("/api/tasks")
        .then()
            .statusCode(200)
            .body("size()", greaterThan(0));
    }
}
```

### フロントエンドテスト / Frontend Testing

#### Jest
- ユニットテスト / Unit testing
- スナップショットテスト / Snapshot testing
- カバレッジレポート / Coverage reporting

#### React Testing Library
```typescript
import { render, screen, fireEvent } from '@testing-library/react';
import TaskItem from './TaskItem';

describe('TaskItem', () => {
  const mockTask = {
    id: 1,
    title: 'Test Task',
    completed: false,
  };
  
  it('should toggle task completion', () => {
    const onToggle = jest.fn();
    render(<TaskItem task={mockTask} onToggle={onToggle} />);
    
    const checkbox = screen.getByRole('checkbox');
    fireEvent.click(checkbox);
    
    expect(onToggle).toHaveBeenCalledWith(1);
  });
  
  it('should display task title', () => {
    render(<TaskItem task={mockTask} />);
    
    expect(screen.getByText('Test Task')).toBeInTheDocument();
  });
});
```

### E2Eテスト / E2E Testing

**Playwright または Cypress**（Phase 2以降）:
- ブラウザ自動化 / Browser automation
- ユーザーフローのテスト / User flow testing
- クロスブラウザテスト / Cross-browser testing

---

## 📊 技術スタックの比較 / Tech Stack Comparison

### なぜこの組み合わせなのか / Why This Combination?

| 技術 / Technology | 代替案 / Alternative | 選択理由 / Reason for Choice |
|------------------|-------------------|--------------------------|
| Java + Spring Boot | Node.js + Express | 型安全性、エンタープライズでの実績 / Type safety, proven in enterprise |
| PostgreSQL | MySQL, MongoDB | ACID保証、豊富な機能 / ACID compliance, rich features |
| React | Vue, Angular | 大規模エコシステム、学習リソース豊富 / Large ecosystem, abundant learning resources |
| TypeScript | JavaScript | 型安全性、開発効率の向上 / Type safety, improved development efficiency |
| Tailwind CSS | Bootstrap, Material-UI | カスタマイズ性、バンドルサイズ最適化 / Customizability, optimized bundle size |

---

## 🚀 将来の技術追加 / Future Technology Additions

### Phase 2以降で検討 / Considering for Phase 2+

- **Redis**: キャッシング、セッション管理 / Caching, session management
- **Elasticsearch**: 高度な検索機能 / Advanced search capabilities
- **WebSocket**: リアルタイム通知 / Real-time notifications
- **GraphQL**: 柔軟なAPIクエリ / Flexible API queries

### Phase 3以降で検討 / Considering for Phase 3+

- **Kubernetes**: コンテナオーケストレーション / Container orchestration
- **CI/CD**: GitHub Actions, Jenkins / Automated deployment
- **Monitoring**: Prometheus, Grafana / System monitoring
- **Cloud Deployment**: AWS, GCP, Azure / Cloud infrastructure

---

このドキュメントは、プロジェクトの進化に応じて更新されます。

This document will be updated as the project evolves.
