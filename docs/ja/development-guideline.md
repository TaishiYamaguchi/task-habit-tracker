# 開発ガイドライン

このドキュメントは、Task Habit Trackerプロジェクトの開発において守るべき品質基準、コーディング規約、およびベストプラクティスを定義します。

## 📋 目次

- [コード品質](#コード品質)
- [セキュリティプラクティス](#セキュリティプラクティス)
- [ドキュメント更新ポリシー](#ドキュメント更新ポリシー)
- [バージョン管理](#バージョン管理)
- [コミットメッセージ規約](#コミットメッセージ規約)
- [ブランチ戦略](#ブランチ戦略)
- [PRレビューチェックリスト](#prレビューチェックリスト)
- [依存関係とセキュリティ更新](#依存関係とセキュリティ更新)

## 🎯 コード品質

### テスト

**基本方針**:
- 新機能には必ずテストを追加する
- バグ修正時は、バグを再現するテストを先に書く
- テストカバレッジは主要機能で70%以上を目標とする

**テストの種類**:

1. **ユニットテスト**
   - 個々のメソッドやクラスの動作を検証
   - モックを活用して依存関係を分離
   - 高速に実行できるよう設計

2. **統合テスト**
   - 複数のコンポーネント間の連携を検証
   - データベースとの統合を含む
   - トランザクションのロールバックで状態を保つ

3. **E2Eテスト**（Phase 2以降）
   - ユーザーの操作フローを検証
   - 主要なユーザーストーリーをカバー

**バックエンドテストの例**:
```java
@SpringBootTest
@Transactional
class TaskServiceTest {
    
    @Autowired
    private TaskService taskService;
    
    @Test
    void createTask_ShouldReturnCreatedTask() {
        // Given
        TaskRequest request = new TaskRequest("Test Task");
        
        // When
        Task result = taskService.createTask(request);
        
        // Then
        assertNotNull(result.getId());
        assertEquals("Test Task", result.getTitle());
    }
}
```

**フロントエンドテストの例**:
```typescript
describe('TaskList Component', () => {
  it('should render tasks correctly', () => {
    const tasks = [{ id: 1, title: 'Test Task', completed: false }];
    render(<TaskList tasks={tasks} />);
    
    expect(screen.getByText('Test Task')).toBeInTheDocument();
  });
});
```

### 命名規則

**Java（バックエンド）**:
- **クラス名**: PascalCase（例: `TaskController`, `UserService`）
- **メソッド名**: camelCase（例: `getUserById`, `createTask`）
- **定数**: UPPER_SNAKE_CASE（例: `MAX_RETRY_COUNT`）
- **パッケージ名**: 小文字、ドット区切り（例: `com.taskhabit.service`）

**TypeScript/JavaScript（フロントエンド）**:
- **コンポーネント名**: PascalCase（例: `TaskList`, `UserProfile`）
- **関数名**: camelCase（例: `handleClick`, `fetchTasks`）
- **定数**: UPPER_SNAKE_CASE（例: `API_BASE_URL`）
- **ファイル名**: kebab-case（例: `task-list.tsx`, `user-service.ts`）

### コメント

**コメントが必要な場合**:
- 複雑なビジネスロジックの説明
- 技術的な制約や回避策の理由
- 公開APIの仕様（JavaDoc/TSDoc）

**コメントが不要な場合**:
- コードを見れば明らかな内容
- 適切な命名で説明できる場合

**良い例**:
```java
/**
 * ユーザーの習慣達成率を計算します。
 * 
 * @param userId ユーザーID
 * @param period 計算期間（日数）
 * @return 達成率（0.0～1.0）
 */
public double calculateCompletionRate(Long userId, int period) {
    // 過去N日間の習慣記録を取得
    List<HabitLog> logs = habitRepository.findRecentLogs(userId, period);
    
    // 達成率を計算（達成数 / 期待数）
    return (double) logs.stream().filter(HabitLog::isCompleted).count() / period;
}
```

### コードスタイル

**フォーマット**:
- **Java**: Google Java Style Guide または Spring Boot推奨スタイル
- **TypeScript**: Prettier + ESLintの推奨設定

**一般的な原則**:
- DRY（Don't Repeat Yourself）: 重複コードを避ける
- SOLID原則: 特に単一責任の原則を重視
- YAGNI（You Aren't Gonna Need It）: 過度な抽象化を避ける

## 🔒 セキュリティプラクティス

### 秘匿情報管理

**絶対にコミットしてはいけない情報**:
- データベースの認証情報
- JWTシークレットキー
- APIキー、トークン
- その他の機密情報

**環境変数の使用**:
```bash
# ❌ 悪い例: ハードコード
String dbPassword = "mysecretpassword";

# ✅ 良い例: 環境変数から取得
String dbPassword = System.getenv("DB_PASSWORD");
```

**`.env` ファイルの管理**:
- `.env` は `.gitignore` に追加
- `.env.example` をテンプレートとして提供
- 本番環境では環境変数を直接設定

### セキュリティ対策

**OWASP Top 10 対策**:

1. **SQLインジェクション**: JPA/Hibernateのパラメータバインディングを使用
2. **XSS（クロスサイトスクリプティング）**: 入力値のサニタイズとエスケープ
3. **認証の不備**: Spring Securityによる堅牢な認証実装
4. **機密データの露出**: パスワードのハッシュ化、JWTの適切な管理
5. **アクセス制御の不備**: ロールベースのアクセス制御（RBAC）

**パスワード管理**:
```java
// BCryptでパスワードをハッシュ化
@Bean
public PasswordEncoder passwordEncoder() {
    return new BCryptPasswordEncoder();
}
```

**JWT管理**:
- 適切な有効期限設定（推奨: 24時間）
- リフレッシュトークンの実装（Phase 2）
- セキュアなトークン保存（httpOnlyクッキー推奨）

### 入力検証

**バックエンド**:
```java
public class TaskRequest {
    @NotBlank(message = "タイトルは必須です")
    @Size(max = 200, message = "タイトルは200文字以内で入力してください")
    private String title;
    
    @Size(max = 1000, message = "説明は1000文字以内で入力してください")
    private String description;
}
```

**フロントエンド**:
- ユーザー入力の検証（フォームバリデーション）
- APIレスポンスの型チェック
- XSS対策（DOMPurifyなどの利用を検討）

## 📚 ドキュメント更新ポリシー

### ドキュメント作成の原則

1. **タイムリーな更新**: コード変更と同時にドキュメントも更新
2. **明確さ**: 技術的背景がない人でも理解できる説明
3. **例示**: 具体的なコード例やユースケースを含める
4. **最新性の維持**: 古くなった情報は削除または更新

### 更新が必要なケース

**必須**:
- 新機能の追加
- APIエンドポイントの変更
- 環境構築手順の変更
- アーキテクチャの大きな変更

**推奨**:
- バグ修正で設計意図が明確になる場合
- パフォーマンス改善の手法追加
- トラブルシューティング情報の追加

### ドキュメントの種類と役割

| ドキュメント | 更新頻度 | 目的 |
|------------|---------|------|
| README.md | プロジェクト初期＋重要な変更時 | プロジェクト概要、クイックスタート |
| development-guideline.md | ガイドライン追加・変更時 | 開発基準の定義 |
| features-roadmap.md | フェーズ完了時 | 機能の進捗管理 |
| architecture.md | アーキテクチャ変更時 | 設計思想の共有 |
| API仕様（Swagger） | API変更時 | APIの使用方法 |

### コード内ドキュメント

**JavaDoc**:
```java
/**
 * タスクを作成します。
 * 
 * @param request タスク作成リクエスト
 * @param userId 作成者のユーザーID
 * @return 作成されたタスク
 * @throws ValidationException リクエストが不正な場合
 */
public Task createTask(TaskRequest request, Long userId) {
    // ...
}
```

**TSDoc**:
```typescript
/**
 * タスク一覧を取得します
 * @param filters - フィルタ条件
 * @returns Promise<Task[]> タスクの配列
 */
async function fetchTasks(filters: TaskFilters): Promise<Task[]> {
  // ...
}
```

## 🌿 バージョン管理

### Gitのベストプラクティス

1. **小さなコミット**: 1つの論理的変更につき1コミット
2. **頻繁なコミット**: 作業が一段落したらコミット
3. **わかりやすいコミットメッセージ**: 変更内容が明確にわかるように
4. **定期的なプッシュ**: 1日の終わりには必ずプッシュ

### `.gitignore` の管理

以下のファイルは必ずignoreする:
- 環境変数ファイル（`.env`）
- ビルド成果物（`target/`, `dist/`, `build/`）
- IDE設定（`.idea/`, `.vscode/`）※共有したい設定は除く
- 依存関係（`node_modules/`）
- ログファイル（`*.log`）

## 💬 コミットメッセージ規約

### Conventional Commits

このプロジェクトでは [Conventional Commits](https://www.conventionalcommits.org/) を採用します。

**フォーマット**:
```
<type>(<scope>): <subject>

<body>

<footer>
```

**Type の種類**:
- `feat`: 新機能
- `fix`: バグ修正
- `docs`: ドキュメントのみの変更
- `style`: コードの意味に影響しない変更（フォーマット、セミコロンなど）
- `refactor`: バグ修正や機能追加を含まないコード変更
- `perf`: パフォーマンス改善
- `test`: テストの追加・修正
- `chore`: ビルドプロセスやツールの変更

**例**:
```bash
# 良い例
feat(auth): JWT認証機能を実装

- Spring Securityの設定を追加
- JWTトークンの生成・検証ロジックを実装
- ログインエンドポイントを追加

Closes #123

# 別の良い例
fix(task): タスク削除時の例外処理を修正

存在しないタスクを削除しようとした際に404を返すように変更

# シンプルな例
docs: README にセットアップ手順を追加
```

**コミットメッセージの書き方**:
- **subject**: 50文字以内、現在形で記述
- **body**: 詳細な説明（必要に応じて）
- **footer**: Issue番号の参照（`Closes #123`、`Refs #456`）

## 🌳 ブランチ戦略

### ブランチモデル

**Git Flow** を簡略化したモデルを採用:

```
main (本番相当)
  └─ develop (開発統合)
       ├─ feature/task-crud (機能開発)
       ├─ feature/habit-tracking (機能開発)
       └─ fix/login-bug (バグ修正)
```

### ブランチの種類

1. **`main` ブランチ**
   - 常に本番デプロイ可能な状態
   - 直接コミット禁止
   - `develop` からのマージのみ

2. **`develop` ブランチ**
   - 開発中の最新コード
   - 機能ブランチの統合先
   - 直接コミット禁止（緊急時を除く）

3. **`feature/*` ブランチ**
   - 新機能開発用
   - `develop` から分岐
   - `develop` へマージ

4. **`fix/*` ブランチ**
   - バグ修正用
   - `develop` から分岐
   - `develop` へマージ

5. **`hotfix/*` ブランチ**（必要に応じて）
   - 本番環境の緊急修正用
   - `main` から分岐
   - `main` と `develop` 両方にマージ

### ブランチ命名規則

```bash
# 機能開発
feature/user-authentication
feature/task-filtering

# バグ修正
fix/login-error
fix/task-delete-issue

# ホットフィックス
hotfix/security-patch
```

### ワークフロー

```bash
# 1. 最新のdevelopを取得
git checkout develop
git pull origin develop

# 2. 機能ブランチを作成
git checkout -b feature/new-feature

# 3. 開発・コミット
git add .
git commit -m "feat: 新機能を追加"

# 4. プッシュしてPR作成
git push origin feature/new-feature
# GitHub上でPRを作成
```

## ✅ PRレビューチェックリスト

### コード品質

- [ ] コードはプロジェクトのコーディング規約に従っているか
- [ ] 重複コードがないか（DRY原則）
- [ ] 適切な命名がされているか
- [ ] 複雑すぎるメソッド/関数はないか（1メソッド30行以内が目安）
- [ ] エラーハンドリングは適切か

### テスト

- [ ] 新機能に対するテストが追加されているか
- [ ] すべてのテストが通過しているか
- [ ] エッジケースがカバーされているか
- [ ] テストの命名が明確か

### セキュリティ

- [ ] 入力値の検証が行われているか
- [ ] 秘匿情報がハードコードされていないか
- [ ] SQLインジェクション対策ができているか
- [ ] XSS対策ができているか

### ドキュメント

- [ ] 必要なドキュメントが更新されているか
- [ ] コードコメントは適切か
- [ ] API仕様が更新されているか（該当する場合）

### Git

- [ ] コミットメッセージは規約に従っているか
- [ ] PRのタイトル・説明は明確か
- [ ] 不要なファイルがコミットされていないか
- [ ] マージ先ブランチは正しいか

### パフォーマンス

- [ ] 不要なデータベースクエリはないか
- [ ] N+1問題が発生していないか
- [ ] 大量のデータでも動作するか

## 📦 依存関係とセキュリティ更新

### 依存関係管理

**原則**:
- 必要最小限の依存関係を保つ
- 定期的に依存関係を更新する（月1回）
- セキュリティアップデートは即座に適用

**バックエンド（Maven）**:
```bash
# 依存関係の確認
mvn dependency:tree

# 古い依存関係の確認
mvn versions:display-dependency-updates
```

**フロントエンド（npm）**:
```bash
# 依存関係の確認
npm list

# 古い依存関係の確認
npm outdated

# 脆弱性チェック
npm audit

# 脆弱性の自動修正
npm audit fix
```

### セキュリティ更新

1. **GitHub Dependabotの活用**
   - 自動的にPRが作成される
   - セキュリティアップデートは優先的にマージ

2. **定期的なチェック**
   - 週1回: セキュリティアラートの確認
   - 月1回: 依存関係の全体的な更新

3. **更新手順**
   ```bash
   # 1. 更新可能なパッケージを確認
   npm outdated
   
   # 2. テスト環境で更新
   npm update
   
   # 3. テスト実行
   npm test
   
   # 4. 問題なければコミット
   git commit -m "chore: 依存関係を更新"
   ```

### バージョン管理ポリシー

- **メジャーバージョン**: 破壊的変更を含む場合は慎重に検討
- **マイナーバージョン**: 新機能追加は積極的に更新
- **パッチバージョン**: バグ修正は即座に適用

---

このガイドラインは、プロジェクトの進化に応じて継続的に更新されます。新しいベストプラクティスを学んだ際は、積極的にこのドキュメントに反映してください。
