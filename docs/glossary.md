# プロジェクト用語集 (Glossary)

## 概要

このドキュメントは、めしマッププロジェクト内で使用される用語の定義を管理します。

**更新日**: 2025-01-15

## ドメイン用語

プロジェクト固有のビジネス概念や機能に関する用語。

### 店舗 (Shop)

**定義**: ユーザーが訪れた、または訪れたい飲食店

**説明**:
Google Places APIから取得した店舗情報をシステム内で管理するエンティティ。
店舗名、住所、座標、カテゴリ、写真URLなどの情報を持つ。

**関連用語**: [登録](#登録-registration)、[Place ID](#place-id)

**使用例**:
- 「店舗を検索する」: Google Places APIを通じて店舗情報を取得する
- 「店舗を登録する」: 検索結果の店舗を自分のリストに追加する

**データモデル**: `packages/shared/src/types/shop.ts`

**英語表記**: Shop

### 登録 (Registration)

**定義**: ユーザーが店舗を自分のリストに追加した記録

**説明**:
匿名ユーザーと店舗を紐付けるエンティティ。
登録日時を持ち、同一ユーザーが同一店舗を重複登録することはできない。

**関連用語**: [店舗](#店舗-shop)、[匿名ユーザー](#匿名ユーザー-anonymous-user)

**使用例**:
- 「登録店舗を地図上に表示する」
- 「登録を解除する」: 店舗を自分のリストから削除する

**関連API**:

| メソッド | エンドポイント | 説明 |
|---------|---------------|------|
| POST | `/api/registrations` | 店舗を登録 |
| GET | `/api/registrations` | 登録一覧を取得 |
| DELETE | `/api/registrations/{id}` | 登録を解除 |

**データモデル**: `packages/shared/src/types/registration.ts`

**英語表記**: Registration

### 匿名ユーザー (Anonymous User)

**定義**: アカウント登録なしでアプリを利用するユーザーの識別子

**説明**:
MVP段階ではユーザー認証を実装せず、ブラウザのLocalStorageに保存された
UUID v4形式の匿名ユーザーIDでユーザーを識別する。
ブラウザのキャッシュをクリアするとIDが失われ、データにアクセスできなくなる。

**関連用語**: [登録](#登録-registration)、[ユーザー認証](#ユーザー認証-user-authentication)

**使用例**:
- 「匿名ユーザーIDを生成する」: 初回起動時にUUIDを生成
- 「匿名ユーザーIDをリクエストヘッダに付与する」

**制約事項**:
- ブラウザのLocalStorageに依存
- 異なるブラウザ・デバイス間での同期不可
- データ消失リスクあり

**英語表記**: Anonymous User

### マーカー (Marker)

**定義**: 地図上に表示される店舗や現在地を示すアイコン

**説明**:
Leafletの地図上に配置されるピン状のアイコン。
タップすると店舗詳細のポップアップが表示される。

**関連用語**: [ポップアップ](#ポップアップ-popup)、[地図](#地図-map)

**使用例**:
- 「登録店舗のマーカーを表示する」
- 「マーカーをタップして詳細を表示する」

**英語表記**: Marker

### ポップアップ (Popup)

**定義**: マーカーをタップした際に表示される店舗詳細情報

**説明**:
店舗名、住所、カテゴリ、画像、Google Mapsへのリンクなどを
表示するモーダルまたはカード形式のUI要素。

**関連用語**: [マーカー](#マーカー-marker)、[店舗](#店舗-shop)

**使用例**:
- 「ポップアップを閉じる」
- 「ポップアップからGoogle Mapsを開く」

**英語表記**: Popup

### ユーザー認証 (User Authentication)

**定義**: メールアドレスとパスワードによるアカウント認証機能

**説明**:
P1（重要）の将来機能として計画。
匿名ユーザーのデータをアカウントに移行し、
複数デバイスでのデータ同期を可能にする。

**関連用語**: [匿名ユーザー](#匿名ユーザー-anonymous-user)、[優先度](#優先度-priority)

**優先度**: P1（重要）- MVP後に実装予定

**英語表記**: User Authentication

### 優先度 (Priority)

**定義**: 機能開発の優先順位を示す指標

**分類**:

| レベル | 名称 | 説明 | リリースへの影響 |
|--------|------|------|-----------------|
| P0 | 必須 | MVP機能。リリースに必須 | なければリリース不可 |
| P1 | 重要 | MVP後に実装すべき重要機能 | 初回リリース後に対応 |
| P2 | できれば | 将来的に検討する機能 | 余裕があれば対応 |

**関連用語**: [MVP](#mvp)

**使用例**:
- 「店舗検索はP0なので最優先で実装する」
- 「ユーザー認証はP1として位置づけられている」
- 「お気に入りフォルダ機能はP2として将来検討」

**本プロジェクトでのP0機能**:
- 店舗検索
- 店舗登録
- 地図表示
- 店舗詳細表示

**英語表記**: Priority

### 検索キーワード (Search Keyword)

**定義**: 店舗検索時にユーザーが入力するテキスト

**制約**:
- 文字数: 1〜100文字
- 空文字列は不可
- 100文字を超える場合はValidationError

**関連用語**: [店舗](#店舗-shop)、[ValidationError](#validationerror)

**使用例**:
- 「渋谷 ラーメン」で検索
- 「新宿 イタリアン」で検索

**英語表記**: Search Keyword

## 技術用語

プロジェクトで使用している技術・フレームワーク・ツールに関する用語。

### React

**定義**: ユーザーインターフェース構築のためのJavaScriptライブラリ

**公式サイト**: https://react.dev/

**本プロジェクトでの用途**:
Next.jsの基盤としてフロントエンドUIを構築。
関数コンポーネントとHooksを使用。

**バージョン**: 19.x

**選定理由**:
- コンポーネントベースの再利用性
- 豊富なエコシステム
- Next.jsとの統合

**関連ドキュメント**: [アーキテクチャ設計書](./architecture.md#テクノロジースタック)

### Next.js

**定義**: Reactベースのフルスタックウェブアプリケーションフレームワーク

**公式サイト**: https://nextjs.org/

**本プロジェクトでの用途**:
フロントエンドアプリケーションのフレームワークとして使用。
App Routerを採用し、Cloudflare Pagesにデプロイ。

**バージョン**: 15.x

**選定理由**:
- Reactベースで開発効率が高い
- Cloudflare Pagesとの相性が良い
- SSR/SSG対応

**関連ドキュメント**: [アーキテクチャ設計書](./architecture.md#テクノロジースタック)

### Hono

**定義**: 軽量で高速なWebフレームワーク

**公式サイト**: https://hono.dev/

**本プロジェクトでの用途**:
バックエンドAPIサーバーのフレームワークとして使用。
Cloudflare Workers上で動作。

**バージョン**: 4.x

**選定理由**:
- 軽量（約12KB）
- Cloudflare Workers最適化
- TypeScriptネイティブ対応

**関連ドキュメント**: [アーキテクチャ設計書](./architecture.md#テクノロジースタック)

### Leaflet

**定義**: オープンソースのJavaScript地図ライブラリ

**公式サイト**: https://leafletjs.com/

**本プロジェクトでの用途**:
地図表示、マーカー配置、ポップアップ表示に使用。
React Leafletラッパーを通じてReactコンポーネントとして利用。

**バージョン**: 1.9.x

**選定理由**:
- 無料・オープンソース
- 軽量で高速
- OpenStreetMapタイル対応

**関連ドキュメント**: [機能設計書](./functional-design.md#技術スタック)

### React Leaflet

**定義**: LeafletをReactコンポーネントとして利用するためのラッパーライブラリ

**公式サイト**: https://react-leaflet.js.org/

**本プロジェクトでの用途**:
MapContainer、TileLayer、Marker、Popup等のコンポーネントを使用し、
Reactの宣言的な記法で地図機能を実装。

**バージョン**: 5.x

**選定理由**:
- React 19対応
- Leafletの全機能をReactで利用可能
- TypeScript型定義が充実

**関連用語**: [Leaflet](#leaflet)、[React](#react)

**関連ドキュメント**: [アーキテクチャ設計書](./architecture.md#テクノロジースタック)

### Supabase

**定義**: オープンソースのFirebase代替サービス（PostgreSQL + 認証 + ストレージ）

**公式サイト**: https://supabase.com/

**本プロジェクトでの用途**:
PostgreSQLデータベースとして使用。
将来的にユーザー認証機能（Supabase Auth）も利用予定。

**バージョン**: supabase-js 2.x

**選定理由**:
- 無料枠あり
- PostgreSQLベース
- 認証機能内蔵（将来利用）
- 自動バックアップ

**関連ドキュメント**: [アーキテクチャ設計書](./architecture.md#テクノロジースタック)

### Google Places API

**定義**: Googleが提供する場所検索・詳細情報取得API

**公式サイト**: https://developers.google.com/maps/documentation/places/web-service

**本プロジェクトでの用途**:
店舗検索機能で使用。Text Search APIで店舗を検索し、
店舗名、住所、座標、カテゴリ、写真を取得。

**コスト**: 月間$200の無料クレジット

**選定理由**:
- 日本の飲食店データが豊富
- 無料クレジットあり
- 信頼性が高い

**関連ドキュメント**: [機能設計書](./functional-design.md#api設計)

### Cloudflare Pages

**定義**: Cloudflareが提供する静的サイト/フロントエンドホスティングサービス

**公式サイト**: https://pages.cloudflare.com/

**本プロジェクトでの用途**:
Next.jsフロントエンドのホスティング。

**選定理由**:
- 無料枠が充実
- グローバルCDN
- クレジットカード登録不要

### Cloudflare Workers

**定義**: Cloudflareが提供するエッジコンピューティングプラットフォーム

**公式サイト**: https://workers.cloudflare.com/

**本プロジェクトでの用途**:
HonoバックエンドAPIのホスティング。

**制約**:
- CPU時間: 10ms/リクエスト（無料枠）
- メモリ: 128MB

**選定理由**:
- 無料枠が充実
- エッジでの低レイテンシ
- Honoとの相性が良い

## 略語・頭字語

### API

**正式名称**: Application Programming Interface

**意味**: アプリケーション間でデータをやり取りするためのインターフェース

**本プロジェクトでの使用**:
- バックエンドAPI: `/api/shops/*`, `/api/registrations/*`
- 外部API: Google Places API

### MVP

**正式名称**: Minimum Viable Product

**意味**: 最小限の機能を持った実用可能なプロダクト

**本プロジェクトでの使用**:
PRDでP0（必須）として定義された機能群。
店舗検索、店舗登録、地図表示、店舗詳細表示の4機能。

### PRD

**正式名称**: Product Requirements Document

**意味**: プロダクト要求定義書

**本プロジェクトでの使用**:
`docs/product-requirements.md` に配置。
プロダクトの目的、ターゲットユーザー、機能要件、非機能要件を定義。

### UUID

**正式名称**: Universally Unique Identifier

**意味**: 世界中で一意であることが保証される識別子

**本プロジェクトでの使用**:
匿名ユーザーID、店舗ID、登録IDにUUID v4形式を採用。

**フォーマット**: `xxxxxxxx-xxxx-4xxx-yxxx-xxxxxxxxxxxx`

### Place ID

**正式名称**: Google Place ID

**意味**: Google Places APIで場所を一意に識別するID

**本プロジェクトでの使用**:
店舗のユニーク識別子として使用。
Google Mapsへのリンク生成に利用。

**例**: `ChIJ...`（可変長文字列）

## アーキテクチャ用語

### モノレポ (Monorepo)

**定義**: 複数のプロジェクト/パッケージを単一のリポジトリで管理する構成

**本プロジェクトでの適用**:
```
meshi-map/
├── apps/
│   ├── web/       # フロントエンド
│   └── api/       # バックエンド
└── packages/
    ├── shared/    # 共通型定義
    └── database/  # Supabaseクライアント
```

**メリット**:
- コード共有が容易
- 一貫したバージョン管理
- 統一されたCI/CD

**関連ドキュメント**: [リポジトリ構造定義書](./repository-structure.md)

### レイヤードアーキテクチャ (Layered Architecture)

**定義**: システムを役割ごとに複数の層に分割し、上位層から下位層への一方向の依存関係を持たせる設計パターン

**本プロジェクトでの適用**:
バックエンド（apps/api）で採用:

```
APIレイヤー (routes/)
    ↓
サービスレイヤー (services/)
    ↓
リポジトリレイヤー (repositories/)
    ↓
データレイヤー (Supabase/Google Places API)
```

**メリット**:
- 関心の分離による保守性向上
- テストが容易
- 変更の影響範囲が限定的

**関連ドキュメント**: [アーキテクチャ設計書](./architecture.md#アーキテクチャパターン)

### APIレイヤー (API Layer)

**定義**: HTTPリクエストを受け付け、レスポンスを返す層

**責務**:
- ルーティング
- リクエストのバリデーション
- 認証・認可チェック
- サービスレイヤーの呼び出し
- レスポンスの整形

**禁止事項**:
- ビジネスロジックの実装
- データベースへの直接アクセス

**本プロジェクトでの実装**:
- `apps/api/src/routes/shops.ts`
- `apps/api/src/routes/registrations.ts`

**関連用語**: [サービスレイヤー](#サービスレイヤー-service-layer)

### サービスレイヤー (Service Layer)

**定義**: ビジネスロジックを実装する層

**責務**:
- ビジネスルールの適用
- 複数リポジトリの調整
- トランザクション管理
- 外部API呼び出しの調整

**禁止事項**:
- APIレイヤーへの依存
- Supabaseクライアントの直接利用（リポジトリ経由）

**本プロジェクトでの実装**:
- `ShopSearchService`: 店舗検索ロジック
- `RegistrationService`: 登録・解除ロジック

**関連用語**: [APIレイヤー](#apiレイヤー-api-layer)、[リポジトリレイヤー](#リポジトリレイヤー-repository-layer)

### リポジトリレイヤー (Repository Layer)

**定義**: データアクセスを抽象化する層

**責務**:
- データベースCRUD操作
- クエリの構築と実行
- データマッピング

**禁止事項**:
- ビジネスロジックの実装
- 複数エンティティにまたがる操作

**本プロジェクトでの実装**:
- `ShopRepository`: 店舗テーブルへのアクセス
- `RegistrationRepository`: 登録テーブルへのアクセス

**関連用語**: [サービスレイヤー](#サービスレイヤー-service-layer)、[データレイヤー](#データレイヤー-data-layer)

### データレイヤー (Data Layer)

**定義**: 実際のデータストアや外部サービスとの接続を管理する層

**責務**:
- データベース接続管理
- 外部API接続管理
- 接続プール管理

**本プロジェクトでの実装**:
- Supabase (PostgreSQL)
- Google Places API

**関連用語**: [リポジトリレイヤー](#リポジトリレイヤー-repository-layer)、[Supabase](#supabase)、[Google Places API](#google-places-api)

## データモデル用語

### shops テーブル

**定義**: 店舗情報を格納するテーブル

**主要フィールド**:

| フィールド | 型 | 制約 |
|-----------|-----|------|
| `id` | UUID | PK |
| `place_id` | TEXT | UNIQUE, NOT NULL |
| `name` | TEXT | NOT NULL, 1-200文字 |
| `address` | TEXT | NOT NULL |
| `latitude` | DECIMAL | NOT NULL, -90〜90 |
| `longitude` | DECIMAL | NOT NULL, -180〜180 |
| `category` | TEXT | NOT NULL |
| `photo_url` | TEXT | NULL許容 |
| `google_maps_url` | TEXT | NOT NULL |
| `created_at` | TIMESTAMP | NOT NULL, DEFAULT now() |
| `updated_at` | TIMESTAMP | NOT NULL, DEFAULT now() |

**関連エンティティ**: [registrations](#registrations-テーブル)

**制約**:
- `place_id` はユニーク
- `name`, `address`, `category` は必須
- `latitude` は -90〜90 の範囲
- `longitude` は -180〜180 の範囲

### registrations テーブル

**定義**: ユーザーと店舗の関連付け（登録）を格納するテーブル

**主要フィールド**:

| フィールド | 型 | 制約 |
|-----------|-----|------|
| `id` | UUID | PK |
| `anonymous_user_id` | UUID | NOT NULL |
| `shop_id` | UUID | FK → shops.id, NOT NULL |
| `created_at` | TIMESTAMP | NOT NULL, DEFAULT now() |

**関連エンティティ**: [shops](#shops-テーブル)

**制約**:
- `anonymous_user_id` + `shop_id` の組み合わせはユニーク
- `shop_id` は shops テーブルへの外部キー

## エラー・例外

### ValidationError

**クラス名**: `ValidationError`

**発生条件**:
- リクエストパラメータが不正な場合
- 検索キーワードが空または長すぎる場合
- 匿名ユーザーIDの形式が不正な場合

**対処方法**:
- ユーザー: エラーメッセージに従って入力を修正
- 開発者: バリデーションルールを確認

**HTTPステータス**: 400 Bad Request

**例**:
```typescript
throw new ValidationError('キーワードは1-100文字で入力してください', 'keyword', keyword);
```

### NotFoundError

**クラス名**: `NotFoundError`

**発生条件**:
- 指定されたIDの店舗が存在しない場合
- 指定されたIDの登録が存在しない場合

**対処方法**:
- ユーザー: 正しいIDを指定
- 開発者: ID生成・参照ロジックを確認

**HTTPステータス**: 404 Not Found

**例**:
```typescript
throw new NotFoundError('Registration', registrationId);
```

### ForbiddenError

**クラス名**: `ForbiddenError`

**発生条件**:
- 他のユーザーの登録にアクセスしようとした場合

**対処方法**:
- ユーザー: 自分の登録のみ操作可能
- 開発者: 認可チェックを確認

**HTTPステータス**: 403 Forbidden

### ExternalApiError

**クラス名**: `ExternalApiError`

**発生条件**:
- Google Places APIの呼び出しが失敗した場合
- API利用上限に達した場合

**対処方法**:
- ユーザー: しばらく待ってから再試行
- 開発者: APIキー、利用量、ネットワークを確認

**HTTPステータス**: 500 Internal Server Error / 503 Service Unavailable

## 索引

### あ行
- [APIレイヤー](#apiレイヤー-api-layer) - アーキテクチャ用語
- [匿名ユーザー](#匿名ユーザー-anonymous-user) - ドメイン用語

### か行
- [Cloudflare Pages](#cloudflare-pages) - 技術用語
- [Cloudflare Workers](#cloudflare-workers) - 技術用語
- [検索キーワード](#検索キーワード-search-keyword) - ドメイン用語
- [Google Places API](#google-places-api) - 技術用語

### さ行
- [サービスレイヤー](#サービスレイヤー-service-layer) - アーキテクチャ用語
- [Supabase](#supabase) - 技術用語
- [shops テーブル](#shops-テーブル) - データモデル用語

### た行
- [データレイヤー](#データレイヤー-data-layer) - アーキテクチャ用語
- [店舗](#店舗-shop) - ドメイン用語
- [登録](#登録-registration) - ドメイン用語

### は行
- [Hono](#hono) - 技術用語
- [ポップアップ](#ポップアップ-popup) - ドメイン用語

### ま行
- [マーカー](#マーカー-marker) - ドメイン用語
- [モノレポ](#モノレポ-monorepo) - アーキテクチャ用語

### や行
- [優先度](#優先度-priority) - ドメイン用語
- [ユーザー認証](#ユーザー認証-user-authentication) - ドメイン用語

### ら行
- [Leaflet](#leaflet) - 技術用語
- [リポジトリレイヤー](#リポジトリレイヤー-repository-layer) - アーキテクチャ用語
- [レイヤードアーキテクチャ](#レイヤードアーキテクチャ-layered-architecture) - アーキテクチャ用語
- [registrations テーブル](#registrations-テーブル) - データモデル用語

### A-Z
- [API](#api) - 略語
- [MVP](#mvp) - 略語
- [Next.js](#nextjs) - 技術用語
- [Place ID](#place-id) - 略語
- [PRD](#prd) - 略語
- [React](#react) - 技術用語
- [React Leaflet](#react-leaflet) - 技術用語
- [UUID](#uuid) - 略語

### エラー
- [ExternalApiError](#externalapierror) - エラー
- [ForbiddenError](#forbiddenerror) - エラー
- [NotFoundError](#notfounderror) - エラー
- [ValidationError](#validationerror) - エラー
