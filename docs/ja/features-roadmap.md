# 機能ロードマップ

このドキュメントでは、Task Habit Trackerの機能を段階的に開発していく計画を詳細に記載します。

## 🗺️ 開発フェーズ概要

各フェーズは、前のフェーズで構築した基盤の上に新機能を追加していく形で進めます。

| フェーズ | 期間 | 主な目標 | ステータス |
|---------|------|----------|-----------|
| Phase 1: MVP | 1～2週間 | 基本的なタスク管理機能 | 🚧 開発中 |
| Phase 2: 機能拡張 | 追加1～2週間 | 習慣トラッキングとUI改善 | 📋 計画中 |
| Phase 3: 高度な機能 | 今後の計画 | 外部連携と高度な機能 | 💡 アイデア |

---

## 🎯 Phase 1: MVP（最小実用製品）

**期間**: 1～2週間  
**目標**: 基本的なタスク管理機能を持つ動作するアプリケーションを完成させる

### 機能詳細

#### 1.1 ユーザー管理・認証

##### バックエンド
- **ユーザー登録**
  - メールアドレスとパスワードで登録
  - パスワードはBCryptでハッシュ化
  - メールアドレスの重複チェック
  - 入力値のバリデーション
  
  ```json
  POST /api/auth/register
  {
    "email": "user@example.com",
    "password": "securePassword123",
    "username": "JohnDoe"
  }
  ```

- **ログイン**
  - メールアドレスとパスワードで認証
  - JWT トークンの発行
  - トークン有効期限: 24時間
  
  ```json
  POST /api/auth/login
  {
    "email": "user@example.com",
    "password": "securePassword123"
  }
  
  Response:
  {
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "expiresIn": 86400
  }
  ```

- **認証ミドルウェア**
  - JWTトークンの検証
  - 保護されたエンドポイントへのアクセス制御
  - ユーザー情報の抽出

##### フロントエンド
- ログインフォーム
- 新規登録フォーム
- トークンの保存（LocalStorage または Cookie）
- 認証状態の管理（Context API）
- 保護されたルートの実装

##### データベーススキーマ
```sql
CREATE TABLE users (
    id BIGSERIAL PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    username VARCHAR(100) NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

#### 1.2 タスク管理（CRUD操作）

##### タスク作成
- **API エンドポイント**: `POST /api/tasks`
- **機能**:
  - タスクのタイトル（必須）
  - 説明（任意）
  - 期日（任意）
  - 優先度（Low, Medium, High）
  - 現在のユーザーに紐付け

```json
POST /api/tasks
{
  "title": "プロジェクト資料を作成",
  "description": "Phase 1の完了報告資料を作成する",
  "dueDate": "2024-12-15",
  "priority": "HIGH"
}

Response:
{
  "id": 1,
  "title": "プロジェクト資料を作成",
  "description": "Phase 1の完了報告資料を作成する",
  "dueDate": "2024-12-15",
  "priority": "HIGH",
  "completed": false,
  "createdAt": "2024-12-01T10:00:00Z",
  "updatedAt": "2024-12-01T10:00:00Z"
}
```

##### タスク取得
- **すべてのタスクを取得**: `GET /api/tasks`
  - ページネーション対応（デフォルト: 20件/ページ）
  - 完了/未完了でフィルタリング可能
  
- **特定のタスクを取得**: `GET /api/tasks/{id}`
  - タスクの詳細情報を取得
  - 存在しない場合は404エラー

```json
GET /api/tasks?page=1&size=20&completed=false

Response:
{
  "content": [
    {
      "id": 1,
      "title": "プロジェクト資料を作成",
      "completed": false,
      ...
    }
  ],
  "page": 1,
  "size": 20,
  "totalElements": 5,
  "totalPages": 1
}
```

##### タスク更新
- **API エンドポイント**: `PUT /api/tasks/{id}`
- **機能**:
  - タイトル、説明、期日、優先度の更新
  - 他のユーザーのタスクは更新不可
  - 存在しないタスクの場合は404エラー

```json
PUT /api/tasks/1
{
  "title": "プロジェクト資料を作成（更新）",
  "description": "詳細な説明を追加",
  "dueDate": "2024-12-20",
  "priority": "MEDIUM"
}
```

##### タスク削除
- **API エンドポイント**: `DELETE /api/tasks/{id}`
- **機能**:
  - タスクの論理削除または物理削除
  - 他のユーザーのタスクは削除不可
  - 成功時は204 No Contentを返す

#### 1.3 タスク完了切り替え

- **API エンドポイント**: `PATCH /api/tasks/{id}/toggle`
- **機能**:
  - 完了/未完了の状態をトグル
  - 完了時刻を記録
  - 統計情報の更新

```json
PATCH /api/tasks/1/toggle

Response:
{
  "id": 1,
  "completed": true,
  "completedAt": "2024-12-05T15:30:00Z"
}
```

##### フロントエンド実装
- チェックボックスまたはボタンで切り替え
- 完了タスクにはスタイル変更（打ち消し線、グレーアウト）
- アニメーション効果

#### 1.4 日ごとのタスクビュー

##### 今日のタスク
- **API エンドポイント**: `GET /api/tasks/today`
- **機能**:
  - 今日期日のタスクを表示
  - 期日なしタスクも含む（オプション）
  - 優先度順でソート

##### ビューの種類
1. **リストビュー**
   - シンプルな一覧表示
   - 優先度別に色分け
   - 完了/未完了の切り替え

2. **カードビュー**（時間があれば）
   - カード形式で表示
   - ドラッグ&ドロップで並び替え

##### UIコンポーネント
```typescript
// TaskList コンポーネント
interface Task {
  id: number;
  title: string;
  description?: string;
  dueDate?: string;
  priority: 'LOW' | 'MEDIUM' | 'HIGH';
  completed: boolean;
}

const TaskList: React.FC<{ tasks: Task[] }> = ({ tasks }) => {
  // タスク一覧を表示
};
```

#### 1.5 Docker環境構築

##### Docker Compose 構成
```yaml
version: '3.8'

services:
  postgres:
    image: postgres:15
    environment:
      POSTGRES_DB: task_habit_tracker
      POSTGRES_USER: ${DB_USER}
      POSTGRES_PASSWORD: ${DB_PASSWORD}
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data

  backend:
    build: ./backend
    ports:
      - "8080:8080"
    environment:
      DB_HOST: postgres
      DB_PORT: 5432
      DB_NAME: task_habit_tracker
    depends_on:
      - postgres

  frontend:
    build: ./frontend
    ports:
      - "3000:3000"
    environment:
      REACT_APP_API_URL: http://localhost:8080
    depends_on:
      - backend

volumes:
  postgres_data:
```

### Phase 1 完了基準

- [ ] ユーザー登録・ログインが動作する
- [ ] タスクのCRUD操作がすべて動作する
- [ ] タスクの完了切り替えができる
- [ ] 今日のタスクが表示される
- [ ] Docker Composeで環境が起動する
- [ ] 基本的なテストが通過する
- [ ] APIドキュメント（Swagger）が生成される

---

## 🚀 Phase 2: 機能拡張

**期間**: 追加1～2週間  
**目標**: 習慣トラッキング機能の追加とUIの大幅改善

### 機能詳細

#### 2.1 習慣トラッキング

##### 習慣の定義
- **習慣とは**: 毎日（または定期的に）実行したい行動
- **例**: 朝のランニング、読書、瞑想、水を飲む

##### データモデル
```sql
CREATE TABLE habits (
    id BIGSERIAL PRIMARY KEY,
    user_id BIGINT NOT NULL REFERENCES users(id),
    title VARCHAR(200) NOT NULL,
    description TEXT,
    frequency VARCHAR(20) DEFAULT 'DAILY', -- DAILY, WEEKLY, CUSTOM
    target_days TEXT, -- 曜日指定（例: "1,3,5" = 月水金）
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    archived BOOLEAN DEFAULT FALSE
);

CREATE TABLE habit_logs (
    id BIGSERIAL PRIMARY KEY,
    habit_id BIGINT NOT NULL REFERENCES habits(id),
    log_date DATE NOT NULL,
    completed BOOLEAN DEFAULT FALSE,
    note TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(habit_id, log_date)
);
```

##### API エンドポイント

**習慣の作成**:
```json
POST /api/habits
{
  "title": "朝のランニング",
  "description": "30分間のジョギング",
  "frequency": "DAILY"
}
```

**今日の習慣記録**:
```json
POST /api/habits/{id}/log
{
  "date": "2024-12-05",
  "completed": true,
  "note": "5km走った"
}
```

**習慣の達成履歴取得**:
```json
GET /api/habits/{id}/logs?from=2024-12-01&to=2024-12-31

Response:
{
  "habitId": 1,
  "logs": [
    { "date": "2024-12-01", "completed": true },
    { "date": "2024-12-02", "completed": true },
    { "date": "2024-12-03", "completed": false }
  ],
  "completionRate": 0.67
}
```

##### 統計情報
- 連続達成日数（ストリーク）
- 月間達成率
- 週間達成率
- 総達成回数

#### 2.2 カレンダービュー＋ヒートマップ

##### カレンダービュー
- 月間カレンダー表示
- 各日にタスク数を表示
- クリックで日別の詳細表示

##### ヒートマップ
- GitHub風のコントリビューショングラフ
- 色の濃さで達成度を可視化
- 1年分のデータを表示
- ツールチップで詳細情報表示

```typescript
// ヒートマップのデータ構造
interface HeatmapData {
  date: string;
  count: number; // 達成した習慣の数
  level: 0 | 1 | 2 | 3 | 4; // 色の濃さレベル
}
```

##### 使用ライブラリ候補
- `react-calendar-heatmap`
- `recharts`（カスタムヒートマップ用）

#### 2.3 カテゴリ/タグ機能

##### データモデル
```sql
CREATE TABLE categories (
    id BIGSERIAL PRIMARY KEY,
    user_id BIGINT NOT NULL REFERENCES users(id),
    name VARCHAR(100) NOT NULL,
    color VARCHAR(7), -- HEXカラーコード（例: #FF5733）
    icon VARCHAR(50),
    UNIQUE(user_id, name)
);

CREATE TABLE tags (
    id BIGSERIAL PRIMARY KEY,
    user_id BIGINT NOT NULL REFERENCES users(id),
    name VARCHAR(50) NOT NULL,
    UNIQUE(user_id, name)
);

CREATE TABLE task_tags (
    task_id BIGINT NOT NULL REFERENCES tasks(id),
    tag_id BIGINT NOT NULL REFERENCES tags(id),
    PRIMARY KEY (task_id, tag_id)
);
```

##### 機能
- **カテゴリ**: タスクを大きく分類（仕事、プライベート、学習など）
- **タグ**: 複数付与可能な柔軟な分類（#重要、#クイック、#会議準備など）

##### API
```json
POST /api/categories
{
  "name": "仕事",
  "color": "#FF5733",
  "icon": "briefcase"
}

POST /api/tasks
{
  "title": "プレゼン資料作成",
  "categoryId": 1,
  "tags": ["重要", "会議準備"]
}
```

#### 2.4 フィルタ・検索機能

##### フィルタ条件
- 完了/未完了
- カテゴリ
- タグ
- 優先度
- 期日（今日、今週、今月、期限切れ）

##### 検索機能
- タイトル・説明での全文検索
- タグでの検索
- 作成日での範囲検索

##### API
```json
GET /api/tasks/search?q=プレゼン&category=仕事&completed=false&priority=HIGH

Response:
{
  "results": [...],
  "totalCount": 5
}
```

##### UIコンポーネント
- フィルタパネル（サイドバー）
- 検索バー（リアルタイム検索）
- 保存済みフィルタ（お気に入り）

#### 2.5 UI/UX改善

##### デザインシステム
- 一貫したカラーパレット
- タイポグラフィの統一
- スペーシングの規則化

##### レスポンシブデザイン
- モバイルファースト
- タブレット対応
- デスクトップ最適化

##### アニメーション
- タスク完了時の視覚的フィードバック
- ページ遷移のスムーズ化
- ローディング状態の表示

##### アクセシビリティ
- キーボードナビゲーション
- スクリーンリーダー対応
- 適切なコントラスト比

### Phase 2 完了基準

- [ ] 習慣の作成・記録ができる
- [ ] ヒートマップで習慣の達成状況を可視化できる
- [ ] カテゴリ・タグでタスクを整理できる
- [ ] フィルタ・検索で目的のタスクを素早く見つけられる
- [ ] モバイルデバイスでも使いやすい
- [ ] 統計情報が正確に表示される

---

## 🌟 Phase 3: 高度な機能追加

**期間**: 今後の計画  
**目標**: より高度な機能と外部サービス連携

### 機能アイデア

#### 3.1 外部API連携

##### 天気API連携
- **用途**: 屋外活動の習慣に天気情報を表示
- **API候補**: OpenWeatherMap API
- **機能**:
  - 今日の天気予報を表示
  - 天気に応じた習慣の提案
  - 雨の日の代替活動提案

##### カレンダー連携
- **用途**: 既存のカレンダーとの同期
- **API候補**: Google Calendar API
- **機能**:
  - タスクをカレンダーにエクスポート
  - カレンダーイベントからタスク作成
  - 双方向同期

#### 3.2 通知機能

##### プッシュ通知
- タスクの期限が近づいた時
- 習慣の実行時刻のリマインド
- 連続達成の記録更新時

##### メール通知
- 週次レポート
- 月次サマリー
- 達成マイルストーン

##### 実装方法
- **バックエンド**: Spring Boot + Firebase Cloud Messaging
- **フロントエンド**: Service Worker + Push API

#### 3.3 チームタスク共有

##### 機能
- チーム/グループの作成
- タスクの共有
- 担当者の割り当て
- コメント機能
- アクティビティフィード

##### データモデル
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
    role VARCHAR(20), -- OWNER, ADMIN, MEMBER
    joined_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (team_id, user_id)
);

CREATE TABLE shared_tasks (
    id BIGSERIAL PRIMARY KEY,
    task_id BIGINT REFERENCES tasks(id),
    team_id BIGINT REFERENCES teams(id),
    assigned_to BIGINT REFERENCES users(id),
    shared_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

#### 3.4 ファイル添付機能

##### 機能
- タスクへのファイル添付
- 画像のプレビュー表示
- PDFの閲覧
- ファイルサイズ制限（10MB）

##### ストレージ
- **開発環境**: ローカルファイルシステム
- **本番環境**: AWS S3 または Google Cloud Storage

##### セキュリティ
- ファイルタイプの検証
- ウイルススキャン（将来的に）
- アクセス制御

#### 3.5 データエクスポート

##### エクスポート形式
- JSON
- CSV
- PDF（レポート形式）

##### エクスポート内容
- すべてのタスク
- 習慣の達成履歴
- 統計情報
- カスタムレポート

#### 3.6 パフォーマンス最適化

##### データベース最適化
- インデックスの追加
- クエリの最適化
- N+1問題の解決

##### フロントエンド最適化
- コード分割
- 遅延ローディング
- キャッシング戦略
- PWA化

##### モニタリング
- パフォーマンスメトリクスの収集
- エラートラッキング
- ユーザー行動分析

#### 3.7 CI/CDパイプライン

##### 自動化
- 自動テスト実行
- 自動デプロイ
- コードカバレッジレポート
- セキュリティスキャン

##### デプロイ環境
- **開発環境**: 自動デプロイ
- **ステージング環境**: PRマージ時に自動デプロイ
- **本番環境**: 手動承認後にデプロイ

---

## 📊 進捗管理

### 現在の状況

| 項目 | 完了率 | 備考 |
|-----|--------|------|
| Phase 1: MVP | 0% | これから開始 |
| Phase 2: 機能拡張 | 0% | Phase 1完了後 |
| Phase 3: 高度な機能 | 0% | Phase 2完了後 |

### 次のステップ

1. **即座に着手**:
   - バックエンドのプロジェクト初期化
   - データベーススキーマの設計
   - ユーザー認証の実装

2. **今週中**:
   - タスクCRUD APIの実装
   - 基本的なフロントエンド構築
   - Docker環境の整備

3. **来週**:
   - UIの洗練
   - テストの追加
   - Phase 1の完成

---

このロードマップは、開発の進行に応じて柔軟に調整されます。新しいアイデアや優先度の変更があれば、このドキュメントを更新してください。
