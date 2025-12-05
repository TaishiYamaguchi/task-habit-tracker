# 開発ガイドライン

このドキュメントでは、Task & Habit Tracker プロジェクトの開発における品質基準、コーディング規約、ベストプラクティスを定義します。

## 目次

1. [コード品質基準](#コード品質基準)
2. [セキュリティプラクティス](#セキュリティプラクティス)
3. [ドキュメント維持方針](#ドキュメント維持方針)
4. [バージョン管理](#バージョン管理)
5. [依存関係管理](#依存関係管理)
6. [Issue/PR運用ガイドライン](#issuepr運用ガイドライン)

---

## コード品質基準

### テスト

#### テストの種類と責務

| テストタイプ | 責務 | ツール |
|-------------|------|--------|
| **単体テスト** | 個々のクラス・関数の動作検証 | JUnit 5, Vitest |
| **統合テスト** | コンポーネント間の連携検証 | Testcontainers, RTL |
| **E2Eテスト** | エンドツーエンドの動作検証 | (将来: Playwright) |

#### テストガイドライン

```java
// Good: 説明的なテスト名
@Test
void createTask_withValidInput_shouldReturnCreatedTask() { ... }

// Bad: 曖昧なテスト名
@Test
void testCreateTask() { ... }
```

- **AAAパターン**: Arrange（準備）→ Act（実行）→ Assert（検証）
- **1テスト1アサート**: 原則として1つの振る舞いをテスト
- **テストの独立性**: 他のテストに依存しない
- **意味のあるテストデータ**: `"test"` ではなく `"valid_username"` のような意図の分かるデータ

#### テストカバレッジ目標

| レイヤー | 目標カバレッジ |
|----------|--------------|
| Service層 | 80%以上 |
| Controller層 | 70%以上 |
| Repository層 | 統合テストでカバー |
| フロントエンドコンポーネント | 主要な振る舞いをカバー |

### 命名規約

#### Java

```java
// クラス名: PascalCase
public class TaskService { }

// メソッド名: camelCase、動詞で始める
public Task createTask(TaskRequest request) { }

// 定数: SCREAMING_SNAKE_CASE
public static final int MAX_TITLE_LENGTH = 255;

// パッケージ名: すべて小文字
package com.taskhabittracker.service;
```

#### TypeScript/React

```typescript
// コンポーネント: PascalCase
const TaskList: React.FC = () => { ... }

// 関数・変数: camelCase
const handleSubmit = () => { ... }

// 型・インターフェース: PascalCase
interface TaskResponse { ... }

// 定数: SCREAMING_SNAKE_CASE または camelCase
const API_BASE_URL = '/api';
```

#### ファイル名

| 種類 | 規約 | 例 |
|------|------|-----|
| Javaクラス | PascalCase | `TaskService.java` |
| Reactコンポーネント | PascalCase | `TaskList.tsx` |
| フック | camelCase | `useTasks.ts` |
| ユーティリティ | camelCase | `dateUtils.ts` |
| テスト | 対象ファイル + `.test` | `TaskService.test.ts` |

### コメント

#### コメントの原則

- **コードで説明できることはコードで**: 良いコードは自己説明的
- **「なぜ」を説明**: 「何をしているか」ではなく「なぜそうしているか」
- **TODO/FIXMEの活用**: 課題を明示的に残す

```java
// Bad: 何をしているか説明している（コードを見れば分かる）
// ユーザーIDでタスクを取得する
List<Task> tasks = taskRepository.findByUserId(userId);

// Good: なぜそうしているか説明している
// ページネーションはPhase 2で実装予定のため、現時点では全件取得
List<Task> tasks = taskRepository.findByUserId(userId);
```

#### Javadoc / JSDoc

- **公開API**: 必ずドキュメントコメントを記載
- **パラメータと戻り値**: 型だけでは分からない情報を記載
- **例外**: スローされる例外を明示

```java
/**
 * 新しいタスクを作成します。
 *
 * @param userId タスクを作成するユーザーのID
 * @param request タスク作成リクエスト
 * @return 作成されたタスク
 * @throws ResourceNotFoundException ユーザーが存在しない場合
 */
public Task createTask(Long userId, TaskCreateRequest request) { ... }
```

---

## セキュリティプラクティス

### シークレット管理

#### 絶対に守るべきルール

⛔ **絶対禁止**:
- パスワード、APIキー、シークレットをコードにハードコード
- シークレットを含むファイルをGitにコミット
- ログにシークレットを出力

✅ **必ず行う**:
- 環境変数でシークレットを管理
- `.env` ファイルを `.gitignore` に追加
- `.env.example` でテンプレートを提供

#### 環境変数の使用

```yaml
# application.yml - 環境変数を参照
spring:
  datasource:
    url: jdbc:postgresql://${DB_HOST}:${DB_PORT}/${DB_NAME}
    username: ${DB_USER}
    password: ${DB_PASSWORD}

jwt:
  secret: ${JWT_SECRET}
```

```typescript
// フロントエンド - Vite環境変数
const apiUrl = import.meta.env.VITE_API_URL;
```

### 入力検証

- **すべての入力を検証**: クライアントとサーバー両方で
- **ホワイトリスト方式**: 許可する値を明示的に定義
- **SQLインジェクション対策**: パラメータバインディングを使用
- **XSS対策**: 出力時にエスケープ

```java
// バリデーション例
public class TaskCreateRequest {
    @NotBlank(message = "タイトルは必須です")
    @Size(max = 255, message = "タイトルは255文字以内で入力してください")
    private String title;
    
    @Size(max = 5000, message = "説明は5000文字以内で入力してください")
    private String description;
}
```

### 認証・認可

- **JWT**: トークンベースの認証
- **パスワード**: BCryptでハッシュ化
- **HTTPS**: 本番環境では必須
- **CORS**: 許可するオリジンを明示的に設定

---

## ドキュメント維持方針

### ドキュメントの種類と更新タイミング

| ドキュメント | 更新タイミング |
|-------------|--------------|
| **README.md** | 機能追加、セットアップ手順変更時 |
| **API仕様書** (OpenAPI) | APIエンドポイント追加・変更時 |
| **設計ドキュメント** | アーキテクチャ変更時 |
| **ADR** | 重要な技術的意思決定時 |

### ドキュメントの原則

1. **コードと同期**: 古いドキュメントは害になる
2. **DRY原則**: 同じ情報を複数箇所に書かない
3. **適切な粒度**: 詳細すぎず、概要すぎず
4. **検索可能**: 明確な構造とキーワード

### READMEの構成

```markdown
# プロジェクト名
- 概要（1-2文）
- 主要機能
- クイックスタート
- ドキュメントへのリンク
```

---

## バージョン管理

### コミットメッセージ規約

**Conventional Commits** 形式を採用:

```
<type>(<scope>): <subject>

<body>

<footer>
```

#### Type一覧

| Type | 説明 | 例 |
|------|------|-----|
| `feat` | 新機能 | `feat(task): add task completion toggle` |
| `fix` | バグ修正 | `fix(auth): correct token expiration` |
| `docs` | ドキュメント | `docs: update API documentation` |
| `style` | フォーマット | `style: apply prettier formatting` |
| `refactor` | リファクタリング | `refactor(service): extract validation logic` |
| `test` | テスト | `test(task): add unit tests for TaskService` |
| `chore` | 雑務 | `chore: update dependencies` |

#### コミットメッセージ例

```
feat(auth): implement JWT authentication

- Add JwtTokenProvider for token generation and validation
- Configure Spring Security with JWT filter
- Add login and register endpoints

Closes #12
```

### ブランチ戦略

```
main (本番)
  │
  └── develop (開発統合)
        │
        ├── feature/task-crud
        ├── feature/user-auth
        └── fix/login-validation
```

#### ブランチ命名規約

| ブランチタイプ | 命名パターン | 例 |
|--------------|------------|-----|
| 機能開発 | `feature/<機能名>` | `feature/task-crud` |
| バグ修正 | `fix/<バグ内容>` | `fix/login-error` |
| ホットフィックス | `hotfix/<内容>` | `hotfix/security-patch` |
| ドキュメント | `docs/<内容>` | `docs/api-guide` |

### プルリクエストチェックリスト

PRを作成する際に確認すべき項目:

- [ ] コードがビルドできる
- [ ] テストが通過する
- [ ] リンターエラーがない
- [ ] **ドキュメントを更新した**（該当する場合）
- [ ] コミットメッセージが規約に従っている
- [ ] 適切なレビュアーを指定した

---

## 依存関係管理

### セキュリティアップデート

| 優先度 | 対応期限 | 例 |
|--------|---------|-----|
| **Critical** | 24時間以内 | 認証バイパス、RCE |
| **High** | 1週間以内 | SQLインジェクション |
| **Medium** | 1ヶ月以内 | XSS |
| **Low** | 次回リリース | 情報漏洩（限定的） |

### 依存関係更新ポリシー

1. **セキュリティパッチ**: 即座に適用
2. **マイナーアップデート**: 月次で評価
3. **メジャーアップデート**: 計画的に対応（破壊的変更の確認）

### ツール

- **Dependabot**: 自動的な依存関係更新PR
- **npm audit / OWASP**: 脆弱性スキャン
- **Snyk** (将来): 継続的なセキュリティモニタリング

---

## Issue/PR運用ガイドライン

### Issue作成ガイドライン

#### 良いIssueの要素

1. **明確なタイトル**: 何についてのIssueか一目で分かる
2. **再現手順**: バグの場合、再現方法を詳細に
3. **期待される動作**: 何が正しい動作か
4. **実際の動作**: 何が起きているか
5. **環境情報**: 該当する場合

```markdown
## バグの概要
ログイン後、ダッシュボードが表示されない

## 再現手順
1. ログインページにアクセス
2. 正しい認証情報でログイン
3. 空白ページが表示される

## 期待される動作
ダッシュボードが表示される

## 環境
- Browser: Chrome 119
- OS: macOS 14
```

### Issueラベル

| ラベル | 説明 |
|--------|------|
| `bug` | バグ報告 |
| `enhancement` | 機能改善 |
| `documentation` | ドキュメント関連 |
| `good first issue` | 初心者向け |
| `help wanted` | 助けが必要 |
| `priority: high` | 優先度高 |

### PRレビューガイドライン

#### レビュアーが確認すべきこと

1. **機能要件**: 仕様通りに動作するか
2. **コード品質**: 命名、構造、可読性
3. **テスト**: 適切なテストがあるか
4. **セキュリティ**: 脆弱性がないか
5. **ドキュメント**: 必要な更新がされているか

#### 建設的なフィードバックの例

```markdown
// Good
「このメソッドは長いので、バリデーションロジックを
 別メソッドに抽出すると可読性が上がりそうです。
 例: `validateTaskRequest(request)`」

// Bad
「このコードは汚い」
```

---

## 関連ドキュメント

- [プロジェクト背景](./project-background.md) - 開発動機と学習目標
- [機能ロードマップ](./features-roadmap.md) - フェーズ別開発計画
- [技術スタック](../tech-stack.md) - 使用技術の詳細
- [アーキテクチャ](../architecture.md) - システム設計
