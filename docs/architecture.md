# システムアーキテクチャ

このドキュメントでは、Task Habit Trackerのシステムアーキテクチャ、設計思想、およびアプリケーションの構造について説明します。

## 📋 目次

- [アーキテクチャ概要](#アーキテクチャ概要)
- [システム構成図](#システム構成図)
- [バックエンドアーキテクチャ](#バックエンドアーキテクチャ)
- [フロントエンドアーキテクチャ](#フロントエンドアーキテクチャ)
- [データフロー](#データフロー)
- [セキュリティアーキテクチャ](#セキュリティアーキテクチャ)
- [拡張性と保守性](#拡張性と保守性)

---

## 🏗️ アーキテクチャ概要

Task Habit Trackerは、以下の原則に基づいて設計されています：

### 設計原則

1. **関心の分離（Separation of Concerns）**
   - バックエンドとフロントエンドの完全な分離
   - レイヤードアーキテクチャによる責務の明確化

2. **RESTful API設計**
   - クライアント・サーバー間の標準的な通信
   - ステートレスな設計

3. **段階的な拡張性**
   - MVPから始めて、段階的に機能を追加
   - マイクロサービスへの移行を見据えた設計

4. **セキュリティファースト**
   - 認証・認可の徹底
   - データ保護とプライバシーの配慮

---

## 📐 システム構成図

```
┌─────────────────────────────────────────────────────────────┐
│                        クライアント層                          │
│  ┌────────────────────────────────────────────────────┐     │
│  │         React Frontend (Port 3000)                  │     │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐         │     │
│  │  │ Pages    │  │Components│  │  Hooks   │         │     │
│  │  └──────────┘  └──────────┘  └──────────┘         │     │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐         │     │
│  │  │  State   │  │  Router  │  │  API     │         │     │
│  │  │Management│  │          │  │  Client  │         │     │
│  │  └──────────┘  └──────────┘  └──────────┘         │     │
│  └────────────────────────────────────────────────────┘     │
└─────────────────────────────────────────────────────────────┘
                            │
                            │ HTTPS/REST API
                            │ (JSON)
                            ↓
┌─────────────────────────────────────────────────────────────┐
│                        API ゲートウェイ層                      │
│  ┌────────────────────────────────────────────────────┐     │
│  │         Spring Boot Backend (Port 8080)            │     │
│  │                                                     │     │
│  │  ┌─────────────────────────────────────────┐      │     │
│  │  │      Spring Security Filter Chain        │      │     │
│  │  │  ・JWT Authentication Filter             │      │     │
│  │  │  ・CORS Filter                           │      │     │
│  │  │  ・Exception Handler                     │      │     │
│  │  └─────────────────────────────────────────┘      │     │
│  └────────────────────────────────────────────────────┘     │
└─────────────────────────────────────────────────────────────┘
                            │
                            ↓
┌─────────────────────────────────────────────────────────────┐
│                      アプリケーション層                         │
│  ┌────────────────────────────────────────────────────┐     │
│  │              Controller Layer                       │     │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐         │     │
│  │  │   Auth   │  │   Task   │  │  Habit   │         │     │
│  │  │Controller│  │Controller│  │Controller│         │     │
│  │  └──────────┘  └──────────┘  └──────────┘         │     │
│  └────────────────────────────────────────────────────┘     │
│                            │                                 │
│  ┌────────────────────────────────────────────────────┐     │
│  │               Service Layer                         │     │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐         │     │
│  │  │   Auth   │  │   Task   │  │  Habit   │         │     │
│  │  │ Service  │  │ Service  │  │ Service  │         │     │
│  │  └──────────┘  └──────────┘  └──────────┘         │     │
│  │        ↓              ↓              ↓             │     │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐         │     │
│  │  │   User   │  │   Task   │  │  Habit   │         │     │
│  │  │   Repo   │  │   Repo   │  │   Repo   │         │     │
│  │  └──────────┘  └──────────┘  └──────────┘         │     │
│  └────────────────────────────────────────────────────┘     │
└─────────────────────────────────────────────────────────────┘
                            │
                            │ JPA/Hibernate
                            ↓
┌─────────────────────────────────────────────────────────────┐
│                        データ永続化層                          │
│  ┌────────────────────────────────────────────────────┐     │
│  │         PostgreSQL Database (Port 5432)            │     │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐         │     │
│  │  │  users   │  │  tasks   │  │  habits  │         │     │
│  │  └──────────┘  └──────────┘  └──────────┘         │     │
│  │  ┌──────────┐  ┌──────────┐                       │     │
│  │  │categories│  │habit_logs│                       │     │
│  │  └──────────┘  └──────────┘                       │     │
│  └────────────────────────────────────────────────────┘     │
└─────────────────────────────────────────────────────────────┘
```

---

## 🔧 バックエンドアーキテクチャ

### レイヤードアーキテクチャ

バックエンドは、明確な責務を持つ3つの主要レイヤーで構成されています。

#### 1. コントローラー層（Controller Layer）

**責務**:
- HTTPリクエストの受け取り
- リクエストデータのバリデーション
- サービス層の呼び出し
- HTTPレスポンスの返却

**実装例**:
```java
@RestController
@RequestMapping("/api/tasks")
@RequiredArgsConstructor
public class TaskController {
    
    private final TaskService taskService;
    
    @GetMapping
    public ResponseEntity<List<TaskDto>> getAllTasks(
        @RequestParam(required = false) Boolean completed,
        @AuthenticationPrincipal UserDetails userDetails
    ) {
        Long userId = ((CustomUserDetails) userDetails).getUserId();
        List<TaskDto> tasks = taskService.getUserTasks(userId, completed);
        return ResponseEntity.ok(tasks);
    }
    
    @PostMapping
    public ResponseEntity<TaskDto> createTask(
        @Valid @RequestBody CreateTaskRequest request,
        @AuthenticationPrincipal UserDetails userDetails
    ) {
        Long userId = ((CustomUserDetails) userDetails).getUserId();
        TaskDto task = taskService.createTask(request, userId);
        return ResponseEntity.status(HttpStatus.CREATED).body(task);
    }
    
    @PutMapping("/{id}")
    public ResponseEntity<TaskDto> updateTask(
        @PathVariable Long id,
        @Valid @RequestBody UpdateTaskRequest request,
        @AuthenticationPrincipal UserDetails userDetails
    ) {
        Long userId = ((CustomUserDetails) userDetails).getUserId();
        TaskDto task = taskService.updateTask(id, request, userId);
        return ResponseEntity.ok(task);
    }
    
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteTask(
        @PathVariable Long id,
        @AuthenticationPrincipal UserDetails userDetails
    ) {
        Long userId = ((CustomUserDetails) userDetails).getUserId();
        taskService.deleteTask(id, userId);
        return ResponseEntity.noContent().build();
    }
}
```

**ベストプラクティス**:
- コントローラーにビジネスロジックを含めない
- @Valid アノテーションでリクエストを検証
- 適切なHTTPステータスコードを返す
- RESTful な URL 設計

#### 2. サービス層（Service Layer）

**責務**:
- ビジネスロジックの実装
- トランザクション管理
- リポジトリ層の呼び出しと調整
- データの変換（Entity ↔ DTO）

**実装例**:
```java
@Service
@RequiredArgsConstructor
@Transactional
public class TaskService {
    
    private final TaskRepository taskRepository;
    private final TaskMapper taskMapper;
    
    @Transactional(readOnly = true)
    public List<TaskDto> getUserTasks(Long userId, Boolean completed) {
        List<Task> tasks;
        if (completed != null) {
            tasks = taskRepository.findByUserIdAndCompleted(userId, completed);
        } else {
            tasks = taskRepository.findByUserId(userId);
        }
        return tasks.stream()
            .map(taskMapper::toDto)
            .collect(Collectors.toList());
    }
    
    public TaskDto createTask(CreateTaskRequest request, Long userId) {
        Task task = Task.builder()
            .userId(userId)
            .title(request.getTitle())
            .description(request.getDescription())
            .dueDate(request.getDueDate())
            .priority(request.getPriority())
            .completed(false)
            .build();
        
        Task savedTask = taskRepository.save(task);
        return taskMapper.toDto(savedTask);
    }
    
    public TaskDto updateTask(Long id, UpdateTaskRequest request, Long userId) {
        Task task = taskRepository.findById(id)
            .orElseThrow(() -> new ResourceNotFoundException("Task not found"));
        
        // 権限チェック
        if (!task.getUserId().equals(userId)) {
            throw new UnauthorizedException("Not authorized to update this task");
        }
        
        // 更新処理
        task.setTitle(request.getTitle());
        task.setDescription(request.getDescription());
        task.setDueDate(request.getDueDate());
        task.setPriority(request.getPriority());
        
        Task updatedTask = taskRepository.save(task);
        return taskMapper.toDto(updatedTask);
    }
    
    public void deleteTask(Long id, Long userId) {
        Task task = taskRepository.findById(id)
            .orElseThrow(() -> new ResourceNotFoundException("Task not found"));
        
        if (!task.getUserId().equals(userId)) {
            throw new UnauthorizedException("Not authorized to delete this task");
        }
        
        taskRepository.delete(task);
    }
}
```

**ベストプラクティス**:
- @Transactional でトランザクション管理
- 読み取り専用の操作には readOnly = true
- ビジネスルールの検証
- 適切な例外のスロー

#### 3. リポジトリ層（Repository Layer）

**責務**:
- データベースとのやり取り
- クエリの実行
- エンティティの永続化

**実装例**:
```java
@Repository
public interface TaskRepository extends JpaRepository<Task, Long> {
    
    // メソッド名からクエリを自動生成
    List<Task> findByUserId(Long userId);
    
    List<Task> findByUserIdAndCompleted(Long userId, Boolean completed);
    
    List<Task> findByUserIdAndDueDate(Long userId, LocalDate dueDate);
    
    // カスタムクエリ
    @Query("SELECT t FROM Task t WHERE t.userId = :userId AND t.dueDate BETWEEN :start AND :end")
    List<Task> findTasksInDateRange(
        @Param("userId") Long userId,
        @Param("start") LocalDate start,
        @Param("end") LocalDate end
    );
    
    // ネイティブクエリ（パフォーマンスが必要な場合）
    @Query(value = "SELECT * FROM tasks WHERE user_id = ?1 AND completed = false ORDER BY priority DESC, due_date ASC LIMIT ?2", 
           nativeQuery = true)
    List<Task> findTopPriorityTasks(Long userId, int limit);
}
```

### エンティティとDTO

**エンティティ（Entity）**:
```java
@Entity
@Table(name = "tasks")
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class Task {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(name = "user_id", nullable = false)
    private Long userId;
    
    @Column(nullable = false, length = 200)
    private String title;
    
    @Column(length = 1000)
    private String description;
    
    @Column(name = "due_date")
    private LocalDate dueDate;
    
    @Enumerated(EnumType.STRING)
    @Column(length = 20)
    private Priority priority;
    
    @Column(nullable = false)
    private Boolean completed = false;
    
    @Column(name = "completed_at")
    private LocalDateTime completedAt;
    
    @Column(name = "created_at", nullable = false, updatable = false)
    private LocalDateTime createdAt;
    
    @Column(name = "updated_at")
    private LocalDateTime updatedAt;
    
    @PrePersist
    protected void onCreate() {
        createdAt = LocalDateTime.now();
        updatedAt = LocalDateTime.now();
    }
    
    @PreUpdate
    protected void onUpdate() {
        updatedAt = LocalDateTime.now();
    }
}
```

**DTO（Data Transfer Object）**:
```java
@Data
@Builder
public class TaskDto {
    private Long id;
    private String title;
    private String description;
    private LocalDate dueDate;
    private Priority priority;
    private Boolean completed;
    private LocalDateTime completedAt;
    private LocalDateTime createdAt;
    private LocalDateTime updatedAt;
}
```

### エラーハンドリング

**グローバル例外ハンドラー**:
```java
@RestControllerAdvice
public class GlobalExceptionHandler {
    
    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleResourceNotFound(ResourceNotFoundException ex) {
        ErrorResponse error = ErrorResponse.builder()
            .status(HttpStatus.NOT_FOUND.value())
            .message(ex.getMessage())
            .timestamp(LocalDateTime.now())
            .build();
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(error);
    }
    
    @ExceptionHandler(UnauthorizedException.class)
    public ResponseEntity<ErrorResponse> handleUnauthorized(UnauthorizedException ex) {
        ErrorResponse error = ErrorResponse.builder()
            .status(HttpStatus.FORBIDDEN.value())
            .message(ex.getMessage())
            .timestamp(LocalDateTime.now())
            .build();
        return ResponseEntity.status(HttpStatus.FORBIDDEN).body(error);
    }
    
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ErrorResponse> handleValidationErrors(MethodArgumentNotValidException ex) {
        List<String> errors = ex.getBindingResult()
            .getFieldErrors()
            .stream()
            .map(error -> error.getField() + ": " + error.getDefaultMessage())
            .collect(Collectors.toList());
        
        ErrorResponse error = ErrorResponse.builder()
            .status(HttpStatus.BAD_REQUEST.value())
            .message("Validation failed")
            .errors(errors)
            .timestamp(LocalDateTime.now())
            .build();
        return ResponseEntity.badRequest().body(error);
    }
}
```

---

## 🎨 フロントエンドアーキテクチャ

### コンポーネント構造

```
src/
├── components/
│   ├── common/           # 共通コンポーネント
│   │   ├── Button.tsx
│   │   ├── Input.tsx
│   │   ├── Modal.tsx
│   │   └── Loading.tsx
│   ├── layout/           # レイアウトコンポーネント
│   │   ├── Header.tsx
│   │   ├── Sidebar.tsx
│   │   └── Footer.tsx
│   ├── task/             # タスク関連コンポーネント
│   │   ├── TaskList.tsx
│   │   ├── TaskItem.tsx
│   │   ├── TaskForm.tsx
│   │   └── TaskFilter.tsx
│   └── habit/            # 習慣関連コンポーネント
│       ├── HabitList.tsx
│       ├── HabitCard.tsx
│       └── HabitHeatmap.tsx
├── pages/                # ページコンポーネント
│   ├── Dashboard.tsx
│   ├── TasksPage.tsx
│   ├── HabitsPage.tsx
│   ├── LoginPage.tsx
│   └── RegisterPage.tsx
├── hooks/                # カスタムフック
│   ├── useAuth.ts
│   ├── useTasks.ts
│   └── useHabits.ts
├── services/             # APIサービス
│   ├── api.ts
│   ├── authService.ts
│   ├── taskService.ts
│   └── habitService.ts
├── store/                # 状態管理
│   ├── authStore.ts
│   ├── taskStore.ts
│   └── habitStore.ts
├── types/                # TypeScript型定義
│   ├── task.ts
│   ├── habit.ts
│   └── user.ts
├── utils/                # ユーティリティ関数
│   ├── dateUtils.ts
│   ├── formatters.ts
│   └── validators.ts
└── App.tsx               # ルートコンポーネント
```

### 状態管理パターン

**Zustand Store の例**:
```typescript
import create from 'zustand';
import { taskService } from '../services/taskService';
import { Task } from '../types/task';

interface TaskStore {
  tasks: Task[];
  loading: boolean;
  error: string | null;
  
  fetchTasks: () => Promise<void>;
  addTask: (task: Omit<Task, 'id'>) => Promise<void>;
  updateTask: (id: number, updates: Partial<Task>) => Promise<void>;
  deleteTask: (id: number) => Promise<void>;
  toggleTask: (id: number) => Promise<void>;
}

export const useTaskStore = create<TaskStore>((set, get) => ({
  tasks: [],
  loading: false,
  error: null,
  
  fetchTasks: async () => {
    set({ loading: true, error: null });
    try {
      const tasks = await taskService.getTasks();
      set({ tasks, loading: false });
    } catch (error) {
      set({ error: error.message, loading: false });
    }
  },
  
  addTask: async (taskData) => {
    try {
      const newTask = await taskService.createTask(taskData);
      set((state) => ({ tasks: [...state.tasks, newTask] }));
    } catch (error) {
      set({ error: error.message });
      throw error;
    }
  },
  
  updateTask: async (id, updates) => {
    try {
      const updatedTask = await taskService.updateTask(id, updates);
      set((state) => ({
        tasks: state.tasks.map(t => t.id === id ? updatedTask : t)
      }));
    } catch (error) {
      set({ error: error.message });
      throw error;
    }
  },
  
  deleteTask: async (id) => {
    try {
      await taskService.deleteTask(id);
      set((state) => ({
        tasks: state.tasks.filter(t => t.id !== id)
      }));
    } catch (error) {
      set({ error: error.message });
      throw error;
    }
  },
  
  toggleTask: async (id) => {
    try {
      const task = get().tasks.find(t => t.id === id);
      if (task) {
        await taskService.toggleTask(id);
        set((state) => ({
          tasks: state.tasks.map(t =>
            t.id === id ? { ...t, completed: !t.completed } : t
          )
        }));
      }
    } catch (error) {
      set({ error: error.message });
      throw error;
    }
  },
}));
```

### APIクライアント

**API サービスの実装**:
```typescript
import axios from 'axios';

const api = axios.create({
  baseURL: process.env.REACT_APP_API_URL || 'http://localhost:8080/api',
  headers: {
    'Content-Type': 'application/json',
  },
});

// リクエストインターセプター（JWT トークンを自動追加）
api.interceptors.request.use(
  (config) => {
    const token = localStorage.getItem('token');
    if (token) {
      config.headers.Authorization = `Bearer ${token}`;
    }
    return config;
  },
  (error) => Promise.reject(error)
);

// レスポンスインターセプター（エラーハンドリング）
api.interceptors.response.use(
  (response) => response,
  (error) => {
    if (error.response?.status === 401) {
      // 認証エラー時はログインページへリダイレクト
      localStorage.removeItem('token');
      window.location.href = '/login';
    }
    return Promise.reject(error);
  }
);

export default api;
```

---

## 🔄 データフロー

### タスク作成のフロー例

```
1. ユーザーがタスク作成フォームに入力
   ↓
2. フロントエンド: バリデーション
   ↓
3. フロントエンド: POST /api/tasks にリクエスト
   ↓
4. バックエンド: JWT トークン検証（Spring Security）
   ↓
5. バックエンド: コントローラーがリクエストを受け取る
   ↓
6. バックエンド: @Valid でリクエストボディを検証
   ↓
7. バックエンド: サービス層でビジネスロジック実行
   ↓
8. バックエンド: リポジトリ層でDBに保存
   ↓
9. バックエンド: エンティティをDTOに変換
   ↓
10. バックエンド: JSONレスポンスを返す
   ↓
11. フロントエンド: レスポンスを受け取り、状態を更新
   ↓
12. フロントエンド: UIを再レンダリング
```

---

## 🔐 セキュリティアーキテクチャ

### 認証フロー

```
1. ユーザーがログイン情報を送信
   POST /api/auth/login
   ↓
2. Spring Security が認証を処理
   ・ユーザー名とパスワードの検証
   ・BCryptでパスワードハッシュを比較
   ↓
3. 認証成功時、JWTトークンを生成
   ・ユーザーIDとロールを含む
   ・有効期限を設定（24時間）
   ・秘密鍵で署名
   ↓
4. トークンをクライアントに返却
   ↓
5. クライアントはトークンを保存（LocalStorage）
   ↓
6. 以降のリクエストでトークンを送信
   Authorization: Bearer <token>
   ↓
7. Spring Security がトークンを検証
   ・署名の検証
   ・有効期限の確認
   ・ユーザー情報の抽出
   ↓
8. 検証成功時、リクエストを処理
```

### 認可（Authorization）

**ロールベースアクセス制御**:
```java
@PreAuthorize("hasRole('USER')")
public TaskDto createTask(CreateTaskRequest request, Long userId) {
    // ユーザーのみがタスクを作成できる
}

@PreAuthorize("hasRole('ADMIN')")
public void deleteAllTasks() {
    // 管理者のみがすべてのタスクを削除できる
}
```

**リソースベースアクセス制御**:
```java
public TaskDto updateTask(Long id, UpdateTaskRequest request, Long userId) {
    Task task = taskRepository.findById(id)
        .orElseThrow(() -> new ResourceNotFoundException("Task not found"));
    
    // 自分のタスクのみ更新可能
    if (!task.getUserId().equals(userId)) {
        throw new UnauthorizedException("Not authorized");
    }
    
    // 更新処理...
}
```

---

## 📈 拡張性と保守性

### マイクロサービスへの移行パス

現在はモノリシックアーキテクチャですが、将来的にマイクロサービスへの移行を考慮した設計になっています：

```
現在: モノリス
┌─────────────────────┐
│  Spring Boot App    │
│  ・Auth             │
│  ・Task Service     │
│  ・Habit Service    │
└─────────────────────┘

将来: マイクロサービス
┌─────────────┐  ┌─────────────┐  ┌─────────────┐
│   Auth      │  │    Task     │  │   Habit     │
│  Service    │  │   Service   │  │  Service    │
└─────────────┘  └─────────────┘  └─────────────┘
       │                 │                 │
       └─────────────────┴─────────────────┘
                         │
                  ┌─────────────┐
                  │  API Gateway│
                  └─────────────┘
```

### コードの保守性

**依存性注入（DI）の活用**:
- テスタビリティの向上
- 疎結合な設計
- モックによる単体テスト

**インターフェースの活用**:
```java
public interface TaskService {
    List<TaskDto> getUserTasks(Long userId, Boolean completed);
    TaskDto createTask(CreateTaskRequest request, Long userId);
    TaskDto updateTask(Long id, UpdateTaskRequest request, Long userId);
    void deleteTask(Long id, Long userId);
}

@Service
public class TaskServiceImpl implements TaskService {
    // 実装...
}
```

### パフォーマンス最適化

**データベースレベル**:
- 適切なインデックスの設定
- クエリの最適化
- コネクションプール

**アプリケーションレベル**:
- ページネーション
- 遅延ローディング
- キャッシング（Phase 2以降）

**フロントエンドレベル**:
- コード分割
- 遅延ローディング
- メモ化（React.memo, useMemo）

---

このアーキテクチャは、プロジェクトの進化に応じて継続的に改善されます。
