# リポジトリ構造定義書 (Repository Structure Document)

## プロジェクト構造

```
meshi-map/
├── apps/                          # アプリケーション
│   ├── web/                       # フロントエンド (Next.js)
│   └── api/                       # バックエンド (Hono)
├── packages/                      # 共有パッケージ
│   ├── shared/                    # 共通型定義・ユーティリティ
│   └── database/                  # Supabaseクライアント
├── docs/                          # プロジェクトドキュメント
├── .claude/                       # Claude Code設定
├── .steering/                     # ステアリングファイル
├── .devcontainer/                 # 開発コンテナ設定
└── [設定ファイル類]
```

## モノレポ構成

### 採用理由

| 利点 | 説明 |
|------|------|
| コード共有の容易さ | `packages/shared` で型定義を共有し、フロントエンド・バックエンド間の型安全性を確保 |
| 依存関係の一元管理 | ルートの `package.json` で全パッケージの依存を管理、バージョン不整合を防止 |
| 将来の拡張性 | React Native等のモバイルアプリを `apps/mobile` として追加可能 |
| 統一されたビルド・テスト | `npm run test` で全パッケージのテストを一括実行可能 |

### 制約・注意点

| 制約 | 対策 |
|------|------|
| ビルド時間の増加 | workspace単位でのビルド（`npm run build -w apps/web`）も可能にする |
| 依存関係の複雑化 | 循環依存を禁止し、依存方向を厳格に管理（本ドキュメント参照） |
| IDE対応 | TypeScriptのプロジェクト参照（`tsconfig.json`の`references`）を活用 |

### npm workspaces

```json
// package.json (ルート)
{
  "name": "meshi-map",
  "private": true,
  "workspaces": [
    "apps/*",
    "packages/*"
  ]
}
```

### パッケージ間の依存関係

```
apps/web (Next.js)
    ├── packages/shared
    └── (packages/database は使用しない - バックエンド経由)

apps/api (Hono)
    ├── packages/shared
    └── packages/database
```

## ディレクトリ詳細

### apps/web/ (フロントエンド)

**役割**: ユーザーインターフェースの提供（地図表示、店舗検索、店舗登録）

```
apps/web/
├── src/
│   ├── app/                       # Next.js App Router
│   │   ├── layout.tsx             # ルートレイアウト
│   │   ├── page.tsx               # ホーム（地図画面）
│   │   └── search/
│   │       └── page.tsx           # 検索画面
│   ├── components/                # Reactコンポーネント
│   │   ├── map/                   # 地図関連
│   │   │   ├── MapView.tsx        # 地図表示
│   │   │   ├── ShopMarker.tsx     # 店舗マーカー
│   │   │   └── ShopPopup.tsx      # 店舗詳細ポップアップ
│   │   ├── search/                # 検索関連
│   │   │   ├── SearchBar.tsx      # 検索バー
│   │   │   └── SearchResultList.tsx # 検索結果一覧
│   │   ├── common/                # 共通UI
│   │   │   ├── Button.tsx
│   │   │   ├── Loading.tsx
│   │   │   └── ErrorMessage.tsx
│   │   └── onboarding/            # オンボーディング
│   │       └── OnboardingDialog.tsx
│   ├── hooks/                     # カスタムフック
│   │   ├── useAnonymousUser.ts    # 匿名ユーザーID管理
│   │   ├── useGeolocation.ts      # 位置情報取得
│   │   └── useRegistrations.ts    # 登録店舗管理
│   ├── lib/                       # ライブラリ・ユーティリティ
│   │   ├── api-client.ts          # APIクライアント
│   │   └── storage.ts             # LocalStorage操作
│   └── styles/                    # スタイル
│       └── globals.css
├── public/                        # 静的ファイル
│   └── icons/
├── next.config.js
├── tailwind.config.js
├── tsconfig.json
└── package.json
```

**命名規則**:
- コンポーネント: PascalCase（`MapView.tsx`）
- フック: camelCase、`use`プレフィックス（`useGeolocation.ts`）
- ユーティリティ: kebab-case（`api-client.ts`）

**依存関係**:
- 依存可能: `packages/shared`
- 依存禁止: `packages/database`、`apps/api`

---

### apps/api/ (バックエンド)

**役割**: APIエンドポイントの提供、外部API（Google Places）との通信、データベース操作

```
apps/api/
├── src/
│   ├── index.ts                   # エントリーポイント
│   ├── routes/                    # APIルート定義
│   │   ├── shops.ts               # /api/shops/*
│   │   └── registrations.ts       # /api/registrations/*
│   ├── services/                  # ビジネスロジック
│   │   ├── ShopSearchService.ts   # 店舗検索
│   │   └── RegistrationService.ts # 店舗登録
│   ├── repositories/              # データアクセス
│   │   ├── ShopRepository.ts      # Shopテーブル操作
│   │   └── RegistrationRepository.ts # Registrationテーブル操作
│   ├── middleware/                # ミドルウェア
│   │   ├── cors.ts                # CORS設定
│   │   ├── errorHandler.ts        # エラーハンドリング
│   │   └── validateUserId.ts      # 匿名ユーザーID検証
│   ├── validators/                # バリデーション
│   │   └── requestValidators.ts   # リクエストバリデーション
│   └── external/                  # 外部API連携
│       └── googlePlaces.ts        # Google Places API
├── wrangler.toml                  # Cloudflare Workers設定
├── tsconfig.json
└── package.json
```

**命名規則**:
- サービス: PascalCase + `Service`サフィックス（`ShopSearchService.ts`）
- リポジトリ: PascalCase + `Repository`サフィックス（`ShopRepository.ts`）
- ルート: kebab-case（`shops.ts`）
- ミドルウェア: camelCase（`errorHandler.ts`）

**依存関係**:
- 依存可能: `packages/shared`、`packages/database`
- 依存禁止: `apps/web`

**レイヤー間の依存**:
```
routes (APIレイヤー)
    ↓
services (サービスレイヤー)
    ↓
repositories (リポジトリレイヤー)
    ↓
database (データレイヤー)
```

---

### packages/shared/ (共有パッケージ)

**役割**: フロントエンド・バックエンド間で共有する型定義とユーティリティ

```
packages/shared/
├── src/
│   ├── types/                     # 型定義
│   │   ├── shop.ts                # Shop型
│   │   ├── registration.ts        # Registration型
│   │   ├── api.ts                 # APIリクエスト/レスポンス型
│   │   └── index.ts               # エクスポート
│   ├── constants/                 # 定数
│   │   ├── api-endpoints.ts       # APIエンドポイント定義
│   │   └── error-codes.ts         # エラーコード
│   ├── utils/                     # ユーティリティ
│   │   └── validation.ts          # 共通バリデーション
│   └── index.ts                   # エントリーポイント
├── tsconfig.json
└── package.json
```

**命名規則**:
- 型定義: kebab-case（`shop.ts`）
- 定数: kebab-case（`api-endpoints.ts`）
- ユーティリティ: kebab-case（`validation.ts`）

**依存関係**:
- 依存可能: 外部ライブラリのみ
- 依存禁止: `apps/*`、`packages/database`

---

### packages/database/ (データベースパッケージ)

**役割**: Supabaseクライアントの初期化とデータベーススキーマ定義

```
packages/database/
├── src/
│   ├── client.ts                  # Supabaseクライアント
│   ├── schema/                    # スキーマ定義
│   │   ├── shops.ts               # Shopsテーブル
│   │   └── registrations.ts       # Registrationsテーブル
│   └── index.ts                   # エクスポート
├── migrations/                    # マイグレーションファイル
│   └── 001_initial_schema.sql
├── tsconfig.json
└── package.json
```

**命名規則**:
- スキーマ: kebab-case、テーブル名と同一（`shops.ts`）
- マイグレーション: `[番号]_[説明].sql`

**依存関係**:
- 依存可能: `packages/shared`（型定義のみ）、`@supabase/supabase-js`
- 依存禁止: `apps/*`

---

### docs/ (ドキュメント)

**役割**: プロジェクトドキュメントの管理

```
docs/
├── ideas/                         # アイデア・下書き
│   └── initial-requirements.md
├── product-requirements.md        # プロダクト要求定義書
├── functional-design.md           # 機能設計書
├── architecture.md                # 技術仕様書
├── repository-structure.md        # リポジトリ構造定義書（本ドキュメント）
├── development-guidelines.md      # 開発ガイドライン
└── glossary.md                    # 用語集
```

---

### .steering/ (ステアリングファイル)

**役割**: 特定の開発作業における「今回何をするか」を定義

```
.steering/
└── [YYYYMMDD]-[task-name]/
    ├── requirements.md            # 今回の作業の要求内容
    ├── design.md                  # 変更内容の設計
    └── tasklist.md                # タスクリスト
```

**命名規則**: `20250115-add-shop-filter` 形式

**Git管理方針**:
- `.steering/` はGit管理下に置き、プロジェクトの開発履歴として保持
- 完了した作業の知見は、必要に応じて `docs/` 配下の永続ドキュメントに反映

---

### .claude/ (Claude Code設定)

**役割**: Claude Code設定とカスタマイズ

```
.claude/
├── commands/                      # スラッシュコマンド
├── skills/                        # タスクモード別スキル
└── agents/                        # サブエージェント定義
```

## テストディレクトリ

### ユニットテスト・統合テスト

各アプリケーション・パッケージ内に `__tests__/` ディレクトリを配置:

```
apps/web/
└── __tests__/                     # ユニットテスト
    ├── components/                # コンポーネントテスト
    │   └── MapView.test.tsx
    └── hooks/                     # フックテスト
        └── useGeolocation.test.ts

apps/api/
└── __tests__/                     # ユニットテスト・統合テスト
    ├── routes/                    # APIルートテスト
    │   └── shops.test.ts
    ├── services/                  # サービステスト
    │   └── ShopSearchService.test.ts
    └── integration/               # 統合テスト
        └── registration-flow.test.ts
```

### E2Eテスト（Playwright）

Playwrightの標準構成に従い、プロジェクトルートに配置:

```
meshi-map/
├── tests/                         # E2Eテストディレクトリ
│   └── e2e/
│       ├── search-and-register.spec.ts
│       ├── onboarding.spec.ts
│       └── registration-flow.spec.ts
└── playwright.config.ts           # Playwright設定
```

**命名規則**:
- ユニットテスト: `[対象ファイル名].test.ts(x)`
- 統合テスト: `[シナリオ].test.ts`
- E2Eテスト: `[シナリオ].spec.ts`

**テストフレームワーク**（architecture.md準拠）:
- ユニットテスト・統合テスト: Vitest
- E2Eテスト: Playwright

## ファイル配置規則

### ソースファイル

| ファイル種別 | 配置先 | 命名規則 | 例 |
|------------|--------|---------|-----|
| Reactコンポーネント | `apps/web/src/components/` | PascalCase | `MapView.tsx` |
| カスタムフック | `apps/web/src/hooks/` | camelCase + use | `useGeolocation.ts` |
| APIルート | `apps/api/src/routes/` | kebab-case | `shops.ts` |
| サービス | `apps/api/src/services/` | PascalCase + Service | `ShopSearchService.ts` |
| リポジトリ | `apps/api/src/repositories/` | PascalCase + Repository | `ShopRepository.ts` |
| 型定義 | `packages/shared/src/types/` | kebab-case | `shop.ts` |
| 定数 | `packages/shared/src/constants/` | kebab-case | `api-endpoints.ts` |

### 設定ファイル

| ファイル | 配置先 | 説明 |
|---------|--------|------|
| `package.json` | 各ディレクトリ | パッケージ設定 |
| `tsconfig.json` | 各ディレクトリ | TypeScript設定 |
| `next.config.js` | `apps/web/` | Next.js設定 |
| `wrangler.toml` | `apps/api/` | Cloudflare Workers設定 |
| `.env.local` | `apps/web/` | ローカル環境変数（Git除外） |
| `.dev.vars` | `apps/api/` | Workers開発用環境変数（Git除外） |

## ルートディレクトリの設定ファイル

```
meshi-map/
├── package.json                   # ルートpackage.json (workspaces)
├── tsconfig.base.json             # 共通TypeScript設定
├── .gitignore                     # Git除外設定
├── .prettierrc                    # Prettier設定
├── .eslintrc.js                   # ESLint設定
├── README.md                      # プロジェクト概要
└── CLAUDE.md                      # Claude Code設定
```

## 依存関係のルール

### レイヤー間の依存（apps/api内）

```
routes (APIレイヤー)
    ↓ (OK)
services (サービスレイヤー)
    ↓ (OK)
repositories (リポジトリレイヤー)
    ↓ (OK)
external / database (データレイヤー)
```

**禁止される依存**:
- `repositories` → `routes` (❌)
- `services` → `routes` (❌)
- `external` → `services` (❌)

### パッケージ間の依存

```
apps/web
    ↓ (OK)
packages/shared

apps/api
    ↓ (OK)
packages/shared
    ↓ (OK)
packages/database
```

**禁止される依存**:
- `apps/web` → `packages/database` (❌ バックエンド経由でアクセス)
- `apps/web` → `apps/api` (❌ 直接importしない)
- `packages/*` → `apps/*` (❌)

### 循環依存の禁止

```typescript
// ❌ 悪い例: 循環依存
// services/ShopSearchService.ts
import { RegistrationService } from './RegistrationService';

// services/RegistrationService.ts
import { ShopSearchService } from './ShopSearchService';  // 循環依存！
```

**解決策**: 共通の型やインターフェースを`packages/shared`に抽出

## スケーリング戦略

### 機能追加時の配置

| 機能規模 | 配置方針 | 例 |
|---------|---------|-----|
| 小規模 | 既存ディレクトリに追加 | 新しいAPI endpoint |
| 中規模 | サブディレクトリを作成 | `components/filter/` |
| 大規模 | 新しいパッケージを作成 | `packages/analytics/` |

### スケールアップの閾値

| 指標 | 閾値（80%到達） | 測定方法 | 測定頻度 | 対策 |
|------|---------------|---------|---------|------|
| ユーザー数 | 500人超過 | Supabase Analytics（匿名ユーザーID数） | 週次 | 有料プランへの移行検討 |
| 店舗数 | 5,000店舗超過 | `SELECT COUNT(*) FROM shops` | 週次 | インデックスの追加、クエリ最適化 |
| 登録数 | 50,000件超過 | `SELECT COUNT(*) FROM registrations` | 週次 | ページネーションの実装 |
| Supabase DB容量 | 400MB超過 | Supabase Dashboard「Database Usage」 | 週次 | データ圧縮、アーカイブ検討 |
| ファイルサイズ | 500行超過 | ESLintルール `max-lines`、PR時自動チェック | PR毎 | ファイル分割実施 |

### ファイルサイズの管理

- **推奨**: 300行以下
- **リファクタリング検討**: 300-500行
- **分割必須**: 500行以上

**分割例**:
```typescript
// Before: 1ファイルに全機能
// components/MapView.tsx (600行)

// After: 責務ごとに分割
components/map/
├── MapView.tsx           # メインコンポーネント (150行)
├── MapControls.tsx       # 地図操作UI (100行)
├── MarkerCluster.tsx     # マーカークラスタリング (150行)
└── hooks/
    └── useMapState.ts    # 地図状態管理 (100行)
```

## 除外設定

### .gitignore

```gitignore
# 依存関係
node_modules/

# ビルド成果物
dist/
.next/
.wrangler/

# 環境変数
.env
.env.local
.dev.vars

# NOTE: .steering/ はGit管理下に置き、開発履歴として保持します

# ログ・キャッシュ
*.log
.cache/

# OS固有
.DS_Store
Thumbs.db

# IDE
.idea/
.vscode/
*.swp
```

### .prettierignore / .eslintignore

```
dist/
.next/
node_modules/
coverage/
*.md
```
