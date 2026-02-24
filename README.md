# Task Habit Tracker

タスクおよび習慣管理アプリケーション / Task and Habit Management Application

## 📖 概要 / Overview

このプロジェクトは、モダンなWebアプリケーション開発のベストプラクティスを学ぶための実践用リポジトリです。タスクおよび習慣管理アプリケーションを構築する過程で、バックエンド（Java/Spring Boot）からフロントエンド（React/TypeScript）、データベース設計、Docker化まで、フルスタック開発のスキルを段階的に習得します。

This project is a hands-on repository for learning modern web application development best practices. Through building a task and habit management application, you'll learn full-stack development skills including backend (Java/Spring Boot), frontend (React/TypeScript), database design, and Docker containerization.

## 🎯 主な機能 / Key Features

### 現在実装中 / Currently in Development

**Phase 1: MVP（最小実用製品）**
- ユーザー登録・ログイン（JWT認証）
- タスクのCRUD操作
- タスク完了切り替え
- 日ごとのタスクビュー

### 今後の計画 / Future Plans

**Phase 2: 機能拡張**
- 習慣トラッキング（毎日の達成記録）
- カレンダービュー＋ヒートマップ
- カテゴリ／タグ機能
- フィルタ・検索機能

**Phase 3: 高度な機能追加**
- 外部API連携
- リアルタイム通知
- チームタスク共有
- ファイル添付機能

詳細は [features-roadmap.md](docs/ja/features-roadmap.md) を参照してください。

## 🛠️ 技術スタック / Tech Stack

### バックエンド / Backend
- **言語**: Java 17+
- **フレームワーク**: Spring Boot 3.x
- **セキュリティ**: Spring Security + JWT認証
- **データベース**: PostgreSQL
- **ビルドツール**: Maven

### フロントエンド / Frontend
- **言語**: TypeScript
- **フレームワーク**: React 18+
- **スタイリング**: Tailwind CSS
- **状態管理**: (検討中)

### インフラ / Infrastructure
- **コンテナ**: Docker & Docker Compose
- **API仕様**: Swagger/OpenAPI

### テスト / Testing
- **バックエンド**: JUnit 5, Mockito
- **フロントエンド**: Jest, React Testing Library

詳細は [tech-stack.md](docs/tech-stack.md) を参照してください。

## 🚀 クイックスタート / Quick Start

### 前提条件 / Prerequisites

- Docker & Docker Compose
- Java 17+ (ローカル開発の場合)
- Node.js 18+ (ローカル開発の場合)

### セットアップ手順 / Setup Instructions

```bash
# リポジトリをクローン / Clone the repository
git clone https://github.com/TaishiYamaguchi/task-habit-tracker.git
cd task-habit-tracker

# 環境変数ファイルを作成 / Create environment file
cp .env.example .env
# .env ファイルを編集して必要な値を設定してください

# Docker Composeで起動 / Start with Docker Compose
docker-compose up -d

# アプリケーションにアクセス / Access the application
# Frontend: http://localhost:3000
# Backend API: http://localhost:8080
# API Documentation: http://localhost:8080/swagger-ui.html
```

※ 現在開発初期段階のため、上記のセットアップ手順は今後変更される可能性があります。

## 📚 ドキュメント / Documentation

- [開発背景と目的](docs/ja/project-background.md) - プロジェクトの背景、目的、学習目標
- [開発ガイドライン](docs/ja/development-guideline.md) - コーディング規約、開発プラクティス
- [機能ロードマップ](docs/ja/features-roadmap.md) - フェーズ別の機能詳細
- [技術スタック詳細](docs/tech-stack.md) - 使用技術の詳細説明
- [アーキテクチャ設計](docs/architecture.md) - システムアーキテクチャ概要
- [データベース設計](docs/database-design.md) - データベーススキーマとER図
- [Agent Skills 開発方針](docs/development/agent-skills.md) - AI支援ツールの活用方針

## 🤖 開発方針 / Development Policy

このプロジェクトでは、GitHub Copilot などの AI 支援ツール（Agent Skills）を積極的に活用し、開発効率とドキュメント品質の向上を図ります。

This project actively uses AI assistance tools (Agent Skills, e.g. GitHub Copilot) to improve development efficiency and documentation quality.

### 基本方針 / Principles

- AI 支援はドキュメント整備を最優先として開始します。  
  AI assistance starts with documentation improvements as the top priority.

- AI は小さな変更（ドキュメント・テスト・リファクタリング）を実装し、Pull Request を作成することがあります。  
  AI may implement small changes (docs / tests / refactors) and open pull requests.

- 「小さな変更」とは、外部から観察できる振る舞いを変えない（behavior-preserving）かつスコープが限定的な変更を指します。数値的な閾値は設けません。  
  "Small changes" means behavior-preserving modifications with a limited, well-defined scope. No numeric thresholds are imposed.

- マージの最終判断は常に人間（メンテナー）が行います。メンテナーがすべての変更に責任を持ちます。  
  Humans review and decide on merges; maintainers are accountable for all changes.

詳細は [docs/development/agent-skills.md](docs/development/agent-skills.md) を参照してください。

For full details, see [docs/development/agent-skills.md](docs/development/agent-skills.md).

## 🤝 コントリビューション / Contributing

このプロジェクトは主に個人学習目的ですが、フィードバックや提案は歓迎します。

Pull Requestを作成する際は、[Pull Request テンプレート](.github/PULL_REQUEST_TEMPLATE.md)に従ってください。

## 📝 ライセンス / License

MIT License

## 👤 作成者 / Author

Taishi Yamaguchi (@TaishiYamaguchi)