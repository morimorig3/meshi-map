# 開発ガイドライン (Development Guidelines)

## コーディング規約

### 命名規則

#### 変数・関数

**TypeScript**:
```typescript
// ✅ 良い例
const shopSearchResults = await searchShops(keyword);
function validateRegistrationData(data: RegistrationData): void { }
const isRegistered = checkIfRegistered(placeId);

// ❌ 悪い例
const data = await search(kw);
function validate(d: any): void { }
const flag = check(id);
```

**原則**:
- 変数: camelCase、名詞または名詞句
- 関数: camelCase、動詞で始める
- 定数: UPPER_SNAKE_CASE
- Boolean: `is`, `has`, `should`, `can`で始める

#### クラス・インターフェース

```typescript
// クラス: PascalCase、名詞 + 役割サフィックス
class ShopSearchService { }
class RegistrationRepository { }

// インターフェース: PascalCase
interface Shop { }
interface RegistrationWithShop { }

// 型エイリアス: PascalCase
type ShopCategory = 'ラーメン' | 'イタリアン' | '寿司';
type Nullable<T> = T | null;
```

#### ファイル名

| 種別 | 命名規則 | 例 |
|------|---------|-----|
| Reactコンポーネント | PascalCase | `MapView.tsx`, `ShopPopup.tsx` |
| カスタムフック | camelCase + use | `useGeolocation.ts`, `useRegistrations.ts` |
| サービス | PascalCase + Service | `ShopSearchService.ts` |
| リポジトリ | PascalCase + Repository | `ShopRepository.ts` |
| 型定義 | kebab-case | `shop.ts`, `registration.ts` |
| 定数 | kebab-case | `api-endpoints.ts`, `error-codes.ts` |
| ユーティリティ | kebab-case | `api-client.ts`, `storage.ts` |

### コードフォーマット

**インデント**: 2スペース

**行の長さ**: 最大100文字

**Prettier設定** (`.prettierrc`):
```json
{
  "semi": true,
  "singleQuote": true,
  "tabWidth": 2,
  "trailingComma": "es5",
  "printWidth": 100
}
```

### コメント規約

**関数・クラスのドキュメント（TSDoc）**:
```typescript
/**
 * 店舗を検索する
 *
 * @param keyword - 検索キーワード（店舗名）
 * @param location - 現在地（検索結果の並び順に使用）
 * @returns 検索結果の店舗リスト
 * @throws {ValidationError} キーワードが空または長すぎる場合
 * @throws {ExternalApiError} Google Places APIエラーの場合
 */
async function searchShops(
  keyword: string,
  location?: { lat: number; lng: number }
): Promise<SearchResultShop[]> {
  // 実装
}
```

**インラインコメント**:
```typescript
// ✅ 良い例: なぜそうするかを説明
// Google Places APIの無料枠を考慮し、1回の検索で最大20件に制限
const MAX_SEARCH_RESULTS = 20;

// ❌ 悪い例: 何をしているかを説明（コードを見れば分かる）
// 検索結果を20件に設定
const MAX_SEARCH_RESULTS = 20;
```

### エラーハンドリング

**カスタムエラークラス**:
```typescript
// packages/shared/src/errors/ValidationError.ts
export class ValidationError extends Error {
  constructor(
    message: string,
    public field: string,
    public value: unknown
  ) {
    super(message);
    this.name = 'ValidationError';
  }
}

// packages/shared/src/errors/NotFoundError.ts
export class NotFoundError extends Error {
  constructor(
    public resource: string,
    public id: string
  ) {
    super(`${resource} not found: ${id}`);
    this.name = 'NotFoundError';
  }
}

// packages/shared/src/errors/ExternalApiError.ts
export class ExternalApiError extends Error {
  constructor(
    public service: string,
    message: string,
    public cause?: Error
  ) {
    super(`${service} API error: ${message}`);
    this.name = 'ExternalApiError';
  }
}
```

**エラーハンドリングパターン**:
```typescript
// ✅ 良い例: 適切なエラーハンドリング
async function getRegistration(id: string, userId: string): Promise<Registration> {
  try {
    const registration = await repository.findById(id);

    if (!registration) {
      throw new NotFoundError('Registration', id);
    }

    if (registration.anonymousUserId !== userId) {
      throw new ForbiddenError('他のユーザーの登録にはアクセスできません');
    }

    return registration;
  } catch (error) {
    if (error instanceof NotFoundError || error instanceof ForbiddenError) {
      throw error; // 予期されるエラーはそのまま伝播
    }

    // 予期しないエラーはラップして伝播
    throw new DatabaseError('登録の取得に失敗しました', error as Error);
  }
}

// ❌ 悪い例: エラーを無視
async function getRegistration(id: string): Promise<Registration | null> {
  try {
    return await repository.findById(id);
  } catch (error) {
    return null; // エラー情報が失われる
  }
}
```

### React / Next.js 固有の規約

**コンポーネントの構造**:
```typescript
// ✅ 良い例: 明確な構造
import { useState, useEffect } from 'react';
import { useRegistrations } from '@/hooks/useRegistrations';
import type { Shop } from '@meshi-map/shared';

interface ShopMarkerProps {
  shop: Shop;
  onClick: (shop: Shop) => void;
}

export function ShopMarker({ shop, onClick }: ShopMarkerProps) {
  // フック
  const [isHovered, setIsHovered] = useState(false);

  // イベントハンドラ
  const handleClick = () => {
    onClick(shop);
  };

  // レンダリング
  return (
    <Marker
      position={[shop.latitude, shop.longitude]}
      onClick={handleClick}
    />
  );
}
```

**カスタムフックの設計**:
```typescript
// ✅ 良い例: 単一責務、再利用可能
export function useGeolocation() {
  const [location, setLocation] = useState<GeolocationPosition | null>(null);
  const [error, setError] = useState<GeolocationPositionError | null>(null);
  const [isLoading, setIsLoading] = useState(true);

  useEffect(() => {
    navigator.geolocation.getCurrentPosition(
      (position) => {
        setLocation(position);
        setIsLoading(false);
      },
      (error) => {
        setError(error);
        setIsLoading(false);
      }
    );
  }, []);

  return { location, error, isLoading };
}
```

### Hono 固有の規約

**ルート定義**:
```typescript
// ✅ 良い例: 責務の分離
// apps/api/src/routes/shops.ts
import { Hono } from 'hono';
import { ShopSearchService } from '../services/ShopSearchService';
import { validateSearchQuery } from '../validators/requestValidators';

const shops = new Hono();

shops.get('/search', async (c) => {
  const keyword = c.req.query('q');

  // バリデーション
  validateSearchQuery(keyword);

  // サービス呼び出し
  const service = new ShopSearchService();
  const results = await service.search(keyword);

  return c.json({ shops: results });
});

export { shops };
```

**ミドルウェア**:
```typescript
// apps/api/src/middleware/validateUserId.ts
import { Context, Next } from 'hono';
import { ValidationError } from '@meshi-map/shared';

const UUID_V4_REGEX = /^[0-9a-f]{8}-[0-9a-f]{4}-4[0-9a-f]{3}-[89ab][0-9a-f]{3}-[0-9a-f]{12}$/i;

export async function validateUserId(c: Context, next: Next) {
  const userId = c.req.header('X-Anonymous-User-Id');

  if (!userId) {
    throw new ValidationError('X-Anonymous-User-Id header is required', 'header', null);
  }

  if (!UUID_V4_REGEX.test(userId)) {
    throw new ValidationError('Invalid anonymous user ID format', 'X-Anonymous-User-Id', userId);
  }

  await next();
}
```

## Git運用ルール

### ブランチ戦略（GitHub Flow）

architecture.mdのCI/CDパイプラインと整合した運用です。

**ブランチ構成**:
```
main (本番環境)
├── feature/* (新機能開発)
├── fix/* (バグ修正)
├── refactor/* (リファクタリング)
└── docs/* (ドキュメント更新)
```

**ブランチ命名規則**:
- `feature/add-shop-filter`: 新機能追加
- `fix/search-error-handling`: バグ修正
- `refactor/api-client`: リファクタリング
- `docs/update-readme`: ドキュメント更新

**運用ルール**:
- `main`: 常に本番デプロイ可能な状態を維持
- 全ての変更はPRを通じて`main`にマージ
- 直接コミット禁止: `main`への直接pushは禁止、必ずPRレビューを経由
- マージ方針: feature→main は squash merge
- PRはCIが全てパスしてからマージ可能

### コミットメッセージ規約

**Conventional Commits**:
```
<type>(<scope>): <subject>

<body>

<footer>
```

**Type一覧**:
| Type | 説明 | 例 |
|------|------|-----|
| feat | 新機能 | `feat(search): 店舗検索機能を追加` |
| fix | バグ修正 | `fix(map): マーカー表示の位置ずれを修正` |
| docs | ドキュメント | `docs: READMEにセットアップ手順を追加` |
| style | フォーマット | `style: Prettierでコードを整形` |
| refactor | リファクタリング | `refactor(api): エラーハンドリングを共通化` |
| test | テスト | `test(service): ShopSearchServiceのテストを追加` |
| chore | ビルド・設定 | `chore: ESLint設定を更新` |

**良いコミットメッセージの例**:
```
feat(registration): 店舗登録機能を追加

ユーザーが検索結果から店舗を登録できるようになりました。

実装内容:
- POST /api/registrations エンドポイント追加
- RegistrationService で重複チェック実装
- フロントエンドに登録ボタンを追加

Closes #15
```

### プルリクエストプロセス

**作成前のチェック**:
- [ ] 全てのテストがパス (`npm run test`)
- [ ] Lintエラーがない (`npm run lint`)
- [ ] 型チェックがパス (`npm run typecheck`)
- [ ] ビルドが成功する (`npm run build`)

**PRテンプレート**:
```markdown
## 概要
[変更内容の簡潔な説明]

## 変更理由
[なぜこの変更が必要か]

## 変更内容
- [変更点1]
- [変更点2]

## テスト
- [ ] ユニットテスト追加/更新
- [ ] 手動テスト実施

## スクリーンショット（UI変更の場合）
[画像]

## 関連Issue
Closes #[Issue番号]

## レビューポイント
[特に見てほしい点]
```

## テスト戦略

### テストの種類と配置

テストファイルの配置については[リポジトリ構造定義書](./repository-structure.md#テストディレクトリ)も参照してください。

| テスト種別 | フレームワーク | 配置先 | 命名規則 |
|-----------|--------------|--------|---------|
| ユニットテスト | Vitest | `apps/*/\_\_tests\_\_/` | `[対象ファイル名].test.ts(x)` |
| 統合テスト | Vitest | `apps/*/\_\_tests\_\_/integration/` | `[シナリオ].test.ts` |
| E2Eテスト | Playwright | `tests/e2e/` (プロジェクトルート) | `[シナリオ].spec.ts` |

### カバレッジ目標（architecture.md準拠）

| カバレッジ種別 | 目標値 | 測定方法 |
|--------------|--------|---------|
| 行カバレッジ | 80%以上 | `vitest --coverage` |
| 分岐カバレッジ | 75%以上 | `vitest --coverage` |
| 関数カバレッジ | 90%以上 | `vitest --coverage` |
| 主要フローE2E | 100%カバー | Playwrightレポート |

### テストの書き方（Given-When-Then）

```typescript
// apps/api/__tests__/services/ShopSearchService.test.ts
import { describe, it, expect, vi, beforeEach } from 'vitest';
import { ShopSearchService } from '../../src/services/ShopSearchService';
import { ValidationError } from '@meshi-map/shared';

describe('ShopSearchService', () => {
  describe('search', () => {
    it('正常なキーワードで店舗を検索できる', async () => {
      // Given: 準備
      const service = new ShopSearchService(mockGooglePlacesClient);
      const keyword = 'ラーメン';

      // When: 実行
      const results = await service.search(keyword);

      // Then: 検証
      expect(results).toBeInstanceOf(Array);
      expect(results.length).toBeGreaterThan(0);
      expect(results[0]).toHaveProperty('placeId');
      expect(results[0]).toHaveProperty('name');
    });

    it('空のキーワードでValidationErrorをスローする', async () => {
      // Given: 準備
      const service = new ShopSearchService(mockGooglePlacesClient);
      const emptyKeyword = '';

      // When/Then: 実行と検証
      await expect(
        service.search(emptyKeyword)
      ).rejects.toThrow(ValidationError);
    });

    it('100文字を超えるキーワードでValidationErrorをスローする', async () => {
      // Given: 準備
      const service = new ShopSearchService(mockGooglePlacesClient);
      const longKeyword = 'a'.repeat(101);

      // When/Then: 実行と検証
      await expect(
        service.search(longKeyword)
      ).rejects.toThrow(ValidationError);
    });
  });
});
```

### モックの使用

```typescript
// 外部API（Google Places）のモック
const mockGooglePlacesClient = {
  textSearch: vi.fn().mockResolvedValue({
    results: [
      {
        place_id: 'ChIJ...',
        name: 'テストラーメン店',
        formatted_address: '東京都渋谷区...',
        geometry: { location: { lat: 35.6812, lng: 139.7671 } },
        types: ['restaurant', 'food'],
      },
    ],
  }),
};

// Supabaseのモック
const mockSupabase = {
  from: vi.fn().mockReturnThis(),
  select: vi.fn().mockReturnThis(),
  insert: vi.fn().mockReturnThis(),
  eq: vi.fn().mockReturnThis(),
  single: vi.fn(),
};
```

## コードレビュー基準

### レビューポイント

**[必須] 機能性**（満たさない場合はマージ不可）:
- [ ] PRDの要件を満たしているか（P0機能は100%満たす必要あり）
- [ ] エッジケースが考慮されているか（空文字列、null、undefined、境界値）
- [ ] エラーハンドリングが適切か（予期されるエラーは全てハンドリング）

**[必須] セキュリティ**（満たさない場合はマージ不可）:
- [ ] 入力検証が適切か（バリデーションルール準拠）
- [ ] APIキーがハードコードされていないか（環境変数のみ使用）
- [ ] XSS/SQLインジェクション対策がされているか

**[推奨] 可読性**（改善が望ましいが必須ではない）:
- [ ] 命名が明確で一貫しているか（命名規則準拠）
- [ ] 複雑なロジックにコメントがあるか（O(n^2)以上、ビジネスロジック）
- [ ] 関数が適切なサイズか（50行以内推奨、100行超は要説明）

**[推奨] 保守性**（改善が望ましいが必須ではない）:
- [ ] 重複コードがないか（DRY原則、3回以上の繰り返しは抽出）
- [ ] レイヤー間の依存関係が適切か（architecture.md準拠）
- [ ] 変更の影響範囲が限定的か（1つの責務に集中）

**[推奨] パフォーマンス**（改善が望ましいが必須ではない）:
- [ ] 不要なAPI呼び出しがないか（キャッシュ活用）
- [ ] 適切なデータ構造を使用しているか（O(1)アクセスが必要ならMapを使用）
- [ ] N+1問題がないか（バッチ処理を検討）

### レビューコメントの書き方

**優先度の明示**:
```markdown
[必須] セキュリティ: APIキーが露出しています。環境変数に移動してください。

[推奨] パフォーマンス: この処理はループの外に移動できます。

[提案] 可読性: この関数名を `searchShopsByKeyword` に変更するのはどうでしょうか？

[質問] この条件分岐の意図を教えてください。
```

## 開発環境セットアップ

### 必要なツール

| ツール | バージョン | インストール方法 |
|--------|-----------|-----------------|
| Node.js | v24.11.0 | nvm or devcontainer |
| npm | 11.x | Node.jsに同梱 |
| Docker | latest | devcontainer利用時 |

### セットアップ手順

```bash
# 1. リポジトリのクローン
git clone https://github.com/your-org/meshi-map.git
cd meshi-map

# 2. 依存関係のインストール
npm install

# 3. 環境変数の設定
cp apps/web/.env.example apps/web/.env.local
cp apps/api/.dev.vars.example apps/api/.dev.vars
# 以下の環境変数設定例を参照して編集

# 4. 開発サーバーの起動
npm run dev           # 全体
npm run dev:web       # フロントエンドのみ (localhost:3000)
npm run dev:api       # バックエンドのみ (localhost:8787)

# 5. テストの実行
npm run test          # 全体
npm run test:web      # フロントエンドのみ
npm run test:api      # バックエンドのみ

# 6. ビルド
npm run build         # 全体
```

### 環境変数の設定

**apps/web/.env.local**:
```bash
# バックエンドAPI URL
NEXT_PUBLIC_API_URL=http://localhost:8787
```

**apps/api/.dev.vars**:
```bash
# Google Places API キー（Google Cloud Consoleから取得）
# https://console.cloud.google.com/apis/credentials
GOOGLE_PLACES_API_KEY=your_api_key_here

# Supabase設定（Supabaseダッシュボードから取得）
# https://supabase.com/dashboard/project/_/settings/api
SUPABASE_URL=https://your-project.supabase.co
SUPABASE_ANON_KEY=your_anon_key_here
```

**注意事項**:
- 実際のAPIキーは絶対にGitにコミットしないこと
- チームメンバーへは別途安全な方法で共有（パスワードマネージャー等）
- 本番環境のキーはCloudflare Dashboard（Secrets）で管理

### npm scripts

```json
{
  "scripts": {
    "dev": "npm run dev --workspaces",
    "dev:web": "npm run dev -w apps/web",
    "dev:api": "npm run dev -w apps/api",
    "build": "npm run build --workspaces",
    "test": "npm run test --workspaces",
    "lint": "eslint .",
    "lint:fix": "eslint . --fix",
    "format": "prettier --write .",
    "typecheck": "tsc --noEmit"
  }
}
```

## 品質自動化

### CI/CD（GitHub Actions）

```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '24'
          cache: 'npm'
      - run: npm ci
      - run: npm run lint
      - run: npm run typecheck

  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '24'
          cache: 'npm'
      - run: npm ci
      - run: npm run test

  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '24'
          cache: 'npm'
      - run: npm ci
      - run: npm run build
```

### Pre-commit フック（Husky + lint-staged）

```json
// package.json
{
  "scripts": {
    "prepare": "husky"
  },
  "lint-staged": {
    "*.{ts,tsx}": [
      "eslint --fix",
      "prettier --write"
    ],
    "*.{json,md}": [
      "prettier --write"
    ]
  }
}
```

```bash
# .husky/pre-commit
npm run lint-staged
npm run typecheck
```

### Commitlint（コミットメッセージ検証）

Conventional Commitsの規約をCIおよびローカルで強制するため、commitlintを導入します。

**設定ファイル** (`.commitlintrc.json`):
```json
{
  "extends": ["@commitlint/config-conventional"],
  "rules": {
    "type-enum": [
      2,
      "always",
      ["feat", "fix", "docs", "style", "refactor", "test", "chore"]
    ],
    "scope-case": [2, "always", "kebab-case"],
    "subject-case": [0],
    "subject-max-length": [2, "always", 72],
    "body-max-line-length": [2, "always", 100]
  }
}
```

**Husky commit-msg フック** (`.husky/commit-msg`):
```bash
npx --no -- commitlint --edit "$1"
```

**セットアップ手順**:
```bash
# 必要なパッケージをインストール
npm install -D @commitlint/cli @commitlint/config-conventional

# Huskyフックを追加
echo 'npx --no -- commitlint --edit "$1"' > .husky/commit-msg
chmod +x .husky/commit-msg
```

**エラー例と対処**:
```bash
# ❌ エラー: type が不正
$ git commit -m "add new feature"
⧗   input: add new feature
✖   subject may not be empty [subject-empty]
✖   type may not be empty [type-empty]

# ✅ 正しいフォーマット
$ git commit -m "feat(search): 店舗検索機能を追加"
```
